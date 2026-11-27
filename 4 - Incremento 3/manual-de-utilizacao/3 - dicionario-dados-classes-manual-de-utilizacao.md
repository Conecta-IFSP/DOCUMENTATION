# Dicionário de classes — Manual de utilização

> Padrão da disciplina · Módulo MAN · Revisado em 25/11/2026

## Resumo

Descreve cada classe do [Diagrama de classes](3%20-%20diagrama-classes-manual-de-utilizacao.md): atributos, tipos, obrigatoriedade, regras e métodos. O conteúdo da Ajuda é declarado no próprio aplicativo, em `src/constants/ajuda.ts`. Ele não é armazenado no banco de dados nem enviado ou recebido pela API. Para alterar uma pergunta, é preciso editar esse arquivo e publicar uma nova versão do aplicativo. O manual em PDF também vai dentro do aplicativo (`assets/manual/manual.pdf`) e é exibido pelo componente `ManualPdf`, descrito na seção 6. Os tipos abaixo são os do TypeScript, como estão no código.

---

## 1. Tipo `TopicoAjuda`

Representa um assunto da tela Ajuda, como Conta e acesso ou Votação. Agrupa as perguntas sobre esse assunto e é exibido como um título com ícone, seguido da lista de perguntas.

| Campo | Tipo | Obrigatório | Descrição |
|---|---|---:|---|
| `id` | `string` | Sim | Identificador do tópico. É usado no endereço `/ajuda?topico=<id>`, que rola a tela até o tópico e abre a primeira pergunta dele. Como é criado: ver "Como os identificadores são criados". |
| `titulo` | `string` | Sim | Nome do assunto exibido na tela. Também é considerado pela busca. |
| `icone` | `IoniconsIconName` | Não | Nome de um ícone da biblioteca Ionicons, exibido ao lado do título. Sem ele, o título aparece sozinho. |
| `perguntas` | `readonly PerguntaAjuda[]` | Sim | Perguntas do tópico, na ordem em que são exibidas. |

### Regras relacionadas ao tópico

O `id` deve usar letras minúsculas, sem acento nem espaço.

O `id` do tópico `conta` não pode ser alterado: o link "Precisa de ajuda?" da tela Entrar abre a Ajuda com `topico=conta`.

Os tópicos são exibidos na ordem da lista `TOPICOS_AJUDA`.

O padrão de escrita do arquivo limita cada tópico a 7 perguntas. Esse limite é uma orientação para quem escreve o conteúdo; o código não o verifica.

Durante uma busca, o tópico que não tem nenhuma pergunta encontrada deixa de ser exibido.

---

## 2. Tipo `PerguntaAjuda`

Representa uma pergunta frequente e a sua resposta. Existe apenas dentro da lista `perguntas` de um `TopicoAjuda`.

| Campo | Tipo | Obrigatório | Descrição |
|---|---|---:|---|
| `id` | `string` | Sim | Identificador da pergunta. Controla quais perguntas estão abertas na tela. Como é criado: ver "Como os identificadores são criados". |
| `pergunta` | `string` | Sim | Texto da pergunta exibido na sanfona. Também é considerado pela busca. |
| `resposta` | `string \| readonly string[]` | Sim | Resposta da pergunta. Um texto é exibido como parágrafo; uma lista é exibida como passos numerados. Também é considerada pela busca. |
| `perfil` | `PerfilAjuda` | Sim | Indica se a pergunta trata de uma ação exclusiva do responsável da comissão. |

### Regras relacionadas à pergunta

O `id` deve ser único em todo o arquivo, e não apenas dentro do tópico.

O padrão de escrita do arquivo limita a resposta em lista a 4 passos, com uma ação curta por passo, e a resposta em texto a 2 ou 3 frases. São orientações para quem escreve o conteúdo; o código não as verifica.

A resposta deve descrever apenas o que o aplicativo faz, com os nomes de botões e telas que aparecem para o usuário.

