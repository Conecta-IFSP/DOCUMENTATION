# Dicionário de classes — Reuniões

> Padrão da disciplina · Módulo REU · Complementa o diagrama de classes · Revisado em 26/11/2026

## Resumo

| Classe                 | Camada               | Responsabilidade                                                                                   |
| ---------------------- | -------------------- | -------------------------------------------------------------------------------------------------- |
| `TelaAgenda`           | Aplicativo           | Mostrar o calendário e as reuniões por dia e responder convites.                                   |
| `TelaInicio`           | Aplicativo           | Mostrar a reunião em andamento, as próximas e os convites, e marcar a entrada pelo cartão "Agora". |
| `FormularioReuniao`    | Aplicativo           | Formulário único de agendar e editar, com validação dos campos.                                    |
| `TelaNovaReuniao`      | Aplicativo           | Enviar a nova reunião e os anexos escolhidos.                                                      |
| `TelaEditarReuniao`    | Aplicativo           | Carregar a reunião e enviar as alterações.                                                         |
| `TelaReuniao`          | Aplicativo           | Mostrar a reunião, o convite, a presença, as votações e a pauta, e acionar as ações.               |
| `TelaRegistrosReuniao` | Aplicativo           | Mostrar a lista de presença e gerar o PDF dela (a aba "Ata" está no módulo ATA).                   |
| `AcessoPresidente`     | Aplicativo           | Mostrar o botão "Link e gestão de presença" só para o responsável.                                 |
| `TelaGestaoPresenca`   | Aplicativo           | Gerar, enviar e revogar o link, corrigir horários e mostrar o histórico.                           |
| `TelaPresencaLink`     | Aplicativo           | Abrir o link de presença e registrar entrada e saída.                                              |
| `anexos`, `api`, `pdf`, `pdfListaPresenca`, `autenticacaoLocal`, `retornoPresenca` | Aplicativo (serviço) | Escolher, enviar e abrir arquivos; chamar a API; entregar PDFs; pedir a identidade do aparelho; guardar o link aberto antes do login. |
| `ReunioesController`   | API                  | Receber as rotas `/reunioes` e repassar ao serviço.                                                |
| `ReunioesService`      | Negócio              | Aplicar autorização, horários, convite, presença, anexos e a saída automática.                     |
| `PresencasController`  | API                  | Receber as rotas `/presencas` e repassar ao serviço.                                               |
| `PresencasService`     | Negócio              | Gerar, validar, enviar e revogar o link, registrar presença por ele e corrigir horários.           |
| `NotificacoesService`  | Negócio              | Gravar as notificações de presença e expirá-las.                                                   |
| `Reuniao`              | Documento            | Representar a reunião guardada no MongoDB.                                                         |
| `Participante`         | Subdocumento         | Relacionar o convidado, a resposta ao convite e a presença.                                        |
| `Presenca`             | Subdocumento         | Registrar entrada, saída, situação e origem.                                                       |
| `Anexo`                | Subdocumento         | Identificar o arquivo da pauta guardado no GridFS.                                                 |
| `AcessoPresenca`       | Objeto embutido      | Guardar o resumo do código do link de presença e a validade.                                       |
| `RegistroCorrecao`     | Objeto embutido      | Guardar cada correção de horário, com motivo e valores anteriores.                                 |
| `Notificacao`          | Documento            | Aviso mostrado na tela "Notificações" de um usuário.                                               |

`Comissao` e `Usuario` estão descritas nos módulos do Incremento 1; abaixo aparecem só os campos que as reuniões usam. `Ata` e `Votacao`, também gravadas dentro de `Reuniao`, estão nos dicionários das [atas](../../4%20-%20Incremento3/atas/3%20-%20dicionario-dados-classes-atas.md) e da [votação](../../4%20-%20Incremento3/votacao/3%20-%20dicionario-dados-classes-votacao.md).

## `Reuniao`

Documento da coleção `reunioes`, definido em `BACKEND/src/reunioes/schemas/reuniao.schema.ts`, com `criado_em` e `atualizado_em` preenchidos pelo Mongoose (`timestamps`).

