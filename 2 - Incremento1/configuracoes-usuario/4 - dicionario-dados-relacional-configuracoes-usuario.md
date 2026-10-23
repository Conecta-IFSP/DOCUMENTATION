# Dicionário relacional — Configurações de usuário

> Padrão do Incremento 1 · Módulo CFG

## Resumo

Detalha campos, tipos, seleção, expiração e descarte dos dados permanentes e temporários da conta.

## Coleção `usuarios`

| Caminho                      | Tipo BSON/lógico | Obrigatório | Índice/padrão    | Descrição e restrição                               |
| ---------------------------- | ---------------- | ----------: | ---------------- | --------------------------------------------------- |
| `_id`                        | `ObjectId`       |         Sim | PK MongoDB       | identificador da conta                              |
| `nome`                       | `String`         |         Sim | —                | nome com espaços externos removidos                 |
| `email`                      | `String`         |         Sim | único            | normalizado em minúsculas                           |
| `senha_hash`                 | `String`         |         Não | `select:false`   | bcrypt custo 10                                     |
| `tipo`                       | `String`         |         Sim | `USUARIO`        | `USUARIO` ou `ADMIN_SISTEMA`                        |
| `tema`                       | `String`         |         Sim | `sistema`        | `sistema`, `claro` ou `escuro`                      |
| `reset_senha_codigo_hash`    | `String`         |         Não | `select:false`   | SHA-256 do código de 7 caracteres                   |
| `reset_senha_token_hash`     | `String`         |         Não | `select:false`   | SHA-256 do token hexadecimal de 64 caracteres       |
| `reset_senha_expira_em`      | `Date`           |         Não | `select:false`   | limite de 30 minutos iniciado na solicitação        |
| `email_pendente`             | `String`         |         Não | `select:false`   | novo endereço normalizado                           |
| `email_pendente_codigo_hash` | `String`         |         Não | `select:false`   | SHA-256 do código de confirmação                    |
| `email_pendente_expira_em`   | `Date`           |         Não | `select:false`   | limite de 30 minutos                                |
| `email_verificado`           | `Boolean`        |         Sim | `true` no schema | contas locais novas informam `false` explicitamente |
| `ativo`                      | `Boolean`        |         Sim | `true`           | conta inativa falha no `JwtAuthGuard`               |
| `provedor_autenticacao`      | `String`         |         Sim | `LOCAL`          | `LOCAL`, `GOOGLE` ou `AMBOS`                        |
| `criado_em`                  | `Date`           |         Sim | timestamp        | criação automática                                  |
| `atualizado_em`              | `Date`           |         Sim | timestamp        | última gravação automática                          |

## Ciclo da recuperação

1. Solicitação define `reset_senha_codigo_hash` e `reset_senha_expira_em` e apaga token antigo.
2. Verificação correta define `reset_senha_token_hash` e apaga o código.
3. Redefinição correta atualiza `senha_hash` e apaga token e expiração.
4. Solicitação posterior substitui os valores anteriores.

## Ciclo da troca de e-mail

1. Solicitação define `email_pendente`, `email_pendente_codigo_hash` e `email_pendente_expira_em`.
2. Confirmação repete a verificação de unicidade, copia `email_pendente` para `email` e apaga os três campos.
