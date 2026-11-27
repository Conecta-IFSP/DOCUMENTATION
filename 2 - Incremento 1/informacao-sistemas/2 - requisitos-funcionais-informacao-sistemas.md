# Requisitos funcionais — Informações do sistema

> Padrão do Incremento 1 · Módulo SIS

## Resumo

Define RF-SIS-01 a RF-SIS-04. Todas as rotas exigem `X-API-Key`, JWT válido, conta ativa e `ADMIN_SISTEMA`.

## Visão geral

| ID        | Requisito              | Rota                          |
| --------- | ---------------------- | ----------------------------- |
| RF-SIS-01 | Consultar status       | `GET /sistema/status`         |
| RF-SIS-02 | Consultar relatórios   | `GET /sistema/relatorios`     |
| RF-SIS-03 | Popular dados de teste | `POST /sistema/popular-teste` |
| RF-SIS-04 | Resetar sistema        | `POST /sistema/reset`         |

## Especificações

### RF-SIS-01 — Consultar status

| Item              | Especificação                                                                              |
| ----------------- | ------------------------------------------------------------------------------------------ |
| **Entrada**       | Requisição autenticada sem corpo.                                                          |
| **Pré-condições** | Três guards aprovados.                                                                     |
| **Processamento** | `SistemaController.status()` → `SistemaService.status()`; módulo `os` e consulta ao banco. |
| **Persistência**  | Nenhuma; leitura para validar disponibilidade.                                             |
| **Saída**         | Memória, CPU, uptime e estado do banco.                                                    |
| **Erros**         | 401 chave/JWT; 403 tipo; erro de serviço se a consulta falhar.                             |

### RF-SIS-02 — Consultar relatórios

| Item              | Especificação                                                |
| ----------------- | ------------------------------------------------------------ |
| **Entrada**       | Requisição autenticada sem corpo.                            |
| **Pré-condições** | Três guards aprovados.                                       |
| **Processamento** | `SistemaService.relatorios()` executa contagens com filtros. |
| **Persistência**  | Somente leitura de `usuarios`, `organizacoes` e `comissoes`. |
| **Saída**         | Totais e subtotais por situação.                             |
| **Erros**         | 401/403 ou falha do banco.                                   |

### RF-SIS-03 — Popular dados de teste

| Item              | Especificação                                                              |
| ----------------- | -------------------------------------------------------------------------- |
| **Entrada**       | Confirmação na interface e `POST /sistema/popular-teste`.                  |
| **Pré-condições** | Admin global autenticado.                                                  |
| **Processamento** | Cria três usuários de cenários distintos, organização aprovada e comissão. |
| **Persistência**  | Inserções em `usuarios`, `organizacoes` e `comissoes`.                     |
| **Saída**         | Status, senha padrão de demonstração e usuários criados.                   |
| **Erros**         | 401/403 ou falha de persistência.                                          |

### RF-SIS-04 — Resetar sistema

| Item              | Especificação                                                                             |
| ----------------- | ----------------------------------------------------------------------------------------- |
| **Entrada**       | Confirmação destrutiva e `POST /sistema/reset`.                                           |
| **Pré-condições** | Admin global autenticado.                                                                 |
| **Processamento** | Remove reuniões, comissões e organizações; remove usuários que não sejam `ADMIN_SISTEMA`. |
| **Persistência**  | Exclusões nas quatro coleções; administradores globais preservados.                       |
| **Saída**         | `{status:'sistema_resetado', resetado_em}`.                                               |
| **Erros**         | 401/403 ou falha de banco.                                                                |

## Limite de escopo

Não existe reinicialização de subsistemas: o backend NestJS é um único processo. A documentação não apresenta uma função inexistente.

## Rastreabilidade

RF-SIS-01 a RF-SIS-04 correspondem a UC-SIS-01 a UC-SIS-04 e CT-SIS-01 a CT-SIS-06.
