# Requisitos funcionais — Reuniões

> Padrão do Incremento 2 · Módulo REU

## Resumo

Define RF-REU-01 a RF-REU-12 com comportamento, entradas, processamento, saídas, pré-condições, pós-condições e efeitos observáveis.

## RF-REU-01 — Agendar reunião

| Campo                  | Descrição                                                                                                                             |
| ---------------------- | ------------------------------------------------------------------------------------------------------------------------------------- |
| **Descrição**          | O sistema deve permitir que o responsável agende reunião para sua comissão.                                                           |
| **Entradas**           | `comissao_id`, `titulo` (3–120), `local` (2–120), `inicio`, `fim`, `pauta?` (até 2.000) e `convidados_ids?`.                          |
| **Origem**             | Tela Nova reunião.                                                                                                                    |
| **Saída**              | Reunião criada com situação `AGENDADA`.                                                                                               |
| **Destino**            | `POST /reunioes`, coleção `reunioes` e tela de reunião.                                                                               |
| **Ação**               | Valida responsável, comissão, organização, horário e convidados; inclui o organizador com convite aceito e os demais como `PENDENTE`. |
| **Pré-condição**       | Usuário ativo, responsável de comissão ativa e aprovado em organização aprovada.                                                      |
| **Pós-condição**       | Reunião aparece na agenda de todos os participantes.                                                                                  |
| **Efeitos colaterais** | Cria documento e timestamps.                                                                                                          |
| **Erros**              | 400 dados/horário; 403 autorização; 404 comissão; 409 gravação concorrente.                                                           |

## RF-REU-02 — Consultar agenda

| Campo                  | Descrição                                                                        |
| ---------------------- | -------------------------------------------------------------------------------- |
| **Descrição**          | O sistema deve listar reuniões em que o usuário consta como participante.        |
| **Entradas**           | Filtros opcionais `de`, `ate` e `comissao_id`.                                   |
| **Origem**             | Agenda ou aba Reuniões da comissão.                                              |
| **Saída**              | Lista ordenada por `inicio`, com comissão, participantes e `sou_dono`.           |
| **Destino**            | `GET /reunioes` e telas de agenda/comissão.                                      |
| **Ação**               | Filtra por participante e período, popula referências e ordena cronologicamente. |
| **Pré-condição**       | Usuário autenticado e ativo.                                                     |
| **Pós-condição**       | Reuniões permitidas exibidas sem alteração.                                      |
| **Efeitos colaterais** | Nenhum.                                                                          |
| **Erros**              | 400 filtro/ID inválido; 401/403 autenticação.                                    |

## RF-REU-03 — Visualizar reunião

| Campo                  | Descrição                                                                   |
| ---------------------- | --------------------------------------------------------------------------- |
| **Descrição**          | O sistema deve exibir dados completos da reunião somente aos participantes. |
| **Entradas**           | ID da reunião.                                                              |
| **Origem**             | Seleção na agenda, início ou comissão.                                      |
| **Saída**              | Reunião populada, estado calculado e situação da ata.                       |
| **Destino**            | `GET /reunioes/:id` e Tela Reunião.                                         |
| **Ação**               | Verifica participação, popula comissão/usuários e informa `sou_dono`.       |
| **Pré-condição**       | Usuário participante e ativo.                                               |
| **Pós-condição**       | Dados exibidos; a tela atualiza a consulta a cada 30 segundos.              |
| **Efeitos colaterais** | Nenhum.                                                                     |
| **Erros**              | 403 não participante; 404 reunião.                                          |

## RF-REU-04 — Cancelar reunião

| Campo                  | Descrição                                                                           |
| ---------------------- | ----------------------------------------------------------------------------------- |
| **Descrição**          | O sistema deve permitir que o organizador cancele reunião ainda não encerrada.      |
| **Entradas**           | ID e `situacao: CANCELADA`.                                                         |
| **Origem**             | Confirmação na Tela Reunião.                                                        |
| **Saída**              | Reunião atualizada.                                                                 |
| **Destino**            | `PATCH /reunioes/:id` e coleção `reunioes`.                                         |
| **Ação**               | Altera apenas a situação, exigindo organizador, estado `AGENDADA` e término futuro. |
| **Pré-condição**       | Usuário criou a reunião.                                                            |
| **Pós-condição**       | Situação `CANCELADA`; convites, presença e ata ficam bloqueados.                    |
| **Efeitos colaterais** | Atualiza timestamp.                                                                 |
| **Erros**              | 400 estado/término; 403 autoria; 404 reunião; 409 estado concorrente.               |

