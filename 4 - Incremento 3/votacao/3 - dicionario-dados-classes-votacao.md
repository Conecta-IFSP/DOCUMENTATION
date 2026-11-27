# Dicionário de classes — Votação na reunião

> Padrão da disciplina · Módulo VOT · Complementa o diagrama de classes · Revisado em 27/11/2026

## Resumo

| Classe               | Camada                  | Responsabilidade                                                                                                   |
| -------------------- | ----------------------- | ------------------------------------------------------------------------------------------------------------------ |
| `TelaReuniao`        | Aplicativo              | Carregar a reunião, descobrir se o usuário é o presidente, calcular se ele pode votar e mostrar a aba "Votações". |
| `VotacoesReuniao`    | Aplicativo (componente) | Mostrar as votações, o formulário de abertura com o tipo, as opções de voto, o voto em branco e o nulo, a confirmação e o resultado. |
| `Confirmacao`        | Aplicativo (componente) | Mostrar o diálogo de confirmação do voto e do encerramento e chamar `confirmar` ou `cancelar`.                    |
| `Pendente`           | Aplicativo (tipo)       | Guardar a mensagem, o rótulo do botão e a ação da confirmação aberta.                                              |
| `api`                | Aplicativo (serviço)    | Enviar a requisição HTTP com `X-API-Key` e o token JWT e devolver o resultado ou a mensagem de erro.               |
| `ReunioesController` | API                     | Receber as rotas HTTP da votação e repassar ao serviço.                                                            |
| `ReunioesService`    | Negócio                 | Aplicar autorização, horário, presença, voto único e o sigilo conforme o tipo da votação.                         |
| `AbrirVotacaoDto`    | API (entrada)           | Validar assunto, descrição, opções e tipo da nova votação.                                                         |
| `VotarDto`           | API (entrada)           | Validar o tipo do voto e, no voto em opção, a opção escolhida.                                                     |
| `VotacaoApresentada` | API (saída)             | Formato devolvido ao aplicativo, sem a lista de votos.                                                             |
| `VotoNominal`        | API (saída)             | Linha da lista de quem votou em quê, devolvida só na votação do tipo aberta encerrada.                            |
| `Votacao`            | Subdocumento            | Guardar assunto, opções, tipo, situação, horários e votos de uma votação.                                          |
| `Voto`               | Subdocumento            | Registrar a escolha de um participante (opção, branco ou nulo) e o horário.                                        |
| `ReuniaoModel`, `ComissaoModel`, `UsuarioModel` | Dados (Model Mongoose) | Ler e gravar os documentos de `reunioes` e ler os de `comissoes` e `usuarios`.                      |
| `Reuniao`, `Participante`, `Presenca` | Dados (documento e subdocumentos) | Reunião em que a votação é gravada, quem participa e a presença que define quem vota.          |
| `Comissao`, `Usuario` | Dados (documentos)     | Comissão em que se procura o presidente e conta de quem abre, vota e aparece em `votos_nominais`.                 |
| `SituacaoVotacao`, `ModoVotacao`, `TipoVoto` | Dados (enumerações) | Situação da votação, tipo (aberta ou fechada) e tipo do voto.                                        |

As classes `Reuniao`, `Participante` e `Presenca` estão descritas por completo no [dicionário de classes das reuniões](../../3%20-%20Incremento%202/reunioes/3%20-%20dicionario-dados-classes-reunioes.md), e `Usuario` e `Comissao`, nos módulos do Incremento 1; abaixo aparecem só os campos que a votação usa.

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

## `Participante`, `Presenca`, `Comissao` e `Usuario` (campos usados pela votação)

