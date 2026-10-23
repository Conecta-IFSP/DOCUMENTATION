# Requisitos funcionais — Cadastro de organização

> Padrão do Incremento 1 · Módulo ORG

## Resumo

Define RF-ORG-01 a RF-ORG-09. Todos usam `X-API-Key`; rotas privadas exigem JWT e os guards/regras indicados.

## Visão geral

| ID        | Requisito                 | Rota principal                           | Ator                         |
| --------- | ------------------------- | ---------------------------------------- | ---------------------------- |
| RF-ORG-01 | Cadastrar organização     | `POST /organizacoes`                     | Usuário                      |
| RF-ORG-02 | Consultar organizações    | `GET /organizacoes`                      | Usuário/Admin do sistema     |
| RF-ORG-03 | Aprovar organização       | `PATCH /organizacoes/:id`                | Admin do sistema             |
| RF-ORG-04 | Revogar organização       | `PATCH /organizacoes/:id`                | Admin do sistema             |
| RF-ORG-05 | Editar organização        | `PATCH /organizacoes/:id`                | Admin do sistema             |
| RF-ORG-06 | Excluir organização       | `DELETE /organizacoes/:id`               | Admin do sistema             |
| RF-ORG-07 | Entrar e renovar código   | `POST /organizacoes/entrar`              | Usuário/Admin da organização |
| RF-ORG-08 | Convidar e responder      | `POST .../convites`, `PATCH .../convite` | Admin/Convidado              |
| RF-ORG-09 | Gerenciar vínculo e papel | `PATCH .../membros/:usuarioId`           | Admin da organização         |

## Especificações

### RF-ORG-01 — Cadastrar organização

| Item              | Especificação                                                                                                             |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------- |
| **Entrada**       | `nome` 2–100, `cidade` 2–60 e `descricao?` até 200.                                                                       |
| **Pré-condições** | Conta ativa do tipo `USUARIO`; nome disponível.                                                                           |
| **Processamento** | `OrganizacoesController.criar()` → `OrganizacoesService.criar()`; valida unicidade, gera código e cria o vínculo inicial. |
| **Persistência**  | `organizacoes`: `PENDENTE`, criador em `criada_por` e `membros` como `ADMIN/APROVADO`.                                    |
| **Saída**         | Organização criada, sem expor campos protegidos indevidos.                                                                |
| **Erros**         | 400 dados inválidos; 409 nome duplicado; 403 tipo não permitido.                                                          |

### RF-ORG-02 — Consultar organizações

| Item              | Especificação                                                                                                      |
| ----------------- | ------------------------------------------------------------------------------------------------------------------ |
| **Entrada**       | `GET /organizacoes?busca=termo`, busca opcional até 100.                                                           |
| **Pré-condições** | JWT válido e conta ativa.                                                                                          |
| **Processamento** | Escapa regex, pesquisa nome/cidade e ordena por `criado_em`; admin global vê todas e usuário vê vínculos próprios. |
| **Persistência**  | Somente leitura de `organizacoes` e população de usuários necessários.                                             |
| **Saída**         | Lista no escopo autorizado.                                                                                        |
| **Erros**         | 401 sessão inválida; 400 busca fora do limite.                                                                     |

### RF-ORG-03 — Aprovar organização

| Item              | Especificação                                                 |
| ----------------- | ------------------------------------------------------------- |
| **Entrada**       | `PATCH /organizacoes/:id` com `status=APROVADA`.              |
| **Pré-condições** | Ator `ADMIN_SISTEMA`; organização existente.                  |
| **Processamento** | `AdminSistemaGuard` autoriza e o service atualiza a situação. |
| **Persistência**  | `organizacoes.status=APROVADA`.                               |
| **Saída**         | Organização atualizada.                                       |
| **Erros**         | 403 sem tipo global; 404 ID inexistente.                      |

### RF-ORG-04 — Revogar organização

