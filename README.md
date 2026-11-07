# Escorpiões em Cotia: Pipeline de Dados (SINAN × INMET)

Pipeline de engenharia de dados que trata as notificações de acidentes por escorpião (SINAN/DATASUS) da região de Cotia (SP) e as cruza com o clima diário (INMET), gerando uma tabela analítica pronta para dashboards e análises.

Projeto Integrador II · Ciência de Dados · Fatec Cotia (2026-2). Esta é a parte de **Engenharia e Dados**, que coordenei.

## O problema

Escorpiões são um problema real de saúde pública na região, e as bases oficiais chegam brutas: o SINAN vem como arquivos `.dbc` nacionais (cerca de 250 mil registros por arquivo, de todos os tipos de animais) com campos codificados, e o INMET vem em CSVs horários com layouts diferentes a cada ano. A pergunta para a qual o pipeline prepara os dados: **como o clima (chuva, temperatura e umidade) se relaciona com a ocorrência de acidentes?**

## Arquitetura (Bronze → Silver → Gold)

| Camada | Conteúdo |
|---|---|
| **Bronze** | Arquivos originais, sem alteração: SINAN `.dbc` (ANIMBR16 a ANIMBR26) e INMET CSV horário (2016–2026, estação Barueri A755) |
| **Silver** | SINAN filtrado (acidentes por escorpião, 12 municípios da região) e decodificado: **7.181 registros × 52 colunas**. INMET agregado por dia (precipitação, temperatura e umidade) |
| **Gold** | SINAN + clima do dia do acidente, via `LEFT JOIN` por data em **SQL (DuckDB)**: 7.181 casos × 56 colunas, **7.176 com clima (99,9%)** |

Municípios da região: Barueri, Carapicuíba, Cotia, Embu das Artes, Itapecerica da Serra, Itapevi, Jandira, Osasco, São Paulo, São Roque, Taboão da Serra e Vargem Grande Paulista.

## Notebooks

Ordem de execução (Google Colab, com o Google Drive montado):

| # | Notebook | O que faz |
|---|---|---|
| 1 | [`01_pipeline_sinan.ipynb`](notebooks/01_pipeline_sinan.ipynb) | Descompacta os `.dbc`, filtra escorpião e região, limpa e decodifica (Bronze → Silver) |
| 2 | [`02_pipeline_inmet.ipynb`](notebooks/02_pipeline_inmet.ipynb) | Unifica os CSVs horários do INMET de 2016 a 2026 em uma base diária (Bronze → Silver) |
| 3 | [`03_gold_sinan_inmet.ipynb`](notebooks/03_gold_sinan_inmet.ipynb) | Cruza SINAN e INMET por data com SQL no DuckDB (Silver → Gold) |

Cada etapa lê a camada anterior e grava a seguinte. Os caminhos são configuráveis por variáveis de ambiente (`SINAN_DIRETORIO_BASE`, `INMET_DIRETORIO_BASE`, `ESCORPIOES_BASE`).

## Destaques técnicos

- **Leitura rápida do SINAN.** O gargalo da primeira versão não era o pandas (a filtragem era só 0,8% do tempo), e sim a leitura do DBF linha a linha com `dbfread`, que convertia cerca de 250 mil registros por arquivo em dicionários Python. A versão atual lê o arquivo como matriz de bytes com NumPy, filtra tipo de acidente e município **antes** de decodificar, descomprime em disco local e guarda cache em Parquet (com `dbfread` como plano B). A execução completa dos 11 arquivos leva **cerca de 0,6 minuto, sem uso do cache** (saída registrada no notebook), contra mais de 10 minutos na versão anterior, com a mesma saída (7.181 registros).
- **Pipeline do INMET robusto a layouts diferentes.** Detecção automática de delimitador, codificação e da linha real do cabeçalho (os arquivos de 2016–2019 diferem dos demais, e um bloco de metadados no topo já enganou a primeira versão), mapeamento de colunas por padrões com prioridade e termos excluídos, e `-9999`/vazio tratados como nulo, nunca como zero. Dia sem leitura de chuva fica nulo, não 0 mm.
- **Decodificação completa do SINAN.** Códigos de município (tabela IBGE com os 5.570 municípios), ocupação (CBO 2002, 2.614 ocupações), categorias da ficha, idade codificada e semanas epidemiológicas convertidos para texto legível.
- **Princípio de qualidade de dados.** Nenhum valor é alterado só porque "parece errado": suspeitos são mantidos, sinalizados e levados à validação. Campo em branco ("Não informado") é diferente do código 9 da ficha ("Ignorado").
- **Validações na própria pipeline.** Logs de códigos fora das tabelas de referência, relatório de cobertura do INMET por ano e conferência de totais na camada Gold.

## Documentação

- [Regras de preparação e limpeza](docs/regras-de-limpeza.md): cada regra aplicada, com status e pendências de validação
- [Dicionário de dados da camada Gold](docs/dicionario-gold.md)
- [Dicionário de dados do INMET](docs/dicionario-inmet.md)

## Limitações conhecidas

- O clima vem de **um único posto** (Barueri A755) para toda a região, então é uma aproximação para os demais municípios.
- A agregação diária usa o dia UTC dos arquivos, não o dia local (diferença de 3 horas).
- 5 casos ficam sem clima (4 acidentes de 2015, anteriores ao início do INMET, e 1 sem dia correspondente).
- Há 34 grupos de possíveis duplicados mantidos e pendentes de validação com a fonte.

## Dados

Os dados individuais do SINAN **não são publicados** neste repositório, por serem registros de saúde. Aqui estão o código, a documentação e a estrutura do pipeline. Para reproduzir:

1. Baixe os arquivos de notificação de acidentes por animais peçonhentos (`ANIMBRxx.dbc`) no [DATASUS](https://datasus.saude.gov.br/) e coloque em `sinan/1_bronze/`.
2. Baixe os dados históricos horários da estação Barueri (A755) no [INMET](https://portal.inmet.gov.br/dadoshistoricos) e coloque em `inmet/1_bronze/`.
3. Rode os três notebooks na ordem.

As tabelas de apoio (municípios IBGE e CBO 2002) são baixadas automaticamente de [kelvins/municipios-brasileiros](https://github.com/kelvins/municipios-brasileiros) e [lucassmacedo/cbo-brasil](https://github.com/lucassmacedo/cbo-brasil), que merecem o crédito.

## Tecnologias

Python · pandas · NumPy · DuckDB (SQL) · Parquet · pyreaddbc · dbfread · Google Colab / Google Drive

## Sobre o desenvolvimento

O código foi desenvolvido com apoio de IA (Claude), com revisão, testes e validação dos resultados por mim.
