# Testes Realizados - Reuniões

Este documento registra os casos de teste do incremento de reuniões: os testes automatizados do servidor e do aplicativo, o teste de ponta a ponta com o servidor e o aplicativo em execução e o roteiro de teste em aparelho.

---

## 1. Ambiente e Execução

| Projeto | Comando | Ferramenta |
|---|---|---|
| BACKEND | `npm test` | Vitest 4.1.11 com Node.js 24.14.0 |
| FRONTEND | `node --test tests/horarios.test.cjs` | Executor de testes do Node.js 24.14.0 |

Os testes automatizados do servidor executam as regras de negócio com as coleções simuladas em memória e com a data e hora atuais fixadas, sem depender de banco de dados ou de rede.

### Resultado da execução em 23/10/2026

| Conjunto | Casos de teste | Execuções | Aprovadas | Reprovadas |
|---|---|---|---|---|
| Reuniões, servidor | CT-01 a CT-23 | 26 | 26 | 0 |
| Servidor completo, todos os módulos | - | 182 | 182 | 0 |
| Sugestão de horários, aplicativo | CT-24 a CT-27 | 4 | 4 | 0 |
| Ponta a ponta | CT-28 e CT-29 | 2 | 2 | 0 |

O CT-06 executa quatro cenários, por isso os 23 casos do servidor correspondem a 26 execuções.

---

## 2. Massas de Dados

### Massa M1 - Regras de reunião

| Item | Valor |
|---|---|
| Data e hora atuais | 16/10/2026, 17:30 (UTC) |
| Comissão | Ativa, com dois integrantes: o responsável e um membro. |
| Organização | `APROVADA`, com o responsável e o membro com vínculo `APROVADO`. |
| Pessoa externa | Usuário que não integra a comissão. |
| Reunião existente | "Fechamento do orçamento", na "Sala do CA", de 16/10/2026 17:00 a 18:30 (UTC), pauta "Despesas do mês", situação `AGENDADA`, organizada pelo responsável. |
| Participantes da reunião | Responsável com convite `ACEITO` e membro com convite `PENDENTE`, ambos sem presença. |
| Nova reunião | Título "Fechamento do orçamento", local "Sala do CA", de 17/10/2026 17:00 a 18:00 (UTC), pauta "Despesas do mês". |

### Massa M2 - Lista de presença

| Item | Valor |
|---|---|
| Reunião | "Reunião de planejamento", de 01/01/2026 10:00 a 11:00 (UTC), situação `AGENDADA`, já encerrada. |
| Participantes | Ana, a responsável e organizadora, e Bruno, membro. Ambos com convite `ACEITO` e presença `CONFIRMADA` das 10:00 às 11:00. |

---

## 3. Casos de Teste Automatizados - Servidor

Arquivos de teste: `BACKEND/src/reunioes/reunioes.regras.spec.ts` (CT-01 a CT-21) e `BACKEND/src/reunioes/atas-presencas.spec.ts` (CT-22 e CT-23).

## CT-01 - Agendar reunião convidando toda a comissão

| Campo | Descrição |
|---|---|
| **ID do Caso de Teste** | CT-01 |
| **Requisito** | RF-01 - Agendar reunião |
| **Tipo** | Automatizado, teste de unidade |
| **Objetivo** | Verificar que, quando o responsável não escolhe convidados, todos os integrantes da comissão são convidados e o responsável fica registrado como organizador com convite aceito. |
| **Pré-Condições** | Massa M1. |
| **Dados de Entrada** | Nova reunião da massa M1, sem `convidados_ids`. |
| **Passos** | 1. O responsável solicita o agendamento da nova reunião. |
| **Resultado Esperado** | A reunião é registrada com local "Sala do CA", situação `AGENDADA` e o responsável em `criada_por`.<br>Os participantes são o responsável, com convite `ACEITO`, e o membro, com convite `PENDENTE`.<br>A resposta traz `sou_dono` igual a `true`. |
| **Resultado Obtido** | Conforme o esperado. |
| **Situação** | Aprovado |

