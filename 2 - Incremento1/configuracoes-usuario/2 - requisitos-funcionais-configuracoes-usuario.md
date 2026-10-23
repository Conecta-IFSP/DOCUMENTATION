# Requisitos funcionais — Configurações de usuário

Todas as rotas recebem `X-API-Key`. As rotas `/usuarios/me...` também exigem `Authorization: Bearer <JWT>`.

## RF-CFG-01 — Consultar e alterar o próprio perfil

| Item | Especificação |
|---|---|
| Entrada | `GET /usuarios/me`; `PATCH /usuarios/me` com `nome?: string` e/ou `tema?: sistema\|claro\|escuro` |
| Validação | `AtualizarPerfilDto`: nome mínimo 2; tema enumerado. O campo `email`, embora presente no DTO, é recusado pelo service e deve usar RF-CFG-02. |
| Processamento | `UsuariosController.buscarMeuPerfil/atualizarMeuPerfil` → `UsuariosService.buscarPorId/atualizarPerfil`; ID vem de `req.usuario.sub`. |
| Persistência | Coleção `usuarios`; leitura por `_id`; atualização de `nome`, `tema` e `atualizado_em`. |
| Saída | Documento do usuário sem campos `select: false`. |
| Interface | `TelaPerfil` usa `useFocusEffect`, `react-hook-form` e chama `sessao.atualizar()` após mudar nome. |

## RF-CFG-02 — Alterar e-mail com confirmação

| Item | Especificação |
|---|---|
| Solicitação | `POST /usuarios/me/email/solicitar`, `{email}`; valida formato, diferença do atual e unicidade. |
| Gravação temporária | `email_pendente`, `email_pendente_codigo_hash=SHA-256(código)` e `email_pendente_expira_em=agora+30min`. |
| Efeito colateral | Nodemailer envia código `XXX-XXXX` ao novo endereço. |
| Confirmação | `POST /usuarios/me/email/confirmar`, `{email,codigo}`; normaliza 7 caracteres, compara hash e expiração, repete verificação de unicidade. |
| Gravação final | Atualiza `email`, define `email_verificado=true` e remove os três campos temporários. |
| Erros | `400` para mesmo e-mail, duplicidade ou código inválido/expirado; `401` sem JWT. |

## RF-CFG-03 — Trocar senha autenticada

| Item | Especificação |
|---|---|
| Rota/entrada | `PATCH /usuarios/me/senha`, `{senha_atual,nova_senha}` |
| Validação | strings; nova senha com mínimo 8; confirmação é validada apenas no frontend e não é enviada. |
| Processamento | Carrega usuário por `_id` com `+senha_hash`; `bcrypt.compare` valida atual e impede nova igual; `bcrypt.hash(nova,10)` gera o valor persistido. |
| Saída | `{mensagem:'Senha alterada com sucesso'}` |
| Erros | `400` se conta não possui senha local, senha atual está errada ou nova repete a atual. |
| Limite | Não remove JWT do aparelho nem invalida tokens emitidos. |

## RF-CFG-04 — Recuperar conta por código

| Etapa | Especificação |
|---|---|
| Solicitar | `POST /auth/recuperar-conta`, `{email}`. Para conta ativa, gera 7 caracteres por `randomBytes`, guarda SHA-256 e expiração de 30 minutos; sempre responde mensagem genérica. |
| Verificar | `POST /auth/verificar-recuperacao`, `{email,codigo}`. Valida hash/expiração; gera token de 32 bytes em hexadecimal, guarda SHA-256 e remove o código. |
| Redefinir | `POST /auth/redefinir-senha`, `{token,nova_senha}`. Exige mínimo 8, localiza conta ativa pelo hash e expiração, grava bcrypt custo 10 e apaga token/expiração. |
| E-mail | Nodemailer envia código; sem SMTP configurado usa transporte JSON e registra o código no terminal de desenvolvimento. |
| Interface | Uma única `TelaRecuperarConta` controla etapas `email`, `codigo`, `senha`, `concluido`; antes da gravação solicita autenticação local quando disponível. |

## RF-CFG-05 — Definir preferência de tema

| Item | Especificação |
|---|---|
| Entrada | Seleção `sistema`, `claro` ou `escuro` em `TelaPerfil`. |
| Interface | `TemaContext.definirPreferencia()` aplica imediatamente e grava a chave `conecta_mais_tema` no SecureStore ou localStorage. |
| API | `PATCH /usuarios/me`, `{tema}`; `TemaUsuario` restringe os três valores. |
| Falha | A tela guarda a preferência anterior; se a API falhar, restaura estado e armazenamento local. |
| Tema sistema | `useColorScheme()` escolhe claro/escuro conforme o aparelho. |

## RF-CFG-06 — Encerrar sessão local

| Item | Especificação |
|---|---|
| Operação | `SessaoContext.sair()` incrementa a geração da sessão, remove `conecta_mais_token` e limpa estado/avisos. |
| Navegação | `Stack.Protected` remove rotas autenticadas quando `usuario` é `null`. |
| Servidor | Não existe endpoint de logout nem lista de revogação. O JWT continua criptograficamente válido até expirar em 1 hora. |

## Matriz de rotas

| Método | Rota | JWT | Controller → service | Coleção/campo principal |
|---|---|---|---|---|
| GET | `/usuarios/me` | Sim | `buscarMeuPerfil` → `buscarPorId` | `usuarios._id` |
| PATCH | `/usuarios/me` | Sim | `atualizarMeuPerfil` → `atualizarPerfil` | `nome`, `tema` |
| POST | `/usuarios/me/email/solicitar` | Sim | `solicitarAlteracaoEmail` | campos `email_pendente*` |
| POST | `/usuarios/me/email/confirmar` | Sim | `confirmarAlteracaoEmail` | `email`, campos pendentes |
| PATCH | `/usuarios/me/senha` | Sim | `alterarMinhaSenha` → `alterarSenha` | `senha_hash` |
| POST | `/auth/recuperar-conta` | Não | `recuperarConta` → `solicitarRecuperacao` | `reset_senha_codigo_hash`, expiração |
| POST | `/auth/verificar-recuperacao` | Não | `verificarRecuperacao` | código → token hash |
| POST | `/auth/redefinir-senha` | Não | `redefinirSenha` | `senha_hash`, limpeza do token |
