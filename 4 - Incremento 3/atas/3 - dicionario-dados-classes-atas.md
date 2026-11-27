# Dicionário de classes — Atas de reunião

> Padrão da disciplina · Módulo ATA · Complementa o diagrama de classes · Revisado em 27/11/2026

## Resumo

| Classe                 | Camada               | Responsabilidade                                                                  |
| ---------------------- | -------------------- | --------------------------------------------------------------------------------- |
| `TelaRegistrosReuniao` | Aplicativo           | Mostrar a ata, editar o rascunho e acionar enviar, aprovar, reprovar e nova versão. |
| `TelaInicio`           | Aplicativo           | Listar as atas pendentes do usuário.                                              |
| `api`                  | Aplicativo (serviço) | Chamar a API com `X-API-Key` e o token da sessão e devolver o `Resultado`.        |
| `pdfAta`               | Aplicativo (serviço) | Montar o HTML da ata aprovada e entregá-lo ao serviço `pdf`.                      |
| `pdf`                  | Aplicativo (serviço) | Gerar o PDF A4 e entregá-lo ao menu de compartilhamento (aplicativo instalado) ou abrir a impressão (Expo Go e web). |
| `diferenca`            | Aplicativo (serviço) | Comparar duas versões do texto e apontar o que mudou.                             |
| `reunioes`             | Aplicativo (utilitário) | Dar o texto e o tom da etiqueta da ata e o número "Ata nº N/AAAA".             |
| `autenticacaoLocal`    | Aplicativo (serviço) | Pedir biometria, Face ID ou PIN antes de aprovar, de reprovar e, quando o presidente esteve presente, de enviar. |
| `Router`, `Print`, `File`, `Sharing` | Aplicativo (biblioteca) | Abrir a ata pendente, gerar o PDF, dar o nome ao arquivo e abrir o menu de compartilhamento. |
| `Window`, `Document`   | Aplicativo (API do navegador) | Abrir a janela com a ata e chamar a impressão, na web.                   |
| `ReunioesController`   | API                  | Receber as rotas HTTP das atas e repassar ao serviço.                             |
| `ReunioesService`      | API                  | Aplicar autorização, prazos, versões, aprovação, reprovação e numeração, e aprovar as atas no fim do prazo. |
| `VersaoAtaDto`, `SalvarAtaDto`, `EnviarAtaDto`, `ReprovarAtaDto` | API (entrada) | Validar o corpo das rotas das atas; os três últimos herdam `versao` do primeiro. |
| `ReuniaoModel`, `ContadorModel` | Dados (Model Mongoose) | Ler e gravar os documentos de `reunioes` e de `contadores`.               |
| `Ata`                  | Subdocumento         | Guardar o texto, a situação, o prazo e as avaliações de uma versão.               |
| `AvaliacaoAta`         | Subdocumento         | Registrar a decisão de um participante, com motivo e horário.                     |
| `Contador`             | Documento            | Guardar o último número de ata usado por comissão e ano.                          |
| `Reuniao`, `Participante`, `Presenca`, `Usuario`, `Comissao` | Documento e subdocumentos | Reunião que guarda a ata, quem participou e esteve presente, e a comissão que numera a ata. |
| `SituacaoAta`, `DecisaoAta` | Enumeração      | Situação da ata e decisão de cada avaliação.                                      |

As classes `Reuniao`, `Participante`, `Presenca`, `Usuario` e `Comissao` estão descritas no [dicionário de classes das reuniões](../../3%20-%20Incremento%202/reunioes/3%20-%20dicionario-dados-classes-reunioes.md); abaixo aparecem só os campos que as atas usam.

## `Ata`

Subdocumento de `Reuniao`, definido em `BACKEND/src/reunioes/schemas/reuniao.schema.ts`. Não tem `_id` próprio (`@Schema({ _id: false })`): ele é identificado pela reunião em que está.