## CT-02 - Agendar reunião com lista de convidados vazia

| Campo | Descrição |
|---|---|
| **ID do Caso de Teste** | CT-02 |
| **Requisito** | RF-01 - Agendar reunião |
| **Tipo** | Automatizado, teste de unidade |
| **Objetivo** | Verificar que uma lista de convidados vazia gera uma reunião somente com o organizador. |
| **Pré-Condições** | Massa M1. |
| **Dados de Entrada** | Nova reunião da massa M1 com `convidados_ids` vazio. |
| **Passos** | 1. O responsável solicita o agendamento. |
| **Resultado Esperado** | Os participantes registrados são apenas o responsável, com convite `ACEITO`. |
| **Resultado Obtido** | Conforme o esperado. |
| **Situação** | Aprovado |

## CT-03 - Agendar reunião com convidados escolhidos

| Campo | Descrição |
|---|---|
| **ID do Caso de Teste** | CT-03 |
| **Requisito** | RF-01 - Agendar reunião |
| **Tipo** | Automatizado, teste de unidade |
| **Objetivo** | Verificar que apenas os convidados escolhidos são registrados e que o organizador não se repete quando também é enviado na lista. |
| **Pré-Condições** | Massa M1. |
| **Dados de Entrada** | Nova reunião da massa M1 com `convidados_ids` contendo o membro e o próprio responsável. |
| **Passos** | 1. O responsável solicita o agendamento. |
| **Resultado Esperado** | Os participantes são, nesta ordem, o responsável e o membro, sem repetição. |
| **Resultado Obtido** | Conforme o esperado. |
| **Situação** | Aprovado |

## CT-04 - Recusar convidado de fora da comissão

| Campo | Descrição |
|---|---|
| **ID do Caso de Teste** | CT-04 |
| **Requisito** | RF-01 - Agendar reunião |
| **Tipo** | Automatizado, teste de unidade |
| **Objetivo** | Verificar que a reunião não é registrada quando um convidado não integra a comissão. |
| **Pré-Condições** | Massa M1. |
| **Dados de Entrada** | Nova reunião da massa M1 com `convidados_ids` contendo a pessoa externa. |
| **Passos** | 1. O responsável solicita o agendamento. |
| **Resultado Esperado** | O sistema responde `400` e nenhuma reunião é registrada. |
| **Resultado Obtido** | Conforme o esperado. |
| **Situação** | Aprovado |

## CT-05 - Validar a lista de convidados

| Campo | Descrição |
|---|---|
| **ID do Caso de Teste** | CT-05 |
| **Requisito** | RF-01 - Agendar reunião |
| **Tipo** | Automatizado, teste de unidade |
| **Objetivo** | Verificar as regras de formato do campo `convidados_ids`. |
| **Pré-Condições** | Nenhuma. |
| **Dados de Entrada** | Nova reunião da massa M1 com `convidados_ids` igual a: nulo; `["invalido"]`; o membro repetido duas vezes; o identificador do membro fora de uma lista; e lista vazia. |
| **Passos** | 1. Validar os dados de entrada de cada variação. |
| **Resultado Esperado** | As quatro primeiras variações são recusadas no campo `convidados_ids`. A lista vazia é aceita. |
| **Resultado Obtido** | Conforme o esperado. |
| **Situação** | Aprovado |

## CT-06 - Impedir agendamento sem permissão

| Campo | Descrição |
|---|---|
| **ID do Caso de Teste** | CT-06 |
| **Requisito** | RF-01 - Agendar reunião |
| **Tipo** | Automatizado, teste de unidade com quatro cenários |
| **Objetivo** | Verificar que a reunião não é registrada quando o usuário não tem permissão para agendar. |
| **Pré-Condições** | Massa M1, alterada em cada cenário. |
| **Dados de Entrada** | Nova reunião da massa M1 nos cenários: 1. usuário com papel `MEMBRO` na comissão; 2. comissão inativa; 3. organização `REVOGADA`; 4. conta do usuário desativada. |
| **Passos** | 1. O usuário solicita o agendamento em cada cenário. |
| **Resultado Esperado** | Cenários 1, 3 e 4: resposta `403`. Cenário 2: resposta `400`. Em nenhum cenário a reunião é registrada. |
| **Resultado Obtido** | Conforme o esperado nos quatro cenários. |
| **Situação** | Aprovado |

