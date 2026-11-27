# Dicionário de classes — Votação na reunião

> Padrão da disciplina · Módulo VOT · Complementa o diagrama de classes · Revisado em 27/11/2026

## Resumo

| Classe               | Camada                  | Responsabilidade                                                                                                   |
| -------------------- | ----------------------- | ------------------------------------------------------------------------------------------------------------------ |
| `TelaReuniao`        | Aplicativo              | Carregar a reunião, descobrir se o usuário é o presidente, calcular se ele pode votar e mostrar a aba "Votações". |
| `VotacoesReuniao`    | Aplicativo (componente) | Mostrar as votações, o formulário de abertura com o tipo, as opções de voto, o voto em branco e o nulo, a confirmação e o resultado. |
| `api`                | Aplicativo (serviço)    | Enviar a requisição HTTP com `X-API-Key` e o token JWT e devolver o resultado ou a mensagem de erro.               |
| `ReunioesController` | API                     | Receber as rotas HTTP da votação e repassar ao serviço.                                                            |
| `ReunioesService`    | Negócio                 | Aplicar autorização, horário, presença, voto único e o sigilo conforme o tipo da votação.                         |
| `AbrirVotacaoDto`    | API (entrada)           | Validar assunto, descrição, opções e tipo da nova votação.                                                         |
| `VotarDto`           | API (entrada)           | Validar o tipo do voto e, no voto em opção, a opção escolhida.                                                     |
| `VotacaoApresentada` | API (saída)             | Formato devolvido ao aplicativo, sem a lista de votos.                                                             |
| `VotoNominal`        | API (saída)             | Linha da lista de quem votou em quê, devolvida só na votação do tipo aberta encerrada.                            |
| `Votacao`            | Subdocumento            | Guardar assunto, opções, tipo, situação, horários e votos de uma votação.                                          |
| `Voto`               | Subdocumento            | Registrar a escolha de um participante (opção, branco ou nulo) e o horário.                                        |

As classes `Reuniao`, `Participante` e `Presenca` estão descritas no [dicionário de classes das reuniões](../../3%20-%20Incremento%202/reunioes/3%20-%20dicionario-dados-classes-reunioes.md), e `Usuario` e `Comissao`, nos módulos do Incremento 1; abaixo aparecem só os campos que a votação usa.

## `Votacao`

Subdocumento de `Reuniao`, definido em `BACKEND/src/reunioes/schemas/reuniao.schema.ts`. Diferente da ata, tem `_id` próprio (`@Schema()`), porque uma reunião tem várias votações e cada uma precisa ser encontrada pela rota.

| Atributo       | Tipo TypeScript/Mongoose        | Tipo BSON           | Obrigatório | Regra                                                                                                                         |
| -------------- | ------------------------------- | ------------------- | ----------: | ----------------------------------------------------------------------------------------------------------------------------- |
| `_id`          | `Types.ObjectId`                | `objectId`          |         Sim | Criado por `ReunioesService.abrirVotacao` com `new Types.ObjectId()`; a formação está no [dicionário relacional](4%20-%20dicionario-dados-relacional-votacao.md). Usado na rota como `votacaoId`. |
| `titulo`       | `string`                        | `string`            |         Sim | Assunto da votação, de 3 a 200 caracteres, sem espaços nas pontas.                                                            |
| `descricao`    | `string`                        | `string`            |         Sim | Até 1.000 caracteres. Padrão: texto vazio.                                                                                    |
| `opcoes`       | `string[]`                      | `array` de `string` |         Sim | De 2 a 10 textos de 1 a 80 caracteres (regra do DTO), sem repetição, ignorando maiúsculas e minúsculas (regra do serviço); o schema só exige a lista. A ordem não muda depois da abertura. |
| `modo`         | `ModoVotacao` (enum string)     | `string`            |         Não | `FECHADA` ou `ABERTA`. Padrão: `FECHADA`, também para as votações gravadas antes do campo existir. Gravado na abertura e nunca alterado. |
| `situacao`     | `SituacaoVotacao` (enum string) | `string`            |         Sim | `ABERTA` ou `ENCERRADA`. Padrão: `ABERTA`.                                                                                    |
| `aberta_por`   | `Types.ObjectId`                | `objectId`          |         Sim | Referência ao presidente que abriu a votação, em `usuarios`.                                                                  |
| `aberta_em`    | `Date`                          | `date`              |         Sim | Instante da abertura, em UTC.                                                                                                 |
| `encerrada_em` | `Date`                          | `date`              |         Não | Instante do encerramento pelo presidente; se ele encerra depois do fim da reunião, guarda o `fim`.                            |
| `votos`        | `Voto[]`                        | `array`             |         Sim | Um item por pessoa que votou. Padrão: lista vazia.                                                                            |