| Atributo        | Tipo TypeScript/Mongoose    | Tipo BSON   | Obrigatório | Regra                                                                                                              |
| --------------- | --------------------------- | ----------- | ----------: | ------------------------------------------------------------------------------------------------------------------ |
| `conteudo`      | `string`                    | `string`    |         Sim | De 20 a 20.000 caracteres. Os parágrafos são separados por quebra de linha.                                        |
| `versao`        | `number` (inteiro)          | `int32`     |         Sim | Começa em 1 e aumenta de 1 a cada salvamento do rascunho e a cada nova versão.                                     |
| `situacao`      | `SituacaoAta` (enum string) | `string`    |         Sim | `RASCUNHO`, `EM_APROVACAO`, `APROVADA`, `REPROVADA` ou `CANCELADA`.                                                |
| `autor_id`      | `Types.ObjectId`            | `objectId`  |         Sim | Identificador do usuário que salvou o texto ou criou a nova versão, em `usuarios`. Não é o objeto do usuário, é a referência a ele. |
| `atualizada_em` | `Date`                      | `date`      |         Sim | Instante da última gravação, em UTC; na aprovação pelo fim do prazo, recebe o valor de `prazo`.                    |
| `enviada_em`    | `Date`                      | `date`      |         Não | Preenchido no envio para aprovação; ausente enquanto a ata é rascunho.                                             |
| `prazo`         | `Date`                      | `date`      |         Não | `enviada_em` mais 24, 72 ou 168 horas. Depois dele, ninguém avalia a versão; quem não avaliou conta como aprovação automática. |
| `aprovadores`   | `Types.ObjectId[]`          | `array`     |         Sim | Lista de identificadores de usuários, fixada no envio com quem tinha presença `CONFIRMADA`. Padrão: lista vazia.   |
| `avaliacoes`    | `AvaliacaoAta[]`            | `array`     |         Sim | Uma entrada por pessoa que já decidiu. Padrão: lista vazia.                                                        |
| `numero`        | `number` (inteiro)          | `int32`     |         Não | Número da ata na comissão, a partir de 1. Atribuído só na aprovação.                                               |
| `ano`           | `number` (inteiro)          | `int32`     |         Não | Ano da aprovação no fuso `America/Sao_Paulo`, usado junto com `numero`.                                            |
| `aprovada_em`   | `Date`                      | `date`      |         Não | Instante em que a última aprovação chegou; na aprovação pelo fim do prazo, igual a `prazo`.                        |
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
| `em`            | `Date`                     | `date`     |         Sim | Instante da decisão, em UTC; na aprovação automática, igual a `Ata.prazo`.         |
| `automatica`    | `boolean`                  | `bool`     |         Não | Verdadeiro quando o servidor registrou a aprovação no fim do prazo, porque a pessoa não respondeu; ausente na decisão tomada pela pessoa. |

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
| `comissao_id`     | `Types.ObjectId`   |         Sim | Comissão em que se confere o presidente e cuja sequência numera a ata.    |
| `presencas_revisao` | `number` (inteiro) |       Não | Contador de mudanças nas presenças, com padrão 0; documentos antigos podem não ter o campo. `gravarAta` só grava se ele não mudou desde a leitura (o valor 0 também aceita o campo ausente). |
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
| `avaliarAta`         | `id`, `versao`, `usuarioId`, avaliação sem `usuario_id` e `em` | reunião atualizada  | Confere versão, elegibilidade, repetição, prazo e trecho; grava a decisão. |
| `concluirSeAprovada` | `reuniao`, `ata`                             | ata                 | Se todos aprovaram, numera e marca `APROVADA`.                                |
| `proximoNumeroAta`   | `comissaoId`, `ano`                          | número              | Incrementa e devolve o contador da comissão no ano.                           |
| `refazerAta`         | `id`, `usuarioId`                            | reunião atualizada  | Move a ata reprovada para o histórico e abre a nova versão.                   |
| `atasPendentes`      | `usuarioId`                                  | lista de reuniões   | Busca o que espera aprovação do usuário e as atas dele reprovadas.            |
| `gravarAta`          | `reuniao`, `ata`, `usuarioId`                | reunião atualizada  | Grava a ata só se a reunião continua `AGENDADA` e a ata e `presencas_revisao` não mudaram desde a leitura; senão, 409 "A ata mudou. Atualize antes de continuar.". |
| `aprovarAtasVencidas` | `agora` (padrão: horário atual)             | quantidade de reuniões encontradas | Nas atas em aprovação com `prazo` até `agora`, registra a aprovação automática de quem não avaliou e conclui a ata. |
| `executarRotinas`    | —                                            | —                   | Chamado ao iniciar a API e a cada 60 segundos: executa `registrarSaidasAutomaticas` e `aprovarAtasVencidas`; a falha de uma não impede a outra. |
| `onApplicationBootstrap` | —                                        | —                   | Chamado pelo NestJS ao iniciar a API: executa `executarRotinas` e agenda a repetição a cada 60 segundos com `setInterval`. |
| `exigirEncerrada`    | `reuniao`                                    | —                   | Recusa com 400 a reunião cancelada ("Esta reunião foi cancelada") e a que ainda não terminou ("Aguarde o término da reunião"). |
| `autorizarResponsavel` | `comissaoId`, `usuarioId`, `acao`          | comissão            | Exige o papel `RESPONSAVEL` na comissão, a comissão ativa e o vínculo `APROVADO` na organização `APROVADA`; nas atas, `acao` é "escrever a ata". |
| `buscarPorId`        | `id`, `usuarioId`                            | reunião apresentada | Carrega a reunião com `carregar`, popula com `popular` e devolve `apresentar`; é a consulta da ata (RF-ATA-06) e a resposta de quem repete a mesma decisão em `avaliarAta`. |
| `carregar`           | `id`, `usuarioId`                            | reunião, participante e `dono` | Chama `validarId` e `validarUsuario`, lê a reunião com `ReuniaoModel.findById` ("Reunião não encontrada", 404) e exige o usuário em `participantes` ("Você não participa desta reunião", 403). |
| `validarUsuario`     | `usuarioId`                                  | —                   | Lê a conta com `UsuarioModel.findById` e recusa a inexistente ou inativa com 403 "Usuário sem acesso ativo". |
| `popular`            | `reuniao`                                    | reunião populada    | Troca `comissao_id` pelo nome e pela organização da comissão e cada `participantes.usuario_id` pelo nome e e-mail. |
| `apresentar`         | `reuniao`, `usuarioId`                       | reunião             | Monta a resposta: retira `acesso_presenca` e `historico_presencas`, prepara as votações para o usuário (módulo VOT) e acrescenta `sou_dono`, verdadeiro para quem agendou. |
| `responder`          | reunião gravada ou nula, `usuarioId`, `conflito` | reunião apresentada | Sem documento gravado, lança 409 com o texto de `conflito`; senão, chama `popular` e `apresentar`. |

