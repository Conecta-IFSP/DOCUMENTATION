# Votação na reunião

> Padrão da disciplina · Módulo VOT · Incremento 3 · Revisado em 27/11/2026

## Resumo

Durante a reunião, o presidente da comissão abre votações sobre os assuntos em pauta. Cada votação tem um assunto, uma descrição opcional, de 2 a 10 opções de voto, que já vêm preenchidas com "Sim", "Não" e "Abstenção", e um tipo, escolhido na abertura: **fechada**, em que ninguém vê quem votou em quê, ou **aberta**, em que todos os participantes veem quem votou em quê depois do encerramento. Quem marcou entrada na reunião, não teve a entrada recusada e ainda não saiu vota uma única vez, em uma das opções, em branco ou nulo, sem poder mudar o voto. Enquanto a votação não é encerrada, todos veem só quantos votos já houve. O resultado, com quantidade e porcentagem de cada opção, dos votos em branco e dos nulos, aparece quando o presidente encerra a votação ou quando a reunião termina; na votação do tipo aberta, aparece também a lista de quem votou em quê.

## Escopo

- abrir uma votação durante a reunião, escolhendo se ela é fechada ou aberta;
- votar em uma das opções, em branco ou nulo;
- encerrar a votação;
- consultar as votações da reunião e o resultado das encerradas, com os votos em branco, os nulos e, na votação do tipo aberta, a lista de quem votou em quê.

Ficam fora deste módulo o agendamento, os convites e a presença, descritos em [Reuniões](../../3%20-%20Incremento%202/reunioes/README.md), e a ata, descrita em [Atas de reunião](../atas/README.md). A presença é a entrada deste módulo: ela define quem pode votar.

O voto em branco, o voto nulo e a escolha entre votação aberta e fechada foram incluídos em 27/11/2026, a partir dos itens 4 e 5 da avaliação do Incremento 3 ([Tratamento das observações da avaliação](../../TRATAMENTO-AVALIACAO.md)). As votações gravadas antes disso são tratadas como fechadas, e os votos delas, como votos em opção.

## Atores

| Ator                    | Quem é no sistema                                                                                                                         | O que faz neste módulo                              |
| ----------------------- | ----------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------- |
| Participante da reunião | Usuário que está na lista de participantes da reunião, com ou sem presença.                                                               | Consulta as votações e o resultado das encerradas, com a lista de quem votou em quê nas do tipo aberta. |
| Participante presente   | Participante que marcou entrada, não teve a entrada recusada e ainda não marcou saída. É uma especialização do participante da reunião.   | Vota nas votações ainda não encerradas, em uma opção, em branco ou nulo. |
| Presidente da comissão  | Membro com papel `RESPONSAVEL` de comissão ativa. É uma especialização do participante da reunião.                                         | Abre votações, escolhendo o tipo, e as encerra.     |

O presidente que marcou entrada também é um participante presente e pode votar nas votações que ele mesmo abriu.

## Índice padronizado

|  Nº | Artefato                                                                  | Finalidade                                              |
| --: | ------------------------------------------------------------------------- | ------------------------------------------------------- |
|   1 | [Casos de uso](1%20-%20casos-uso-votacao.md)                              | Diagrama, atores e requisitos de usuário.               |
|   2 | [Requisitos funcionais](2%20-%20requisitos-funcionais-votacao.md)         | Cada caso de uso no formulário padrão.                  |
|   3 | [Diagrama de classes](3%20-%20diagrama-classes-votacao.md)                | Tela, componente, API, serviço e modelo envolvidos.     |
|   3 | [Dicionário de classes](3%20-%20dicionario-dados-classes-votacao.md)      | Atributos, tipos e operações de cada classe.            |
|   4 | [Diagrama relacional](4%20-%20diagrama-relacional-votacao.md)             | Visão lógica dos dados da votação no MongoDB.           |
|   4 | [Dicionário relacional](4%20-%20dicionario-dados-relacional-votacao.md)   | Campo a campo: tipo, nulo, chaves e geração de valores. |
|   5 | [Requisitos não funcionais](5%20-%20requisitos-nao-funcionais-votacao.md) | Critérios de qualidade com medida e verificação.        |
|   6 | [Diagramas de sequência](6%20-%20diagramas-sequencia-votacao.md)          | Mensagens entre tela, API, serviço e banco.             |
|   7 | [Testes](7%20-%20testes-votacao.md)                                       | Testes automatizados e roteiros manuais.                |

## Rastreabilidade

| Caso de uso | Requisito | Sequência | Testes                                                                                                       |
| ----------- | --------- | --------- | ------------------------------------------------------------------------------------------------------------ |
| UC-VOT-01   | RF-VOT-01 | SD-VOT-01 | CT-VOT-01, CT-VOT-02, CT-VOT-03, CT-VOT-10, CT-VOT-15, CT-VOT-22, CT-VOT-23, CT-VOT-24, CT-VOT-30            |
| UC-VOT-02   | RF-VOT-02 | SD-VOT-02 | CT-VOT-04, CT-VOT-05, CT-VOT-06, CT-VOT-11, CT-VOT-12, CT-VOT-16, CT-VOT-20, CT-VOT-21, CT-VOT-25, CT-VOT-26, CT-VOT-29 |
| UC-VOT-03   | RF-VOT-03 | SD-VOT-03 | CT-VOT-07, CT-VOT-13, CT-VOT-27, CT-VOT-28                                                                   |
| UC-VOT-04   | RF-VOT-04 | SD-VOT-04 | CT-VOT-08, CT-VOT-09, CT-VOT-13, CT-VOT-14, CT-VOT-17, CT-VOT-18, CT-VOT-19, CT-VOT-27, CT-VOT-28            |

## Base de conferência

BACKEND e FRONTEND na branch `homolog`, conferidos em 27/11/2026: backend `d1f1aeb` e frontend `10abe49`, commits que incluíram o voto em branco, o voto nulo e o tipo da votação (aberta ou fechada). Reconferidos na mesma data sobre o backend `6e0d3f5`, que só acrescenta variáveis ao `.env.example`, e o frontend `4f53a25`, nas branches `main` e `homolog`: neste módulo, o `1d3e977` muda o aviso de abertura para "Votação iniciada." e a linha do cartão para "iniciada às", e o `db5b2fe` faz a folha "Nova votação" acompanhar o teclado (`FolhaInferior` com o `KeyboardAvoidingView` do `react-native-keyboard-controller`); os outros commits mudam a entrega do PDF, a presença, o desbloqueio na abertura e um estilo do cartão de "Suas organizações", fora deste módulo. A revisão anterior, de 26/11/2026, conferiu o backend `caca2d2` e o frontend `6ff2d50`. A votação entrou no backend em `c2ea38c` e a tela de votação no frontend em `83ccd12`. Os arquivos principais são `BACKEND/src/reunioes/reunioes.service.ts`, `BACKEND/src/reunioes/reunioes.controller.ts`, `BACKEND/src/reunioes/dto/votacao.dto.ts`, `BACKEND/src/reunioes/schemas/reuniao.schema.ts`, `BACKEND/src/reunioes/votacoes.spec.ts`, `FRONTEND/src/components/VotacoesReuniao.tsx`, `FRONTEND/src/app/reunioes/[id].tsx`, `FRONTEND/src/services/api.ts` e `FRONTEND/src/constants/ajuda.ts`.
