# Cadastro de usuário

> Incremento 1 · Módulo USR · Documentação técnica e funcional

## Resumo

Abrange cadastro local com confirmação de e-mail, login local/Google e participação do usuário em organizações por código, convite ou aprovação.

## Escopo

- cadastrar e confirmar conta local;
- autenticar por senha ou Google;
- solicitar entrada ou responder convite;
- convidar, analisar, promover, rebaixar e remover membros.

## Índice padronizado

|  Nº | Artefato                                                                           | Finalidade                                                 |
| --: | ---------------------------------------------------------------------------------- | ---------------------------------------------------------- |
|   1 | [Casos de uso](1%20-%20casos-uso-cadastro-usuario.md)                              | Atores, resumo, fluxos e exceções.                         |
|   2 | [Requisitos funcionais](2%20-%20requisitos-funcionais-cadastro-usuario.md)         | Comportamentos verificáveis.                               |
|   3 | [Diagrama de classes](3%20-%20diagrama-classes-cadastro-usuario.md)                | Estrutura de domínio e serviços.                           |
|   3 | [Dicionário de classes](3%20-%20dicionario-dados-classes-cadastro-usuario.md)      | Significado de classes, atributos e operações.             |
|   4 | [Diagrama relacional](4%20-%20diagrama-relacional-cadastro-usuario.md)             | Visão lógica dos dados MongoDB.                            |
|   4 | [Dicionário relacional](4%20-%20dicionario-dados-relacional-cadastro-usuario.md)   | Campos, tipos, chaves e restrições.                        |
|   5 | [Requisitos não funcionais](5%20-%20requisitos-nao-funcionais-cadastro-usuario.md) | Critérios mensuráveis de qualidade.                        |
|   6 | [Diagramas de sequência](6%20-%20diagramas-sequencia-cadastro-usuario.md)          | Interação entre telas, controllers, services e provedores. |
|   7 | [Testes](7%20-%20testes-cadastro-usuario.md)                                       | Evidências e roteiro de verificação.                       |

## Rastreabilidade

`UC-USR-01..09 ↔ RF-USR-01..07 ↔ SD-USR-01..07 ↔ CT-USR`

## Base de conferência

FRONTEND `792fab4` e BACKEND `6f2ffc4`, conferidos em 23/10/2026.