Cada método do `ReunioesController` só repassa os dados ao método de mesmo nome do `ReunioesService`, com `req.usuario.sub` como `usuarioId`; `aprovarAta` repassa só `dto.versao`. As rotas de cada método estão no [diagrama de classes](3%20-%20diagrama-classes-atas.md).

## Classes do aplicativo

| Classe                 | Membro                                     | Tipo ou assinatura                                                                          | Regra ou o que faz                                                                                                   |
| ---------------------- | ------------------------------------------ | ------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------- |
| `TelaRegistrosReuniao` | `reuniao`                                  | `Reuniao \| null`, estado; começa nulo                                                      | Reunião devolvida pela API, com a ata e as versões canceladas.                                                       |
|                        | `aba`                                      | `'ata' \| 'presenca'`, estado; começa em `'ata'`                                            | Aba visível.                                                                                                         |
|                        | `prazo`                                    | `number`, estado; começa em `72`                                                            | Prazo escolhido na folha de envio, em horas: 24, 72 ou 168.                                                          |
|                        | `trecho`                                   | `string \| null`, estado; começa nulo                                                       | Parágrafo tocado para apontar na reprovação; tocar de novo no mesmo parágrafo volta a nulo.                          |
|                        | `justificativa`                            | `string`, estado; começa vazio                                                              | Texto da folha de reprovação, limitado a 500 caracteres pelo campo.                                                  |
|                        | `carregar()`                               | `() => Promise<void>`                                                                       | Busca a reunião e a comissão e decide se o usuário é o presidente.                                                   |
|                        | `gravar(caminho, metodo, corpo, mensagem)` | `(string, 'PATCH' \| 'POST', unknown, string) => Promise<boolean>`                          | Chama a API, troca a reunião na tela e mostra a mensagem; ignora o toque enquanto outra gravação está em curso. Devolve verdadeiro no sucesso. |
|                        | `abrirEnvio()`                             | `() => void`                                                                                | Guarda o horário usado no cálculo da data limite e abre a folha de envio.                                            |
|                        | `enviar()`                                 | `() => Promise<void>`                                                                       | Fecha a folha, pede a identidade quando o presidente esteve presente e envia.                                        |
|                        | `aprovar()`                                | `() => Promise<void>`                                                                       | Pede a identidade do aparelho, ou a confirmação na tela quando o aparelho não tem bloqueio, e aprova.               |
|                        | `reprovar()`                               | `() => Promise<void>`                                                                       | Pede a identidade do aparelho e envia o trecho e a justificativa; no sucesso, fecha a folha e limpa a justificativa. |
|                        | `refazer()`                                | `() => void`                                                                                | Abre a confirmação "A versão N será cancelada…" e, confirmada, cria a nova versão.                                  |
|                        | `baixarPdf()`                              | `() => Promise<void>`                                                                       | Chama `gerarPdfAta`; se ela lançar erro, mostra "Não foi possível gerar o PDF da ata.".                              |
|                        | `handleSubmit(aoValidar)`                  | `(Function) => Function`, do `useForm` do `react-hook-form`                                 | Valida o campo `conteudo` e, válido, chama `aoValidar`, que grava o rascunho com `gravar('/ata', 'PATCH', ...)`.      |
|                        | `definirReuniao`, `definirResponsavel`, `definirPrazo`, `definirTrecho`, `definirReprovacaoAberta`, `definirJustificativa`, `definirConfirmacao`, `definirGerandoPdf`, `definirSucesso`, `definirErro` | `(valor) => void`, com o valor do tipo do estado: `Reuniao \| null`, `boolean`, `number`, `string \| null`, `boolean`, `string`, `Pendente \| null`, `boolean`, `string` e `string` | Atualizadores de estado devolvidos pelo `useState` do React; cada um troca o valor e faz a tela ser desenhada de novo. `Pendente` tem `mensagem`, `rotulo` e `acao`. |
| `TelaInicio`           | `atas`                                     | `AtaPendente[]`, estado; começa vazio                                                       | Atas pendentes; só é trocada quando a consulta dá certo.                                                             |
|                        | `carregar(continuar)`                      | `(continuar?: () => boolean) => Promise<void>`                                              | Consulta as organizações, a agenda e as atas pendentes ao mesmo tempo; o administrador do sistema não consulta as duas últimas. |
|                        | `definirAtas(valor)`                       | `(AtaPendente[]) => void`                                                                   | Atualizador do estado `atas`, chamado só quando `atasPendentes.ok` é verdadeiro.                                    |
| `api`                  | `api(caminho, metodo, corpo)`              | `<T>(string, 'GET' \| 'POST' \| 'PATCH' \| 'DELETE', unknown?) => Promise<Resultado<T>>`     | Envia `X-API-Key` e o token; no erro, devolve a mensagem da API (a primeira, quando vem uma lista).                  |
| `pdfAta`               | `gerarPdfAta(reuniao)`                     | `(Reuniao) => Promise<void>`                                                                | Monta o HTML e chama `entregarPdf` com o nome "Ata nº N-AAAA".                                                       |
|                        | `htmlDaAta(reuniao)`                       | `(Reuniao) => string`                                                                       | Documento da ata, com todo texto do usuário escapado por `escapar`.                                                  |
| `pdf`                  | `entregarPdf(html, nome)`                  | `(string, string) => Promise<void>`                                                         | Na web, `imprimirNaWeb`; no Expo Go, `Print.printAsync`; no aplicativo instalado, `Print.printToFileAsync`, depois espera `nomearPdf` e só então chama `Sharing.shareAsync` com o endereço devolvido. |
|                        | `pdfPelaImpressao()`                       | `() => boolean`                                                                             | Verdadeiro na web e no Expo Go. As telas das atas não a chamam.                                                      |
|                        | `imprimirNaWeb(html)`                      | `(string) => void`                                                                          | Chama `window.open('', '_blank')`; sem janela, lança "Permita abrir uma nova janela para gerar o PDF.". Escreve o documento com `document.open`, `write` e `close` da janela aberta e chama `focus` e `print`. |
|                        | `nomearPdf(gerado, nome)`                  | `(File, string) => Promise<string>`                                                         | Desde o frontend `8cf4839`, é `async`: cria o `File` com o nome seguido de ".pdf" em `Paths.cache`, apaga o de mesmo nome se existir, espera `gerado.move(arquivo)` e devolve `arquivo.uri`. Se qualquer desses passos lançar erro, inclusive o `move`, o `catch` devolve `gerado.uri`, com o nome dado pelo `Print` (RF-ATA-08). |
| `reunioes`             | `situacaoDaAta(ata)`                       | `(Ata \| undefined) => { texto: string; tom: TomEtiqueta }`                                 | "Não registrada" sem ata; "Rascunho", "Em aprovação · A de N" (aprovações e aprovadores), "Reprovada", "Cancelada" ou "Aprovada", cada um com o seu tom. |
|                        | `numeroDaAta(ata)`                         | `(Ata \| undefined) => string`                                                              | "Ata nº N/AAAA" quando a ata tem `numero` e `ano`; senão, "Ata".                                                   |
| `diferenca`            | `paragrafos(texto)`                        | `(string) => string[]`                                                                      | Separa o texto nas quebras de linha, tira os espaços das pontas e descarta as linhas vazias.                        |
|                        | `mudancas(antes, depois)`                  | `(string, string) => Mudanca[]`                                                             | Compara os parágrafos e, nos alterados, as palavras, até o limite `LIMITE_COMPARACAO` (RNF-ATA-07).                  |
| `autenticacaoLocal`    | `dispositivoProtegido()`                   | `() => Promise<boolean>`                                                                    | Falso na web e em aparelho sem biometria, PIN ou senha de bloqueio.                                                 |
|                        | `autenticarComDispositivo()`               | `() => Promise<boolean>`                                                                    | Em aparelho sem bloqueio, devolve verdadeiro sem pedir nada; nos outros, abre o pedido do aparelho e devolve se deu certo. |