## `Voto`

Subdocumento de `Votacao`, sem `_id` próprio (`@Schema({ _id: false })`). Cada usuário tem no máximo um voto por votação.

| Atributo     | Tipo TypeScript/Mongoose | Tipo BSON  | Obrigatório | Regra                                                                                       |
| ------------ | ------------------------ | ---------- | ----------: | ------------------------------------------------------------------------------------------- |
| `usuario_id` | `Types.ObjectId`         | `objectId` |         Sim | Referência ao usuário em `usuarios`. Não se repete na mesma votação. Só é devolvido pela API em `votos_nominais`, na votação do tipo aberta encerrada. |
| `tipo`       | `TipoVoto` (enum string) | `string`   |         Não | `OPCAO`, `BRANCO` ou `NULO`. Padrão: `OPCAO`, também para os votos gravados antes do campo existir. |
| `opcao`      | `number` (inteiro)       | `int32`    |         Não | Posição da opção escolhida em `Votacao.opcoes`, de 0 até a quantidade de opções menos 1. Gravada só quando `tipo` é `OPCAO`; ausente nos votos em branco e nulos. |
| `votado_em`  | `Date`                   | `date`     |         Sim | Instante do voto, em UTC.                                                                   |

## `VotacaoApresentada`

Objeto montado por `ReunioesService.apresentarVotacao` para cada item de `votacoes`, na resposta de qualquer rota que devolve uma reunião. Não é gravado.

| Atributo                                              | Tipo                    | Regra                                                                                                   |
| ----------------------------------------------------- | ----------------------- | ------------------------------------------------------------------------------------------------------- |
| `_id`, `titulo`, `descricao`, `opcoes`, `aberta_em`   | iguais aos de `Votacao` | Copiados da votação gravada; `descricao` ausente vira texto vazio.                                      |
| `modo`                                                | `ModoVotacao`           | O valor gravado; sem ele, `FECHADA`.                                                                    |
| `situacao`                                            | `SituacaoVotacao`       | `ENCERRADA` quando gravada assim ou quando o `fim` da reunião já passou; senão `ABERTA`.                |
| `encerrada_em`                                        | `Date` ou `null`        | O valor gravado; se a votação está encerrada pelo fim da reunião, o `fim`; senão `null`.                |
| `total_votos`                                         | `number`                | Quantidade de itens em `votos`, inclusive os votos em branco e os nulos.                                |
| `meu_voto`                                            | `number`, `BRANCO`, `NULO` ou `null` | Escolha de quem consulta: `opcao` quando o voto é de tipo `OPCAO` (ou não tem tipo), senão o próprio `tipo`; `null` se ele não votou. |
| `resultado`                                           | `number[]` ou `null`    | Com a votação encerrada, a quantidade de votos em cada opção, na ordem de `opcoes`, sem os brancos e os nulos; aberta, `null`. |
| `brancos`                                             | `number` ou `null`      | Com a votação encerrada, a quantidade de votos de tipo `BRANCO`; aberta, `null`.                        |
| `nulos`                                               | `number` ou `null`      | Com a votação encerrada, a quantidade de votos de tipo `NULO`; aberta, `null`.                          |
| `votos_nominais`                                      | `VotoNominal[]` ou `null` | Com a votação encerrada e `modo` igual a `ABERTA`, um item por voto, em ordem alfabética de `nome`; nos demais casos, `null`. |

## `VotoNominal`

Item de `VotacaoApresentada.votos_nominais`, montado por `apresentarVotacao`. Não é gravado.

| Atributo     | Tipo                       | Regra                                                                                                   |
| ------------ | -------------------------- | ------------------------------------------------------------------------------------------------------- |
| `usuario_id` | `ObjectId`                 | `usuario_id` do voto gravado.                                                                           |
| `nome`       | `string`                   | Nome do participante com esse `usuario_id`, lido dos participantes populados; quando ele não está na lista ou a conta não existe mais, "Usuário indisponível". |
| `voto`       | `number`, `BRANCO` ou `NULO` | A mesma escolha usada em `meu_voto`: a posição da opção ou o tipo do voto.                            |

## `Reuniao` (campos usados pela votação)

