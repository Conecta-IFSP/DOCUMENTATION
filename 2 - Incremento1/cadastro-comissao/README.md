# Cadastro de comissão

> Incremento 1 · Módulo COM · Documentação técnica e funcional

## Resumo

Abrange consulta, criação, edição, ativação e composição de comissões. A regra central é manter exatamente um responsável elegível em cada comissão.

## Escopo

- listar e consultar comissões permitidas;
- cadastrar e editar comissões;
- desativar e reativar sem apagar histórico;
- adicionar e remover membros;
- substituir o responsável atomicamente.

## Índice padronizado

|  Nº | Artefato                                                                            | Finalidade                                            |
| --: | ----------------------------------------------------------------------------------- | ----------------------------------------------------- |
|   1 | [Casos de uso](1%20-%20casos-uso-cadastro-comissao.md)                              | Atores, resumo, fluxos e exceções.                    |
|   2 | [Requisitos funcionais](2%20-%20requisitos-funcionais-cadastro-comissao.md)         | Comportamentos verificáveis.                          |
|   3 | [Diagrama de classes](3%20-%20diagrama-classes-cadastro-comissao.md)                | Estrutura de domínio e serviços.                      |
|   3 | [Dicionário de classes](3%20-%20dicionario-dados-classes-cadastro-comissao.md)      | Significado de classes, atributos e operações.        |
|   4 | [Diagrama relacional](4%20-%20diagrama-relacional-cadastro-comissao.md)             | Visão lógica dos dados MongoDB.                       |
|   4 | [Dicionário relacional](4%20-%20dicionario-dados-relacional-cadastro-comissao.md)   | Campos, tipos, chaves e restrições.                   |
|   5 | [Requisitos não funcionais](5%20-%20requisitos-nao-funcionais-cadastro-comissao.md) | Critérios mensuráveis de qualidade.                   |
|   6 | [Diagramas de sequência](6%20-%20diagramas-sequencia-cadastro-comissao.md)          | Interação entre telas, controllers, services e banco. |
|   7 | [Testes](7%20-%20testes-cadastro-comissao.md)                                       | Evidências e roteiro de verificação.                  |

## Rastreabilidade

`UC-COM-01..09 ↔ RF-COM-01..09 ↔ SD-COM-01..09 ↔ CT-COM`

## Base de conferência

FRONTEND `homolog` (`792fab4`) e BACKEND `homolog` (`6f2ffc4`), conferidos em 23/10/2026.
