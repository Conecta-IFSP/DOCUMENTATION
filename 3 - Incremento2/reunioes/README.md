# Reuniões

> Incremento 2 · Módulo REU · Documentação técnica e funcional

## Resumo

Abrange agendamento, agenda, convites, presença, lista de presença e ata versionada. As regras centrais restringem o agendamento ao responsável da comissão, protegem operações concorrentes e exigem aprovação da ata pelos participantes com presença confirmada.

## Escopo

- agendar, consultar, visualizar e cancelar reuniões;
- responder convites e registrar entrada e saída;
- confirmar ou recusar registros de presença;
- gerar e compartilhar a lista de presença;
- salvar, publicar e aprovar uma ata versionada.

## Índice padronizado

|  Nº | Artefato                                                                   | Finalidade                                          |
| --: | -------------------------------------------------------------------------- | --------------------------------------------------- |
|   1 | [Casos de uso](1%20-%20casos-uso-reunioes.md)                              | Atores, resumo, fluxos e exceções.                  |
|   2 | [Requisitos funcionais](2%20-%20requisitos-funcionais-reunioes.md)         | Comportamentos verificáveis.                        |
|   3 | [Diagrama de classes](3%20-%20diagrama-classes-reunioes.md)                | Estrutura de domínio, serviços e telas.             |
|   3 | [Dicionário de classes](3%20-%20dicionario-dados-classes-reunioes.md)      | Significado de classes, atributos e operações.      |
|   4 | [Diagrama relacional](4%20-%20diagrama-relacional-reunioes.md)             | Visão lógica dos documentos MongoDB.                |
|   4 | [Dicionário relacional](4%20-%20dicionario-dados-relacional-reunioes.md)   | Campos, tipos, chaves e restrições.                 |
|   5 | [Requisitos não funcionais](5%20-%20requisitos-nao-funcionais-reunioes.md) | Critérios mensuráveis de qualidade.                 |
|   6 | [Diagramas de sequência](6%20-%20diagramas-sequencia-reunioes.md)          | Interação entre telas, controller, service e banco. |
|   7 | [Testes](7%20-%20testes-reunioes.md)                                       | Evidências e roteiro de verificação.                |

## Rastreabilidade

`UC-REU-01..12 ↔ RF-REU-01..12 ↔ SD-REU-01..12 ↔ CT-REU`

## Base de conferência

FRONTEND `792fab4` e BACKEND `6f2ffc4`, conferidos em 23/10/2026.