## CT-07 - Recusar horário inválido

| Campo | Descrição |
|---|---|
| **ID do Caso de Teste** | CT-07 |
| **Requisito** | RF-01 - Agendar reunião |
| **Tipo** | Automatizado, teste de unidade |
| **Objetivo** | Verificar que o sistema recusa término anterior ao início e reunião que já terminou. |
| **Pré-Condições** | Massa M1. |
| **Dados de Entrada** | 1. Nova reunião com término em 17/10/2026 16:00 (UTC), antes do início.<br>2. Nova reunião de 15/10/2026 10:00 a 11:00 (UTC), no passado. |
| **Passos** | 1. O responsável solicita o agendamento de cada reunião. |
| **Resultado Esperado** | As duas solicitações recebem `400`, e nenhuma reunião é registrada. |
| **Resultado Obtido** | Conforme o esperado. |
| **Situação** | Aprovado |

## CT-08 - Validar os campos da nova reunião

| Campo | Descrição |
|---|---|
| **ID do Caso de Teste** | CT-08 |
| **Requisito** | RF-01 - Agendar reunião |
| **Tipo** | Automatizado, teste de unidade |
| **Objetivo** | Verificar as regras de formato dos campos e a remoção de espaços nas extremidades da pauta. |
| **Pré-Condições** | Nenhuma. |
| **Dados de Entrada** | `comissao_id` "x", `titulo` " a ", `local` " ", `inicio` "16/10", `fim` vazio e `pauta` "  Despesas do mês\nPróximos passos  ". |
| **Passos** | 1. Validar os dados de entrada. |
| **Resultado Esperado** | São recusados os campos `comissao_id`, `titulo`, `local`, `inicio` e `fim`.<br>A pauta é aceita e fica "Despesas do mês\nPróximos passos". |
| **Resultado Obtido** | Conforme o esperado. |
| **Situação** | Aprovado |

## CT-09 - Recusar pauta inválida

| Campo | Descrição |
|---|---|
| **ID do Caso de Teste** | CT-09 |
| **Requisito** | RF-01 - Agendar reunião |
| **Tipo** | Automatizado, teste de unidade |
| **Objetivo** | Verificar que a pauta deve ser texto com no máximo 2000 caracteres. |
| **Pré-Condições** | Nenhuma. |
| **Dados de Entrada** | Reunião válida com pauta igual a: 1. a lista `["Item"]`; 2. um texto de 2001 caracteres. |
| **Passos** | 1. Validar os dados de entrada de cada variação. |
| **Resultado Esperado** | Nas duas variações, apenas o campo `pauta` é recusado. |
| **Resultado Obtido** | Conforme o esperado. |
| **Situação** | Aprovado |

## CT-10 - Listar reuniões do participante por período

| Campo | Descrição |
|---|---|
| **ID do Caso de Teste** | CT-10 |
| **Requisito** | RF-02 - Consultar agenda de reuniões |
| **Tipo** | Automatizado, teste de unidade |
| **Objetivo** | Verificar que a consulta seleciona apenas as reuniões do usuário e aplica os filtros de período e de comissão. |
| **Pré-Condições** | Massa M1. |
| **Dados de Entrada** | Usuário: o membro. Filtros: `de` 01/10/2026 00:00, `ate` 31/10/2026 23:59:59 (UTC) e `comissao_id` da comissão da massa. |
| **Passos** | 1. O membro consulta a agenda com os filtros. |
| **Resultado Esperado** | A consulta à coleção `reunioes` usa `participantes.usuario_id` igual ao membro, `fim` maior ou igual a `de`, `inicio` menor ou igual a `ate` e `comissao_id` igual ao informado. |
| **Resultado Obtido** | Conforme o esperado. |
| **Situação** | Aprovado |

## CT-11 - Bloquear quem não participa da reunião

