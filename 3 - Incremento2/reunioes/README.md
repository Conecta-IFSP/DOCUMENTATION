# Reuniões

> Padrão da disciplina · Módulo REU · Incremento 2 · Revisado em 26/11/2026

## Resumo

O presidente da comissão agenda as reuniões da comissão, escolhe quem convidar, escreve a pauta e anexa arquivos para leitura prévia. Cada convidado vê a reunião na agenda e aceita ou recusa o convite. Durante a reunião, cada participante marca a própria entrada e a própria saída, confirmando a identidade no aparelho, e quem agendou confirma ou recusa essas presenças. Quem marcou entrada e não marcou saída recebe a saída automaticamente no término da reunião. O presidente também pode gerar um link de presença, válido só durante a reunião, com o qual cada convidado registra entrada e saída já confirmadas, e pode corrigir os horários de um participante, informando um motivo que fica guardado num histórico. Depois do término, qualquer participante consulta a lista de presença e gera o PDF dela.

## Escopo

- agendar, consultar, visualizar, editar e cancelar reuniões, inclusive de dia inteiro e sem local;
- responder ao convite, inclusive desistir depois de aceitar;
- registrar entrada e saída, com a saída automática no término;
- confirmar ou recusar a presença registrada por um participante;
- gerar, enviar e revogar o link de presença, e registrar presença por ele;
- corrigir horários de entrada e saída, com motivo e histórico;
- consultar a lista de presença e gerar o PDF dela;
- anexar, baixar e remover arquivos da pauta.

Ficam fora deste módulo a ata, descrita em [Atas de reunião](../../4%20-%20Incremento3/atas/README.md), e as votações, descritas em [Votação na reunião](../../4%20-%20Incremento3/votacao/README.md). A presença registrada aqui é a entrada dos dois: ela define quem aprova a ata e quem vota.

## Atores

| Ator                    | Quem é no sistema                                                                            | O que faz neste módulo                                                                                                                        |
| ----------------------- | -------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------- |
| Participante da reunião | Usuário que está na lista de participantes da reunião.                                       | Consulta a agenda e a reunião, responde ao convite, marca entrada e saída, registra presença pelo link, vê a lista de presença e baixa anexos. |
| Presidente da comissão  | Membro da comissão com papel `RESPONSAVEL`. É uma especialização do participante da reunião. | Agenda, edita e cancela reuniões, confirma ou recusa presenças, cuida dos anexos, gera, envia e revoga o link de presença e corrige horários.   |

## Índice padronizado

|  Nº | Artefato                                                                   | Finalidade                                              |
| --: | -------------------------------------------------------------------------- | ------------------------------------------------------- |
|   1 | [Casos de uso](1%20-%20casos-uso-reunioes.md)                              | Diagrama, atores e requisitos de usuário.               |
|   2 | [Requisitos funcionais](2%20-%20requisitos-funcionais-reunioes.md)         | Cada caso de uso no formulário padrão.                  |
|   3 | [Diagrama de classes](3%20-%20diagrama-classes-reunioes.md)                | Telas, API, serviços e modelo envolvidos.               |
|   3 | [Dicionário de classes](3%20-%20dicionario-dados-classes-reunioes.md)      | Atributos, tipos e operações de cada classe.            |
|   4 | [Diagrama relacional](4%20-%20diagrama-relacional-reunioes.md)             | Visão lógica dos dados da reunião no MongoDB.           |
|   4 | [Dicionário relacional](4%20-%20dicionario-dados-relacional-reunioes.md)   | Campo a campo: tipo, nulo, chaves e geração de valores. |
|   5 | [Requisitos não funcionais](5%20-%20requisitos-nao-funcionais-reunioes.md) | Critérios de qualidade com medida e verificação.        |
|   6 | [Diagramas de sequência](6%20-%20diagramas-sequencia-reunioes.md)          | Mensagens entre tela, API, serviço e banco.             |
|   7 | [Testes](7%20-%20testes-reunioes.md)                                       | Testes automatizados e roteiros executados.             |

## Rastreabilidade