---

## Como os identificadores são criados

Os dois identificadores deste módulo são textos fixos, escritos à mão no arquivo `src/constants/ajuda.ts` por quem redige o conteúdo. Nenhum deles é gerado pelo aplicativo, pelo servidor ou pelo banco.

| Identificador | Quem cria | Formato | Valor inicial | Incremento | Unicidade |
|---|---|---|---|---|---|
| `TopicoAjuda.id` | A pessoa que escreve o tópico, no próprio arquivo. | Uma palavra em letras minúsculas, sem acento e sem espaço. Valores atuais: `conta`, `organizacoes`, `reunioes`, `presenca`, `ata`, `votacao` e `relatorio`. | Não há: não é um número nem uma sequência. | Não há. Um tópico novo recebe uma palavra nova, escolhida por quem escreve. | Único entre os tópicos. O código não verifica; a conferência é feita na revisão do arquivo. |
| `PerguntaAjuda.id` | A pessoa que escreve a pergunta, no próprio arquivo. | Palavras em letras minúsculas, sem acento, separadas por hífen: um prefixo de assunto e um resumo da pergunta, como `ata-aprovar` e `conta-esqueci-senha`. | Não há: não é um número nem uma sequência. | Não há. Uma pergunta nova recebe um texto novo. | Único no arquivo inteiro. O código não verifica; em 13/11/2026 as 42 perguntas tinham 42 identificadores diferentes. |

O prefixo do identificador da pergunta nem sempre é o `id` do tópico em que ela está hoje: três perguntas do tópico `presenca` têm o prefixo `reunioes-` (`reunioes-presenca`, `reunioes-confirmar-presenca` e `reunioes-lista-presenca`). Isso não afeta o funcionamento, porque o identificador só é comparado por igualdade.

---

## 3. Enumeração `PerfilAjuda`

Indica a quem se aplica uma pergunta. É declarada como o tipo `'responsavel' | 'todos'`.

| Valor | Descrição |
|---|---|
| `responsavel` | A pergunta trata de uma ação exclusiva do responsável da comissão. A pergunta exibe o selo "Responsável". |
| `todos` | A pergunta vale para qualquer usuário. Nenhum selo é exibido. |

### Regras relacionadas ao perfil

O perfil não restringe quem vê a pergunta. Todas as perguntas são exibidas para todos os usuários, com ou sem login. O selo apenas informa.

---

## 4. Módulo `ajuda`

Representa o arquivo `src/constants/ajuda.ts`, que declara os tipos acima e as constantes lidas pela tela Ajuda.

| Constante | Tipo | Descrição |
|---|---|---|
| `TOPICOS_AJUDA` | `readonly TopicoAjuda[]` | Todo o conteúdo da tela Ajuda, na ordem de exibição. |
| `SELO_PERFIL` | `Record<PerfilAjuda, string \| null>` | Texto do selo de cada perfil: `responsavel` → `'Responsável'`; `todos` → `null`, que significa nenhum selo. |

### Conteúdo atual de `TOPICOS_AJUDA`

| `id` | `titulo` | `icone` | Perguntas | Com perfil `responsavel` | Respostas em lista |
|---|---|---|---:|---:|---:|
| `conta` | Conta e acesso | `person-circle-outline` | 4 | 0 | 2 |
| `organizacoes` | Organizações e comissões | `business-outline` | 6 | 0 | 3 |
| `reunioes` | Reuniões | `calendar-outline` | 6 | 3 | 3 |
| `presenca` | Presença | `checkmark-done-outline` | 6 | 3 | 6 |
| `ata` | Ata | `document-text-outline` | 7 | 2 | 3 |
| `votacao` | Votação | `checkbox-outline` | 7 | 1 | 2 |
| `relatorio` | Relatório individual | `bar-chart-outline` | 6 | 0 | 2 |
| **Total** | | | **42** | **9** | **21** |

---

## 5. Classes do Aplicativo

