# Dicionário relacional — Cadastro de comissão

> Padrão do Incremento 1 · Tipos lógicos e correspondência com MongoDB/Mongoose.

## Resumo

| Estrutura lógica  | Estrutura física      | Finalidade                   |
| ----------------- | --------------------- | ---------------------------- |
| `COMISSAO`        | `comissoes`           | Dados da comissão.           |
| `COMISSAO_MEMBRO` | `comissoes.membros[]` | Usuários e papéis embutidos. |

## COMISSAO

| Campo            | Tipo       | Chave/Nulo   | Restrição                                   |
| ---------------- | ---------- | ------------ | ------------------------------------------- |
| `_id`            | `ObjectId` | PK, não nulo | Gerado pelo MongoDB.                        |
| `nome`           | `string`   | não nulo     | 2 a 100 caracteres.                         |
| `descricao`      | `string`   | nulo         | Até 1.000 caracteres.                       |
| `organizacao_id` | `ObjectId` | FK, não nulo | Organização aprovada.                       |
| `ativo`          | `boolean`  | não nulo     | Padrão `true`.                              |
| `membros`        | `array`    | não nulo     | Usuários únicos; exatamente um responsável. |
| `criado_em`      | `Date`     | não nulo     | Automático.                                 |
| `atualizado_em`  | `Date`     | não nulo     | Automático.                                 |

## COMISSAO_MEMBRO

| Campo         | Tipo       | Chave/Nulo   | Restrição                      |
| ------------- | ---------- | ------------ | ------------------------------ |
| `comissao_id` | `ObjectId` | PK/FK lógica | Implícito no documento pai.    |
| `usuario_id`  | `ObjectId` | PK/FK lógica | Não pode se repetir na equipe. |
| `papel`       | `enum`     | não nulo     | `RESPONSAVEL` ou `MEMBRO`.     |

## Restrições

- o responsável deve estar ativo e aprovado na organização;
- membro comum também precisa ser elegível na organização;
- comissão inativa preserva a equipe, mas bloqueia sua alteração;
- atualizações de equipe usam comparação do estado anterior para detectar concorrência.

## Rastreabilidade

RF-COM-03 e RF-COM-07 a RF-COM-09.