| Caso de uso | Requisito | Sequência | Testes                                                       |
| ----------- | --------- | --------- | ------------------------------------------------------------ |
| UC-REU-01   | RF-REU-01 | SD-REU-01 | CT-REU-01, CT-REU-14, CT-REU-15, CT-REU-23, CT-REU-26        |
| UC-REU-02   | RF-REU-02 | SD-REU-02 | CT-REU-02, CT-REU-15                                         |
| UC-REU-03   | RF-REU-03 | SD-REU-03 | CT-REU-03, CT-REU-15, CT-REU-17                              |
| UC-REU-04   | RF-REU-04 | SD-REU-04 | CT-REU-04, CT-REU-16                                         |
| UC-REU-05   | RF-REU-05 | SD-REU-05 | CT-REU-05, CT-REU-16, CT-REU-27                              |
| UC-REU-06   | RF-REU-06 | SD-REU-06 | CT-REU-06, CT-REU-17, CT-REU-27, CT-REU-40                   |
| UC-REU-07   | RF-REU-07 | SD-REU-07 | CT-REU-07, CT-REU-17, CT-REU-27, CT-REU-29, CT-REU-31        |
| UC-REU-08   | RF-REU-08 | SD-REU-08 | CT-REU-08, CT-REU-17                                         |
| UC-REU-09   | RF-REU-09 | SD-REU-09 | CT-REU-09, CT-REU-18, CT-REU-30, CT-REU-31, CT-REU-32        |
| UC-REU-13   | RF-REU-13 | SD-REU-13 | CT-REU-21, CT-REU-24                                         |
| UC-REU-14   | RF-REU-14 | SD-REU-14 | CT-REU-22, CT-REU-25                                         |
| UC-REU-15   | RF-REU-15 | SD-REU-15 | CT-REU-22, CT-REU-25                                         |
| UC-REU-16   | RF-REU-16 | SD-REU-16 | CT-REU-22, CT-REU-25                                         |
| UC-REU-17   | RF-REU-17 | SD-REU-17 | CT-REU-13, CT-REU-33, CT-REU-37, CT-REU-38                   |
| UC-REU-18   | RF-REU-18 | SD-REU-18 | CT-REU-37, CT-REU-38                                         |
| UC-REU-19   | RF-REU-19 | SD-REU-19 | CT-REU-34, CT-REU-38                                         |
| UC-REU-20   | RF-REU-20 | SD-REU-20 | CT-REU-35, CT-REU-38                                         |
| UC-REU-21   | RF-REU-21 | SD-REU-21 | CT-REU-36, CT-REU-38                                         |
| UC-REU-22   | RF-REU-22 | SD-REU-22 | CT-REU-36, CT-REU-38                                         |

Os números 10, 11 e 12 não são usados neste módulo, nem como caso de uso nem como requisito ou diagrama: a ata é documentada como UC-ATA-01 a UC-ATA-08. Pelo mesmo motivo, CT-REU-10 a CT-REU-12 não têm teste próprio.

## Base de conferência

BACKEND e FRONTEND na branch `homolog`, conferidos em 26/11/2026 (backend `811cf02`, frontend `8fc0a5a`; o pedido de identidade em `marcarEntrada`, em `FRONTEND/src/app/(abas)/inicio.tsx`, entrou em `3a2ece4`). Os arquivos principais são `BACKEND/src/reunioes/reunioes.service.ts`, `BACKEND/src/reunioes/reunioes.controller.ts`, `BACKEND/src/reunioes/schemas/reuniao.schema.ts`, `BACKEND/src/reunioes/anexos.ts`, `BACKEND/src/presencas/presencas.service.ts`, `BACKEND/src/presencas/presencas.controller.ts`, `BACKEND/src/notificacoes/notificacoes.service.ts`, `FRONTEND/src/app/reunioes/[id].tsx`, `FRONTEND/src/app/reunioes/registros.tsx`, `FRONTEND/src/app/reunioes/acesso.tsx`, `FRONTEND/src/app/presenca.tsx`, `FRONTEND/src/app/(abas)/agenda.tsx`, `FRONTEND/src/app/(abas)/inicio.tsx`, `FRONTEND/src/components/FormularioReuniao.tsx`, `FRONTEND/src/services/anexos.ts` e `FRONTEND/src/services/pdfListaPresenca.ts`.

A entrega do Incremento 2 foi conferida em 23/10/2026 sobre FRONTEND `792fab4` e BACKEND `6f2ffc4`.