| Campo | Descrição |
|---|---|
| **ID do Caso de Teste** | CT-11 |
| **Requisito** | RF-03 - Visualizar reunião |
| **Tipo** | Automatizado, teste de unidade |
| **Objetivo** | Verificar que um usuário que não consta entre os participantes não consegue visualizar a reunião. |
| **Pré-Condições** | Massa M1. |
| **Dados de Entrada** | Usuário: a pessoa externa. Reunião existente da massa M1. |
| **Passos** | 1. A pessoa externa solicita a reunião. |
| **Resultado Esperado** | O sistema responde `403`. |
| **Resultado Obtido** | Conforme o esperado. |
| **Situação** | Aprovado |

## CT-12 - Cancelar somente pelo organizador e antes do término

| Campo | Descrição |
|---|---|
| **ID do Caso de Teste** | CT-12 |
| **Requisito** | RF-04 - Cancelar reunião |
| **Tipo** | Automatizado, teste de unidade |
| **Objetivo** | Verificar que apenas o organizador cancela a reunião, que a gravação exige a reunião ainda agendada e não encerrada, e que uma reunião encerrada não é cancelada. |
| **Pré-Condições** | Massa M1. No passo 3, a data e hora atuais são 16/10/2026, 19:00 (UTC), depois do término. |
| **Dados de Entrada** | `situacao` igual a `CANCELADA`, enviada pelo membro e depois pelo responsável. |
| **Passos** | 1. O membro solicita o cancelamento.<br>2. O responsável solicita o cancelamento.<br>3. O responsável solicita o cancelamento depois do término. |
| **Resultado Esperado** | Passo 1: resposta `403`.<br>Passo 2: a reunião é alterada com as condições `situacao` igual a `AGENDADA` e `fim` posterior às 17:30, e o valor `situacao` igual a `CANCELADA`.<br>Passo 3: resposta `400`, sem gravação. |
| **Resultado Obtido** | Conforme o esperado. |
| **Situação** | Aprovado |

## CT-13 - Aceitar somente o cancelamento na alteração da reunião

| Campo | Descrição |
|---|---|
| **ID do Caso de Teste** | CT-13 |
| **Requisito** | RF-04 - Cancelar reunião |
| **Tipo** | Automatizado, teste de unidade |
| **Objetivo** | Verificar que a requisição de alteração da reunião aceita apenas a situação `CANCELADA`. |
| **Pré-Condições** | Nenhuma. |
| **Dados de Entrada** | 1. `situacao` igual a `CANCELADA`.<br>2. `titulo` "Outro título" e `inicio` 17/10/2026 10:00 (UTC), sem `situacao`.<br>3. `situacao` igual a `AGENDADA`. |
| **Passos** | 1. Validar os dados de entrada de cada variação. |
| **Resultado Esperado** | Variação 1: aceita.<br>Variações 2 e 3: recusadas no campo `situacao`. |
| **Resultado Obtido** | Conforme o esperado. |
| **Situação** | Aprovado |

## CT-14 - Responder convite

| Campo | Descrição |
|---|---|
| **ID do Caso de Teste** | CT-14 |
| **Requisito** | RF-05 - Responder convite |
| **Tipo** | Automatizado, teste de unidade |
| **Objetivo** | Verificar que o convidado registra a resposta e que o organizador, quem já registrou entrada e reuniões canceladas não aceitam a resposta. |
| **Pré-Condições** | Massa M1. Na resposta 3, o membro tem presença `AGUARDANDO` desde as 17:00. |
| **Dados de Entrada** | 1. O membro recusa o convite da reunião agendada.<br>2. O responsável, organizador da reunião, recusa o convite.<br>3. O membro, já com entrada registrada, recusa o convite.<br>4. O membro aceita o convite com a reunião cancelada. |
| **Passos** | 1. Enviar cada resposta. |
| **Resultado Esperado** | Resposta 1: o convite do membro passa para `RECUSADO`, com a condição de que ele continue sem presença.<br>Resposta 2: `403`.<br>Respostas 3 e 4: `400`. |
| **Resultado Obtido** | Conforme o esperado. |
| **Situação** | Aprovado |

