# Dicionário de Dados - Modelo Relacional - Cadastro de Organização

## ORGANIZACAO

| Coluna | Tipo | Chave | Nulo | Descrição |
|---|---|---|---|---|
| `_id` | ObjectId | PK | Não | Identificador da organização. |
| `nome` | varchar(100) | UNIQUE | Não | De 2 a 100 caracteres. Único, sem distinguir maiúsculas. |
| `descricao` | varchar(200) | — | Sim | Texto livre de até 200 caracteres. |
| `status` | enum | — | Não | PENDENTE, APROVADA ou REVOGADA. Padrão: PENDENTE. |
| `criada_por` | ObjectId | FK → USUARIO._id | Não | Conta que cadastrou a organização. |
| `criado_em` | timestamp | — | Não | Data do cadastro, gravada pelo sistema. |
| `atualizado_em` | timestamp | — | Não | Data da última alteração, gravada pelo sistema. |

## Valores de `status`

| Valor | Significado |
|---|---|
| `PENDENTE` | Cadastrada pelo usuário, aguardando autorização (RF-01). |
| `APROVADA` | Autorizada pelo administrador do sistema; aceita pedidos de acesso e comissões (RF-03). |
| `REVOGADA` | Recusada ou suspensa; só nesta situação pode ser excluída (RF-04 e RF-06). |