| Item              | Especificação                                                   |
| ----------------- | --------------------------------------------------------------- |
| **Entrada**       | `PATCH /organizacoes/:id` com `status=REVOGADA`.                |
| **Pré-condições** | Ator `ADMIN_SISTEMA`; confirmação na interface.                 |
| **Processamento** | Atualiza a situação após autorização.                           |
| **Persistência**  | `organizacoes.status=REVOGADA`.                                 |
| **Saída**         | Organização bloqueada para novos acessos e gestão de comissões. |
| **Erros**         | 403 sem permissão; 404 inexistente.                             |

### RF-ORG-05 — Editar organização

| Item              | Especificação                                                 |
| ----------------- | ------------------------------------------------------------- |
| **Entrada**       | `PATCH /organizacoes/:id` com nome, cidade e/ou descrição.    |
| **Pré-condições** | Ator `ADMIN_SISTEMA`; organização existente.                  |
| **Processamento** | Valida campos, verifica unicidade e grava apenas os enviados. |
| **Persistência**  | Campos informados e `atualizado_em`.                          |
| **Saída**         | Organização atualizada.                                       |
| **Erros**         | 400 entrada inválida; 409 nome duplicado; 403/404.            |

### RF-ORG-06 — Excluir organização

| Item              | Especificação                                                 |
| ----------------- | ------------------------------------------------------------- |
| **Entrada**       | `DELETE /organizacoes/:id`.                                   |
| **Pré-condições** | `ADMIN_SISTEMA`, confirmação explícita e situação `REVOGADA`. |
| **Processamento** | Exclui organização e depois suas comissões.                   |
| **Persistência**  | Remove de `organizacoes` e `comissoes`.                       |
| **Saída**         | Confirmação de exclusão.                                      |
| **Erros**         | 400 se não revogada; 403/404.                                 |

### RF-ORG-07 — Entrar por código e renovar código

| Item              | Especificação                                                                            |
| ----------------- | ---------------------------------------------------------------------------------------- |
| **Entrada**       | `{codigo}` em `POST /organizacoes/entrar`; renovação em `POST /organizacoes/:id/codigo`. |
| **Pré-condições** | Entrada em organização aprovada; renovação por `ADMIN/APROVADO`.                         |
| **Processamento** | Normaliza dez caracteres; entrada cria/reabre pendência; renovação gera valor único.     |
| **Persistência**  | `membros[]` ou `codigo_acesso`.                                                          |
| **Saída**         | Organização/vínculo atualizado ou novo código apenas ao administrador.                   |
| **Erros**         | 404 código/organização; 409 vínculo incompatível; 403 sem papel.                         |

### RF-ORG-08 — Convidar e responder convite

| Item              | Especificação                                                                                  |
| ----------------- | ---------------------------------------------------------------------------------------------- |
| **Entrada**       | E-mail em `POST /organizacoes/:id/convites`; `{aceitar}` em `PATCH /organizacoes/:id/convite`. |
| **Pré-condições** | Organização aprovada; convite por admin aprovado; alvo ativo e comum.                          |
| **Processamento** | Cria `CONVIDADO`; resposta aceita como `APROVADO` ou remove o vínculo.                         |
| **Persistência**  | `organizacoes.membros[]`.                                                                      |
| **Saída**         | Organização atualizada.                                                                        |
| **Erros**         | 404 usuário/convite; 409 estado ou vínculo; 403 sem autorização.                               |

### RF-ORG-09 — Gerenciar status e papel do membro

| Item              | Especificação                                                                   |
| ----------------- | ------------------------------------------------------------------------------- |
| **Entrada**       | `status=APROVADO                                                                | REJEITADO`ou`papel=ADMIN | MEMBRO` nas rotas de membro. |
| **Pré-condições** | Solicitante `ADMIN/APROVADO`; alvo no estado permitido.                         |
| **Processamento** | Atualiza com filtro do estado original; impede rebaixar o último administrador. |
| **Persistência**  | `membros.$.status`, `papel` e `aprovado_em`.                                    |
| **Saída**         | Organização atualizada.                                                         |
| **Erros**         | 400 transição inválida; 403 permissão; 404 alvo; 409 concorrência.              |

## Rastreabilidade

RF-ORG-01 a RF-ORG-09 correspondem a UC-ORG-01 a UC-ORG-09 e aos testes CT-ORG.