## Models do Mongoose

Cada model é o `Model` que o NestJS injeta no `ReunioesService` com `@InjectModel`. `find`, `findById` e `findOneAndUpdate` devolvem uma `Query`; `select`, `populate`, `sort` e `exec` são dessa `Query` e aparecem na classe do model porque as sequências os chamam na linha de vida dele, na ordem do encadeamento.

| Classe          | Coleção      | Atributo no service              | Métodos chamados                                                                                                   |
| --------------- | ------------ | -------------------------------- | ------------------------------------------------------------------------------------------------------------------ |
| `ReuniaoModel`  | `reunioes`   | `ReunioesService.reuniaoModel`   | `find(filtro)`, `findById(id)`, `findOneAndUpdate(filtro, alteracao, opcoes)`, `updateOne(filtro, alteracao)`, `select(campos)`, `populate(caminho, campos)`, `sort(ordem)`, `exec()` |
| `ContadorModel` | `contadores` | `ReunioesService.contadorModel`  | `findOneAndUpdate(filtro, alteracao, opcoes)`, com `$inc` de 1 em `valor`, `upsert` e `returnDocument: 'after'`      |

## Bibliotecas e objetos do navegador

| Classe     | Origem               | Membros usados                                                                                       | O que faz nas atas                                                                                   |
| ---------- | -------------------- | ---------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------- |
| `Router`   | `expo-router` (`useRouter`) | `push(rota)`                                                                                  | Abre a tela de registros da ata tocada na seção "Atas" da tela inicial.                              |
| `Print`    | `expo-print`         | `printAsync(opcoes)`: `Promise<void>`; `printToFileAsync(opcoes)`: `Promise<FilePrintResult>`, com `uri` | No Expo Go, abre a tela de impressão; no aplicativo instalado, grava o PDF A4 no cache.          |
| `File`     | `expo-file-system` 58 | `File(...partes)`, `uri`, `exists`, `delete()` (síncrono) e `move(destino)`, que devolve `Promise<void>` | Representa o PDF gerado e o de destino em `nomearPdf`, que apaga o destino antigo e espera o `move`. |
| `Sharing`  | `expo-sharing`       | `shareAsync(url, opcoes)`: `Promise<void>`                                                           | Abre o menu de compartilhamento com `mimeType` `application/pdf`, UTI `com.adobe.pdf` e `dialogTitle` igual ao nome. |
| `Window`   | API do navegador     | `open(url, alvo)`, `document`, `focus()`, `print()`                                                  | Na web, `window.open` abre a janela; `focus` e `print` são chamados na janela aberta.                |
| `Document` | API do navegador     | `open()`, `write(html)`, `close()`                                                                   | `document` da janela aberta, que recebe só o HTML da ata.                                            |

