# Regras de preparação e limpeza dos dados

Projeto Integrador II · Escorpiões em Cotia · Engenharia e Dados · Sprint 2 (atualizado em 18/10/2026)

**Princípio:** nenhum valor é alterado só porque "parece errado". Valores suspeitos são mantidos, sinalizados e levados à validação.

## SINAN

| Problema | Regra aplicada | Status |
|---|---|---|
| Campos categóricos vazios (inclui pulos lógicos da ficha) | Texto "Não informado" (diferente de "Ignorado", código 9 da ficha) | Aplicada |
| Campos numéricos vazios (nº de ampolas) | Preenchido com 0 | **A validar:** 80 casos com soroterapia = Sim e 0 ampolas; proposta: "Não informado" |
| Ano de nascimento vazio (28 registros) | Preenchido com 0 | **A validar:** 0 não é ano válido; não calcular idade a partir desse campo |
| Códigos categóricos (sexo, raça, escolaridade, gestante, sim/não) | Decodificados para texto conforme o dicionário | Aplicada (tabela de escolaridade a validar com a fonte oficial) |
| Códigos IBGE de município | Convertidos para nome (5.570 municípios); fora de SP recebe sufixo da UF; código inexistente vira "Município ignorado (UF)" | Aplicada |
| Ocupação (CBO) | Convertida pela CBO 2002 (2.614 ocupações) + códigos especiais do SINAN | Parcial: 5 códigos sem descrição ficam como "Ocupação não catalogada" |
| Idade codificada (4 dígitos: unidade + quantidade) | Convertida para texto ("55 anos", "3 meses") | Aplicada; pode divergir do ano de nascimento (não corrigido) |
| Formato de datas | Padronizadas para AAAA-MM-DD; inválidas ficam "Não informado" | Aplicada |
| Semana epidemiológica (ano + semana) | Mantido apenas o número da semana (1–53) | Aplicada |
| Colunas sem uso analítico | 23 colunas administrativas/constantes removidas na leitura | Aplicada (a Bronze preserva o original) |
| Filtro de escorpião e região | `TP_ACIDENT = 3` e município de notificação **ou** residência entre os 12 da região | Aplicada |
| Duplicidades | Removidas apenas as linhas 100% idênticas | 34 grupos de possíveis duplicados mantidos, pendentes de validação |
| Datas fora do esperado | Mantidas sem alteração (4 acidentes em 2015; um outlier de data de sintomas em 2003) | A validar |
| Óbito sem data | Mantido "Não informado" | A validar com a fonte |
| Ordenação | Por data da notificação | Aplicada |

## INMET

| Problema | Regra aplicada |
|---|---|
| Layouts diferentes entre anos (vírgula × ponto e vírgula, codificações, cabeçalhos) | Detecção automática de delimitador, codificação e linha real do cabeçalho |
| Linhas de metadados no topo (ex.: dados da estação) | Cabeçalho real = primeira linha que começa com DATA, sem ":", e contém HORA/PRECIPITAÇÃO/TEMPERATURA/UMIDADE |
| Nomes de colunas variam por ano | Mapeamento por lista de padrões com prioridade e termos excluídos (MAX, MIN, ORVALHO, SENSAÇÃO) |
| Valores ausentes (`-9999` e vazios) | Convertidos para nulo; nunca viram zero. Dia sem leitura de chuva fica nulo (`min_count=1`) |
| Agregação diária | Precipitação = soma; temperatura e umidade = média. O dia é o dia UTC do arquivo |
| Datas repetidas entre arquivos | Mantida a linha com mais campos preenchidos |

## Gold

| Problema | Regra aplicada |
|---|---|
| Junção SINAN × INMET | `LEFT JOIN` por Data do Acidente = DATA; preserva todos os casos e a ordem; coluna "Clima Disponível" (Sim/Não) |
| Um único posto (Barueri A755) para toda a região | Sem correção; o clima é uma aproximação para os demais municípios |

## Pendências de validação

- Ano de nascimento e nº de ampolas: decidir se vazio passa de 0 para "Não informado".
- Conferir os 34 grupos de possíveis duplicados com a vigilância/fonte.
- Validar a tabela de escolaridade e os códigos 99999x de ocupação; 5 códigos CBO sem descrição.
- Confirmar o óbito sem data e o outlier de data de sintomas (2003).
- Decidir se o clima deve usar dia local (UTC-3) em vez de dia UTC.
- Decidir se os casos de 2015 (sem clima) ficam nas análises.