## CT-15 - Registrar entrada do membro e do organizador

| Campo | Descrição |
|---|---|
| **ID do Caso de Teste** | CT-15 |
| **Requisito** | RF-06 - Registrar entrada |
| **Tipo** | Automatizado, teste de unidade |
| **Objetivo** | Verificar que a entrada do membro aguarda confirmação e que a do organizador já fica confirmada. |
| **Pré-Condições** | Massa M1, durante a reunião. |
| **Dados de Entrada** | Entrada do membro e, depois, do responsável. |
| **Passos** | 1. O membro registra a entrada.<br>2. O responsável registra a entrada. |
| **Resultado Esperado** | Passo 1: a presença é registrada com entrada às 17:30, situação `AGUARDANDO` e origem `PROPRIO`.<br>Passo 2: a presença é registrada com situação `CONFIRMADA`. |
| **Resultado Obtido** | Conforme o esperado. |
| **Situação** | Aprovado |

## CT-16 - Impedir entrada de quem recusou o convite

| Campo | Descrição |
|---|---|
| **ID do Caso de Teste** | CT-16 |
| **Requisito** | RF-06 - Registrar entrada |
| **Tipo** | Automatizado, teste de unidade |
| **Objetivo** | Verificar que o participante que recusou o convite não registra entrada. |
| **Pré-Condições** | Massa M1, durante a reunião, com o convite do membro `RECUSADO`. |
| **Dados de Entrada** | Entrada do membro. |
| **Passos** | 1. O membro registra a entrada. |
| **Resultado Esperado** | O sistema responde `400`, e nenhuma gravação é feita. |
| **Resultado Obtido** | Conforme o esperado. |
| **Situação** | Aprovado |

## CT-17 - Aceitar entrada somente durante a reunião

| Campo | Descrição |
|---|---|
| **ID do Caso de Teste** | CT-17 |
| **Requisito** | RF-06 - Registrar entrada |
| **Tipo** | Automatizado, teste de unidade |
| **Objetivo** | Verificar que a entrada só é aceita entre o início e o término de uma reunião não cancelada. |
| **Pré-Condições** | Massa M1. |
| **Dados de Entrada** | Entrada do membro com a data e hora atuais em: 1. 16:50, antes do início; 2. 17:00, no início; 3. 18:31, depois do término; 4. 17:30, com a reunião cancelada. |
| **Passos** | 1. O membro registra a entrada em cada momento. |
| **Resultado Esperado** | Momento 2: entrada registrada. Momentos 1, 3 e 4: resposta `400`. |
| **Resultado Obtido** | Conforme o esperado. |
| **Situação** | Aprovado |

## CT-18 - Impedir entrada repetida

| Campo | Descrição |
|---|---|
| **ID do Caso de Teste** | CT-18 |
| **Requisito** | RF-06 - Registrar entrada |
| **Tipo** | Automatizado, teste de unidade |
| **Objetivo** | Verificar que o participante não registra a entrada duas vezes. |
| **Pré-Condições** | Massa M1, com o membro já tendo presença `AGUARDANDO` desde as 17:00. |
| **Dados de Entrada** | Nova entrada do membro. |
| **Passos** | 1. O membro registra a entrada novamente. |
| **Resultado Esperado** | O sistema responde `409`, e nenhuma gravação é feita. |
| **Resultado Obtido** | Conforme o esperado. |
| **Situação** | Aprovado |

## CT-19 - Registrar saída

| Campo | Descrição |
|---|---|
| **ID do Caso de Teste** | CT-19 |
| **Requisito** | RF-07 - Registrar saída |
| **Tipo** | Automatizado, teste de unidade |
| **Objetivo** | Verificar que a saída exige entrada e que a hora registrada não passa do término. |
| **Pré-Condições** | Massa M1. No passo 2, o membro tem presença desde as 17:00 e a data e hora atuais são 19:00. |
| **Dados de Entrada** | 1. Saída do membro sem entrada.<br>2. Saída do membro com entrada, depois do término. |
| **Passos** | 1. O membro registra a saída sem entrada.<br>2. O membro registra a saída com entrada. |
| **Resultado Esperado** | Passo 1: resposta `400`.<br>Passo 2: a saída é registrada às 18:30, horário do término. |
| **Resultado Obtido** | Conforme o esperado. |
| **Situação** | Aprovado |

