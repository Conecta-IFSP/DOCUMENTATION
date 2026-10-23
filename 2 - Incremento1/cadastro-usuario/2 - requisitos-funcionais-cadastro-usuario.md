# Requisitos funcionais — Cadastro e acesso de usuário

## RF-USR-01 — Cadastrar conta local e confirmar e-mail

`TelaCadastro` usa `react-hook-form` para validar nome, e-mail, senha de no mínimo 8 e confirmação. Envia `POST /usuarios`. `CreateUsuarioDto` valida; `UsuariosService.criar()` normaliza o e-mail, verifica unicidade e grava bcrypt custo 10 com `email_verificado=false`. `AuthService.enviarCodigoVerificacao()` grava SHA-256 do código e expiração de 30 min e envia por Nodemailer. `POST /auth/verificar-cadastro` recebe e-mail/código, compara os campos ocultos e confirma a conta.

## RF-USR-02 — Autenticar com e-mail e senha

`POST /auth/login` recebe `{email,senha}`. Exige conta existente, ativa, confirmada e com `senha_hash`; usa `bcrypt.compare` e retorna `{access_token,usuario}`. O JWT contém `sub`, `email`, `tipo` e expira em 1 hora. O app valida `/usuarios/me` antes de considerar a sessão iniciada e salva o token no SecureStore/localStorage.

## RF-USR-03 — Autenticar com Google

O aplicativo usa Google Sign-In nativo ou Expo Auth Session. `POST /auth/google` aceita `id_token` ou authorization code/redirect URI/plataforma. `AuthService` valida `aud`, `iss`, `exp` e e-mail verificado via Google; localiza por `google_id` ou e-mail, vincula como `AMBOS` quando necessário ou cria conta `GOOGLE`. Retorna o mesmo formato de login local.

## RF-USR-04 — Solicitar acesso por código

`POST /organizacoes/entrar` recebe código de 10 caracteres. `OrganizacoesService.entrarComCodigo()` normaliza, busca `organizacoes.codigo_acesso` em organização `APROVADA` e chama `adicionarMembro()`. Novo vínculo é `MEMBRO/PENDENTE`; vínculo rejeitado é reaberto; membro/pendência/convite existente gera conflito.

## RF-USR-05 — Convidar e responder convite

Administrador aprovado envia e-mail a `POST /organizacoes/:id/convites`. O service busca usuário ativo que não seja `ADMIN_SISTEMA` e grava/reaproveita vínculo `CONVIDADO`. O destinatário responde em `PATCH /organizacoes/:id/convite` com `{aceitar}`; aceitação grava `APROVADO` e `aprovado_em`, recusa remove o vínculo.

## RF-USR-06 — Analisar solicitação

Administrador aprovado chama `PATCH /organizacoes/:id/membros/:usuarioId` com `APROVADO` ou `REJEITADO`. Apenas `PENDENTE` pode ser analisado, exceto `REJEITADO→APROVADO`. A atualização filtra o estado original e as permissões para não sobrescrever mudança concorrente; falha retorna `409`.

## RF-USR-07 — Gerenciar papel e remoção

`PATCH /organizacoes/:id/membros/:usuarioId/papel` promove/rebaixa membro aprovado e impede zero administradores. `DELETE /organizacoes/:id/membros/:usuarioId` remove somente membro comum e, depois, usa `$pull` em todas as comissões da organização. Conta e outros vínculos permanecem.

## Matriz de persistência

| Requisito | Coleção/campos |
|---|---|
| RF-USR-01 | `usuarios`: nome, email, senha_hash, verificacao_email_*, email_verificado, provedor_autenticacao |
| RF-USR-02/03 | leitura/atualização de `usuarios`; JWT não é persistido |
| RF-USR-04/05/06/07 | `organizacoes.membros[]`: usuario_id, papel, status, solicitado_em, aprovado_em |
| RF-USR-07 | `comissoes.membros[]` na remoção de acesso |

Todas as chamadas exigem `X-API-Key`; vínculos exigem JWT, `USUARIO`, conta ativa e autorização de domínio.
