# Dicionário de classes — Atas de reunião

> Padrão da disciplina · Módulo ATA · Complementa o diagrama de classes · Revisado em 10/11/2026

## Resumo

| Classe                 | Camada               | Responsabilidade                                                                  |
| ---------------------- | -------------------- | --------------------------------------------------------------------------------- |
| `TelaRegistrosReuniao` | Aplicativo           | Mostrar a ata, editar o rascunho e acionar enviar, aprovar, reprovar e nova versão. |
| `TelaInicio`           | Aplicativo           | Listar as atas pendentes do usuário.                                              |
| `pdfAta`               | Aplicativo (serviço) | Montar o PDF da ata aprovada e abrir o compartilhamento.                          |
| `diferenca`            | Aplicativo (serviço) | Comparar duas versões do texto e apontar o que mudou.                             |
| `autenticacaoLocal`    | Aplicativo (serviço) | Pedir biometria, Face ID ou PIN antes de aprovar.                                 |
| `ReunioesController`   | API                  | Receber as rotas HTTP das atas e repassar ao serviço.                             |
| `ReunioesService`      | Negócio              | Aplicar autorização, prazos, versões, aprovação, reprovação e numeração.          |
| `Ata`                  | Subdocumento         | Guardar o texto, a situação, o prazo e as avaliações de uma versão.               |
| `AvaliacaoAta`         | Subdocumento         | Registrar a decisão de um participante, com motivo e horário.                     |
| `Contador`             | Documento            | Guardar o último número de ata usado por comissão e ano.                          |

As classes `Reuniao`, `Participante`, `Presenca`, `Usuario` e `Comissao` estão descritas no [dicionário de classes das reuniões](../../3%20-%20Incremento2/reunioes/3%20-%20dicionario-dados-classes-reunioes.md); abaixo aparecem só os campos que as atas usam.

## `Ata`

Subdocumento de `Reuniao`, definido em `BACKEND/src/reunioes/schemas/reuniao.schema.ts`. Não tem `_id` próprio (`@Schema({ _id: false })`): ele é identificado pela reunião em que está.

| Atributo        | Tipo TypeScript/Mongoose    | Tipo BSON   | Obrigatório | Regra                                                                                                              |
| --------------- | --------------------------- | ----------- | ----------: | ------------------------------------------------------------------------------------------------------------------ |
| `conteudo`      | `string`                    | `string`    |         Sim | De 20 a 20.000 caracteres. Os parágrafos são separados por quebra de linha.                                        |
| `versao`        | `number` (inteiro)          | `int32`     |         Sim | Começa em 1 e aumenta de 1 a cada salvamento do rascunho e a cada nova versão.                                     |
| `situacao`      | `SituacaoAta` (enum string) | `string`    |         Sim | `RASCUNHO`, `EM_APROVACAO`, `APROVADA`, `REPROVADA` ou `CANCELADA`.                                                |
| `autor_id`      | `Types.ObjectId`            | `objectId`  |         Sim | Identificador do usuário que salvou o texto, em `usuarios`. Não é o objeto do usuário, é a referência a ele.       |
| `atualizada_em` | `Date`                      | `date`      |         Sim | Instante da última gravação, em UTC.                                                                               |
| `enviada_em`    | `Date`                      | `date`      |         Não | Preenchido no envio para aprovação; ausente enquanto a ata é rascunho.                                             |
| `prazo`         | `Date`                      | `date`      |         Não | `enviada_em` mais 24, 72 ou 168 horas. Serve para avisar; o sistema não encerra a aprovação sozinho.               |
| `aprovadores`   | `Types.ObjectId[]`          | `array`     |         Sim | Lista de identificadores de usuários, fixada no envio com quem tinha presença `CONFIRMADA`. Padrão: lista vazia.   |
| `avaliacoes`    | `AvaliacaoAta[]`            | `array`     |         Sim | Uma entrada por pessoa que já decidiu. Padrão: lista vazia.                                                        |
| `numero`        | `number` (inteiro)          | `int32`     |         Não | Número da ata na comissão, a partir de 1. Atribuído só na aprovação.                                               |
| `ano`           | `number` (inteiro)          | `int32`     |         Não | Ano da aprovação no fuso `America/Sao_Paulo`, usado junto com `numero`.                                            |
| `aprovada_em`   | `Date`                      | `date`      |         Não | Instante em que a última aprovação chegou.                                                                         |
| `cancelada_em`  | `Date`                      | `date`      |         Não | Preenchido quando a ata reprovada vai para `atas_canceladas`.                                                      |

Os campos `enviada_em`, `prazo`, `numero`, `ano`, `aprovada_em` e `cancelada_em` são opcionais porque dependem da fase: um rascunho não tem nenhum deles.

## `AvaliacaoAta`

Subdocumento de `Ata`, também sem `_id` próprio. Cada usuário tem no máximo uma avaliação por versão.

| Atributo        | Tipo TypeScript/Mongoose   | Tipo BSON  | Obrigatório | Regra                                                                              |
| --------------- | -------------------------- | ---------- | ----------: | ---------------------------------------------------------------------------------- |
| `usuario_id`    | `Types.ObjectId`           | `objectId` |         Sim | Referência ao usuário em `usuarios`; precisa estar em `Ata.aprovadores`.           |
| `decisao`       | `DecisaoAta` (enum string) | `string`   |         Sim | `APROVADA` ou `REPROVADA`.                                                         |
| `trecho`        | `string`                   | `string`   |         Não | Parágrafo apontado na reprovação; até 20.000 caracteres e precisa existir no texto. |
| `justificativa` | `string`                   | `string`   |         Não | Motivo da reprovação, até 500 caracteres. Texto em branco é guardado como ausente. |
| `em`            | `Date`                     | `date`     |         Sim | Instante da decisão, em UTC.                                                       |