## CT-20 - Marcar presente quem não registrou entrada

| Campo | Descrição |
|---|---|
| **ID do Caso de Teste** | CT-20 |
| **Requisito** | RF-08 - Confirmar ou recusar presença |
| **Tipo** | Automatizado, teste de unidade |
| **Objetivo** | Verificar que apenas o organizador marca como presente quem não registrou entrada, e somente depois do início. |
| **Pré-Condições** | Massa M1, com o membro sem presença. |
| **Dados de Entrada** | 1. O membro tenta confirmar a própria presença.<br>2. O responsável confirma às 16:50, antes do início.<br>3. O responsável recusa a presença do membro, que não tem entrada.<br>4. O responsável confirma às 17:30. |
| **Passos** | 1. Enviar cada solicitação na ordem dos dados de entrada. |
| **Resultado Esperado** | Solicitação 1: `403`. Solicitações 2 e 3: `400`.<br>Solicitação 4: a presença do membro é criada com entrada às 17:00, situação `CONFIRMADA` e origem `RESPONSAVEL`. |
| **Resultado Obtido** | Conforme o esperado. |
| **Situação** | Aprovado |

## CT-21 - Confirmar entrada que aguardava

| Campo | Descrição |
|---|---|
| **ID do Caso de Teste** | CT-21 |
| **Requisito** | RF-08 - Confirmar ou recusar presença |
| **Tipo** | Automatizado, teste de unidade |
| **Objetivo** | Verificar que o organizador confirma uma presença `AGUARDANDO`. |
| **Pré-Condições** | Massa M1, com o membro tendo presença `AGUARDANDO` desde as 17:00. |
| **Dados de Entrada** | `situacao` igual a `CONFIRMADA` para o membro. |
| **Passos** | 1. O responsável confirma a presença do membro. |
| **Resultado Esperado** | A situação da presença do membro passa para `CONFIRMADA`. |
| **Resultado Obtido** | Conforme o esperado. |
| **Situação** | Aprovado |

## CT-22 - Gerar lista de presença

| Campo | Descrição |
|---|---|
| **ID do Caso de Teste** | CT-22 |
| **Requisito** | RF-09 - Gerar lista de presença |
| **Tipo** | Automatizado, teste de unidade |
| **Objetivo** | Verificar os nomes, as situações e a duração da lista, sem criar presença para quem não registrou entrada. |
| **Pré-Condições** | Massa M2, com Bruno sem presença. |
| **Dados de Entrada** | Solicitação da lista por Ana. |
| **Passos** | 1. Ana solicita a lista de presença. |
| **Resultado Esperado** | Ana aparece com situação `CONFIRMADA` e 60 minutos.<br>Bruno aparece com situação `SEM_REGISTRO`, entrada, saída e minutos vazios. |
| **Resultado Obtido** | Conforme o esperado. |
| **Situação** | Aprovado |

## CT-23 - Recusar a lista de presença para quem não participa

| Campo | Descrição |
|---|---|
| **ID do Caso de Teste** | CT-23 |
| **Requisito** | RF-09 - Gerar lista de presença |
| **Tipo** | Automatizado, teste de unidade |
| **Objetivo** | Verificar que um usuário que não consta entre os participantes não obtém a lista de presença. |
| **Pré-Condições** | Massa M2. |
| **Dados de Entrada** | Solicitação da lista por um usuário que não participa da reunião. |
| **Passos** | 1. O usuário solicita a lista de presença. |
| **Resultado Esperado** | O sistema responde `403`. |
| **Resultado Obtido** | Conforme o esperado. |
| **Situação** | Aprovado |

---

## 4. Casos de Teste Automatizados - Aplicativo

