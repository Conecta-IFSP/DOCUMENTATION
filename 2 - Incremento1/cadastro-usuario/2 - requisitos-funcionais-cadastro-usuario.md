# Requisitos funcionais — Cadastro de usuário

> Padrão do Incremento 1 · Módulo USR

## Resumo

Define RF-USR-01 a RF-USR-07. Todas as chamadas exigem `X-API-Key`; operações de vínculo também exigem JWT, conta ativa e tipo `USUARIO`.

## Visão geral

| ID        | Requisito                         | Rota principal                                     |
| --------- | --------------------------------- | -------------------------------------------------- |
| RF-USR-01 | Cadastrar e confirmar conta local | `POST /usuarios`, `POST /auth/verificar-cadastro`  |
| RF-USR-02 | Autenticar com senha              | `POST /auth/login`                                 |
| RF-USR-03 | Autenticar com Google             | `POST /auth/google`                                |
| RF-USR-04 | Solicitar acesso por código       | `POST /organizacoes/entrar`                        |
| RF-USR-05 | Convidar e responder              | `POST .../convites`, `PATCH .../convite`           |
| RF-USR-06 | Analisar solicitação              | `PATCH .../membros/:usuarioId`                     |
| RF-USR-07 | Gerenciar papel e remoção         | `PATCH .../papel`, `DELETE .../membros/:usuarioId` |

## Especificações

### RF-USR-01 — Cadastrar e confirmar conta local

| Item              | Especificação                                                                                                                                                                   |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Entrada**       | Nome, e-mail, senha mínima de oito e confirmação; depois e-mail e código.                                                                                                       |
| **Pré-condições** | E-mail disponível.                                                                                                                                                              |
| **Processamento** | `UsuariosService.criar()` normaliza e usa bcrypt custo 10; `AuthService` gera código, guarda SHA-256 por 30 min e envia por Nodemailer; a confirmação compara hash e expiração. |
| **Persistência**  | `usuarios` com `LOCAL`, `email_verificado=false`; após confirmar, flag verdadeira e campos temporários removidos.                                                               |
| **Saída**         | Cadastro iniciado e, depois, confirmação concluída.                                                                                                                             |
| **Erros**         | 400 entrada/código; 409 duplicidade; 503 SMTP com remoção da conta provisória.                                                                                                  |

### RF-USR-02 — Autenticar com senha

| Item              | Especificação                                                                                        |
| ----------------- | ---------------------------------------------------------------------------------------------------- |
| **Entrada**       | `{email, senha}` em `POST /auth/login`.                                                              |
| **Pré-condições** | Conta ativa, confirmada e com `senha_hash`.                                                          |
| **Processamento** | Busca campos protegidos, executa `bcrypt.compare` e emite JWT com `sub`, `email`, `tipo` e uma hora. |
| **Persistência**  | JWT não é persistido; o app guarda no SecureStore/localStorage.                                      |
| **Saída**         | `{access_token, usuario}` e validação posterior em `GET /usuarios/me`.                               |
| **Erros**         | 401 genérico para credencial/estado inválido.                                                        |

### RF-USR-03 — Autenticar com Google

| Item              | Especificação                                                                                       |
| ----------------- | --------------------------------------------------------------------------------------------------- |
| **Entrada**       | `id_token` ou authorization code com redirect URI/plataforma.                                       |
| **Pré-condições** | Cliente Google configurado e e-mail Google verificado.                                              |
| **Processamento** | Valida `aud`, `iss`, `exp`; localiza por `google_id`/e-mail; cria `GOOGLE` ou vincula como `AMBOS`. |
| **Persistência**  | `usuarios.google_id`, `provedor_autenticacao` e avatar protegido.                                   |
| **Saída**         | Mesmo formato do login local.                                                                       |
| **Erros**         | 400 token, emissor, audiência ou expiração inválidos.                                               |

### RF-USR-04 — Solicitar acesso por código

| Item              | Especificação                                                       |
| ----------------- | ------------------------------------------------------------------- |
| **Entrada**       | Código de dez caracteres em `POST /organizacoes/entrar`.            |
| **Pré-condições** | Organização aprovada e vínculo permitido.                           |
| **Processamento** | Normaliza, busca `codigo_acesso` e cria ou reabre vínculo.          |
| **Persistência**  | `organizacoes.membros[]` como `MEMBRO/PENDENTE`.                    |
| **Saída**         | Organização atualizada.                                             |
| **Erros**         | 404 código; 409 membro/solicitação em análise/convite já existente. |

### RF-USR-05 — Convidar e responder

| Item              | Especificação                                                                   |
| ----------------- | ------------------------------------------------------------------------------- |
| **Entrada**       | E-mail do alvo; depois `{aceitar}` pelo convidado.                              |
| **Pré-condições** | Admin aprovado; alvo ativo, comum e sem vínculo incompatível.                   |
| **Processamento** | Convite grava `CONVIDADO`; resposta aceita como `APROVADO` ou remove o vínculo. |
| **Persistência**  | `organizacoes.membros[]`.                                                       |
| **Saída**         | Organização atualizada.                                                         |
| **Erros**         | 403 permissão; 404 usuário/convite; 409 estado.                                 |

### RF-USR-06 — Analisar solicitação

| Item              | Especificação                                                                |
| ----------------- | ---------------------------------------------------------------------------- |
| **Entrada**       | `APROVADO` ou `REJEITADO`.                                                   |
| **Pré-condições** | Admin aprovado; alvo `PENDENTE`, ou `REJEITADO` para reaprovação.            |
| **Processamento** | Atualiza com filtro do estado original para impedir sobrescrita concorrente. |
| **Persistência**  | `membros.$.status` e `aprovado_em`.                                          |
| **Saída**         | Organização atualizada.                                                      |
| **Erros**         | 403 permissão; 404 alvo; 409 transição/concorrência.                         |

### RF-USR-07 — Gerenciar papel e remoção

| Item              | Especificação                                                                                        |
| ----------------- | ---------------------------------------------------------------------------------------------------- |
| **Entrada**       | Papel `ADMIN                                                                                         | MEMBRO` ou ID a remover. |
| **Pré-condições** | Admin aprovado; alvo aprovado; remoção somente de membro comum.                                      |
| **Processamento** | Altera papel preservando ao menos um admin; remoção usa `$pull` também nas comissões da organização. |
| **Persistência**  | `organizacoes.membros[]` e `comissoes.membros[]`.                                                    |
| **Saída**         | Organização atualizada ou confirmação.                                                               |
| **Erros**         | 400 último admin; 403/404; 409 concorrência.                                                         |

## Rastreabilidade

RF-USR-01 a RF-USR-07 cobrem UC-USR-01 a UC-USR-09 e CT-USR-01 a CT-USR-09.