| Atributo              | Tipo TypeScript/Mongoose        | Tipo BSON  | Obrigatório | Regra                                                                                                                  |
| --------------------- | ------------------------------- | ---------- | ----------: | ---------------------------------------------------------------------------------------------------------------------- |
| `_id`                 | `Types.ObjectId`                | `objectId` |         Sim | Criado na inserção; a formação está no [dicionário relacional](4%20-%20dicionario-dados-relacional-reunioes.md).       |
| `comissao_id`         | `Types.ObjectId`                | `objectId` |         Sim | Referência à comissão, em `comissoes`; ativa no agendamento.                                                           |
| `titulo`              | `string`                        | `string`   |         Sim | De 3 a 120 caracteres, sem espaços nas pontas.                                                                         |
| `local`               | `string`                        | `string`   |         Sim | Vazio quando não informado; preenchido, de 2 a 120 caracteres. Padrão: texto vazio.                                    |
| `inicio`              | `Date`                          | `date`     |         Sim | Instante em UTC. Em reunião de dia inteiro, 00:00 do dia no horário do aparelho de quem agendou.                       |
| `fim`                 | `Date`                          | `date`     |         Sim | Maior que `inicio` e, ao agendar, maior que o horário atual. Em dia inteiro, 23:59:59.999 do dia.                      |
| `dia_inteiro`         | `boolean`                       | `bool`     |         Sim | Padrão: falso. Esconde os horários na tela e muda o rótulo para "Dia inteiro".                                         |
| `horario_alterado_em` | `Date`                          | `date`     |         Não | Instante da última mudança de data ou horário feita na edição.                                                         |
| `pauta`               | `string`                        | `string`   |         Sim | Até 2.000 caracteres. Padrão: texto vazio.                                                                             |
| `anexos`              | `Anexo[]`                       | `array`    |         Sim | De 0 a 10 itens. Padrão: lista vazia.                                                                                  |
| `criada_por`          | `Types.ObjectId`                | `objectId` |         Sim | Quem agendou; só ele edita, cancela, confirma presenças e cuida dos anexos.                                            |
| `situacao`            | `SituacaoReuniao` (enum string) | `string`   |         Sim | `AGENDADA` ou `CANCELADA`. Padrão: `AGENDADA`.                                                                         |
| `participantes`       | `Participante[]`                | `array`    |         Sim | Usuários sem repetição; inclui quem agendou.                                                                           |
| `presencas_revisao`   | `number` (inteiro)              | `int32`    |         Sim | Contador de mudanças nas presenças. Padrão: 0. Soma 1 a cada entrada, saída, confirmação, recusa ou correção.          |
| `acesso_presenca`     | `AcessoPresenca`                | `object`   |         Não | Link de presença ativo. Não sai nas consultas comuns (`select: false`).                                                |
| `historico_presencas` | `RegistroCorrecao[]`            | `array`    |         Sim | Correções de horário, da mais antiga para a mais recente. Padrão: lista vazia. Não sai nas consultas comuns.           |
| `criado_em`           | `Date`                          | `date`     |         Sim | Preenchido pelo Mongoose na inserção.                                                                                  |
| `atualizado_em`       | `Date`                          | `date`     |         Sim | Atualizado pelo Mongoose a cada gravação; a edição o usa para não sobrescrever alterações feitas no meio do caminho.   |

## `Participante`

Subdocumento de `Reuniao`, sem `_id` próprio (`@Schema({ _id: false })`).

| Atributo     | Tipo TypeScript/Mongoose        | Tipo BSON  | Obrigatório | Regra                                                                   |
| ------------ | ------------------------------- | ---------- | ----------: | ----------------------------------------------------------------------- |
| `usuario_id` | `Types.ObjectId`                | `objectId` |         Sim | Referência ao usuário em `usuarios`; integrante da comissão no convite. |
| `convite`    | `RespostaConvite` (enum string) | `string`   |         Sim | `PENDENTE`, `ACEITO` ou `RECUSADO`. Padrão: `PENDENTE`.                 |
| `presenca`   | `Presenca`                      | `object`   |         Não | Ausente até a entrada ser marcada, registrada pelo link ou corrigida.   |

