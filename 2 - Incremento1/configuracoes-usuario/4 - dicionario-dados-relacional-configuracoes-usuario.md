# Dicionário de Dados - Modelo Relacional - Configurações de Usuário

## USUARIO

| Coluna | Tipo | Chave | Nulo | Descrição |
|---|---|---|---|---|
| `_id` | ObjectId | PK | Não | Identificador da conta. |
| `nome` | varchar | — | Não | Nome da pessoa usuária. |
| `email` | varchar | UNIQUE | Não | E-mail normalizado para minúsculas; não pode pertencer a outra conta. |
| `senha_hash` | varchar | — | Não | Hash bcrypt da senha atual; não é retornado nas consultas comuns. |
| `senhaHash` | varchar | — | Sim | Hash legado, mantido temporariamente para contas antigas. |
| `tipo` | enum | — | Não | `USUARIO` ou `ADMIN_SISTEMA`. |
| `tema` | enum | — | Não | `claro`, `escuro` ou `sistema`; padrão `sistema`. |
| `reset_senha_token_hash` | varchar | — | Sim | SHA-256 do token de recuperação. Nunca armazena o token original. |
| `reset_senha_expira_em` | timestamp | — | Sim | Prazo do token; definido para 30 minutos após a solicitação. |
| `ativo` | boolean | — | Não | Contas inativas não podem autenticar nem solicitar recuperação. |
| `criado_em` | timestamp | — | Não | Data de criação, preenchida pelo Mongoose. |
| `atualizado_em` | timestamp | — | Não | Data da última alteração, preenchida pelo Mongoose. |

## Valores de `tema`

| Valor | Significado |
|---|---|
| `claro` | Usa sempre a paleta clara. |
| `escuro` | Usa sempre a paleta escura. |
| `sistema` | Acompanha a preferência clara ou escura configurada no aparelho. |

## Ciclo dos campos de recuperação

| Momento | `reset_senha_token_hash` | `reset_senha_expira_em` |
|---|---|---|
| Sem recuperação pendente | Ausente | Ausente |
| Após solicitar recuperação | SHA-256 do novo token | Horário atual + 30 minutos |
| Após redefinir a senha | Removido | Removido |
| Após expirar | Permanece sem validade até nova solicitação ou limpeza | Data anterior ao horário atual |