### Classe `TelaAjuda`

Representa a tela Ajuda (`src/app/ajuda.tsx`), que pode ser aberta com ou sem login. Os campos abaixo são o estado da tela. Todos são descartados quando o usuário sai dela.

| Campo | Tipo | Obrigatório | Descrição |
|---|---|---:|---|
| `topico` | `string` | Não | Parâmetro do endereço que indica o tópico a abrir, por exemplo `conta`. |
| `busca` | `string` | Sim | Texto digitado no campo Buscar. Começa vazio. |
| `abertas` | `ReadonlySet<string>` | Sim | Identificadores das perguntas abertas. Começa vazio ou com a primeira pergunta do tópico recebido em `topico`. |
| `topicosVisiveis` | `readonly TopicoAjuda[]` | Sim | Tópicos exibidos: o resultado de `filtrarTopicosAjuda` aplicado a `TOPICOS_AJUDA` com o texto de `busca`. |
| `encontradas` | `number` | Sim | Quantidade de perguntas em `topicosVisiveis`, exibida durante a busca. |

| Método | Retorno | Descrição |
|---|---|---|
| `TelaAjuda()` | `JSX.Element` | Função do componente: monta a tela com o título, o campo Buscar, os tópicos e o botão do manual, que é sempre exibido. |
| `primeiraPerguntaDo(topicoId: string \| undefined)` | `string \| undefined` | Procura em `TOPICOS_AJUDA` o tópico cujo `id` é igual a `topicoId` e devolve o `id` da primeira pergunta dele. Devolve `undefined` quando não há tópico com esse `id`. É declarada no mesmo arquivo, fora do componente. |
| `definirBusca(texto: string)` | `void` | Guarda em `busca` o texto digitado. É chamada pelo `onChangeText` do campo Buscar e, com texto vazio, pelo ícone "Limpar busca". |
| `rolarParaTopico()` | `void` | Rola a tela até o tópico recebido em `topico`, uma única vez por tópico. |
| `alternar(perguntaId: string, aberta: boolean)` | `void` | Com `aberta` igual a `true`, acrescenta `perguntaId` ao conjunto `abertas`; com `false`, retira. |
| `voltar()` | `void` | Retorna à tela anterior ou, se não houver uma, abre o Início (com login) ou a tela Entrar (sem login). |

### Classe `Sanfona`

Representa uma pergunta que se abre e se fecha (`src/components/Sanfona.tsx`). A tela Ajuda exibe uma sanfona para cada pergunta visível.

| Campo | Tipo | Obrigatório | Descrição |
|---|---|---:|---|
| `pergunta` | `string` | Sim | Texto exibido no cabeçalho da sanfona. |
| `resposta` | `string \| readonly string[]` | Sim | Texto corrido ou lista de passos, exibida numerada. |
| `aberta` | `boolean` | Não | Indica se a resposta está visível. A tela Ajuda sempre informa esse valor; sem ele, o próprio componente guarda o estado. |
| `abertaInicial` | `boolean` | Não | Estado inicial quando `aberta` não é informado. O valor padrão é `false`. |
| `aoAlternar` | `(aberta: boolean) => void` | Não | Função chamada quando o usuário toca na pergunta. No diagrama aparece como o método `aoAlternar(aberta: boolean)`. |
| `selo` | `string \| null` | Não | Texto do selo, obtido de `SELO_PERFIL`. Com valor, o selo é exibido e o leitor de tela anuncia "Exclusivo do Responsável". |
| `primeira` | `boolean` | Não | Indica a primeira pergunta do tópico, que não tem linha divisória acima. O valor padrão é `false`. |

O método `alternar` inverte o estado e chama `aoAlternar`. A abertura e o fechamento são animados em 220 ms; quando o aparelho está com a opção de reduzir movimento ativada, a animação não é feita.

### Classes `CampoTexto`, `EstadoVazio` e `Botao`