## RF-REU-05 — Responder convite

| Campo                  | Descrição                                                                                  |
| ---------------------- | ------------------------------------------------------------------------------------------ |
| **Descrição**          | O sistema deve registrar aceite ou recusa do participante.                                 |
| **Entradas**           | ID e `aceito` booleano.                                                                    |
| **Origem**             | Agenda, início ou Tela Reunião.                                                            |
| **Saída**              | Reunião com convite `ACEITO` ou `RECUSADO`.                                                |
| **Destino**            | `PATCH /reunioes/:id/convite`.                                                             |
| **Ação**               | Localiza o participante e atualiza sua resposta com condição concorrente.                  |
| **Pré-condição**       | Reunião agendada e futura; usuário convidado e não organizador; sem presença para recusar. |
| **Pós-condição**       | Resposta persistida.                                                                       |
| **Efeitos colaterais** | Atualiza timestamp.                                                                        |
| **Erros**              | 400 estado/presença; 403 organizador ou usuário externo; 409 alteração concorrente.        |

## RF-REU-06 — Registrar entrada

| Campo                  | Descrição                                                                         |
| ---------------------- | --------------------------------------------------------------------------------- |
| **Descrição**          | O sistema deve registrar uma única entrada durante a reunião.                     |
| **Entradas**           | ID da reunião e usuário autenticado.                                              |
| **Origem**             | Ação “Marcar entrada”.                                                            |
| **Saída**              | Presença com `entrada`, `marcada_por` e `situacao`.                               |
| **Destino**            | `POST /reunioes/:id/entrada`.                                                     |
| **Ação**               | Registra horário atual; organizador fica `CONFIRMADA`, demais ficam `AGUARDANDO`. |
| **Pré-condição**       | Convite não recusado; reunião em andamento; presença ausente.                     |
| **Pós-condição**       | Presença criada para o participante.                                              |
| **Efeitos colaterais** | Atualiza timestamp.                                                               |
| **Erros**              | 400 janela/convite; 403 não participante; 409 entrada duplicada.                  |

## RF-REU-07 — Registrar saída

| Campo                  | Descrição                                                            |
| ---------------------- | -------------------------------------------------------------------- |
| **Descrição**          | O sistema deve registrar a saída do participante que possui entrada. |
| **Entradas**           | ID da reunião e usuário autenticado.                                 |
| **Origem**             | Ação “Marcar saída”.                                                 |
| **Saída**              | Presença com `saida`.                                                |
| **Destino**            | `POST /reunioes/:id/saida`.                                          |
| **Ação**               | Usa o menor horário entre o instante atual e `fim`.                  |
| **Pré-condição**       | Entrada existente e saída ausente.                                   |
| **Pós-condição**       | Período de permanência encerrado.                                    |
| **Efeitos colaterais** | Atualiza timestamp.                                                  |
| **Erros**              | 400 sem entrada; 403 não participante; 409 saída duplicada.          |

## RF-REU-08 — Confirmar ou recusar presença

| Campo                  | Descrição                                                                                         |
| ---------------------- | ------------------------------------------------------------------------------------------------- |
| **Descrição**          | O sistema deve permitir ao organizador validar presença.                                          |
| **Entradas**           | ID da reunião, ID do participante e `situacao` (`CONFIRMADA` ou `RECUSADA`).                      |
| **Origem**             | Lista de participantes da Tela Reunião.                                                           |
| **Saída**              | Presença atualizada.                                                                              |
| **Destino**            | `PATCH /reunioes/:id/presencas/:usuarioId`.                                                       |
| **Ação**               | Atualiza presença existente ou cria presença confirmada, com origem `RESPONSAVEL`, após o início. |
| **Pré-condição**       | Usuário é organizador; reunião não cancelada e iniciada.                                          |
| **Pós-condição**       | Situação final persistida.                                                                        |
| **Efeitos colaterais** | Pode criar entrada igual ao início para presença confirmada pelo responsável.                     |
| **Erros**              | 400 momento/estado; 403 autoria; 404 participante; 409 concorrência.                              |