## DTOs e validação

| DTO              | Campo           | Validação                                                       | Mensagem de erro                                        |
| ---------------- | --------------- | --------------------------------------------------------------- | ------------------------------------------------------- |
| `VersaoAtaDto`   | `versao`        | inteiro, mínimo 0; é o corpo de `aprovarAta` e a classe mãe dos outros três | "Informe a versão da ata"                   |
| `SalvarAtaDto`   | `conteudo`      | texto de 20 a 20.000 caracteres, aparado                        | "Descreva a ata" (não é texto), "Descreva a ata com pelo menos 20 caracteres" ou "A ata deve ter no máximo 20000 caracteres" |
|                  | `versao`        | inteiro, mínimo 0                                               | "Informe a versão da ata"                               |
| `EnviarAtaDto`   | `versao`        | inteiro, mínimo 0                                               | "Informe a versão da ata"                               |
|                  | `prazo_horas`   | um entre 24, 72 e 168                                           | "Escolha um prazo de 24 horas, 3 dias ou 7 dias"        |
| `ReprovarAtaDto` | `versao`        | inteiro, mínimo 0                                               | "Informe a versão da ata"                               |
|                  | `trecho`        | opcional, texto de até 20.000 caracteres, aparado; em branco vira ausente | "Trecho inválido"                             |
|                  | `justificativa` | opcional, texto de até 500 caracteres, aparado; em branco vira ausente | "Justificativa inválida" (não é texto) ou "A justificativa deve ter no máximo 500 caracteres" |

## Rastreabilidade

RF-ATA-01 a RF-ATA-08; [Diagrama de classes](3%20-%20diagrama-classes-atas.md) e [Dicionário relacional](4%20-%20dicionario-dados-relacional-atas.md).