Componentes compartilhados com outras telas. As tabelas mostram só as propriedades usadas pela tela Ajuda, com o valor que ela informa.

| Classe | Campo | Tipo | Obrigatório | Valor na tela Ajuda |
|---|---|---|---:|---|
| `CampoTexto` | `rotulo` | `string` | Sim | "Buscar" |
| `CampoTexto` | `icone` | `IoniconsIconName` | Não | `search-outline` |
| `CampoTexto` | `compacto` | `boolean` | Não | `true` |
| `CampoTexto` | `value` | `string` | Não | O estado `busca`. |
| `CampoTexto` | `onChangeText` | `(texto: string) => void` | Não | `definirBusca`. |
| `EstadoVazio` | `icone` | `IoniconsIconName` | Sim | `search-outline` |
| `EstadoVazio` | `mensagem` | `string` | Sim | "Tente outra palavra, como o nome da tela ou do botão. Se não encontrar, consulte o manual completo." |
| `Botao` | `titulo` | `string` | Sim | "Ver manual completo (PDF)" |
| `Botao` | `icone` | `IoniconsIconName` | Não | `document-outline` |
| `Botao` | `variante` | `'primario' \| 'secundario' \| 'perigo'` | Não | `secundario` |
| `Botao` | `pilula` | `boolean` | Não | `true` |
| `Botao` | `aoTocar` | `() => void` | Sim | Função que chama `router.push('/manual')` e abre a tela "Manual de utilização". |

O campo Buscar também recebe a ação "Limpar busca", exibida no fim do campo só quando há texto.

### Módulo `buscaAjuda`

Representa o arquivo `src/utils/busca-ajuda.ts`, com as funções da busca da tela Ajuda.

| Função | Retorno | Descrição |
|---|---|---|
| `normalizarBusca(texto: string)` | `string` | Remove acentos e espaços nas pontas e converte para minúsculas, para que "Votação", "VOTACAO" e "votacao" sejam iguais. |
| `filtrarTopicosAjuda(topicos: readonly TopicoAjuda[], busca: string)` | `readonly TopicoAjuda[]` | Mantém só as perguntas que contêm todas as palavras digitadas no título do tópico, na pergunta ou na resposta, e retira os tópicos que ficarem sem perguntas. A comparação é por trecho (`includes`), não por palavra inteira: "ata" também encontra "data". Com a busca vazia, devolve a própria lista recebida. Não altera a lista original. |

### Classe `Router`

Objeto de navegação da biblioteca Expo Router, obtido com `useRouter()`. As tabelas mostram só os métodos usados pelo R4.

| Método | Retorno | Descrição |
|---|---|---|
| `push(href: string)` | `void` | Abre uma tela por cima da atual. O botão "Ver manual completo (PDF)" chama `router.push('/manual')`. |
| `canGoBack()` | `boolean` | Indica se existe uma tela anterior para voltar. |
| `back()` | `void` | Volta à tela anterior. |
| `replace(href: string)` | `void` | Troca a tela atual por outra, sem guardar a atual no histórico. Usado ao voltar quando não há tela anterior. |

---

## 6. Classes do manual dentro do aplicativo

### Classe `TelaManual`

Representa a tela "Manual de utilização" (`src/app/manual.tsx`, rota `/manual`). Não tem estado próprio. A rota é declarada em `src/app/_layout.tsx` fora dos grupos `Stack.Protected`, por isso abre com ou sem login.

| Método | Retorno | Descrição |
|---|---|---|
| `TelaManual()` | `JSX.Element` | Monta o `TelaComCabecalho` com `titulo` "Manual de utilização", `divisor` e `rolagem={false}`, e coloca o `ManualPdf` dentro dele. |

### Classe `TelaComCabecalho` (propriedades usadas pelo manual)

Componente compartilhado por quase todas as telas (`src/components/TelaComCabecalho.tsx`). Mostra o cabeçalho com a seta de voltar e o título, e o conteúdo da tela abaixo. A tabela mostra só as propriedades usadas pela `TelaManual`.

