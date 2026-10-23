# Dicionário de classes — Reuniões

> Padrão do Incremento 2 · Complementa o diagrama de classes do módulo

## Resumo

| Classe                 | Camada               | Responsabilidade                                          |
| ---------------------- | -------------------- | --------------------------------------------------------- |
| `ReunioesController`   | API                  | Receber rotas HTTP protegidas e delegar ao service.       |
| `ReunioesService`      | Negócio              | Aplicar autorização, tempo, presença, versão e aprovação. |
| `Reuniao`              | Domínio/persistência | Representar o encontro armazenado no MongoDB.             |
| `Participante`         | Valor embutido       | Relacionar convidado, resposta e presença.                |
| `Presenca`             | Valor embutido       | Registrar entrada, saída, situação e origem.              |
| `Ata`                  | Valor embutido       | Guardar texto versionado e aprovações.                    |
| `Comissao`             | Domínio relacionado  | Fornecer responsável, integrantes e organização.          |
| `Usuario`              | Domínio relacionado  | Identificar participantes e validar conta ativa.          |
| `TelaRegistrosReuniao` | Aplicativo           | Editar/aprovar ata e consultar lista de presença.         |

## Classes e operações

### `ReunioesController`

| Operação               | Rota                                       | Service              |
| ---------------------- | ------------------------------------------ | -------------------- |
| `listar()`             | `GET /reunioes`                            | `listar`             |
| `criar()`              | `POST /reunioes`                           | `criar`              |
| `buscarPorId()`        | `GET /reunioes/:id`                        | `buscarPorId`        |
| `atualizar()`          | `PATCH /reunioes/:id`                      | `atualizar`          |
| `responderConvite()`   | `PATCH /reunioes/:id/convite`              | `responderConvite`   |
| `marcarEntrada()`      | `POST /reunioes/:id/entrada`               | `marcarEntrada`      |
| `marcarSaida()`        | `POST /reunioes/:id/saida`                 | `marcarSaida`        |
| `atualizarPresenca()`  | `PATCH /reunioes/:id/presencas/:usuarioId` | `atualizarPresenca`  |
| `gerarListaPresenca()` | `GET /reunioes/:id/lista-presenca`         | `gerarListaPresenca` |
| `salvarAta()`          | `PATCH /reunioes/:id/ata`                  | `salvarAta`          |
| `publicarAta()`        | `POST /reunioes/:id/ata/publicar`          | `publicarAta`        |
| `aprovarAta()`         | `POST /reunioes/:id/ata/aprovar`           | `aprovarAta`         |

### `Reuniao`

| Atributo        | Tipo              | Obrigatório | Regra                                                |
| --------------- | ----------------- | ----------: | ---------------------------------------------------- |
| `_id`           | `ObjectId`        |         Sim | Gerado pelo MongoDB.                                 |
| `comissao_id`   | `ObjectId`        |         Sim | Referência a comissão ativa no agendamento.          |
| `titulo`        | `string`          |         Sim | 3 a 120 caracteres no DTO.                           |
| `local`         | `string`          |         Sim | 2 a 120 caracteres no DTO.                           |
| `inicio`        | `Date`            |         Sim | Instante em UTC.                                     |
| `fim`           | `Date`            |         Sim | Posterior ao início e, na criação, ao momento atual. |
| `pauta`         | `string`          |         Sim | Padrão vazio; até 2.000 caracteres.                  |
| `criada_por`    | `ObjectId`        |         Sim | Organizador da reunião.                              |
| `situacao`      | `SituacaoReuniao` |         Sim | `AGENDADA` ou `CANCELADA`.                           |
| `participantes` | `Participante[]`  |         Sim | Usuários únicos; inclui o organizador.               |
| `ata`           | `Ata`             |         Não | Criada somente após o encerramento.                  |
| `criado_em`     | `Date`            |         Sim | Timestamp automático.                                |
| `atualizado_em` | `Date`            |         Sim | Timestamp automático.                                |

### `Participante`

| Atributo     | Tipo              | Obrigatório | Regra                                           |
| ------------ | ----------------- | ----------: | ----------------------------------------------- |
| `usuario_id` | `ObjectId`        |         Sim | Integrante da comissão na criação.              |
| `convite`    | `RespostaConvite` |         Sim | `PENDENTE`, `ACEITO` ou `RECUSADO`.             |
| `presenca`   | `Presenca`        |         Não | Ausente até entrada ou marcação do organizador. |

### `Presenca`

| Atributo      | Tipo               | Obrigatório | Regra                                     |
| ------------- | ------------------ | ----------: | ----------------------------------------- |
| `entrada`     | `Date`             |         Sim | Uma única entrada.                        |
| `saida`       | `Date`             |         Não | Não ultrapassa `Reuniao.fim`.             |
| `situacao`    | `SituacaoPresenca` |         Sim | `AGUARDANDO`, `CONFIRMADA` ou `RECUSADA`. |
| `marcada_por` | `OrigemPresenca`   |         Sim | `PROPRIO` ou `RESPONSAVEL`.               |

### `Ata`

| Atributo        | Tipo         | Obrigatório | Regra                                                  |
| --------------- | ------------ | ----------: | ------------------------------------------------------ |
| `conteudo`      | `string`     |         Sim | 20 a 20.000 caracteres.                                |
| `versao`        | `number`     |         Sim | Inteiro a partir de 1; incrementado a cada salvamento. |
| `situacao`      | `string`     |         Sim | `RASCUNHO`, `EM_APROVACAO` ou `APROVADA`.              |
| `autor_id`      | `ObjectId`   |         Sim | Responsável que salvou o texto.                        |
| `atualizada_em` | `Date`       |         Sim | Instante da última versão.                             |
| `aprovadores`   | `ObjectId[]` |         Sim | Presentes confirmados congelados na publicação.        |
| `aprovacoes`    | `ObjectId[]` |         Sim | Subconjunto único de `aprovadores`.                    |

## DTOs e validação

| DTO                          | Campos principais                                         |
| ---------------------------- | --------------------------------------------------------- |
| `CreateReuniaoDto`           | comissão, título, local, início, fim, pauta e convidados. |
| `UpdateReuniaoDto`           | somente `situacao: CANCELADA`.                            |
| `ResponderConviteReuniaoDto` | `aceito: boolean`.                                        |
| `AtualizarPresencaDto`       | `situacao` igual a `CONFIRMADA` ou `RECUSADA`.            |
| `SalvarAtaDto`               | `conteudo` e `versao`.                                    |
| `VersaoAtaDto`               | `versao` inteira, mínima zero.                            |

## Rastreabilidade

RF-REU-01 a RF-REU-12; diagrama de classes e dicionário relacional.
