# Análise e Previsão da Variação de Temperatura com PySpark

Projeto desenvolvido para a disciplina de **Big Data**, com o objetivo de aplicar técnicas de processamento distribuído, análise exploratória de dados, pré-processamento e modelagem preditiva utilizando o ecossistema Apache Spark.

O projeto utiliza dados meteorológicos horários do estado de São Paulo, obtidos a partir do Kaggle, com foco na previsão de temperatura futura usando três modelos de aprendizado de máquina.

## Objetivo do Projeto

O objetivo principal é construir um fluxo de Big Data capaz de:

- Carregar e processar um dataset meteorológico de grande volume (> 1 GB).
- Converter arquivos CSV para o formato Parquet.
- Realizar análise exploratória dos dados utilizando PySpark.
- Tratar valores ausentes, inconsistências e outliers.
- Criar features temporais de defasagem e janelas móveis para contexto climático.
- Treinar três modelos preditivos e comparar seus desempenhos.

### Alvos de Previsão

Cada modelo é treinado para dois objetivos distintos:

| Alvo | Descrição |
|---|---|
| `temperatura_amanha` | Temperatura média do dia seguinte (°C) |
| `temperatura_media_proximos_7_dias` | Temperatura média dos próximos 7 dias (°C) |

## Dataset Utilizado

**Hourly Weather Surface Brazil — Southeast Region**  
Disponível em: https://www.kaggle.com/datasets/PROPPG-PPG/hourly-weather-surface-brazil-southeast-region

A base contém registros meteorológicos horários da região Sudeste do Brasil. Para reduzir o custo computacional e manter o foco do projeto, a análise foi concentrada nos dados do estado de **São Paulo (SP)**.

## Tecnologias Utilizadas

- Docker (`jupyter/pyspark-notebook`)
- Apache Spark / PySpark
- PySpark MLlib
- Parquet
- Plotly
- Python

## Estrutura do Projeto

```text
.
├── data/
│   ├── raw/
│   │   └── arquivos_csv_originais
│   └── processed/
│       ├── weather_sp_parquet/
│       ├── weather_sp_preprocessado/
│       ├── weather_sp_dataset_amanha/
│       ├── weather_sp_dataset_semana/
│       ├── weather_sp_amanha_train/
│       ├── weather_sp_amanha_test/
│       ├── weather_sp_semana_train/
│       └── weather_sp_semana_test/
│
├── models/
│   ├── random_forest_amanha/
│   ├── random_forest_semana/
│   ├── linear_regression_amanha/
│   ├── linear_regression_semana/
│   ├── mlp_faixa_termica_amanha/
│   └── mlp_faixa_termica_semana/
│
├── notebooks/
│   ├── 00_csv_to_parquet.ipynb
│   ├── 01_eda_weather.ipynb
│   ├── 02_preprocessamento.ipynb
│   ├── 03_modelagem_random_forest.ipynb
│   ├── 04_modelo_linear_regression.ipynb
│   └── 05_modelo_redes_neurais.ipynb
│
├── .gitignore
└── README.md
```

## Descrição dos Notebooks

### 00_csv_to_parquet.ipynb

Converte os arquivos CSV originais para Parquet, filtrando apenas os dados do estado de São Paulo.

### 01_eda_weather.ipynb

Análise exploratória dos dados com PySpark. Cobre estatísticas descritivas, valores nulos, análise de temperatura por ano/mês/hora, correlação entre variáveis e análise por estação meteorológica. Desenvolvida inteiramente com PySpark, sem bibliotecas externas de visualização.

### 02_preprocessamento.ipynb

Prepara a base para modelagem. As principais etapas são:

- Normalização dos nomes das colunas e conversão de tipos.
- Tratamento de valores sentinela (`-9999`) e remoção de outliers por IQR.
- Criação de variáveis temporais (`ano`, `mes`, `dia`, `hora_num`).
- Agregação diária por estação meteorológica.
- Criação das colunas-alvo: `temperatura_amanha` e `temperatura_media_proximos_7_dias`.
- Criação de features de defasagem e janelas móveis (temperatura ontem, média 3 dias, média 7 dias, etc.).
- Classificação de estações por `tipo_area`, `macro_regiao_sp` e `faixa_altitude`.
- **Split temporal:** treino com anos anteriores a 2018, teste com 2018 em diante.
- Imputação com mediana calculada exclusivamente no treino (sem data leakage).