| Campo | Tipo | Obrigatório | Descrição |
|---|---|---:|---|
| `titulo` | `string` | Sim | Texto do cabeçalho. Na `TelaManual`: "Manual de utilização". |
| `divisor` | `boolean` | Não | Com `true`, mostra uma linha sob o cabeçalho. O valor padrão é `false`. Na `TelaManual`: `true`. |
| `rolagem` | `boolean` | Não | Criada para o manual. Com `true` (valor padrão), o conteúdo fica dentro de uma área com rolagem, margens laterais e largura máxima de 560 px, como nas outras telas. Com `false`, o conteúdo ocupa todo o resto da tela, sem rolagem e sem margens, para telas que rolam por conta própria. Na `TelaManual`: `false`, para que a rolagem e o zoom sejam os da `WebView`. |
| `children` | `ReactNode` | Sim | Conteúdo da tela. Na `TelaManual`: o `ManualPdf`. |
| `aoVoltar` | `() => void` | Não | Função da seta de voltar. A `TelaManual` não informa, e vale a função interna `voltar`. |

| Método | Retorno | Descrição |
|---|---|---|
| `voltar()` | `void` | Função usada quando `aoVoltar` não é informada: se `router.canGoBack()` é verdadeiro, executa `router.back()`; se não, `router.replace('/inicio')`. |

### Classe `ManualPdf`

Componente exportado por `src/components/ManualPdf.tsx`. Só controla o reinício do visualizador.

| Campo | Tipo | Obrigatório | Descrição |
|---|---|---:|---|
| `tentativa` | `number` | Sim | Estado que começa em 0 e soma 1 a cada toque em "Tentar de novo". |

| Método | Retorno | Descrição |
|---|---|---|
| `ManualPdf()` | `JSX.Element` | Lê o tema com `useTema()` e monta um `Visualizador` com a chave `` `${cores.fundo}-${tentativa}` `` e com `aoTentarDeNovo`, que soma 1 a `tentativa`. Quando a chave muda (troca de tema ou nova tentativa), o `Visualizador` anterior é descartado e um novo é montado do zero. |

### Classe `Visualizador`

Componente interno de `src/components/ManualPdf.tsx`. Prepara o conteúdo, mostra a `WebView` e trata o progresso e os erros.

| Campo | Tipo | Obrigatório | Descrição |
|---|---|---:|---|
| `aoTentarDeNovo` | `() => void` | Sim | Propriedade recebida do `ManualPdf`; chamada pelo botão "Tentar de novo". |
| `conteudo` | `Conteudo \| null` | Sim | Estado. Começa `null`; recebe a página HTML (ou o seu endereço) devolvida por `obterConteudo`. Enquanto é `null`, a `WebView` não é montada. |
| `estado` | `Estado` | Sim | Estado. Começa em `{ fase: 'preparando' }`. |

| Método | Retorno | Descrição |
|---|---|---|
| `Visualizador(aoTentarDeNovo: () => void)` | `JSX.Element` | Na montagem, chama `obterConteudo(cores.fundo)`; no sucesso guarda o resultado em `conteudo`, na falha passa `estado` para `erro`. Com `estado` em `erro`, mostra o `EstadoVazio` (ícone `document-text-outline`, mensagem `MENSAGEM_ERRO`) e o `Botao` "Tentar de novo" (ícone `refresh`, variante `secundario`). Senão, mostra a `WebView` com fundo `cores.fundo`; por cima, "Abrindo o manual…" enquanto não há página, e a faixa "Carregando página X de T" enquanto a fase é `renderizando` e `pagina` é menor que `total`. |
| `aoReceberMensagem(evento: WebViewMessageEvent)` | `void` | Converte `evento.nativeEvent.data` de JSON para `MensagemWebView` (ignora a mensagem se não for JSON válido). `progresso` → `estado` = `renderizando` com `pagina` e `total`; `pronto` → `estado` = `pronto`; `erro` → chama `falhar(detalhe)`. |
| `falhar(detalhe: unknown)` | `void` | Escreve o detalhe no console (`console.warn`) e passa `estado` para `{ fase: 'erro' }`. É chamada também por `onError`, `onContentProcessDidTerminate` (iOS) e `onRenderProcessGone` (Android) da `WebView`. |