## RF-REU-09 — Gerar lista de presença

| Campo                  | Descrição                                                                                      |
| ---------------------- | ---------------------------------------------------------------------------------------------- |
| **Descrição**          | O sistema deve gerar relatório sem criar registros ausentes.                                   |
| **Entradas**           | ID da reunião.                                                                                 |
| **Origem**             | Aba Presença dos registros.                                                                    |
| **Saída**              | Título, período, geração e participantes com situação, horários e minutos.                     |
| **Destino**            | `GET /reunioes/:id/lista-presenca` e Tela Registros.                                           |
| **Ação**               | Calcula minutos inteiros quando há entrada e saída; usa `SEM_REGISTRO` quando não há presença. |
| **Pré-condição**       | Usuário participa da reunião.                                                                  |
| **Pós-condição**       | Relatório exibido/compartilhável.                                                              |
| **Efeitos colaterais** | Nenhum.                                                                                        |
| **Erros**              | 403 não participante; 404 reunião.                                                             |

## RF-REU-10 — Salvar rascunho da ata

| Campo                  | Descrição                                                                 |
| ---------------------- | ------------------------------------------------------------------------- |
| **Descrição**          | O sistema deve criar ou atualizar a ata com controle otimista de versão.  |
| **Entradas**           | `conteudo` (20–20.000) e `versao` atualmente exibida.                     |
| **Origem**             | Aba Ata da Tela Registros.                                                |
| **Saída**              | Ata em `RASCUNHO`, com versão incrementada.                               |
| **Destino**            | `PATCH /reunioes/:id/ata` e subdocumento `ata`.                           |
| **Ação**               | Confere responsável, encerramento, situação e versão; grava autor e data. |
| **Pré-condição**       | Reunião encerrada, não cancelada; ata inexistente ou rascunho.            |
| **Pós-condição**       | Nova versão persistida.                                                   |
| **Efeitos colaterais** | Incrementa `ata.versao` e atualiza `ata.atualizada_em`.                   |
| **Erros**              | 400 conteúdo/estado; 403 autoria; 409 versão divergente.                  |

## RF-REU-11 — Enviar ata para aprovação

| Campo                  | Descrição                                                                                         |
| ---------------------- | ------------------------------------------------------------------------------------------------- |
| **Descrição**          | O sistema deve publicar o rascunho para aprovação dos presentes confirmados.                      |
| **Entradas**           | Versão atual da ata.                                                                              |
| **Origem**             | Confirmação na Aba Ata.                                                                           |
| **Saída**              | Ata em `EM_APROVACAO`.                                                                            |
| **Destino**            | `POST /reunioes/:id/ata/publicar`.                                                                |
| **Ação**               | Congela o texto, copia participantes `CONFIRMADA` para `aprovadores` e inicia `aprovacoes` vazia. |
| **Pré-condição**       | Responsável, reunião encerrada, rascunho atual e ao menos um presente confirmado.                 |
| **Pós-condição**       | Ata deixa de ser editável.                                                                        |
| **Efeitos colaterais** | Fixa o conjunto de aprovadores da versão.                                                         |
| **Erros**              | 400 sem rascunho/aprovador; 403 autoria; 409 versão divergente.                                   |

## RF-REU-12 — Aprovar ata

| Campo                  | Descrição                                                                                               |
| ---------------------- | ------------------------------------------------------------------------------------------------------- |
| **Descrição**          | O sistema deve registrar aprovação individual e concluir por unanimidade.                               |
| **Entradas**           | Versão atual da ata.                                                                                    |
| **Origem**             | Ação “Aprovar ata”.                                                                                     |
| **Saída**              | Ata com lista de aprovações e situação atualizada.                                                      |
| **Destino**            | `POST /reunioes/:id/ata/aprovar`.                                                                       |
| **Ação**               | Inclui o usuário uma única vez; quando todos os aprovadores constam em `aprovacoes`, define `APROVADA`. |
| **Pré-condição**       | Ata `EM_APROVACAO`; usuário em `aprovadores`; versão atual.                                             |
| **Pós-condição**       | Aprovação registrada ou ata concluída.                                                                  |
| **Efeitos colaterais** | Repetição pelo mesmo usuário é idempotente.                                                             |
| **Erros**              | 400 estado; 403 não aprovador; 409 versão divergente/escrita concorrente.                               |