## `Presenca`

Subdocumento de `Participante`, sem `_id` próprio.

| Atributo           | Tipo TypeScript/Mongoose         | Tipo BSON | Obrigatório | Regra                                                                                                   |
| ------------------ | -------------------------------- | --------- | ----------: | ------------------------------------------------------------------------------------------------------- |
| `entrada`          | `Date`                           | `date`    |         Sim | Horário do servidor na marcação; `inicio` da reunião quando quem agendou marca a pessoa como presente.  |
| `saida`            | `Date`                           | `date`    |         Não | Nunca maior que `Reuniao.fim`.                                                                          |
| `saida_automatica` | `boolean`                        | `bool`    |         Não | Verdadeiro quando o servidor registrou a saída no término; ausente nos demais casos.                    |
| `situacao`         | `SituacaoPresenca` (enum string) | `string`  |         Sim | `AGUARDANDO`, `CONFIRMADA` ou `RECUSADA`.                                                               |
| `marcada_por`      | `OrigemPresenca` (enum string)   | `string`  |         Sim | `PROPRIO`, `RESPONSAVEL` ou `LINK`.                                                                     |

## `Anexo`

Subdocumento de `Reuniao`. O `_id` não é gerado pelo subdocumento (`@Schema({ _id: false })` com `_id` declarado): ele recebe o identificador do arquivo no GridFS.

| Atributo     | Tipo TypeScript/Mongoose | Tipo BSON  | Obrigatório | Regra                                                                     |
| ------------ | ------------------------ | ---------- | ----------: | ------------------------------------------------------------------------- |
| `_id`        | `Types.ObjectId`         | `objectId` |         Sim | Igual ao `_id` do arquivo em `anexos.files`.                              |
| `nome`       | `string`                 | `string`   |         Sim | Nome original, convertido para utf8, sem espaços nas pontas, até 200 caracteres. |
| `tipo`       | `string`                 | `string`   |         Sim | Tipo MIME, um dos 11 de `TIPOS_ANEXO`.                                    |
| `tamanho`    | `number` (inteiro)       | `int32`    |         Sim | Tamanho em bytes, de 1 a 10.485.760.                                      |
| `enviado_em` | `Date`                   | `date`     |         Sim | Instante do envio, em UTC.                                                |

## `AcessoPresenca`

Objeto gravado em `Reuniao.acesso_presenca` por `PresencasService.gerar`. No máximo um por reunião: gerar de novo substitui o anterior.

| Atributo         | Tipo TypeScript | Tipo BSON | Obrigatório | Regra                                                                                             |
| ---------------- | --------------- | --------- | ----------: | ------------------------------------------------------------------------------------------------- |
| `hash`           | `string`        | `string`  |         Sim | Resumo SHA-256 do código do link, em 64 caracteres hexadecimais. O código não é gravado.         |
| `inicio`         | `Date`          | `date`    |         Sim | `inicio` da reunião no momento da geração; se a reunião mudar de horário, o link deixa de valer. |
| `fim`            | `Date`          | `date`    |         Sim | `fim` da reunião no momento da geração; o link expira nesse instante.                            |
| `responsavel_id` | `string`        | `string`  |         Sim | Identificador, em texto, do responsável que gerou; precisa continuar sendo o responsável.        |
| `criado_em`      | `Date`          | `date`    |         Sim | Instante da geração.                                                                              |

## `RegistroCorrecao`

Cada item de `Reuniao.historico_presencas`, acrescentado por `PresencasService.corrigir`. Nunca é alterado nem apagado.

