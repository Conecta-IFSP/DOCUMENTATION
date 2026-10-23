# Dicionário de classes — Cadastro de comissão

> Padrão do Incremento 1 · Complementa o diagrama de classes do módulo.

## Resumo

| Classe                | Camada               | Responsabilidade                                                      |
| --------------------- | -------------------- | --------------------------------------------------------------------- |
| `ComissoesController` | API                  | Receber rotas HTTP e delegar ao service.                              |
| `ComissoesService`    | Negócio              | Validar autorização, elegibilidade, responsável único e concorrência. |
| `Comissao`            | Domínio/persistência | Representar a comissão armazenada no MongoDB.                         |
| `MembroComissao`      | Valor embutido       | Relacionar usuário à comissão com um papel.                           |
| `Organizacao`         | Domínio relacionado  | Fornecer escopo e autorização organizacional.                         |
| `Usuario`             | Domínio relacionado  | Identificar membros elegíveis.                                        |

## Classes e operações

### `ComissoesController`

| Operação                      | Entrada principal   | Saída                     |
| ----------------------------- | ------------------- | ------------------------- |
| `listar()`                    | usuário autenticado | comissões permitidas      |
| `buscarPorId(id)`             | ID e usuário        | comissão autorizada       |
| `criar(dto)`                  | `CreateComissaoDto` | comissão criada           |
| `atualizar(id,dto)`           | `UpdateComissaoDto` | comissão atualizada       |
| `adicionarMembro(id,dto)`     | usuário alvo        | equipe atualizada         |
| `definirResponsavel(id,dto)`  | novo responsável    | equipe com um responsável |
| `removerMembro(id,usuarioId)` | IDs                 | comissão atualizada       |

### `Comissao`

| Atributo         | Tipo               | Obrigatório | Regra                                        |
| ---------------- | ------------------ | ----------: | -------------------------------------------- |
| `_id`            | `ObjectId`         |         Sim | Gerado pelo MongoDB.                         |
| `nome`           | `string`           |         Sim | 2 a 100 caracteres.                          |
| `descricao`      | `string`           |         Não | Até 1.000 caracteres.                        |
| `organizacao_id` | `ObjectId`         |         Sim | Referência à organização aprovada.           |
| `ativo`          | `boolean`          |         Sim | Padrão `true`; desativação é lógica.         |
| `membros`        | `MembroComissao[]` |         Sim | Usuários únicos e exatamente um responsável. |
| `criado_em`      | `Date`             |         Sim | Timestamp automático.                        |
| `atualizado_em`  | `Date`             |         Sim | Timestamp automático.                        |

### `MembroComissao`

| Atributo     | Tipo            | Obrigatório | Regra                                    |
| ------------ | --------------- | ----------: | ---------------------------------------- |
| `usuario_id` | `ObjectId`      |         Sim | Usuário ativo e aprovado na organização. |
| `papel`      | `PapelComissao` |         Sim | `RESPONSAVEL` ou `MEMBRO`.               |

## Rastreabilidade

As classes sustentam RF-COM-01 a RF-COM-09 e SD-COM-01 a SD-COM-09.