| Atributo         | Tipo              | Obrigatório | Regra                                                                       |
| ---------------- | ----------------- | ----------: | --------------------------------------------------------------------------- |
| `votacoes`       | `Votacao[]`       |         Sim | Votações da reunião, na ordem de abertura. Padrão: lista vazia.             |
| `participantes`  | `Participante[]`  |         Sim | Define quem consulta as votações; a presença define quem vota. O `nome` do usuário populado de cada participante é o usado em `votos_nominais`. |
| `inicio` e `fim` | `Date`            |         Sim | A votação só é aberta entre os dois e só recebe votos antes de `fim`.       |
| `situacao`       | `SituacaoReuniao` |         Sim | Nada da votação muda com a reunião `CANCELADA`.                             |
| `comissao_id`    | `ObjectId`        |         Sim | Comissão em que se procura o presidente.                                    |

## Componente `VotacoesReuniao`

Definido em `FRONTEND/src/components/VotacoesReuniao.tsx`. Recebe da `TelaReuniao` as propriedades abaixo.

| Propriedade     | Tipo               | Significado                                                                                                   |
| --------------- | ------------------ | ------------------------------------------------------------------------------------------------------------- |
| `reuniao`       | `Reuniao`          | Reunião com as votações apresentadas.                                                                         |
| `presidente`    | `boolean`          | Verdadeiro quando o usuário é `RESPONSAVEL` da comissão ativa; mostra "Abrir votação" e "Encerrar votação".  |
| `emAndamento`   | `boolean`          | Reunião não cancelada, já começada e não terminada; condição para mostrar "Abrir votação".                    |
| `motivoSemVoto` | `string` ou `null` | Texto mostrado no lugar das opções quando o usuário não pode votar.                                           |
| `salvando`      | `string` ou `null` | Chave da operação em andamento; desativa os botões.                                                           |
| `executar`      | função             | Executa a chamada, mostra a mensagem de sucesso ou de erro e atualiza a reunião na tela.                     |

O estado `modo` guarda o tipo marcado na folha "Nova votação"; começa em `FECHADA` e volta a `FECHADA` sempre que a folha é aberta. O arquivo também declara, fora do componente, as constantes e a função abaixo.

| Constante ou função | Conteúdo                                                                                                                                         |
| ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------ |
| `MODOS`             | As duas escolhas da seção "Tipo de votação": `FECHADA`, com o título "Fechada", o ícone de cadeado e o texto "Ninguém vê quem votou em quê."; e `ABERTA`, com o título "Aberta", o ícone de olho e o texto "Todos veem quem votou em quê quando a votação é encerrada.". |
| `VOTOS_ESPECIAIS`   | Os dois botões abaixo das opções: `BRANCO`, com o título "Votar em branco", e `NULO`, com o título "Votar nulo".                                 |
| `descreverVoto`     | Recebe a votação e a escolha e devolve `em branco` para `BRANCO`, `nulo` para `NULO` e `em "<opção>"` para a posição de uma opção; o texto completa a confirmação do voto e a faixa `Você votou <escolha>`. |

| Método                | O que faz                                                                                                                                         |
| --------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| `abrirFormulario`     | Limpa os campos, volta as opções para "Sim", "Não" e "Abstenção", volta o tipo para `FECHADA` e abre a folha "Nova votação".                     |
| `validar`             | Confere assunto (mínimo 3 caracteres), opções preenchidas, pelo menos 2 opções e opções sem repetição; devolve a mensagem do primeiro problema ou texto vazio. |
| `enviar`              | Chama `validar` e, sem problemas, envia `POST /reunioes/:id/votacoes` com assunto, descrição, opções e `modo`; fecha a folha quando dá certo.     |
| `pedirVoto`           | Recebe a posição de uma opção, `BRANCO` ou `NULO`; abre a confirmação do voto e, confirmada, envia `POST /reunioes/:id/votacoes/:votacaoId/votos` com `{ opcao }` ou `{ tipo }`. |
| `pedirEncerramento`   | Abre a confirmação do encerramento e, confirmada, envia `POST /reunioes/:id/votacoes/:votacaoId/encerrar`.                                       |
| `renderizarAberta`    | Mostra as opções e os botões "Votar em branco" e "Votar nulo", a faixa `Você votou <escolha>` ou a faixa com o motivo de não poder votar.        |
| `renderizarResultado` | Mostra cada opção e as linhas "Branco" e "Nulo" com quantidade, porcentagem e barra; destaca a opção mais votada e marca o voto do usuário; com `votos_nominais`, mostra a lista "Quem votou em quê". |

## Enumerações

| Enumeração        | Valores                    | Onde aparece                                                                          |
| ----------------- | -------------------------- | ------------------------------------------------------------------------------------- |
| `SituacaoVotacao` | `ABERTA`, `ENCERRADA`      | `Votacao.situacao` e `VotacaoApresentada.situacao`                                    |
| `ModoVotacao`     | `ABERTA`, `FECHADA`        | `Votacao.modo`, `AbrirVotacaoDto.modo` e `VotacaoApresentada.modo`                    |
| `TipoVoto`        | `OPCAO`, `BRANCO`, `NULO`  | `Voto.tipo` e `VotarDto.tipo`; `BRANCO` e `NULO` também em `meu_voto` e `VotoNominal.voto` |