| Atributo          | Tipo TypeScript    | Tipo BSON | Obrigatório | Regra                                                                                              |
| ----------------- | ------------------ | --------- | ----------: | -------------------------------------------------------------------------------------------------- |
| `participante_id` | `string`           | `string`  |         Sim | Identificador, em texto, do participante corrigido.                                               |
| `responsavel_id`  | `string`           | `string`  |         Sim | Identificador, em texto, do responsável que corrigiu.                                             |
| `alterada_em`     | `Date`             | `date`    |         Sim | Instante da correção.                                                                              |
| `motivo`          | `string`           | `string`  |         Sim | De 5 a 500 caracteres, sem espaços nas pontas.                                                     |
| `antes`           | `Presenca` ou nulo | `object`  |         Sim | Entrada, saída, situação e origem anteriores; nulo quando a pessoa não tinha presença.            |
| `depois`          | `Presenca`         | `object`  |         Sim | Presença gravada: horários informados, `situacao = CONFIRMADA` e `marcada_por = RESPONSAVEL`.      |

## `Notificacao` (campos usados pelo link de presença)

Documento da coleção `notificacoes`, definido em `BACKEND/src/notificacoes/notificacao.schema.ts`.

| Atributo     | Tipo TypeScript/Mongoose | Tipo BSON  | Obrigatório | Regra                                                                                     |
| ------------ | ------------------------ | ---------- | ----------: | ----------------------------------------------------------------------------------------- |
| `usuario_id` | `Types.ObjectId`         | `objectId` |         Sim | Destinatário.                                                                             |
| `chave`      | `string`                 | `string`   |         Sim | `PRESENCA:<reunião>:<resumo do código>`; o par `usuario_id` e `chave` é único.            |
| `tipo`       | `string`                 | `string`   |         Sim | `PRESENCA` para o link.                                                                   |
| `titulo`     | `string`                 | `string`   |         Sim | Título da reunião.                                                                        |
| `mensagem`   | `string`                 | `string`   |         Sim | "O responsável solicitou seu registro de presença. Abra para confirmar durante a reunião." |
| `destino`    | `string`                 | `string`   |         Sim | `/presenca?id=<reunião>&codigo=<código>`.                                                 |
| `expira_em`  | `Date`                   | `date`     |         Não | `fim` da reunião; vira o horário atual quando o link é trocado ou revogado.               |
| `lida_em`    | `Date`                   | `date`     |         Não | Preenchido quando o usuário toca em "Abrir".                                              |

## Enumerações

| Enumeração         | Valores                                | Onde aparece             |
| ------------------ | -------------------------------------- | ------------------------ |
| `SituacaoReuniao`  | `AGENDADA`, `CANCELADA`                | `Reuniao.situacao`       |
| `RespostaConvite`  | `PENDENTE`, `ACEITO`, `RECUSADO`       | `Participante.convite`   |
| `SituacaoPresenca` | `AGUARDANDO`, `CONFIRMADA`, `RECUSADA` | `Presenca.situacao`      |
| `OrigemPresenca`   | `LINK`, `PROPRIO`, `RESPONSAVEL`       | `Presenca.marcada_por`   |

## Operações do `ReunioesService`

