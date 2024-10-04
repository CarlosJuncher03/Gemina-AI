# Mage e IA Generativa: O Futuro da Engenharia de Dados

Esta documentação apresenta o passo a passo da utilização do Mage, integrado com IA generativa, Mkdocs, PostgreSQL e Python, para a criação de um fluxo de dados contínuo e escalável. O objetivo é automatizar e otimizar a engenharia de dados, proporcionando uma pipeline eficiente e robusta.

## Mage: Uma Ferramenta Poderosa e Multifuncional

Mage é uma ferramenta desenvolvida em Python que permite realizar tanto ETL quanto ELT, seja em tempo real (Real-Time) ou em Batch. Ela oferece flexibilidade na transformação de dados, podendo ser feita diretamente em SQL ou utilizando Python.

Com Mage, é possível executar:

- ETL e ELT
- Processos em Batch e CDC (Change Data Capture)
- Operações em Real-Time
- Cargas completas (Full Load) e incrementais
- Integração com DBT
- Transformações via SQL ou Python

Além dessas funcionalidades, Mage oferece uma ampla gama de possibilidades para criação de pipelines de dados personalizadas e eficientes. 

Para mais informações, consulte a [documentação oficial do projeto](https://docs.mage.ai/introduction/overview).

![Screenshot_1](https://github.com/user-attachments/assets/5aa9ecb3-d71f-41d2-9008-8756421d871d)


### Data Integration com Mage

O projeto consiste em utilizar a funcionalidade **Data Integration** do Mage para executar um ELT incremental do banco de dados MariaDB para o armazém de dados, a cada 1 minuto. O objetivo é capturar apenas as alterações (dados modificados), garantindo uma sincronização eficiente e em tempo real.

### Configurando Mage

A configuração do Mage é simples. Para criar um pipeline de **Data Integration**, siga os passos abaixo:

1. **Adicionar Pipeline**: Selecione a opção de criar uma nova pipeline para integração de dados.

![Screenshot_2](https://github.com/user-attachments/assets/e5d3124d-007a-4dab-aa2c-1e4994e91292)


2. **Adicionar Origem**: Selecione o tipo de origem de dados (por exemplo, MariaDB) e insira as credenciais necessárias.

![Screenshot_3](https://github.com/user-attachments/assets/2206eb7f-a277-429f-b6c4-9997b926b844)


3. **Adicionar Destino**: Escolha o destino para onde os dados serão carregados, como um Data Warehouse.Defina se a sincronização será incremental (apenas alterações) ou full (todos os dados

![Screenshot_4](https://github.com/user-attachments/assets/3babf71d-5ff3-4172-98d9-b2dd44c70f10)


4. **Após essas etapas, basta configurar o período de execução desejado (por exemplo, a cada 1 minuto).



## PostgreSQL: Um Banco Relacional Potente e Open Source

PostgreSQL é um banco de dados robusto, amplamente utilizado na área de dados por sua alta capacidade de processamento, armazenamento e análise. Mesmo que seu foco não seja um banco dimensional, ele ainda consegue solucionar uma grande parte dos desafios no gerenciamento de dados, tornando-se uma das principais escolhas para armazéns de dados.

### Dados Carregados

- Com a execução da pipeline no Mage, os dados são carregados diretamente no PostgreSQL, permitindo uma integração contínua e eficiente.

![Screenshot_6](https://github.com/user-attachments/assets/cc20093e-0b74-4786-9a26-120fe36fdc64)


## Modelo: Usando IA para gerar as views e documentação

No código abaixo, utilizamos o modelo Gemini para criar as análises das views e a documentação.

### Código em Python

```python
import psycopg2
import google.generativeai as genai
import os
import subprocess

# Configure a chave da API diretamente no código
genai.configure(api_key="")

# Parâmetros de geração otimizados para velocidade e maior número de tokens
generation_config = {
    "temperature": 0.7,
    "top_p": 1.0,
    "top_k": 100,
    "max_output_tokens": 10024,
    "response_mime_type": "text/plain",
}

# Parâmetros de moderação da geração (configurações de segurança)
safety_settings = [
    {"category": "HARM_CATEGORY_HARASSMENT", "threshold": "BLOCK_NONE"},
    {"category": "HARM_CATEGORY_HATE_SPEECH", "threshold": "BLOCK_NONE"},
    {"category": "HARM_CATEGORY_SEXUALLY_EXPLICIT", "threshold": "BLOCK_NONE"},
    {"category": "HARM_CATEGORY_DANGEROUS_CONTENT", "threshold": "BLOCK_NONE"},
]

# Inicialize o modelo Gemini 1.5 Flash
model = genai.GenerativeModel(
    model_name="gemini-1.5-flash",
    safety_settings=safety_settings,
    generation_config=generation_config,
)

# Parâmetros de conexão com o banco de dados PostgreSQL
db_params = {
    "dbname": "DataLake",
    "user": "postgres",
    "password": "1234",
    "host": "127.0.0.1",
    "port": "5432",
    "options": "-c search_path=vendas_produtos"
}

def connect_db(params):
    try:
        conn = psycopg2.connect(**params)
        print("Conexão ao banco de dados realizada com sucesso.")
        return conn
    except Exception as e:
        print(f"Erro ao conectar ao banco de dados: {e}")
        return None

# Extrair dados das tabelas
def extract_data(conn):
    try:
        cur = conn.cursor()

        cur.execute("""select 
                        p.codigo,
                        p.codigobarras,
                        p.descricao
                        from vendas_produtos.produtos p LIMIT 5;""")
        produtos_data = cur.fetchall()

        cur.execute("""SELECT * FROM vendas_produtos.vendasprodutos LIMIT 5;""")
        vendas_data = cur.fetchall()

        cur.close()
        print("Dados extraídos com sucesso.")
        return produtos_data, vendas_data
    except Exception as e:
        print(f"Erro ao extrair dados: {e}")
        return None, None

# Ajuste no prompt para views preditivas, especificando a relação correta entre as tabelas
def send_to_gemini_preditiva(produtos_data, vendas_data):
    prompt = f"""
    Aqui estão os dados de amostra da tabela 'produtos':
    {produtos_data}

    E aqui estão os dados de amostra da tabela 'vendasprodutos':
    {vendas_data}

    A relação entre as tabelas 'produtos' e 'vendasprodutos' é feita pela coluna 'codigo' na tabela 'produtos'
    e pela coluna 'codigoproduto' na tabela 'vendasprodutos'. Use essas colunas para fazer as junções corretas.

    Preciso que você gere 5 views SQL preditivas que façam previsões de vendas e lucro futuro.

    Outra questão: são views para serem rodadas em um PostgreSQL, as views precisam ser criadas no schema camada_analitica 
    e as tabelas estão no vendas_produtos. Então já ajuste as views para virem corretas para onde criar e de onde puxar as tabelas.

    Outro ponto: não invente campos nem nada, utilize os campos que eu te mandei para criar as views. Ao criar
    um join ou cálculos, não utilize nomes de campos que não existem nas tabelas do banco.

    Para cada view, forneça:
    - Nome da view
    - O que essa view está analisando
    - SQL da view
    """

    chat_session = model.start_chat()
    response = chat_session.send_message(prompt)
    
    # Print the raw response from the API
    print("Resposta da API para views preditivas:")
    print(response.text)

    return response.text

# Ajuste no prompt para views prescritivas, especificando a relação correta entre as tabelas
def send_to_gemini_prescritiva(produtos_data, vendas_data):
    prompt = f"""
    Aqui estão os dados de amostra da tabela 'produtos':
    {produtos_data}

    E aqui estão os dados de amostra da tabela 'vendasprodutos':
    {vendas_data}

    A relação entre as tabelas 'produtos' e 'vendasprodutos' é feita pela coluna 'codigo' na tabela 'produtos'
    e pela coluna 'codigoproduto' na tabela 'vendasprodutos'. Use essas colunas para fazer as junções corretas.

    Preciso que você gere 5 views SQL prescritivas que recomendem ações baseadas nas vendas e no lucro.

    Outra questão: são views para serem rodadas em um PostgreSQL, as views precisam ser criadas no schema camada_analitica 
    e as tabelas estão no vendas_produtos. Então já ajuste as views para virem corretas para onde criar e de onde puxar as tabelas.

    Outro ponto: não invente campos nem nada, utilize os campos que eu te mandei para criar as views. Ao criar
    um join ou cálculos, não utilize nomes de campos que não existem nas tabelas do banco.

    Para cada view, forneça:
    - Nome da view
    - O que essa view está analisando
    - SQL da view
    """

    chat_session = model.start_chat()
    response = chat_session.send_message(prompt)
    
    # Print the raw response from the API
    print("Resposta da API para views prescritivas:")
    print(response.text)

    return response.text

# Salvar a resposta da API diretamente em arquivos .md dentro da pasta docs do projeto MkDocs
def save_response_as_markdown(response_text, view_type, docs_dir):
    markdown_file = os.path.join(docs_dir, f"{view_type}_views.md")
    
    # Salvar diretamente a resposta completa no arquivo .md
    with open(markdown_file, "w", encoding="utf-8") as md_f:
        md_f.write(response_text)

    print(f"Arquivo {markdown_file} salvo com sucesso.")

# Função para criar e configurar o projeto MkDocs
def create_mkdocs_project():
    project_name = "my_mkdocs_project"
    
    # Criar o projeto MkDocs
    if not os.path.exists(project_name):
        subprocess.run(["mkdocs", "new", project_name])
        print(f"Projeto MkDocs '{project_name}' criado com sucesso.")
    
    # Caminho para a pasta docs dentro do projeto
    docs_dir = os.path.join(project_name, "docs")
    
    return project_name, docs_dir

# Atualizar o arquivo mkdocs.yml com os novos arquivos .md
def update_mkdocs_config(project_name):
    config_file = os.path.join(project_name, "mkdocs.yml")
    
    # Adicionar as views no menu de navegação
    with open(config_file, "a", encoding="utf-8") as config:
        config.write("\nnav:\n")
        config.write("  - Home: index.md\n")
        config.write("  - Views Preditivas: preditiva_views.md\n")
        config.write("  - Views Prescritivas: prescritiva_views.md\n")
    
    print(f"Arquivo {config_file} atualizado com sucesso.")

# Função principal
def main():
    # Criar o projeto MkDocs e obter o caminho da pasta docs
    project_name, docs_dir = create_mkdocs_project()

    conn = connect_db(db_params)

    if conn:
        produtos_data, vendas_data = extract_data(conn)

        if produtos_data and vendas_data:
            # Gerar e salvar views preditivas
            preditiva_response = send_to_gemini_preditiva(produtos_data, vendas_data)
            save_response_as_markdown(preditiva_response, "preditiva", docs_dir)

            # Gerar e salvar views prescritivas
            prescritiva_response = send_to_gemini_prescritiva(produtos_data, vendas_data)
            save_response_as_markdown(prescritiva_response, "prescritiva", docs_dir)

            # Atualizar o arquivo mkdocs.yml
            update_mkdocs_config(project_name)

            print("Projeto MkDocs criado e atualizado com as views preditivas e prescritivas.")
        else:
            print("Erro ao extrair dados das tabelas.")

        conn.close()
    else:
        print("Falha ao conectar ao banco de dados.")

if __name__ == "__main__":
    main()
```

## Resultado

Ao rodar o código, uma pasta chamada **`mkdocs`** será criada. Dentro dela, será possível subir um servidor local com a documentação gerada.

![Screenshot_10](https://github.com/user-attachments/assets/50e1db74-dd0f-43b7-a210-a88b327eeba1)


### Acessando a pasta e iniciando o servidor

Após acessar a pasta **`my_mkdocs_project`**, você verá a estrutura de diretórios e arquivos como a seguir:

```bash
PS D:\Projetos\ELT\Mage IA\my_mkdocs_project> ls

    Diretório: D:\Projetos\ELT\Mage IA\my_mkdocs_project

Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----        04/10/2024     01:16                docs
-a----        04/10/2024     01:16            136 mkdocs.yml
```
### Iniciando o Servidor

Para iniciar o servidor local, execute o seguinte comando:

```bash
PS D:\Projetos\ELT\Mage IA\my_mkdocs_project> mkdocs serve
O terminal exibirá as seguintes informações:

Copiar código
INFO    -  Building documentation...
INFO    -  Cleaning site directory
INFO    -  Documentation built in 0.14 seconds
INFO    -  [01:17:57] Watching paths for changes: 'docs', 'mkdocs.yml'
INFO    -  [01:17:57] Serving on http://127.0.0.1:8000/
```
Agora, você pode acessar a documentação localmente através do endereço: http://127.0.0.1:8000/.

### Acessando a Documentação

Para acessar a documentação gerada, basta abrir o navegador e inserir o endereço:

[http://127.0.0.1:8000/](http://127.0.0.1:8000/)

Após isso, a documentação estará disponível para visualização.

![Screenshot_11](https://github.com/user-attachments/assets/f74f123e-6c82-445a-9fb8-817b73792b98)
![Screenshot_12](https://github.com/user-attachments/assets/7efea529-864c-4373-afdd-18b0758c952e)
![Screenshot_13](https://github.com/user-attachments/assets/f3a7db7a-3e02-49e9-b83e-55a1a8a586b1)


## Conclusão

O projeto tem como objetivo gerar visualizações e análises de dados utilizando modelos avançados, como o **Gemini** ou o **ChatGPT-4**, que possuem grande capacidade de análise de dados de forma automática e precisa, com mínima intervenção humana.

