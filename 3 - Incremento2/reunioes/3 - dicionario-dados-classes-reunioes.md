# Dicionário de classes — Reuniões

> Padrão da disciplina · Módulo REU · Complementa o diagrama de classes · Revisado em 10/11/2026

## Resumo

| Classe                 | Camada               | Responsabilidade                                          |
| ---------------------- | -------------------- | --------------------------------------------------------- |
| `ReunioesController`   | API                  | Receber rotas HTTP protegidas e delegar ao service.       |
| `ReunioesService`      | Negócio              | Aplicar autorização, tempo, presença, versão e aprovação. |
| `Reuniao`              | Domínio/persistência | Representar o encontro armazenado no MongoDB.             |
| `Participante`         | Valor embutido       | Relacionar convidado, resposta e presença.                |
| `Presenca`             | Valor embutido       | Registrar entrada, saída, situação e origem.              |
| `Anexo`                | Valor embutido       | Identificar o arquivo da pauta guardado no GridFS.        |
| `Ata`                  | Valor embutido       | Guardar texto versionado e avaliações; detalhada no [módulo ATA](../../4%20-%20Incremento3/atas/3%20-%20dicionario-dados-classes-atas.md). |
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
| `adicionarAnexo()`     | `POST /reunioes/:id/anexos`                | `adicionarAnexo`     |
| `abrirAnexo()`         | `GET /reunioes/:id/anexos/:anexoId`        | `abrirAnexo`         |
| `removerAnexo()`       | `DELETE /reunioes/:id/anexos/:anexoId`     | `removerAnexo`       |

As rotas da ata estão no [dicionário de classes das atas](../../4%20-%20Incremento3/atas/3%20-%20dicionario-dados-classes-atas.md).

O método `atualizar` do service recebe o `PATCH /reunioes/:id` e encaminha para `cancelar`, quando vem só `situacao`, ou para `editar`, quando vêm os demais campos.

### `Reuniao`

| Atributo        | Tipo              | Obrigatório | Regra                                                |
| --------------- | ----------------- | ----------: | ---------------------------------------------------- |
| `_id`           | `ObjectId`        |         Sim | Criado pelo driver do MongoDB ao inserir: 12 bytes (hora da criação, valor sorteado e contador). Não é sequencial. |
| `comissao_id`   | `ObjectId`        |         Sim | Referência a comissão ativa no agendamento.          |
| `titulo`        | `string`          |         Sim | 3 a 120 caracteres no DTO.                           |
| `local`         | `string`          |         Sim | Padrão vazio; quando preenchido, 2 a 120 caracteres. |
| `inicio`        | `Date`            |         Sim | Instante em UTC. Em reunião de dia inteiro, 00:00 do dia. |
| `fim`           | `Date`            |         Sim | Posterior ao início e, na criação, ao momento atual. Em reunião de dia inteiro, 23:59:59.999 do dia. |
| `dia_inteiro`   | `boolean`         |         Sim | Padrão `false`. Esconde os horários na tela e muda o rótulo para "Dia inteiro". |
| `horario_alterado_em` | `Date`      |         Não | Instante da última mudança de data ou horário na edição. |
| `pauta`         | `string`          |         Sim | Padrão vazio; até 2.000 caracteres.                  |
| `anexos`        | `Anexo[]`         |         Sim | Padrão vazio; no máximo 10 itens.                    |
| `criada_por`    | `ObjectId`        |         Sim | Organizador da reunião.                              |
| `situacao`      | `SituacaoReuniao` |         Sim | `AGENDADA` ou `CANCELADA`.                           |
| `participantes` | `Participante[]`  |         Sim | Usuários únicos; inclui o organizador.               |
| `ata`           | `Ata`             |         Não | Criada somente após o encerramento.                  |
| `atas_canceladas` | `Ata[]`         |         Sim | Padrão vazio; versões reprovadas da ata.             |
| `criado_em`     | `Date`            |         Sim | Timestamp automático.                                |
| `atualizado_em` | `Date`            |         Sim | Timestamp automático; usado como guarda na edição.   |

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

### `Anexo`

| Atributo     | Tipo       | Obrigatório | Regra                                                                        |
| ------------ | ---------- | ----------: | ---------------------------------------------------------------------------- |
| `_id`        | `ObjectId` |         Sim | É o identificador do arquivo no GridFS; não é gerado como campo novo.        |
| `nome`       | `string`   |         Sim | Nome original, convertido para utf8 e cortado em 200 caracteres.             |
| `tipo`       | `string`   |         Sim | Tipo MIME do arquivo, limitado à lista permitida em `anexos.ts`.             |
| `tamanho`    | `number`   |         Sim | Tamanho em bytes, no mínimo 1 e no máximo 10.485.760 (10 MB).                |
| `enviado_em` | `Date`     |         Sim | Instante do envio.                                                           |

### `Ata`

Descrita por completo no [dicionário de classes das atas](../../4%20-%20Incremento3/atas/3%20-%20dicionario-dados-classes-atas.md). Resumo dos atributos:

| Atributo        | Tipo             | Obrigatório | Regra                                                                   |
| --------------- | ---------------- | ----------: | ----------------------------------------------------------------------- |
| `conteudo`      | `string`         |         Sim | 20 a 20.000 caracteres.                                                 |
| `versao`        | `number`         |         Sim | Inteiro a partir de 1; aumenta a cada salvamento e a cada nova versão.  |
| `situacao`      | `SituacaoAta`    |         Sim | `RASCUNHO`, `EM_APROVACAO`, `APROVADA`, `REPROVADA` ou `CANCELADA`.     |
| `autor_id`      | `ObjectId`       |         Sim | Responsável que salvou o texto.                                         |
| `atualizada_em` | `Date`           |         Sim | Instante da última gravação.                                            |
| `aprovadores`   | `ObjectId[]`     |         Sim | Presentes confirmados, congelados no envio.                             |
| `avaliacoes`    | `AvaliacaoAta[]` |         Sim | Uma decisão por aprovador, com motivo e horário.                        |

## DTOs e validação

| DTO                          | Campos principais                                         |
| ---------------------------- | --------------------------------------------------------- |
| `CreateReuniaoDto`           | comissão, título, local opcional, início, fim, `dia_inteiro` opcional, pauta e convidados. |
| `UpdateReuniaoDto`           | `situacao: CANCELADA` ou, para edição, título, local, início, fim, `dia_inteiro`, pauta e convidados, todos opcionais. |
| `ResponderConviteReuniaoDto` | `aceito: boolean`.                                        |
| `AtualizarPresencaDto`       | `situacao` igual a `CONFIRMADA` ou `RECUSADA`.            |

Os DTOs da ata (`SalvarAtaDto`, `EnviarAtaDto`, `ReprovarAtaDto` e `VersaoAtaDto`) estão no módulo ATA.

## Rastreabilidade

RF-REU-01 a RF-REU-09 e RF-REU-13 a RF-REU-16; diagrama de classes e dicionário relacional.