| Método                       | Parâmetros                                        | Retorno               | O que faz                                                                                                   |
| ---------------------------- | ------------------------------------------------- | --------------------- | ----------------------------------------------------------------------------------------------------------- |
| `criar`                      | `CreateReuniaoDto`, `usuarioId`                   | reunião criada        | Autoriza o presidente, valida horários e convidados e insere a reunião.                                     |
| `listar`                     | `usuarioId`, filtros `de`, `ate`, `comissao_id`   | lista de reuniões     | Busca as reuniões do usuário no período, ordenadas por `inicio` crescente.                                  |
| `buscarPorId`                | `id`, `usuarioId`                                 | reunião               | Carrega, popula e apresenta a reunião.                                                                      |
| `atualizar`                  | `id`, `UpdateReuniaoDto`, `usuarioId`             | reunião atualizada    | Encaminha para `cancelar` (só `situacao`) ou para `editar` (demais campos).                                 |
| `editar`                     | `id`, campos, `usuarioId`                         | reunião atualizada    | Aplica a edição, volta convites a pendente quando o horário muda e grava comparando `atualizado_em`.        |
| `cancelar`                   | `id`, `usuarioId`                                 | reunião atualizada    | Muda `situacao` para `CANCELADA`.                                                                           |
| `responderConvite`           | `id`, `aceito`, `usuarioId`                       | reunião atualizada    | Grava `ACEITO` ou `RECUSADO` no convite do usuário.                                                         |
| `marcarEntrada`              | `id`, `usuarioId`                                 | reunião atualizada    | Cria a presença com a hora do servidor.                                                                     |
| `marcarSaida`                | `id`, `usuarioId`                                 | reunião atualizada    | Grava a saída, limitada ao `fim`.                                                                           |
| `atualizarPresenca`          | `id`, `participanteId`, `situacao`, `usuarioId`   | reunião atualizada    | Confirma ou recusa a presença, ou cria a presença confirmada.                                               |
| `gerarListaPresenca`         | `id`, `usuarioId`                                 | lista de presença     | Monta a lista com nomes, situação, horários e minutos.                                                      |
| `adicionarAnexo`             | `id`, arquivo, `usuarioId`                        | reunião atualizada    | Grava o arquivo no GridFS e o acrescenta à lista.                                                           |
| `abrirAnexo`                 | `id`, `anexoId`, `usuarioId`                      | anexo e fluxo         | Abre a leitura do arquivo no GridFS.                                                                        |
| `removerAnexo`               | `id`, `anexoId`, `usuarioId`                      | reunião atualizada    | Retira o anexo da lista e apaga o arquivo.                                                                  |
| `onApplicationBootstrap`     | —                                                 | —                     | Chama `executarRotinas` ao iniciar e agenda a repetição a cada 60 segundos.                                 |
| `executarRotinas`            | —                                                 | —                     | Executa `registrarSaidasAutomaticas` e `aprovarAtasVencidas` (ATA); a falha de uma não impede a outra.      |
| `registrarSaidasAutomaticas` | `agora` (padrão: horário atual)                   | quantidade de reuniões | Grava a saída no `fim` de quem entrou e não saiu, nas reuniões encerradas sem ata enviada.                  |
| `carregar`, `carregarComoDono` | `id`, `usuarioId`                               | reunião e participante | Validam o identificador, a conta e a participação; o segundo exige quem agendou.                           |
| `autorizarResponsavel`       | `comissaoId`, `usuarioId`, ação                   | comissão              | Exige papel `RESPONSAVEL`, comissão ativa e vínculo aprovado em organização aprovada.                       |
| `responder`                  | reunião gravada ou nula, `usuarioId`, conflito    | reunião apresentada   | Lança o conflito (409) quando a gravação não encontrou o documento; senão popula e apresenta.               |

## Operações do `PresencasService`

| Método         | Parâmetros                               | Retorno                | O que faz                                                                                       |
| -------------- | ---------------------------------------- | ---------------------- | ----------------------------------------------------------------------------------------------- |
| `gestao`       | `id`, `usuarioId`                        | situação da gestão     | Confere o responsável e informa `link_ativo`, horários, participantes e `revisao`.              |
| `gerar`        | `id`, `usuarioId`                        | código, link e validade | Sorteia o código, grava o resumo e expira as notificações anteriores.                          |
| `revogar`      | `id`, `usuarioId`                        | mensagem               | Remove `acesso_presenca` e expira as notificações.                                              |
| `consultar`    | `id`, `codigo`, `usuarioId`              | dados da reunião       | Valida o link e devolve título, comissão, horários, convite e presença do usuário.              |
| `registrar`    | `id`, `RegistrarAcessoDto`, `usuarioId`  | mensagem               | Registra entrada `CONFIRMADA` pelo link ou a saída.                                             |
| `enviar`       | `id`, `EnviarAcessoDto`, `usuarioId`     | mensagem               | Cria notificações ou envia e-mails aos convidados.                                              |
| `corrigir`     | `id`, `alvoId`, `CorrigirHorarioDto`, `usuarioId` | mensagem      | Substitui a presença e acrescenta o registro ao histórico.                                      |
| `historico`    | `id`, `usuarioId`                        | lista de correções     | Lê `historico_presencas`.                                                                        |
| `validarLink`  | reunião, comissão, `codigo`              | nada                   | Exige link gravado, resumo igual, mesmo responsável, mesmos horários e reunião não terminada.   |
| `editavel`     | reunião                                  | nada                   | Recusa reunião cancelada ou com ata enviada.                                                     |
| `filtro`       | reunião                                  | filtro                 | Monta o filtro de gravação: `_id`, `AGENDADA`, mesmos `inicio` e `fim` e a mesma ata.           |