### Módulo `manualPdf`

Constantes e funções declaradas no nível do arquivo `src/components/ManualPdf.tsx`, fora dos componentes.

| Constante | Tipo | Descrição |
|---|---|---|
| `ASSET_PDF`, `ASSET_PDFJS`, `ASSET_WORKER` | `number` | Referências do empacotador (Metro) para `assets/manual/manual.pdf`, `assets/pdfjs/pdf.min.js.txt` e `assets/pdfjs/pdf.worker.min.js.txt`, obtidas com `require`. |
| `CARREGAR_POR_ARQUIVO` | `boolean` | Valor atual: `false` (página entregue à `WebView` como texto). Com `true`, a página é gravada em `Paths.cache` e carregada por endereço. Alternativa para o caso de tela branca no Android; não ativada e não testada. |
| `MENSAGEM_ERRO` | `string` | "Não foi possível abrir o manual. Tente de novo; se continuar, consulte as perguntas da Ajuda." |
| `cacheConteudo` | `Map<string, Promise<Conteudo>>` | Cache em memória, com uma entrada por cor de fundo (tema claro e escuro). Começa vazio, perde a entrada cuja preparação falhou e é descartado quando o aplicativo é fechado. |

| Função | Retorno | Descrição |
|---|---|---|
| `lerAsset(modulo: number)` | `Promise<File>` | `Asset.fromModule(modulo)`, `downloadAsync()` e `new File(asset.localUri ?? asset.uri)`. |
| `escaparScript(codigo: string)` | `string` | Troca cada `</script` (sem diferenciar maiúsculas) por `<\/script`, para o código poder ser escrito dentro de uma marcação `<script>`. |
| `montarHtml(pdfBase64: string, pdfJs: string, worker: string, corFundo: string)` | `string` | Monta a página HTML: configuração de tela com zoom até 4 vezes, fundo `corFundo`, área `#paginas` com 8 px de margem e de espaço entre páginas, o worker em `<script type="text/plain" id="pdf-worker">`, o PDF.js e o `scriptVisualizador` com o PDF em base64. |
| `prepararConteudo(corFundo: string)` | `Promise<Conteudo>` | Lê os três arquivos com `lerAsset`, o PDF com `base64()` e os dois scripts com `text()`, e chama `montarHtml`. Devolve `{ html }`; com `CARREGAR_POR_ARQUIVO` igual a `true`, grava `manual-conecta-<cor sem símbolos>.html` em `Paths.cache` e devolve `{ uri }`. |
| `obterConteudo(corFundo: string)` | `Promise<Conteudo>` | Devolve a promessa guardada em `cacheConteudo` para `corFundo`; se não houver, chama `prepararConteudo`, guarda a promessa e a retira do cache se ela falhar. |

### Tipos `Conteudo`, `MensagemWebView` e `Estado`

| Tipo | Declaração | Descrição |
|---|---|---|
| `Conteudo` | `{ html: string } \| { uri: string }` | O que a `WebView` recebe em `source`: a página como texto ou o endereço do arquivo gravado. |
| `MensagemWebView` | `{ tipo: 'progresso'; pagina: number; total: number } \| { tipo: 'pronto'; total: number } \| { tipo: 'erro'; detalhe: string }` | Mensagens que o `scriptVisualizador` envia ao aplicativo por `postMessage`, em JSON. |
| `Estado` | `{ fase: 'preparando' } \| { fase: 'renderizando'; pagina: number; total: number } \| { fase: 'pronto' } \| { fase: 'erro' }` | Fase da tela do manual. |