Arquivo de teste: `FRONTEND/tests/horarios.test.cjs`. Os casos verificam as sugestões de horário dos campos Início e Término da tela de agendamento.

## CT-24 - Sugerir início a partir da próxima meia hora

| Campo | Descrição |
|---|---|
| **ID do Caso de Teste** | CT-24 |
| **Requisito** | RF-01 - Agendar reunião |
| **Tipo** | Automatizado, teste de unidade |
| **Objetivo** | Verificar que, para o dia atual, a primeira sugestão de início é o próximo horário de meia hora. |
| **Pré-Condições** | Data e hora atuais: 20/10/2026, 14:12. |
| **Dados de Entrada** | Dia escolhido: 20/10/2026. |
| **Passos** | 1. Gerar as sugestões de início. |
| **Resultado Esperado** | A primeira sugestão é 14:30. |
| **Resultado Obtido** | Conforme o esperado. |
| **Situação** | Aprovado |

## CT-25 - Sugerir início em data futura

| Campo | Descrição |
|---|---|
| **ID do Caso de Teste** | CT-25 |
| **Requisito** | RF-01 - Agendar reunião |
| **Tipo** | Automatizado, teste de unidade |
| **Objetivo** | Verificar que, para uma data futura, as sugestões começam no horário comercial, independentemente da hora atual. |
| **Pré-Condições** | Data e hora atuais: 20/10/2026, 23:50. |
| **Dados de Entrada** | Dia escolhido: 21/10/2026. |
| **Passos** | 1. Gerar as sugestões de início. |
| **Resultado Esperado** | A primeira sugestão é 08:00. |
| **Resultado Obtido** | Conforme o esperado. |
| **Situação** | Aprovado |

## CT-26 - Sugerir término dentro do mesmo dia

| Campo | Descrição |
|---|---|
| **ID do Caso de Teste** | CT-26 |
| **Requisito** | RF-01 - Agendar reunião |
| **Tipo** | Automatizado, teste de unidade |
| **Objetivo** | Verificar que as sugestões de término usam durações de 30 a 180 minutos e não passam para o dia seguinte. |
| **Pré-Condições** | Dia escolhido: 20/10/2026. |
| **Dados de Entrada** | 1. Início às 23:00.<br>2. Início às 23:40. |
| **Passos** | 1. Gerar as sugestões de término para cada início. |
| **Resultado Esperado** | Início 23:00: única sugestão 23:30.<br>Início 23:40: nenhuma sugestão. |
| **Resultado Obtido** | Conforme o esperado. |
| **Situação** | Aprovado |

## CT-27 - Não sugerir horários no fim do dia

| Campo | Descrição |
|---|---|
| **ID do Caso de Teste** | CT-27 |
| **Requisito** | RF-01 - Agendar reunião |
| **Tipo** | Automatizado, teste de unidade |
| **Objetivo** | Verificar que o sistema não sugere 24:00 nem horários do dia seguinte. |
| **Pré-Condições** | Data e hora atuais: 20/10/2026, 23:59. |
| **Dados de Entrada** | Dia escolhido: 20/10/2026. |
| **Passos** | 1. Gerar as sugestões de início. |
| **Resultado Esperado** | Nenhuma sugestão. |
| **Resultado Obtido** | Conforme o esperado. |
| **Situação** | Aprovado |

---

## 5. Casos de Teste de Ponta a Ponta

## CT-28 - Fluxo completo com servidor e aplicativo em execução

