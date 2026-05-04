# Análise e Previsão da Variação de Temperatura com PySpark

Projeto desenvolvido para a disciplina de **Big Data**, com o objetivo de aplicar técnicas de processamento distribuído, análise exploratória de dados, pré-processamento e modelagem preditiva utilizando o ecossistema Apache Spark.

O projeto utiliza dados meteorológicos horários da região Sudeste do Brasil, obtidos a partir do Kaggle, com foco na análise da variação da temperatura ao longo dos anos e na preparação de uma base para modelos de aprendizado de máquina.

## Objetivo do Projeto

O objetivo principal é construir um fluxo de Big Data capaz de:

- Carregar e processar um dataset meteorológico de grande volume.
- Converter arquivos CSV para o formato Parquet.
- Realizar análise exploratória dos dados utilizando PySpark.
- Tratar valores ausentes, inconsistências e outliers.
- Preparar uma base limpa para modelos preditivos.
- Treinar modelos de regressão para previsão da temperatura do ar.

A variável-alvo considerada no projeto é a **temperatura do ar**, representada originalmente pela coluna:

```text
TEMPERATURA DO AR - BULBO SECO, HORARIA (°C)
```

Após o tratamento, essa variável é renomeada para:

```text
temperatura
```

## Dataset Utilizado

O dataset utilizado foi obtido no Kaggle:

**Hourly Weather Surface Brazil - Southeast Region**  
Disponível em: https://www.kaggle.com/datasets/PROPPG-PPG/hourly-weather-surface-brazil-southeast-region

A base contém registros meteorológicos horários da região Sudeste do Brasil, incluindo informações como:

- Data e hora da medição
- Região e estado
- Estação meteorológica
- Temperatura do ar
- Temperatura máxima e mínima
- Temperatura do ponto de orvalho
- Umidade relativa do ar
- Pressão atmosférica
- Precipitação
- Radiação global
- Velocidade, rajada e direção do vento

Para reduzir o custo computacional local e manter o foco temporal do projeto, a análise foi concentrada nos dados do estado de **São Paulo (SP)**.

## Tecnologias Utilizadas

- Docker
- Jupyter Notebook
- Apache Spark
- PySpark
- Parquet
- Python

O projeto foi desenvolvido utilizando a imagem Docker:

```bash
jupyter/pyspark-notebook
```

## Estrutura do Projeto

```text
.
├── data/
│   ├── raw/
│   │   └── arquivos_csv_originais
│   └── processed/
│       ├── weather_sp_parquet/
│       └── weather_sp_preprocessado/
│
├── notebooks/
│   ├── 00_csv_to_parquet.ipynb
│   ├── 01_eda_weather.ipynb
│   └── 02_preprocessamento.ipynb
│
├── .gitignore
└── README.md
```

## Observação sobre os Dados

Os arquivos `.csv` e `.parquet` não são versionados no GitHub devido ao tamanho elevado da base.

As pastas de dados são mantidas apenas para organização local:

```text
data/raw/
data/processed/
```

O usuário deve baixar o dataset manualmente no Kaggle e colocar os arquivos CSV em:

```text
data/raw/
```

## Como Executar o Projeto

### 1. Clonar o repositório

```bash
git clone <url-do-repositorio>
cd <nome-do-repositorio>
```

### 2. Baixar o dataset

Baixe o dataset no Kaggle:

https://www.kaggle.com/datasets/PROPPG-PPG/hourly-weather-surface-brazil-southeast-region

Depois coloque os arquivos CSV em:

```text
data/raw/
```

### 3. Subir o ambiente com Docker

No Windows PowerShell, dentro da pasta raiz do projeto, execute:

```powershell
docker run --name spark-weather-container -p 8888:8888 -p 4040:4040 -p 7077:7077 -v ${PWD}:/home/jovyan/work jupyter/pyspark-notebook
```

No Linux ou macOS:

```bash
docker run --name spark-weather-container -p 8888:8888 -p 4040:4040 -p 7077:7077 -v $(pwd):/home/jovyan/work jupyter/pyspark-notebook
```

Após iniciar o container, acesse no navegador o link exibido no terminal, geralmente semelhante a:

```text
http://127.0.0.1:8888/lab?token=...
```

### 4. Executar os notebooks

Execute os notebooks na seguinte ordem:

```text
1. 00_csv_to_parquet.ipynb
2. 01_eda_weather.ipynb
3. 02_preprocessamento.ipynb
```