### Script `scriptVisualizador`

Código JavaScript escrito por `montarHtml` dentro da página HTML e executado pela `WebView`, não pelo React Native. No código, é uma função anônima executada imediatamente; o nome serve só para o diagrama.

| Campo | Tipo | Descrição |
|---|---|---|
| `PDF_BASE64` | `string \| null` | O PDF em base64. Vira `null` depois de convertido em bytes, para liberar memória. |
| `terminou` | `boolean` | Começa `false`. Vira `true` no primeiro erro ou ao terminar; a partir daí nenhuma mensagem é enviada. |
| `vigia` | `number \| null` | Identificador do temporizador de 30 s (tempo limite). |
| `CODIGO_WORKER` | `string` | Código do worker do PDF.js, lido da marcação `#pdf-worker`. |

| Função | Retorno | Descrição |
|---|---|---|
| `enviar(mensagem: MensagemWebView)` | `void` | Envia a mensagem ao aplicativo com `window.ReactNativeWebView.postMessage(JSON.stringify(mensagem))`. |
| `falhar(erro: unknown)` | `void` | Se `terminou` ainda é `false`, marca `terminou`, cancela o temporizador e envia `{ tipo: 'erro', detalhe }`. |
| `renovarVigia()` | `void` | Reinicia o temporizador de 30 s. Ao vencer, se a página está oculta (`document.visibilityState` igual a `hidden`, aplicativo em segundo plano), renova de novo; se não, chama `falhar('tempo esgotado ao abrir o manual')`. Também é chamada quando a página volta a ficar visível. |
| `criarWorker()` | `Worker \| null` | Cria um `Worker` a partir de `CODIGO_WORKER`, com `Blob` e `URL.createObjectURL`. Devolve `null` se não conseguir. |
| `esperarWorkerPronto(worker: Worker, ms: number)` | `Promise<boolean>` | Resolve `true` na primeira mensagem do worker (o PDF.js envia "ready" ao carregar) e `false` em erro do worker ou depois de `ms` milissegundos. É chamada com 3000. |
| `rodarWorkerNaThreadPrincipal()` | `void` | Acrescenta à página uma marcação `<script>` com `CODIGO_WORKER`, que define `window.pdfjsWorker`; com ela, o PDF.js funciona sem worker. Não faz nada se `window.pdfjsWorker` já existe. |
| `abrirDocumento(bytes: Uint8Array)` | `Promise<PDFDocumentProxy>` | Usa o worker se `esperarWorkerPronto` devolve `true` (entregue em `pdfjsLib.GlobalWorkerOptions.workerPort`); senão encerra o worker e chama `rodarWorkerNaThreadPrincipal()`. Depois chama `pdfjsLib.getDocument({ data: bytes, isEvalSupported: false })`. |
| `base64ParaBytes(base64: string)` | `Uint8Array` | Decodifica o base64 com `atob` e copia cada caractere para um byte. |
| `canvasParaUrl(canvas: HTMLCanvasElement)` | `Promise<string>` | Converte o desenho em JPEG com qualidade 0,85: com `canvas.toBlob`, devolve um endereço `blob:`; sem ele, ou se o `Blob` vier vazio, devolve um endereço `data:` de `toDataURL`. |
| `renderizar()` | `Promise<void>` | Inicia o tempo limite, converte o PDF em bytes, abre o documento e desenha as páginas de 1 a `numPages` em um único `canvas` reaproveitado, com largura igual a (largura da área − 16 px) × min(`devicePixelRatio`, 2). Cada página vira uma `<img>` com a descrição "Página N de T", e depois de cada uma envia `progresso`. No fim, zera o `canvas`, libera o documento e envia `pronto`. Qualquer erro vai para `falhar`. |

### Bibliotecas usadas pelo manual

