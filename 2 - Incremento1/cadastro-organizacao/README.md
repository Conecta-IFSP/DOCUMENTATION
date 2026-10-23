# Cadastro de organização

> Incremento 1 · Módulo ORG · Documentação técnica e funcional

## Resumo

Abrange cadastro e moderação global de organizações, pesquisa, códigos de acesso, convites e administração de papéis internos.

## Escopo

- cadastrar, pesquisar, aprovar, revogar, editar e excluir organizações;
- entrar por código e renovar o código;
- convidar e responder convite;
- analisar solicitações e administrar papéis.

## Índice padronizado

|  Nº | Artefato                                                                               | Finalidade                                            |
| --: | -------------------------------------------------------------------------------------- | ----------------------------------------------------- |
|   1 | [Casos de uso](1%20-%20casos-uso-cadastro-organizacao.md)                              | Atores, resumo, fluxos e exceções.                    |
|   2 | [Requisitos funcionais](2%20-%20requisitos-funcionais-cadastro-organizacao.md)         | Comportamentos verificáveis.                          |
|   3 | [Diagrama de classes](3%20-%20diagrama-classes-cadastro-organizacao.md)                | Estrutura de domínio e serviços.                      |
|   3 | [Dicionário de classes](3%20-%20dicionario-dados-classes-cadastro-organizacao.md)      | Significado de classes, atributos e operações.        |
|   4 | [Diagrama relacional](4%20-%20diagrama-relacional-cadastro-organizacao.md)             | Visão lógica dos dados MongoDB.                       |
|   4 | [Dicionário relacional](4%20-%20dicionario-dados-relacional-cadastro-organizacao.md)   | Campos, tipos, chaves e restrições.                   |
|   5 | [Requisitos não funcionais](5%20-%20requisitos-nao-funcionais-cadastro-organizacao.md) | Critérios mensuráveis de qualidade.                   |
|   6 | [Diagramas de sequência](6%20-%20diagramas-sequencia-cadastro-organizacao.md)          | Interação entre telas, controllers, services e banco. |
|   7 | [Testes](7%20-%20testes-cadastro-organizacao.md)                                       | Evidências e roteiro de verificação.                  |

## Rastreabilidade

`UC-ORG-01..09 ↔ RF-ORG-01..09 ↔ SD-ORG-01..08 ↔ CT-ORG`

## Base de conferência

FRONTEND `homolog` (`792fab4`) e BACKEND `homolog` (`6f2ffc4`), conferidos em 23/10/2026.
