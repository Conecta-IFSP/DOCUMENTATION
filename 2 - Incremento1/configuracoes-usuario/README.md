# Configurações de usuário

> Incremento 1 · Módulo CFG · Documentação técnica e funcional

## Resumo

Abrange perfil, troca confirmada de e-mail, troca e recuperação de senha, tema e saída da sessão. A recuperação usa código digitado no app e funciona no Expo Go sem link `localhost`.

## Escopo

- alterar nome do perfil;
- trocar e-mail por confirmação;
- trocar ou recuperar senha;
- escolher tema com rollback em falha;
- encerrar a sessão local.

## Índice padronizado

|  Nº | Artefato                                                                                | Finalidade                                     |
| --: | --------------------------------------------------------------------------------------- | ---------------------------------------------- |
|   1 | [Casos de uso](1%20-%20casos-uso-configuracoes-usuario.md)                              | Atores, resumo, fluxos e exceções.             |
|   2 | [Requisitos funcionais](2%20-%20requisitos-funcionais-configuracoes-usuario.md)         | Comportamentos verificáveis.                   |
|   3 | [Diagrama de classes](3%20-%20diagrama-classes-configuracoes-usuario.md)                | Estrutura de telas, contextos e services.      |
|   3 | [Dicionário de classes](3%20-%20dicionario-dados-classes-configuracoes-usuario.md)      | Significado de classes, atributos e operações. |
|   4 | [Diagrama relacional](4%20-%20diagrama-relacional-configuracoes-usuario.md)             | Visão lógica dos dados MongoDB.                |
|   4 | [Dicionário relacional](4%20-%20dicionario-dados-relacional-configuracoes-usuario.md)   | Campos, tipos, chaves e restrições.            |
|   5 | [Requisitos não funcionais](5%20-%20requisitos-nao-funcionais-configuracoes-usuario.md) | Critérios mensuráveis de qualidade.            |
|   6 | [Diagramas de sequência](6%20-%20diagramas-sequencia-configuracoes-usuario.md)          | Interação entre telas, contexts, API e e-mail. |
|   7 | [Testes](7%20-%20testes-configuracoes-usuario.md)                                       | Evidências e roteiro de verificação.           |

## Rastreabilidade

`UC-CFG-01..06 ↔ RF-CFG-01..06 ↔ SD-CFG-01..06 ↔ CT-CFG`

## Base de conferência

FRONTEND `homolog` (`792fab4`) e BACKEND `homolog` (`6f2ffc4`), conferidos em 23/10/2026.