| Biblioteca | Versão | Onde é declarada | Uso no R4 |
|---|---|---|---|
| `react-native-webview` | 14.0.1 | `package.json` | Componente `WebView`, que exibe a página HTML do manual. Já vem no Expo Go. Propriedades usadas: `source`, `originWhitelist={['*']}`, `onMessage`, `onError`, `onContentProcessDidTerminate`, `onRenderProcessGone`, `javaScriptEnabled`, `scalesPageToFit`, `setBuiltInZoomControls`, `setDisplayZoomControls={false}`, `setSupportMultipleWindows={false}`, `allowFileAccess` e `allowingReadAccessToURL` (as duas últimas só com `CARREGAR_POR_ARQUIVO`). |
| `expo-asset` | ~58.0.21 | `package.json` e plugin em `app.json` | `Asset.fromModule` e `downloadAsync`, para obter no aparelho os arquivos empacotados. |
| `expo-file-system` | ~58.0.14 | `package.json` | `File` (`base64`, `text`, `create`, `write`) e `Paths.cache`. |
| `expo-router` | ~58.0.30 | `package.json` | `useRouter()` (`push`, `back`, `canGoBack`, `replace`) e as rotas `ajuda` e `manual`. |
| PDF.js (`pdfjsLib`) | 3.11.174 | Não está no `package.json` | Arquivos `legacy/build/pdf.min.js` e `legacy/build/pdf.worker.min.js` do pacote `pdfjs-dist`, copiados para `assets/pdfjs/` com extensão `.txt` e embutidos na página. Origem e forma de atualizar em `assets/pdfjs/README.md`. Usado: `GlobalWorkerOptions.workerPort`, `getDocument`, `numPages`, `getPage`, `getViewport`, `render` e `cleanup`. |

---

## 7. Arquivos do manual empacotados no aplicativo

| Arquivo no FRONTEND | Tamanho | Origem | Descrição |
|---|---:|---|---|
| `assets/manual/manual.pdf` | 1.220.899 bytes | Cópia de `manual-usuario/manual.pdf` deste repositório (mesmo conteúdo, conferido por hash SHA-256 `35a838f8…7b963097` em 25/11/2026, na versão corrigida pela revisão de linguagem do RNF-MAN-05). | Manual do usuário, 27 páginas A4. |
| `assets/pdfjs/pdf.min.js.txt` | 377.116 bytes | `pdfjs-dist@3.11.174`, `legacy/build/pdf.min.js`. | Leitor PDF.js. |
| `assets/pdfjs/pdf.worker.min.js.txt` | 1.133.660 bytes | `pdfjs-dist@3.11.174`, `legacy/build/pdf.worker.min.js`. | Worker do PDF.js. |
| `assets/pdfjs/README.md` | — | Escrito pela equipe. | Origem, versão e como atualizar os dois arquivos do PDF.js. |

A extensão `.txt` dos arquivos do PDF.js e a extensão `.pdf` foram incluídas em `resolver.assetExts` no `metro.config.js`, para que o Metro os trate como arquivos a empacotar, e não como código. Para trocar o manual, o novo PDF substitui `assets/manual/manual.pdf` no FRONTEND e `manual-usuario/manual.pdf` neste repositório, com o mesmo nome, e é preciso publicar uma nova versão do aplicativo (RNF-MAN-03).

## Rastreabilidade

`TopicoAjuda`, `PerguntaAjuda`, `PerfilAjuda`, `TelaAjuda`, `Sanfona`, `CampoTexto`, `EstadoVazio` e `buscaAjuda` atendem ao RF-MAN-01. `Botao`, `Router`, `TelaManual`, `TelaComCabecalho` (opção `rolagem`), `ManualPdf`, `Visualizador`, o módulo `manualPdf`, os tipos `Conteudo`, `MensagemWebView` e `Estado`, o `scriptVisualizador`, as bibliotecas `WebView`, `Asset`, `File` e `pdfjsLib` e os arquivos da seção 7 atendem ao RF-MAN-02.
