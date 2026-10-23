# Dicionário de classes — Configurações de usuário

## `Usuario`

Schema Mongoose da coleção `usuarios`. Os campos sensíveis marcados com `select: false` só são lidos explicitamente pelo service responsável.

| Atributo                     | Tipo                   | Visibilidade padrão | Restrição/finalidade                              |
| ---------------------------- | ---------------------- | ------------------- | ------------------------------------------------- |
| `_id`                        | `ObjectId`             | Pública             | Identificador MongoDB                             |
| `nome`                       | `String`               | Pública             | obrigatório, `trim`; DTO de perfil exige mínimo 2 |
| `email`                      | `String`               | Pública             | obrigatório, único, minúsculo, `trim`             |
| `senha_hash`                 | `String?`              | Oculta              | bcrypt custo 10; ausente em conta somente Google  |
| `tipo`                       | `TipoUsuario`          | Pública             | `USUARIO` ou `ADMIN_SISTEMA`                      |
| `tema`                       | `TemaUsuario`          | Pública             | padrão `sistema`                                  |
| `reset_senha_codigo_hash`    | `String?`              | Oculta              | SHA-256 do código de recuperação                  |
| `reset_senha_token_hash`     | `String?`              | Oculta              | SHA-256 do token criado após validar o código     |
| `reset_senha_expira_em`      | `Date?`                | Oculta              | validade compartilhada do processo, 30 minutos    |
| `email_pendente`             | `String?`              | Oculta              | novo endereço aguardando confirmação              |
| `email_pendente_codigo_hash` | `String?`              | Oculta              | SHA-256 do código enviado ao novo endereço        |
| `email_pendente_expira_em`   | `Date?`                | Oculta              | agora + 30 minutos                                |
| `email_verificado`           | `Boolean`              | Pública             | necessário no login local                         |
| `ativo`                      | `Boolean`              | Pública             | padrão `true`; guard recusa conta inativa         |
| `provedor_autenticacao`      | `ProvedorAutenticacao` | Pública             | `LOCAL`, `GOOGLE` ou `AMBOS`                      |
| `criado_em`/`atualizado_em`  | `Date`                 | Pública             | timestamps automáticos                            |

## Controllers e services

| Classe/método                             | Responsabilidade                                            |
| ----------------------------------------- | ----------------------------------------------------------- |
| `UsuariosController.buscarMeuPerfil()`    | entrega ao service o `sub` extraído pelo `JwtAuthGuard`     |
| `UsuariosController.atualizarMeuPerfil()` | recebe `AtualizarPerfilDto` e altera apenas a própria conta |
| `UsuariosService.atualizarPerfil()`       | permite nome/tema e recusa e-mail direto                    |
| `UsuariosService.alterarSenha()`          | compara senha atual/nova e grava bcrypt                     |
| `AuthService.solicitarAlteracaoEmail()`   | garante diferença/unicidade e envia código                  |
| `AuthService.confirmarAlteracaoEmail()`   | valida código/expiração e efetiva o endereço                |
| `AuthService.solicitarRecuperacao()`      | envia mensagem genérica e código quando a conta existe      |
| `AuthService.verificarRecuperacao()`      | consome código e cria token temporário                      |
| `AuthService.redefinirSenha()`            | consome token e grava nova senha                            |
| `AuthService.gerarCodigoVerificacao()`    | gera 7 caracteres de `A-Z0-9` com `randomBytes`             |
| `AuthService.hashToken()`                 | calcula SHA-256 em hexadecimal                              |

## DTOs

| DTO                          | Campos                      | Restrições                                                             |
| ---------------------------- | --------------------------- | ---------------------------------------------------------------------- |
| `AtualizarPerfilDto`         | `nome?`, `email?`, `tema?`  | nome ≥2; e-mail válido; tema enumerado; e-mail é recusado pelo service |
| `AlterarSenhaDto`            | `senha_atual`, `nova_senha` | strings; nova ≥8                                                       |
| `SolicitarAlteracaoEmailDto` | `email`                     | e-mail válido                                                          |
| `ConfirmarAlteracaoEmailDto` | `email`, `codigo`           | e-mail válido; código com 7 ou 8 caracteres contando hífen             |
| `RecuperarContaDto`          | `email`                     | e-mail válido                                                          |
| `VerificarRecuperacaoDto`    | `email`, `codigo`           | e-mail válido; código com 7 ou 8 caracteres                            |
| `RedefinirSenhaDto`          | `token`, `nova_senha`       | token string; senha ≥8                                                 |

## Classes do aplicativo

| Componente/módulo            | Responsabilidade                                                                           |
| ---------------------------- | ------------------------------------------------------------------------------------------ |
| `TelaPerfil`                 | carregar perfil ao foco, editar nome, tema e abrir ações protegidas                        |
| `TelaTrocarEmail`            | controlar etapas novo e-mail → código → concluído                                          |
| `TelaAlterarSenha`           | validar senha atual/nova/confirmação e apresentar resultado                                |
| `TelaRecuperarConta`         | controlar quatro etapas da recuperação sem link externo                                    |
| `TemaContext`                | resolver tema efetivo e persistir a chave `conecta_mais_tema`                              |
| `SessaoContext`              | salvar/remover JWT e revalidar `/usuarios/me`                                              |
| `autenticarComDispositivo()` | solicitar biometria/credencial quando cadastrada; retornar `true` na web ou sem credencial |