| Classe         | Atributo       | Tipo                | Obrigatório | Regra                                                                                                  |
| -------------- | -------------- | ------------------- | ----------: | ------------------------------------------------------------------------------------------------------ |
| `Participante` | `usuario_id`   | `ObjectId`          |         Sim | Referência ao usuário em `usuarios`; só participante da reunião consulta as votações.                 |
| `Participante` | `convite`      | `RespostaConvite`   |         Sim | Não é conferido pela votação.                                                                          |
| `Participante` | `presenca`     | `Presenca`          |         Não | Sem presença, o participante não vota.                                                                 |
| `Presenca`     | `entrada`      | `Date`              |         Sim | Horário em que a entrada foi marcada.                                                                  |
| `Presenca`     | `saida`        | `Date`              |         Não | Com saída marcada, o participante não vota mais.                                                       |
| `Presenca`     | `situacao`     | `SituacaoPresenca`  |         Sim | Vota quem está com `AGUARDANDO` ou `CONFIRMADA`; `RECUSADA` não vota.                                  |
| `Comissao`     | `_id`          | `ObjectId`          |         Sim | Lido por `exigirPresidente` com `ComissaoModel.findById(reuniao.comissao_id)`.                         |
| `Comissao`     | `ativo`        | `boolean`           |         Sim | Com a comissão inativa, ninguém abre nem encerra votações.                                             |
| `Comissao`     | `membros`      | `MembroComissao[]`  |         Sim | O presidente é o membro com `papel` igual a `RESPONSAVEL` e o `usuario_id` de quem chama.              |
| `Usuario`      | `_id`          | `ObjectId`          |         Sim | Comparado com `aberta_por`, `Voto.usuario_id` e `Participante.usuario_id`.                             |
| `Usuario`      | `nome`         | `string`            |         Sim | Lido pelo `populate` dos participantes e mostrado em `votos_nominais`.                                 |
| `Usuario`      | `ativo`        | `boolean`           |         Sim | Conferido por `validarUsuario`; conta inativa não usa a votação.                                       |

## Models do Mongoose

Os models são criados pelo `MongooseModule` a partir dos schemas e injetados pelo NestJS no `ReunioesService` com `@InjectModel`. No código, o tipo de cada atributo é `Model<...Document>`; no diagrama, cada model é uma classe com o nome do schema seguido de `Model` e só os métodos chamados pela votação. Cada método devolve uma `Query`, executada com `exec()`.

| Classe          | Guardado em                       | Método                                                | Uso na votação                                                                                                                         |
| --------------- | --------------------------------- | ----------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------- |
| `ReuniaoModel`  | `ReunioesService.reuniaoModel`    | `findById(id: string)`                                | `carregar` lê a reunião da rota; sem documento, erro 404 "Reunião não encontrada".                                                    |
| `ReuniaoModel`  | `ReunioesService.reuniaoModel`    | `findOneAndUpdate(filtro, alteracao, opcoes)`         | `abrirVotacao`, `votar` e `encerrarVotacao` gravam só se o filtro ainda vale no banco, com `returnDocument: 'after'`; sem documento, `responder` lança o conflito 409. |
| `ComissaoModel` | `ReunioesService.comissaoModel`   | `findById(id: ObjectId)`                              | `exigirPresidente` lê a comissão da reunião; sem documento, erro 404 "Comissão não encontrada".                                       |
| `UsuarioModel`  | `ReunioesService.usuarioModel`    | `findById(id: string)`                                | `validarUsuario`, chamado por `carregar`, confere se a conta existe e está ativa.                                                     |

## Classe `TelaReuniao` (o que a votação usa)

Componente padrão de `FRONTEND/src/app/reunioes/[id].tsx`, rota `/reunioes/:id`.

| Atributo     | Tipo                               | Obrigatório | Regra                                                                                          |
| ------------ | ---------------------------------- | ----------: | ---------------------------------------------------------------------------------------------- |
| `reuniao`    | `Reuniao \| null`                  |         Não | Reunião apresentada pela API; nula até a primeira resposta.                                    |
| `aba`        | `'presenca' \| 'votacoes' \| 'pauta'` |      Sim | Começa em `presenca`; com `votacoes`, a tela mostra o `VotacoesReuniao`.                       |
| `presidente` | `boolean`                          |         Sim | Começa falso; verdadeiro com a comissão ativa e o usuário como `RESPONSAVEL`.                  |
| `salvando`   | `string \| null`                   |         Não | Chave da operação em andamento, como `abrir-votacao` ou `votar-<id da votação>`; nula sem operação. |

