# Dicionário de dados: camada Gold (casos + clima)

Arquivo gerado: `3_gold/gold_casos_clima_2016_2026.csv` (consulta em `gold_escorpioes_clima.sql`). Cada linha é um caso de escorpionismo do SINAN acrescido do clima do dia do acidente (INMET, estação Barueri A755), por `LEFT JOIN` em Data do Acidente = DATA.

**7.181 casos × 56 colunas; 7.176 com clima (99,9%).** As 52 primeiras colunas são idênticas à Silver do SINAN; as 4 últimas são o clima.

## Convenções

- **Ignorado:** código 9 da ficha SINAN (informação não obtida).
- **Não informado:** campo em branco na fonte.
- **Não se aplica:** pulo lógico da ficha.
- Datas no formato AAAA-MM-DD; separador do CSV: ponto e vírgula; codificação UTF-8 com BOM.

## Campos do SINAN

| Campo (cabeçalho) | Código SINAN | Tipo | Descrição | Observações |
|---|---|---|---|---|
| Data da Notificação | DT_NOTIFIC | Data | Data em que o caso foi notificado | Ordenação da base. Sem vazios |
| Semana da Notificação | SEM_NOT | Inteiro (1–53) | Semana epidemiológica da notificação | O ano fica em coluna própria |
| Ano da Notificação | NU_ANO | Inteiro | Ano da notificação | 2016 a 2026 (2026 parcial, até set/2026) |
| Município de Notificação | ID_MUNICIP | Texto | Município da unidade notificadora (convertido do código IBGE) | Fora de SP recebe sufixo da UF; "Município ignorado (UF)" = código inexistente |
| Data dos Primeiros Sintomas | DT_SIN_PRI | Data | Início dos sintomas | Há 1 outlier (2003), mantido |
| Semana dos Primeiros Sintomas | SEM_PRI | Inteiro (1–53) | Semana epidemiológica do início dos sintomas | |
| Ano de Nascimento | ANO_NASC | Inteiro | Ano de nascimento do paciente | 28 registros com 0 (ausente), a validar |
| Idade | NU_IDADE_N | Texto | Idade decodificada (original: 4 dígitos = unidade + quantidade) | Pode divergir do ano de nascimento; não corrigido |
| Sexo | CS_SEXO | Categórico | Sexo do paciente | Masculino, Feminino, Ignorado |
| Gestante | CS_GESTANT | Categórico | Situação gestacional | Trimestres, "Idade gestacional ignorada", Não, Não se aplica, Ignorado |
| Raça/Cor | CS_RACA | Categórico | Raça/cor autodeclarada | Branca, Preta, Amarela, Parda, Indígena, Ignorado |
| Escolaridade | CS_ESCOL_N | Categórico | Nível de escolaridade | Tabela de códigos ainda não validada com a fonte oficial |
| Município de Residência | ID_MN_RESI | Texto | Município de residência (chave do filtro regional) | |
| Data da Investigação | DT_INVEST | Data | Data da investigação do caso | |
| Ocupação | ID_OCUPA_N | Texto | Ocupação (CBO 2002 + códigos especiais do SINAN) | Cerca de 58% "Não informado"; 5 códigos CBO não catalogados |
| Data do Acidente | ANT_DT_ACI | Data | Data da picada. **Chave de junção com o clima** | Mínimo em 2015 (4 casos, sem clima); máximo em set/2026 |
| Município de Ocorrência | ANT_MUNIC_ | Texto | Município onde ocorreu o acidente | |
| Tempo entre Picada e Atendimento | ANT_TEMPO_ | Categórico | Intervalo até o atendimento | 0 a 1h, 1 a 3h, 3 a 6h, 6 a 12h, 12 a 24h, 24h ou mais, Ignorado |
| Local da Picada | ANT_LOCA_1 | Categórico | Parte do corpo atingida | Cabeça, Braço, Antebraço, Mão, Dedo da mão, Tronco, Coxa, Perna, Pé, Dedo do pé, Ignorado |
| Manifestação Local | MCLI_LOCAL | Sim/Não | Houve manifestação local? | |
| Dor, Edema, Equimose, Necrose, Outras Manifestações Locais | CLI_DOR, CLI_EDEMA, CLI_EQUIMO, CLI_NECROS, CLI_LOCAL_ | Sim/Não | Manifestações locais específicas | |
| Especificação de Outras Manifestações Locais | CLI_LOCA_1 | Texto livre | Descrição da manifestação local adicional | Digitação livre, sem padronização |
| Manifestação Sistêmica | MCLI_SIST | Sim/Não | Houve manifestação sistêmica? | |
| Neuroparalítica, Hemorrágica, Vagal, Miolítica/Hemolítica, Renal | CLI_NEURO, CLI_HEMORR, CLI_VAGAIS, CLI_MIOLIT, CLI_RENAL | Sim/Não | Tipo de manifestação sistêmica | Em branco quando não há manifestação sistêmica (≈95% "Não informado": pulo lógico) |
| Tempo de Coagulação | CLI_TEMPO_ | Categórico | Resultado do tempo de coagulação | Normal, Alterado, Não realizado |
| Tipo de Animal | TP_ACIDENT | Categórico | Animal causador | Constante ("Escorpião") após o filtro |
| Classificação do Caso | TRA_CLASSI | Categórico | Gravidade clínica | Leve, Moderado, Grave, Ignorado |
| Soroterapia Aplicada | CON_SOROTE | Sim/Não | Foi aplicado soro antiescorpiônico? | |
| Nº de Ampolas de Soro Antiescorpiônico | NU_AMPOL_9 | Inteiro | Quantidade de ampolas | Vazio vira 0; 80 casos com soro = Sim e 0 ampolas, a validar |
| Complicação Local | COM_LOC | Sim/Não | Houve complicação local? | |
| Infecção Secundária, Necrose Extensa, Síndrome Comportamental, Déficit Funcional, Amputação | COM_SECUND, COM_NECROS, COM_COMPOR, COM_DEFICT, COM_APUTAC | Sim/Não | Tipo de complicação local | Em branco quando não há complicação local (pulo lógico) |
| Complicação Sistêmica | COM_SISTEM | Sim/Não | Houve complicação sistêmica? | |
| Insuficiência Renal, Insuficiência Respiratória/Edema Pulmonar, Septicemia, Choque | COM_RENAL, COM_EDEMA, COM_SEPTIC, COM_CHOQUE | Sim/Não | Tipo de complicação sistêmica | Em branco quando não há complicação sistêmica (pulo lógico) |
| Acidente Relacionado ao Trabalho | DOENCA_TRA | Sim/Não | O acidente ocorreu em atividade de trabalho? | |
| Evolução do Caso | EVOLUCAO | Categórico | Desfecho | Cura, Óbito por acidente por animal peçonhento, Óbito por outra causa, Ignorado |
| Data do Óbito | DT_OBITO | Data | Data do óbito, quando aplicável | 3 óbitos e só 2 datas |
| Data de Encerramento | DT_ENCERRA | Data | Data de encerramento do caso | 263 "Não informado" |

"Sim/Não" = Sim, Não, Ignorado ou Não informado.

## Campos de clima (INMET, estação Barueri A755)

| Campo | Tipo | Descrição | Observações |
|---|---|---|---|
| Precipitação Total (mm) | Decimal | Chuva acumulada no dia do acidente (soma das 24 leituras horárias, dia UTC) | Nulo se o dia não tem nenhuma leitura |
| Temperatura Média (°C) | Decimal | Média das temperaturas horárias do ar (bulbo seco) no dia | 1 casa decimal |
| Umidade Relativa Média (%) | Decimal | Média da umidade relativa horária no dia | Arredondada em 0 casas |
| Clima Disponível | Categórico | Indica se houve dado de clima para a data do acidente | Sim ou Não; "Não" = 5 casos (4 de 2015 + 1 sem dia correspondente) |