`ModoVotacao` e `SituacaoVotacao` têm um valor com o mesmo texto, `ABERTA`, com sentidos diferentes: no modo, todos veem quem votou em quê depois do encerramento; na situação, a votação ainda recebe votos.

## Operações do `ReunioesService`

| Método              | Parâmetros                              | Retorno              | O que faz                                                                              |
| ------------------- | --------------------------------------- | -------------------- | -------------------------------------------------------------------------------------- |
| `abrirVotacao`      | `id`, `AbrirVotacaoDto`, `usuarioId`    | reunião atualizada   | Confere presidente, reunião em andamento e opções; acrescenta a votação com o `modo` recebido ou `FECHADA`. |
| `votar`             | `id`, `votacaoId`, `VotarDto`, `usuarioId` | reunião atualizada | Confere votação aberta, opção (só no tipo `OPCAO`), presença e voto único; acrescenta o voto com o `tipo` e, no tipo `OPCAO`, a `opcao`. |
| `encerrarVotacao`   | `id`, `votacaoId`, `usuarioId`          | reunião atualizada   | Confere presidente e votação aberta; marca `ENCERRADA`.                                |
| `exigirPresidente`  | `reuniao`, `usuarioId`                  | nada                 | Busca a comissão e exige o usuário como `RESPONSAVEL`, com a comissão ativa.           |
| `buscarVotacao`     | `reuniao`, `votacaoId`                  | votação              | Valida o identificador e procura a votação na reunião.                                 |
| `apresentarVotacao` | `votacao`, `fim`, `usuarioId`, `nomes`  | `VotacaoApresentada` | Monta a resposta sem a lista de votos, considera encerrada a votação de reunião terminada, conta brancos e nulos à parte e, só no modo `ABERTA` encerrado, monta `votos_nominais` com os nomes de `nomes`. |
| `apresentar`        | `reuniao`, `usuarioId`                  | reunião              | Copia a reunião, monta `nomes` (`Map` do `_id` de cada participante populado para o `nome`) e troca `votacoes` pelo formato de apresentação. |
| `responder`         | reunião gravada ou nula, `usuarioId`, conflito | reunião       | Lança o conflito (409) quando a gravação não encontrou o documento; senão popula e apresenta. |

## DTOs e validação

| DTO               | Campo       | Validação                                                      | Mensagem de erro                                                                                                                                                                         |
| ----------------- | ----------- | -------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `AbrirVotacaoDto` | `titulo`    | texto aparado de 3 a 200 caracteres                            | "Informe o assunto da votação", "Informe um assunto com pelo menos 3 caracteres", "O assunto deve ter até 200 caracteres"                                                                |
|                   | `descricao` | opcional, texto aparado de até 1.000 caracteres                | "A descrição deve ser um texto", "A descrição deve ter até 1000 caracteres"                                                                                                              |
|                   | `opcoes`    | lista de 2 a 10 textos aparados, cada um de 1 a 80 caracteres  | "Informe as opções de voto", "Informe pelo menos 2 opções de voto", "Informe no máximo 10 opções de voto", "Cada opção deve ser um texto", "Preencha todas as opções de voto", "Cada opção deve ter até 80 caracteres" |
|                   | `modo`      | opcional, um dos valores de `ModoVotacao`                      | "Escolha se a votação é aberta ou fechada"                                                                                                                                               |
| `VotarDto`        | `tipo`      | opcional, um dos valores de `TipoVoto`                         | "Tipo de voto inválido"                                                                                                                                                                  |
|                   | `opcao`     | inteiro, mínimo 0; conferido só quando `tipo` falta ou é `OPCAO` | "Escolha uma opção de voto"                                                                                                                                                            |

A repetição de opções não é regra do DTO: é conferida por `abrirVotacao`, com a mensagem "Não repita opções de voto". O limite superior de `opcao` também é conferido no serviço, com "Opção de voto inválida", porque depende da votação gravada. Com `tipo` igual a `BRANCO` ou `NULO`, o DTO não confere `opcao`, e o serviço não a grava.

## Rastreabilidade

RF-VOT-01 a RF-VOT-04; [Diagrama de classes](3%20-%20diagrama-classes-votacao.md) e [Dicionário relacional](4%20-%20dicionario-dados-relacional-votacao.md).