## DTOs e validação

| DTO                          | Campo            | Validação                                                         | Mensagem de erro                                                                                         |
| ---------------------------- | ---------------- | ----------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------- |
| `CreateReuniaoDto`           | `comissao_id`    | identificador do MongoDB                                          | "Selecione uma comissão válida"                                                                          |
|                              | `titulo`         | texto aparado de 3 a 120 caracteres                               | "Informe o título", "Informe um título com pelo menos 3 caracteres", "O título deve ter até 120 caracteres" |
|                              | `local`          | opcional; vazio ou texto aparado de 2 a 120 caracteres            | "O local deve ser um texto", "O local deve ter pelo menos 2 caracteres", "O local deve ter até 120 caracteres" |
|                              | `inicio`, `fim`  | data e hora ISO 8601 estrita                                      | "Informe um início válido", "Informe um término válido"                                                  |
|                              | `dia_inteiro`    | opcional, verdadeiro ou falso                                     | "Informe se a reunião dura o dia inteiro"                                                                |
|                              | `pauta`          | opcional, texto aparado de até 2.000 caracteres                   | "A pauta deve ser um texto", "A pauta deve ter até 2000 caracteres"                                      |
|                              | `convidados_ids` | opcional; lista sem repetição de identificadores do MongoDB      | "Informe a lista de convidados", "Não repita convidados", "Convidado inválido"                           |
| `UpdateReuniaoDto`           | `situacao`       | opcional, só `CANCELADA`                                          | "A reunião só pode ser cancelada"                                                                        |
|                              | demais campos    | os mesmos do `CreateReuniaoDto`, todos opcionais                  | as mesmas                                                                                                |
| `ResponderConviteReuniaoDto` | `aceito`         | verdadeiro ou falso                                               | "Informe se o convite foi aceito"                                                                        |
| `AtualizarPresencaDto`       | `situacao`       | `CONFIRMADA` ou `RECUSADA`                                        | "A presença deve ser CONFIRMADA ou RECUSADA"                                                             |
| `CodigoAcessoDto`            | `codigo`         | exatamente 64 caracteres de 0 a 9 e de a a f                      | "Link de presença inválido"                                                                              |
| `RegistrarAcessoDto`         | `acao`           | `ENTRADA` ou `SAIDA`                                              | mensagem padrão do class-validator, em inglês                                                            |
| `EnviarAcessoDto`            | `canal`          | `APP` ou `EMAIL`                                                  | mensagem padrão do class-validator, em inglês                                                            |
| `CorrigirHorarioDto`         | `revisao`        | inteiro, mínimo 0                                                 | mensagem padrão do class-validator, em inglês                                                            |
|                              | `entrada`        | data e hora ISO 8601 estrita                                      | mensagem padrão do class-validator, em inglês                                                            |
|                              | `saida`          | opcional, data e hora ISO 8601 estrita                            | mensagem padrão do class-validator, em inglês                                                            |
|                              | `motivo`         | texto aparado de 5 a 500 caracteres                               | mensagem padrão do class-validator, em inglês                                                            |

O `ValidationPipe` da API descarta campos que não estão no DTO (`whitelist`). As regras que dependem do banco, como convidado integrante da comissão ou horário dentro da reunião, são conferidas nos services.

## Rastreabilidade

RF-REU-01 a RF-REU-09 e RF-REU-13 a RF-REU-22; [Diagrama de classes](3%20-%20diagrama-classes-reunioes.md) e [Dicionário relacional](4%20-%20dicionario-dados-relacional-reunioes.md).