## `Contador`

Documento da coleção `contadores`, definido em `BACKEND/src/reunioes/schemas/contador.schema.ts`.

| Atributo | Tipo TypeScript/Mongoose | Tipo BSON | Obrigatório | Regra                                                                                 |
| -------- | ------------------------ | --------- | ----------: | ------------------------------------------------------------------------------------- |
| `_id`    | `string`                 | `string`  |         Sim | Identifica a sequência: `atas:<id da comissão>:<ano>`, por exemplo `atas:66f…0f:2026`. |
| `valor`  | `number` (inteiro)       | `int32`   |         Sim | Último número usado. Começa em 1 no primeiro uso e sobe de 1 em 1.                    |

Diferente das demais coleções, o `_id` aqui é um texto montado pelo sistema, não um `ObjectId` do MongoDB. Isso permite achar a sequência de uma comissão num ano sem consulta extra.

## `Reuniao` (campos usados pelas atas)

| Atributo          | Tipo               | Obrigatório | Regra                                                                     |
| ----------------- | ------------------ | ----------: | ------------------------------------------------------------------------- |
| `ata`             | `Ata`              |         Não | Versão vigente. Só existe depois do primeiro salvamento.                  |
| `atas_canceladas` | `Ata[]`            |         Sim | Versões reprovadas que deram lugar a outras, da mais antiga para a nova.  |
| `participantes`   | `Participante[]`   |         Sim | Define quem consulta a ata; os de presença `CONFIRMADA` definem quem aprova. |
| `fim`             | `Date`             |         Sim | A ata só existe depois desse instante.                                    |
| `situacao`        | `SituacaoReuniao`  |         Sim | A ata só muda enquanto a reunião está `AGENDADA`.                         |

## Enumerações

| Enumeração    | Valores                                                           | Onde aparece      |
| -------------- | ----------------------------------------------------------------- | ----------------- |
| `SituacaoAta`  | `RASCUNHO`, `EM_APROVACAO`, `APROVADA`, `REPROVADA`, `CANCELADA`  | `Ata.situacao`    |
| `DecisaoAta`   | `APROVADA`, `REPROVADA`                                           | `AvaliacaoAta.decisao` |

`CANCELADA` só aparece nas atas de `atas_canceladas`; a ata vigente nunca usa esse valor.

## Operações do `ReunioesService`

| Método               | Parâmetros                                   | Retorno             | O que faz                                                                     |
| -------------------- | -------------------------------------------- | ------------------- | ----------------------------------------------------------------------------- |
| `salvarAta`          | `id`, `SalvarAtaDto`, `usuarioId`            | reunião atualizada  | Cria ou atualiza o rascunho, conferindo a versão.                             |
| `publicarAta`        | `id`, `EnviarAtaDto`, `usuarioId`            | reunião atualizada  | Congela o texto, define aprovadores e prazo.                                  |
| `aprovarAta`         | `id`, `versao`, `usuarioId`                  | reunião atualizada  | Chama `avaliarAta` com `APROVADA`.                                            |
| `reprovarAta`        | `id`, `ReprovarAtaDto`, `usuarioId`          | reunião atualizada  | Chama `avaliarAta` com `REPROVADA`.                                           |
| `avaliarAta`         | `id`, `versao`, `usuarioId`, avaliação       | reunião atualizada  | Confere versão, elegibilidade e repetição; grava a decisão.                   |
| `concluirSeAprovada` | `reuniao`, `ata`                             | ata                 | Se todos aprovaram, numera e marca `APROVADA`.                                |
| `proximoNumeroAta`   | `comissaoId`, `ano`                          | número              | Incrementa e devolve o contador da comissão no ano.                           |
| `refazerAta`         | `id`, `usuarioId`                            | reunião atualizada  | Move a ata reprovada para o histórico e abre a nova versão.                   |
| `atasPendentes`      | `usuarioId`                                  | lista de reuniões   | Busca o que espera aprovação do usuário e as atas dele reprovadas.            |
| `gravarAta`          | `reuniao`, `ata`, `usuarioId`                | reunião atualizada  | Grava a ata só se ela não tiver mudado desde a leitura.                       |

## DTOs e validação

| DTO              | Campo           | Validação                                                       | Mensagem de erro                                        |
| ---------------- | --------------- | --------------------------------------------------------------- | ------------------------------------------------------- |
| `SalvarAtaDto`   | `conteudo`      | texto de 20 a 20.000 caracteres, aparado                        | "Descreva a ata com pelo menos 20 caracteres"           |
|                  | `versao`        | inteiro, mínimo 0                                               | "Informe a versão da ata"                               |
| `EnviarAtaDto`   | `versao`        | inteiro, mínimo 0                                               | "Informe a versão da ata"                               |
|                  | `prazo_horas`   | um entre 24, 72 e 168                                           | "Escolha um prazo de 24 horas, 3 dias ou 7 dias"        |
| `ReprovarAtaDto` | `versao`        | inteiro, mínimo 0                                               | "Informe a versão da ata"                               |
|                  | `trecho`        | opcional, texto de até 20.000 caracteres                        | "Trecho inválido"                                       |
|                  | `justificativa` | opcional, texto de até 500 caracteres; em branco vira ausente   | "A justificativa deve ter no máximo 500 caracteres"     |

## Rastreabilidade

RF-ATA-01 a RF-ATA-08; [Diagrama de classes](3%20-%20diagrama-classes-atas.md) e [Dicionário relacional](4%20-%20dicionario-dados-relacional-atas.md).
