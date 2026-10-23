# Informações do sistema

> Incremento 1 · Módulo SIS · Documentação técnica e funcional

## Resumo

Abrange monitoramento do backend, relatórios administrativos, dados de demonstração e reset protegido do ambiente.

## Escopo

- consultar status do servidor e banco;
- consultar contagens consolidadas;
- criar dados controlados de teste;
- resetar dados operacionais, preservando administradores globais.

## Índice padronizado

|  Nº | Artefato                                                                              | Finalidade                                                 |
| --: | ------------------------------------------------------------------------------------- | ---------------------------------------------------------- |
|   1 | [Casos de uso](1%20-%20casos-uso-informacao-sistemas.md)                              | Atores, resumo, fluxos e exceções.                         |
|   2 | [Requisitos funcionais](2%20-%20requisitos-funcionais-informacao-sistemas.md)         | Comportamentos verificáveis.                               |
|   3 | [Diagrama de classes](3%20-%20diagrama-classes-informacao-sistemas.md)                | Estrutura de controller e service.                         |
|   3 | [Dicionário de classes](3%20-%20dicionario-dados-classes-informacao-sistemas.md)      | Significado de classes, atributos e operações.             |
|   4 | [Diagrama relacional](4%20-%20diagrama-relacional-informacao-sistemas.md)             | Coleções consultadas e removidas.                          |
|   4 | [Dicionário relacional](4%20-%20dicionario-dados-relacional-informacao-sistemas.md)   | Dados agregados e efeito do reset.                         |
|   5 | [Requisitos não funcionais](5%20-%20requisitos-nao-funcionais-informacao-sistemas.md) | Critérios mensuráveis de qualidade.                        |
|   6 | [Diagramas de sequência](6%20-%20diagramas-sequencia-informacao-sistemas.md)          | Interação entre tela, guards, controller, service e banco. |
|   7 | [Testes](7%20-%20testes-informacao-sistemas.md)                                       | Evidências e roteiro de verificação.                       |

## Rastreabilidade

`UC-SIS-01..04 ↔ RF-SIS-01..04 ↔ SD-SIS-01..04 ↔ CT-SIS`

## Base de conferência

FRONTEND `792fab4` e BACKEND `6f2ffc4`, conferidos em 23/10/2026.