| Método                                                    | Retorno            | O que faz                                                                                                                            |
| --------------------------------------------------------- | ------------------ | ------------------------------------------------------------------------------------------------------------------------------------ |
| `buscar()`                                                | `Promise<void>`    | Lê `GET /reunioes/:id` e, com a comissão, `GET /comissoes/:comissaoId`, e guarda a reunião e se o usuário é o presidente. Repete a cada 30 segundos com a tela aberta. |
| `executar(chave, acao, mensagem)`                         | `Promise<boolean>` | Não faz nada e devolve falso se outra operação está em andamento; senão guarda a chave em `salvando`, chama `acao` e mostra `mensagem` ou o erro. Devolve verdadeiro quando a API aceitou. |
| `definirReuniao(valor: Reuniao \| null)`                  | `void`             | Guarda a reunião devolvida pela API.                                                                                                 |
| `definirAba(valor: Aba)`                                  | `void`             | Troca a aba mostrada; é chamada pelo controle de abas da tela.                                                                       |
| `definirErro(valor: string)`                              | `void`             | Mostra a faixa de erro com a mensagem devolvida pela API; texto vazio a esconde.                                                     |
| `definirSucesso(valor: string)`                           | `void`             | Mostra a faixa de sucesso, como "Votação iniciada.", "Voto registrado." ou "Votação encerrada.".                                      |
| `definirPresidente(valor: boolean)`                       | `void`             | Guarda se o usuário é o presidente da comissão.                                                                                      |

Os métodos `definir…` são as funções de atualização de estado criadas por `useState`; cada uma troca um atributo e faz a tela ser desenhada de novo.

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
| `VotacoesReuniao(reuniao, presidente, emAndamento, motivoSemVoto, salvando, executar)` | Função do componente, com as propriedades da tabela acima; devolve o `JSX.Element` com o botão "Abrir votação", os cartões das votações, a folha "Nova votação" e a `Confirmacao`. |
| `definirFormulario(valor: boolean)` | Abre (`true`) ou fecha (`false`) a folha "Nova votação".                                                                             |
| `definirModo(valor: ModoVotacao)` | Marca o tipo escolhido em "Tipo de votação"; é chamada no toque em "Fechada" ou "Aberta".                                              |
| `definirErroFormulario(valor: string)` | Mostra na folha a faixa com o problema devolvido por `validar`; texto vazio a esconde.                                          |
| `definirConfirmacao(valor: Pendente \| null)` | Abre a `Confirmacao` com o `Pendente` recebido; com `null`, fecha o diálogo.                                             |
| `abrirFormulario`     | Limpa os campos, volta as opções para "Sim", "Não" e "Abstenção", volta o tipo para `FECHADA` e abre a folha "Nova votação".                     |
| `validar`             | Confere assunto (mínimo 3 caracteres), opções preenchidas, pelo menos 2 opções e opções sem repetição; devolve a mensagem do primeiro problema ou texto vazio. |
| `enviar`              | Chama `validar` e, sem problemas, envia `POST /reunioes/:id/votacoes` com assunto, descrição, opções e `modo`; fecha a folha quando dá certo.     |
| `pedirVoto`           | Recebe a posição de uma opção, `BRANCO` ou `NULO`; abre a confirmação do voto e, confirmada, envia `POST /reunioes/:id/votacoes/:votacaoId/votos` com `{ opcao }` ou `{ tipo }`. |
| `pedirEncerramento`   | Abre a confirmação do encerramento e, confirmada, envia `POST /reunioes/:id/votacoes/:votacaoId/encerrar`.                                       |
| `renderizarAberta`    | Mostra as opções e os botões "Votar em branco" e "Votar nulo", a faixa `Você votou <escolha>` ou a faixa com o motivo de não poder votar.        |
| `renderizarResultado` | Mostra cada opção e as linhas "Branco" e "Nulo" com quantidade, porcentagem e barra; destaca a opção mais votada e marca o voto do usuário; com `votos_nominais`, mostra a lista "Quem votou em quê". |

Os métodos `definir…` são as funções de atualização de estado criadas por `useState`.

## Componente `Confirmacao` e tipo `Pendente`

`Confirmacao` é o diálogo de `FRONTEND/src/components/Confirmacao.tsx`, compartilhado com outras telas; abaixo aparecem as propriedades que o `VotacoesReuniao` informa. `Pendente` é o tipo declarado em `VotacoesReuniao.tsx` para o estado `confirmacao`.

