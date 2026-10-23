# Dicionário de classes — Cadastro de organização

> Padrão do Incremento 1 · Complementa o diagrama de classes do módulo.

## Resumo

| Classe                   | Camada               | Responsabilidade                                              |
| ------------------------ | -------------------- | ------------------------------------------------------------- |
| `OrganizacoesController` | API                  | Expor cadastro, consulta, análise e gestão de vínculos.       |
| `OrganizacoesService`    | Negócio              | Aplicar autorização, estados, código, convite e concorrência. |
| `Organizacao`            | Domínio/persistência | Armazenar dados e membros da organização.                     |
| `MembroOrganizacao`      | Valor embutido       | Representar papel e situação do vínculo.                      |
| `Usuario`                | Domínio relacionado  | Identificar criador, convidado e integrante.                  |

## Classes e operações

### `OrganizacoesController`

| Operação               | Rota principal                                     | Responsabilidade              |
| ---------------------- | -------------------------------------------------- | ----------------------------- |
| `criar()`              | `POST /organizacoes`                               | Cadastrar organização.        |
| `listar()`             | `GET /organizacoes`                                | Consultar por escopo e busca. |
| `atualizar()`          | `PATCH /organizacoes/:id`                          | Alterar dados ou situação.    |
| `remover()`            | `DELETE /organizacoes/:id`                         | Excluir organização revogada. |
| `entrar()`             | `POST /organizacoes/entrar`                        | Criar solicitação por código. |
| `convidar()`           | `POST /organizacoes/:id/convites`                  | Criar convite.                |
| `responderConvite()`   | `PATCH /organizacoes/:id/convite`                  | Aceitar ou recusar.           |
| `alterarPapelMembro()` | `PATCH /organizacoes/:id/membros/:usuarioId/papel` | Promover ou rebaixar.         |

### `Organizacao`

| Atributo        | Tipo                  | Obrigatório | Regra                                      |
| --------------- | --------------------- | ----------: | ------------------------------------------ |
| `_id`           | `ObjectId`            |         Sim | Gerado pelo MongoDB.                       |
| `nome`          | `string`              |         Sim | Único sem diferenciar maiúsculas.          |
| `cidade`        | `string`              |         Sim | Usada também na busca.                     |
| `descricao`     | `string`              |         Não | Até 200 caracteres na entrada.             |
| `status`        | `StatusOrganizacao`   |         Sim | `PENDENTE`, `APROVADA` ou `REVOGADA`.      |
| `criada_por`    | `ObjectId`            |         Sim | Usuário que cadastrou.                     |
| `codigo_acesso` | `string`              |         Sim | Único, dez caracteres e oculto por padrão. |
| `membros`       | `MembroOrganizacao[]` |         Sim | Vínculos embutidos e sem usuário repetido. |

### `MembroOrganizacao`

| Atributo        | Tipo                      | Obrigatório | Regra                                               |
| --------------- | ------------------------- | ----------: | --------------------------------------------------- |
| `usuario_id`    | `ObjectId`                |         Sim | Referência ao usuário.                              |
| `papel`         | `PapelOrganizacao`        |         Sim | `ADMIN` ou `MEMBRO`.                                |
| `status`        | `StatusMembroOrganizacao` |         Sim | `PENDENTE`, `APROVADO`, `REJEITADO` ou `CONVIDADO`. |
| `solicitado_em` | `Date`                    |         Sim | Data de criação do vínculo.                         |
| `aprovado_em`   | `Date`                    |         Não | Preenchida na aprovação.                            |

## Rastreabilidade

As classes sustentam RF-ORG-01 a RF-ORG-09 e SD-ORG-01 a SD-ORG-08.
