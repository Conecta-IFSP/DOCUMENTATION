# Reuniões

> Padrão da disciplina · Módulo REU · Incremento 2 · Revisado em 10/11/2026

## Resumo

Abrange agendamento, agenda, edição, convites, presença, lista de presença e anexos da pauta. As regras centrais restringem o agendamento e a edição ao responsável da comissão e protegem operações concorrentes.

A ata não é mais descrita aqui: ela foi reescrita no Incremento 3 e está em [Atas de reunião](../../4%20-%20Incremento3/atas/README.md).

## Escopo

- agendar, consultar, visualizar, editar e cancelar reuniões, inclusive de dia inteiro e sem local;
- responder convites e registrar entrada e saída;
- confirmar ou recusar registros de presença;
- gerar e compartilhar a lista de presença;
- anexar, baixar e remover arquivos da pauta.

## Índice padronizado

|  Nº | Artefato                                                                   | Finalidade                                          |
| --: | -------------------------------------------------------------------------- | --------------------------------------------------- |
|   1 | [Casos de uso](1%20-%20casos-uso-reunioes.md)                              | Diagrama, atores e requisitos de usuário.           |
|   2 | [Requisitos funcionais](2%20-%20requisitos-funcionais-reunioes.md)         | Cada caso de uso no formulário padrão.              |
|   3 | [Diagrama de classes](3%20-%20diagrama-classes-reunioes.md)                | Estrutura de domínio, serviços e telas.             |
|   3 | [Dicionário de classes](3%20-%20dicionario-dados-classes-reunioes.md)      | Significado de classes, atributos e operações.      |
|   4 | [Diagrama relacional](4%20-%20diagrama-relacional-reunioes.md)             | Visão lógica dos documentos MongoDB.                |
|   4 | [Dicionário relacional](4%20-%20dicionario-dados-relacional-reunioes.md)   | Campos, tipos, chaves e restrições.                 |
|   5 | [Requisitos não funcionais](5%20-%20requisitos-nao-funcionais-reunioes.md) | Critérios mensuráveis de qualidade.                 |
|   6 | [Diagramas de sequência](6%20-%20diagramas-sequencia-reunioes.md)          | Interação entre telas, controller, service e banco. |
|   7 | [Testes](7%20-%20testes-reunioes.md)                                       | Evidências e roteiro de verificação.                |

## Rastreabilidade

`UC-REU-01..09 ↔ RF-REU-01..09 ↔ SD-REU-01..09 ↔ CT-REU` e `UC-REU-13..16 ↔ RF-REU-13..16 ↔ SD-REU-13..16 ↔ CT-REU`.

Os números UC-REU-10 a UC-REU-12 pertenciam à ata e não foram reaproveitados, para que a documentação entregue no Incremento 2 continue legível. A ata agora é UC-ATA-01 a UC-ATA-08.

## Base de conferência

Entrega do Incremento 2: FRONTEND `792fab4` e BACKEND `6f2ffc4`, conferidos em 23/10/2026.

Revisão do Incremento 3: branch `homolog` dos dois repositórios, conferida em 09/11/2026, com as correções pedidas na avaliação do Incremento 2 — reunião de dia inteiro, local opcional, edição da reunião e anexos da pauta.
