# Dicionário de dados: INMET (estação Barueri A755)

Campos do arquivo horário original de medições climáticas do INMET. O pipeline aproveita apenas três deles, agregados por dia (marcados com ✅).

| Campo no CSV | Descrição | Tipo | Usado no pipeline |
|---|---|---|---|
| Data | Data da medição | Data | ✅ (chave diária) |
| Hora UTC | Hora da medição em UTC | Texto | Não (só a data é usada) |
| PRECIPITAÇÃO TOTAL, HORÁRIO (mm) | Chuva acumulada na hora, em mm | Decimal | ✅ soma diária |
| PRESSAO ATMOSFERICA AO NIVEL DA ESTACAO, HORARIA (mB) | Pressão ao nível da estação | Decimal | Não |
| PRESSÃO ATMOSFERICA MAX. / MIN. NA HORA ANT. (AUT) (mB) | Pressão máxima e mínima na hora anterior | Decimal | Não |
| RADIACAO GLOBAL (Kj/m²) | Radiação solar global acumulada | Decimal | Não |
| TEMPERATURA DO AR - BULBO SECO, HORARIA (°C) | Temperatura do ar medida no horário | Decimal | ✅ média diária |
| TEMPERATURA DO PONTO DE ORVALHO (°C) | Temperatura do ponto de orvalho | Decimal | Não |
| TEMPERATURA MÁXIMA / MÍNIMA NA HORA ANT. (AUT) (°C) | Temperatura máxima e mínima na hora anterior | Decimal | Não |
| TEMPERATURA ORVALHO MAX. / MIN. NA HORA ANT. (AUT) (°C) | Orvalho máximo e mínimo na hora anterior | Decimal | Não |
| UMIDADE REL. MAX. / MIN. NA HORA ANT. (AUT) (%) | Umidade máxima e mínima na hora anterior | Decimal | Não |
| UMIDADE RELATIVA DO AR, HORARIA (%) | Umidade relativa medida no horário | Decimal | ✅ média diária |
| VENTO, DIREÇÃO HORARIA (gr) (° (gr)) | Direção do vento | Decimal | Não |
| VENTO, RAJADA MAXIMA (m/s) | Rajada máxima de vento | Decimal | Não |
| VENTO, VELOCIDADE HORARIA (m/s) | Velocidade do vento | Decimal | Não |

## Base diária gerada (Silver)

`inmet/2_silver/inmet_diario_2016_2026.csv`, com as colunas `DATA`, `PRECIPITACAO_TOTAL_MM`, `TEMPERATURA_MEDIA_C` e `UMIDADE_RELATIVA_MEDIA`.

- Precipitação: soma das leituras do dia; dia sem nenhuma leitura fica nulo (não 0 mm).
- Temperatura e umidade: média das leituras do dia.
- Ausências (`-9999` e vazios) viram nulo.
- O dia é o dia UTC do arquivo, não o dia local.