### 03_modelagem_random_forest.ipynb

Treina dois modelos **Random Forest Regressor** (previsão de amanhã e dos próximos 7 dias).

Avaliação com **MAE**, **RMSE** e **R²**. Análise da importância das variáveis e visualizações por tipo de área, macro região, mês e altitude com Plotly.

### 04_modelo_linear_regression.ipynb

Treina dois modelos **Linear Regression** com o mesmo par de alvos.

Avaliação com **MAE**, **RMSE** e **R²**. Análise dos coeficientes do modelo e visualizações por tipo de área, macro região, mês e altitude com Plotly.

### 05_modelo_redes_neurais.ipynb

Treina dois modelos de **Redes Neurais** usando `MultilayerPerceptronClassifier` do MLlib.

Como o MLlib possui rede neural nativa apenas para classificação, o alvo contínuo é convertido em quatro faixas térmicas calculadas pelos quartis da base de treino:

| Classe | Categoria | Interpretação |
|---:|---|---|
| 0 | mais_fria | abaixo do Q1 |
| 1 | amena_baixa | entre Q1 e Q2 |
| 2 | amena_alta | entre Q2 e Q3 |
| 3 | mais_quente | acima do Q3 |

Arquitetura da rede: `[n_features → 32 → 16 → 4]`. As features são normalizadas com `StandardScaler` antes do treinamento.

Avaliação com **Accuracy**, **F1-score**, **Weighted Precision**, **Weighted Recall** e matriz de confusão. Visualizações por tipo de área, macro região, mês e faixa de altitude com Plotly.

## Split Temporal

A separação treino/teste segue uma lógica temporal para evitar data leakage:

- **Treino:** anos anteriores a 2018
- **Teste:** anos de 2018 em diante

A imputação de valores ausentes usa medianas calculadas exclusivamente na base de treino e aplicadas na base de teste.

## Features Consideradas

As features finais incluem variáveis meteorológicas, geográficas e temporais de defasagem:

```text
Temporais:
  ano, mes, dia, hora_num

Geográficas:
  latitude, longitude, altitude, tipo_area_idx, macro_regiao_sp_idx, faixa_altitude_idx

Meteorológicas:
  umidade, umidade_maxima, umidade_minima
  pressao, pressao_maxima, pressao_minima
  precipitacao, radiacao
  velocidade_vento, rajada_vento, direcao_vento

Defasagem e janelas móveis (criadas no pré-processamento):
  temp_media_ontem
  temp_media_ultimos_3_dias
  temp_media_ultimos_7_dias
  umidade_media_ultimos_7_dias
  precipitacao_acumulada_ultimos_7_dias
```

## Observação sobre os Dados

Os arquivos `.csv` e `.parquet` não são versionados no GitHub devido ao tamanho elevado da base.

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

Coloque os arquivos CSV em:

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

Acesse no navegador o link exibido no terminal:

```text
http://127.0.0.1:8888/lab?token=...
```

### 4. Executar os notebooks em ordem

```text
1. 00_csv_to_parquet.ipynb
2. 01_eda_weather.ipynb
3. 02_preprocessamento.ipynb
4. 03_modelagem_random_forest.ipynb
5. 04_modelo_linear_regression.ipynb
6. 05_modelo_redes_neurais.ipynb
```

## Organização dos Dados

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

## Status do Projeto

```text
[x] Ambiente com Docker e PySpark
[x] Conversão de CSV para Parquet
[x] Análise exploratória
[x] Pré-processamento e feature engineering
[x] Modelagem com Random Forest Regressor
[x] Modelagem com Linear Regression
[x] Modelagem com Redes Neurais (MultilayerPerceptronClassifier)
[x] Avaliação e comparação dos modelos
[ ] Preparação dos slides
```

## Autores

Projeto desenvolvido para a disciplina de Big Data.

```text
- Ana Lucia de Souza
- Guilherme Massayuki Yokoda de Moraes
- Leonardo Rossi de Oliveira
- Tiago Tavares de Lima Gonçalves
```