| Classe        | Atributo ou método  | Tipo               | Obrigatório | Regra                                                                                                                  |
| ------------- | ------------------- | ------------------ | ----------: | ---------------------------------------------------------------------------------------------------------------------- |
| `Confirmacao` | `mensagem`          | `string`           |         Não | `confirmacao?.mensagem`; o diálogo fica visível enquanto há mensagem.                                                  |
| `Confirmacao` | `rotuloConfirmar`   | `string`           |         Não | "Votar" ou "Encerrar". Padrão: "Confirmar".                                                                            |
| `Confirmacao` | `perigo`            | `boolean`          |         Não | Verdadeiro no encerramento: o botão de confirmar fica na cor de erro. Padrão: falso.                                  |
| `Confirmacao` | `carregando`        | `boolean`          |         Sim | Verdadeiro enquanto `salvando` não é nulo; mostra o indicador e impede fechar o diálogo.                              |
| `Confirmacao` | `confirmar()`       | `void`             |         Sim | Chamado no toque do botão de confirmar; o `VotacoesReuniao` passa uma função que executa `confirmacao.acao()`.        |
| `Confirmacao` | `cancelar()`        | `void`             |         Sim | Chamado no toque em cancelar; o `VotacoesReuniao` passa uma função que executa `definirConfirmacao(null)`.            |
| `Pendente`    | `mensagem`          | `string`           |         Sim | "Votar <escolha>? O voto não pode ser alterado depois." ou "Encerrar a votação "<assunto>"? Ninguém mais poderá votar." |
| `Pendente`    | `rotulo`            | `string`           |         Sim | "Votar" ou "Encerrar".                                                                                                 |
| `Pendente`    | `perigo`            | `boolean`          |         Não | Verdadeiro só no encerramento.                                                                                          |
| `Pendente`    | `acao()`            | `Promise<unknown>` |         Sim | Função criada por `pedirVoto` ou `pedirEncerramento`: chama `executar` da `TelaReuniao` com a requisição e, no fim, `definirConfirmacao(null)`. |

## Enumerações

| Enumeração        | Valores                    | Onde aparece                                                                          |
| ----------------- | -------------------------- | ------------------------------------------------------------------------------------- |
| `SituacaoVotacao` | `ABERTA`, `ENCERRADA`      | `Votacao.situacao` e `VotacaoApresentada.situacao`                                    |
| `ModoVotacao`     | `ABERTA`, `FECHADA`        | `Votacao.modo`, `AbrirVotacaoDto.modo` e `VotacaoApresentada.modo`                    |
| `TipoVoto`        | `OPCAO`, `BRANCO`, `NULO`  | `Voto.tipo` e `VotarDto.tipo`; `BRANCO` e `NULO` também em `meu_voto` e `VotoNominal.voto` |

`ModoVotacao` e `SituacaoVotacao` têm um valor com o mesmo texto, `ABERTA`, com sentidos diferentes: no modo, todos veem quem votou em quê depois do encerramento; na situação, a votação ainda recebe votos.

## Operações do `ReunioesController`

Definido em `BACKEND/src/reunioes/reunioes.controller.ts`, com o atributo `reunioesService: ReunioesService`. Cada método repassa ao serviço o `usuarioId` lido de `req.usuario.sub`; as rotas estão no [diagrama de classes](3%20-%20diagrama-classes-votacao.md).

| Método            | Parâmetros                                  | Retorno            | O que faz                                                         |
| ----------------- | ------------------------------------------- | ------------------ | ----------------------------------------------------------------- |
| `abrirVotacao`    | `id`, `AbrirVotacaoDto`, `req`              | `Promise<Reuniao>` | Recebe `POST /reunioes/:id/votacoes` e chama `abrirVotacao`.      |
| `votar`           | `id`, `votacaoId`, `VotarDto`, `req`        | `Promise<Reuniao>` | Recebe `POST /reunioes/:id/votacoes/:votacaoId/votos` e chama `votar`. |
| `encerrarVotacao` | `id`, `votacaoId`, `req`                    | `Promise<Reuniao>` | Recebe `POST /reunioes/:id/votacoes/:votacaoId/encerrar` e chama `encerrarVotacao`. |
| `buscarPorId`     | `id`, `req`                                 | `Promise<Reuniao>` | Recebe `GET /reunioes/:id` e chama `buscarPorId`.                 |

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
