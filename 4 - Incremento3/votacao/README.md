# Votação na reunião

> Padrão da disciplina · Módulo VOT · Incremento 3 · Revisado em 13/11/2026

## Resumo

Durante a reunião, o presidente da comissão abre votações sobre os assuntos em pauta. Cada votação tem um assunto, uma descrição opcional e de 2 a 10 opções de voto, que já vêm preenchidas com "Sim", "Não" e "Abstenção". Quem marcou entrada na reunião e ainda não saiu vota uma única vez, sem poder mudar o voto. Enquanto a votação está aberta, todos veem só quantos votos já houve. O resultado por opção, com quantidade e porcentagem, aparece quando o presidente encerra a votação ou quando a reunião termina. Ninguém vê em quem cada pessoa votou.

## Escopo

- abrir uma votação durante a reunião;
- votar em uma das opções;
- encerrar a votação;
- consultar as votações da reunião e o resultado das encerradas.

Ficam fora deste módulo o agendamento, os convites e a presença, descritos em [Reuniões](../../3%20-%20Incremento2/reunioes/README.md), e a ata, descrita em [Atas de reunião](../atas/README.md). A presença é a entrada deste módulo: ela define quem pode votar.

## Atores

| Ator | Quem é no sistema | O que faz neste módulo |
| --- | --- | --- |
| Participante da reunião | Usuário que está na lista de participantes da reunião, com ou sem presença. | Consulta as votações e o resultado das encerradas. |
| Participante presente | Participante que marcou entrada, não teve a entrada recusada e ainda não marcou saída. É uma especialização do participante da reunião. | Vota nas votações abertas. |
| Presidente da comissão | Membro da comissão com papel `RESPONSAVEL`. É uma especialização do participante da reunião. | Abre e encerra votações. |

O presidente que marcou entrada também é um participante presente e pode votar nas votações que ele mesmo abriu.

## Índice padronizado

|  Nº | Artefato | Finalidade |
| --: | --- | --- |
|   1 | [Casos de uso](1%20-%20casos-uso-votacao.md) | Diagrama, atores e requisitos de usuário. |
|   2 | [Requisitos funcionais](2%20-%20requisitos-funcionais-votacao.md) | Cada caso de uso no formulário padrão. |
|   3 | [Diagrama de classes](3%20-%20diagrama-classes-votacao.md) | Telas, API, serviço e modelo envolvidos. |
|   3 | [Dicionário de classes](3%20-%20dicionario-dados-classes-votacao.md) | Atributos, tipos e operações de cada classe. |
|   4 | [Diagrama relacional](4%20-%20diagrama-relacional-votacao.md) | Visão lógica dos dados da votação no MongoDB. |
|   4 | [Dicionário relacional](4%20-%20dicionario-dados-relacional-votacao.md) | Campo a campo: tipo, nulo, chaves e geração de valores. |
|   5 | [Requisitos não funcionais](5%20-%20requisitos-nao-funcionais-votacao.md) | Critérios de qualidade com medida e verificação. |
|   6 | [Diagramas de sequência](6%20-%20diagramas-sequencia-votacao.md) | Mensagens entre tela, API, serviço e banco. |
|   7 | [Testes](7%20-%20testes-votacao.md) | Testes automatizados e roteiros manuais. |

## Rastreabilidade

| Caso de uso | Requisito | Sequência | Testes |
| --- | --- | --- | --- |
| UC-VOT-01 | RF-VOT-01 | SD-VOT-01 | CT-VOT-01, CT-VOT-02, CT-VOT-03, CT-VOT-10 |
| UC-VOT-02 | RF-VOT-02 | SD-VOT-02 | CT-VOT-04, CT-VOT-05, CT-VOT-06, CT-VOT-11, CT-VOT-12 |
| UC-VOT-03 | RF-VOT-03 | SD-VOT-03 | CT-VOT-07, CT-VOT-13 |
| UC-VOT-04 | RF-VOT-04 | SD-VOT-04 | CT-VOT-08, CT-VOT-09, CT-VOT-13 |

## Base de conferência

BACKEND e FRONTEND na branch `homolog`, conferidos em 13/11/2026: BACKEND `08a5b3e`, com a votação incluída em `c2ea38c`; FRONTEND `66806e4`, com a tela de votação incluída em `83ccd12`. Os arquivos principais são `BACKEND/src/reunioes/reunioes.service.ts`, `BACKEND/src/reunioes/reunioes.controller.ts`, `BACKEND/src/reunioes/dto/votacao.dto.ts`, `BACKEND/src/reunioes/schemas/reuniao.schema.ts`, `BACKEND/src/reunioes/votacoes.spec.ts`, `FRONTEND/src/components/VotacoesReuniao.tsx`, `FRONTEND/src/app/reunioes/[id].tsx` e `FRONTEND/src/services/api.ts`.