## Descrição dos Notebooks

### 00_csv_to_parquet.ipynb

Este notebook realiza a primeira etapa do pipeline.

Principais responsabilidades:

- Ler os arquivos CSV originais.
- Filtrar os dados do estado de São Paulo.
- Converter a base para o formato Parquet.
- Salvar a base processada em `data/processed/weather_sp_parquet`.

O uso do formato Parquet melhora a performance de leitura e processamento no Spark, pois os dados passam a ser armazenados em formato colunar.

### 01_eda_weather.ipynb

Este notebook realiza a análise exploratória dos dados.

Principais análises realizadas:

- Visualização inicial da base.
- Verificação do schema.
- Contagem de linhas e colunas.
- Análise de valores nulos.
- Estatísticas descritivas.
- Análise de temperatura por ano.
- Análise de temperatura por mês.
- Análise de temperatura por hora.
- Análise por estação meteorológica.
- Correlação entre variáveis meteorológicas.
- Análise aproximada de quartis e outliers.

Como restrição do projeto, a EDA foi desenvolvida utilizando apenas recursos do **PySpark**, sem bibliotecas externas de visualização.

### 02_preprocessamento.ipynb

Este notebook prepara a base para a etapa de modelagem.

Principais etapas:

- Normalização dos nomes das colunas.
- Renomeação das variáveis principais.
- Conversão de colunas numéricas.
- Criação de variáveis temporais.
- Remoção de registros sem variável-alvo.
- Tratamento de valores sentinela.
- Remoção de valores fisicamente inválidos.
- Remoção de outliers por IQR.
- Cálculo de valores nulos.
- Remoção de colunas com muitos valores ausentes.
- Imputação de valores nulos com mediana.
- Geração da base final para modelagem.

A base final é salva em:

```text
data/processed/weather_sp_preprocessado/
```

## Modelos Planejados

A próxima etapa do projeto será a modelagem preditiva.

Os modelos previstos são:

- Random Forest Regressor
- Rede Neural
- Regressão Linear

## Variável-Alvo

A variável-alvo do projeto é:

```text
temperatura
```

Ela representa a temperatura do ar em graus Celsius.

## Features Consideradas

Após o pré-processamento, algumas das variáveis candidatas para modelagem são:

```text
ano
mes
dia
hora_num
latitude
longitude
height
temperatura_maxima
temperatura_minima
temperatura_orvalho
umidade
pressao
precipitacao
radiacao
velocidade_vento
rajada_vento
direcao_vento
```

Essas variáveis podem ser ajustadas nos notebooks de modelagem conforme a necessidade de cada algoritmo.

## Organização dos Dados

Os dados brutos e processados não devem ser enviados para o GitHub.

Exemplo de configuração no `.gitignore`:

```gitignore
data/raw/*
data/processed/*

!data/raw/.gitkeep
!data/processed/.gitkeep

*.csv
*.parquet
_SUCCESS
*.crc
part-*
```

## Resultados Esperados

Ao final do projeto, espera-se obter:

- Uma base meteorológica processada em PySpark.
- Uma análise exploratória da variação de temperatura ao longo dos anos.
- Uma base pré-processada para modelagem.
- Modelos capazes de prever a temperatura do ar com base em variáveis meteorológicas e temporais.
- Comparação de desempenho entre os modelos utilizados.

## Status do Projeto

Status atual:

```text
[x] Ambiente com Docker e PySpark
[x] Conversão de CSV para Parquet
[x] Análise exploratória inicial
[x] Pré-processamento
[ ] Modelagem com Random Forest
[ ] Modelagem com Rede Neural
[ ] Modelagem com Regressão Linear
[ ] Avaliação final dos modelos
[ ] Preparação dos slides
```

## Autores

Projeto desenvolvido para a disciplina de Big Data.

Integrantes:

```text
- Ana Lucia de Souza
- Guilherme Massayuki Yokoda de Moraes
- Leonardo Rossi de Oliveira
- Tiago Tavares de Lima Gonçalves
```

## Considerações Finais

Este projeto utiliza técnicas de Big Data para processar e analisar dados meteorológicos de grande volume. A escolha do PySpark permite trabalhar com uma base extensa de forma mais eficiente, simulando um fluxo próximo ao utilizado em ambientes reais de engenharia de dados e ciência de dados.
