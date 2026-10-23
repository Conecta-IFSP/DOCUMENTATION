# Dicionário de Dados - Modelo Relacional - Cadastro de Organização

## ORGANIZACAO

| Coluna          | Tipo           | Chave            | Nulo | Descrição                                                                                             |
| --------------- | -------------- | ---------------- | ---- | ----------------------------------------------------------------------------------------------------- |
| `_id`           | ObjectId       | PK               | Não  | Identificador da organização.                                                                         |
| `nome`          | varchar(100)   | UNIQUE           | Não  | De 2 a 100 caracteres. Único, sem distinguir maiúsculas.                                              |
| `cidade`        | varchar        | —                | Não  | Cidade informada no cadastro e usada na busca textual.                                                |
| `descricao`     | varchar(200)   | —                | Sim  | Texto livre de até 200 caracteres.                                                                    |
| `status`        | enum           | —                | Não  | PENDENTE, APROVADA ou REVOGADA. Padrão: PENDENTE.                                                     |
| `criada_por`    | ObjectId       | FK → USUARIO._id | Não  | Conta que cadastrou a organização.                                                                    |
| `codigo_acesso` | char(10)       | UNIQUE           | Não  | Código aleatório de entrada; oculto nas consultas comuns e visível somente ao administrador aprovado. |
| `membros`       | subdocumento[] | —                | Não  | Vínculos embutidos; inicia com o criador como `ADMIN/APROVADO`.                                       |
| `criado_em`     | timestamp      | —                | Não  | Data do cadastro, gravada pelo sistema.                                                               |
| `atualizado_em` | timestamp      | —                | Não  | Data da última alteração, gravada pelo sistema.                                                       |

## Valores de `status`

| Valor      | Significado                                                                             |
| ---------- | --------------------------------------------------------------------------------------- |
| `PENDENTE` | Cadastrada pelo usuário, aguardando autorização (RF-01).                                |
| `APROVADA` | Autorizada pelo administrador do sistema; aceita pedidos de acesso e comissões (RF-03). |
| `REVOGADA` | Recusada ou suspensa; só nesta situação pode ser excluída (RF-04 e RF-06).              |

## ORGANIZACAO_MEMBRO (visão lógica do array `membros`)

| Coluna          | Tipo      | Chave            | Nulo | Descrição                                                    |
| --------------- | --------- | ---------------- | ---- | ------------------------------------------------------------ |
| `usuario_id`    | ObjectId  | FK → USUARIO._id | Não  | Usuário vinculado. Não pode se repetir na mesma organização. |
| `papel`         | enum      | —                | Não  | `ADMIN` ou `MEMBRO`.                                         |
| `status`        | enum      | —                | Não  | `PENDENTE`, `APROVADO`, `REJEITADO` ou `CONVIDADO`.          |
| `solicitado_em` | timestamp | —                | Não  | Criação do vínculo.                                          |
| `aprovado_em`   | timestamp | —                | Sim  | Preenchido quando o acesso é aprovado.                       |