| Campo | Descrição |
|---|---|
| **ID do Caso de Teste** | CT-28 |
| **Requisito** | RF-01, RF-03, RF-09 e RNF-01 |
| **Tipo** | Automatizado, ponta a ponta |
| **Objetivo** | Verificar o funcionamento integrado da API e das telas com um banco de dados real. |
| **Pré-Condições** | Servidor e versão web do aplicativo em execução local, com um banco MongoDB temporário, apagado ao final. Um responsável e um membro cadastrados na mesma comissão. Envio de e-mails desativado. |
| **Dados de Entrada** | Reunião "Reunião de teste", na "Sala 1", para o dia seguinte das 10:00 às 11:00, com o membro como único convidado escolhido. |
| **Passos** | 1. Enviar uma requisição sem o cabeçalho `X-API-Key`.<br>2. Enviar uma requisição com uma chave incorreta.<br>3. O responsável agenda a reunião.<br>4. Para simular uma reunião encerrada, alterar no banco o horário da reunião para o dia anterior e registrar a presença `CONFIRMADA` do membro.<br>5. O responsável consulta a lista de presença.<br>6. Abrir as telas Agenda, Comissão, Nova reunião, Reunião e Lista de presença em um navegador sem interface com tela de celular (390 × 844). |
| **Resultado Esperado** | Passos 1 e 2: resposta `401`.<br>Passo 3: reunião com 2 participantes, o organizador e o convidado.<br>Passo 5: a lista traz o membro com presença `CONFIRMADA`.<br>Passo 6: as telas abrem sem erro no console e sem resposta de erro da API. |
| **Resultado Obtido** | Conforme o esperado. |
| **Situação** | Aprovado |

## CT-29 - Bloquear origens não autorizadas

| Campo | Descrição |
|---|---|
| **ID do Caso de Teste** | CT-29 |
| **Requisito** | RNF-03 - Origens Autorizadas |
| **Tipo** | Manual, requisição HTTP |
| **Objetivo** | Verificar que a API só autoriza chamadas de navegador das origens permitidas. |
| **Pré-Condições** | Servidor em execução local. |
| **Dados de Entrada** | `OPTIONS /reunioes` com `Access-Control-Request-Method: GET` e: 1. `Origin: https://exemplo.com`; 2. `Origin: http://localhost:8081`. |
| **Passos** | 1. Enviar cada requisição e ler os cabeçalhos da resposta. |
| **Resultado Esperado** | Requisição 1: resposta sem `Access-Control-Allow-Origin`.<br>Requisição 2: `Access-Control-Allow-Origin` igual a `http://localhost:8081`. |
| **Resultado Obtido** | Requisição 1: `204`, sem `Access-Control-Allow-Origin`. Requisição 2: `204`, com `Access-Control-Allow-Origin: http://localhost:8081`. |
| **Situação** | Aprovado |

---

## 6. Roteiro de Teste em Aparelho

Executado na apresentação, em um aparelho Android e um aparelho iOS, com o aplicativo conectado ao servidor. As colunas de resultado são preenchidas durante a execução.

| ID | Requisito | Passos | Resultado esperado | Android | iOS |
|---|---|---|---|---|---|
| CT-30 | RF-01 | Como responsável, agendar uma reunião para hoje com dois convidados e pauta. | A tela da reunião abre com os dados informados. | | |
| CT-31 | RF-02 | Abrir a Agenda, trocar de dia e expandir o mês. | O calendário marca o dia da reunião, e a lista mostra as reuniões do dia escolhido. | | |
| CT-32 | RF-05 | Como convidado, aceitar um convite pela agenda e recusar outro pela tela da reunião. | Os convites aparecem como aceito e recusado. | | |
| CT-33 | RF-06, RF-07 | Durante a reunião, marcar entrada e depois saída. | O cartão "Sua presença" mostra a entrada e depois o período. | | |
| CT-34 | RF-08 | Como organizador, confirmar uma entrada e marcar presente quem não registrou. | Os dois participantes aparecem em Presentes. | | |
| CT-35 | RF-09 | Depois do término, abrir a lista de presença e compartilhá-la. | A lista mostra presentes e ausentes, e o menu de compartilhamento abre. | | |
| CT-36 | RF-04 | Agendar outra reunião e cancelá-la. | A reunião aparece como cancelada. | | |
| CT-37 | RNF-08 | Com a reunião aberta em dois aparelhos, marcar entrada em um deles. | A entrada aparece no outro aparelho em até 30 segundos. | | |
| CT-38 | RF-01 | Entrar com um usuário que não é responsável de nenhuma comissão e abrir a Agenda. | A opção "Nova reunião" não aparece. | | |
