# Diagrama de classes — Manual de utilização

> Padrão da disciplina · Módulo MAN · Revisado em 25/11/2026

## Resumo

O conteúdo da Ajuda é estático: ele é escrito no próprio aplicativo, em `src/constants/ajuda.ts`, e não vem do servidor. O manual em PDF também vai dentro do aplicativo, em `assets/manual/manual.pdf`, e é exibido pelo componente `ManualPdf` com o leitor PDF.js embutido. Por isso o R4 só tem classes no pacote **Aplicativo**; os pacotes **API** e **Dados**, usados nos outros módulos, ficam vazios, porque não há controller, service, rota da API nem documento no banco. Cada atributo traz o seu tipo, e cada método traz o tipo dos parâmetros e do retorno. Toda classe e todo método do diagrama existem no código, e são os mesmos chamados nos [Diagramas de sequência](6%20-%20diagramas-sequencia-manual-de-utilizacao.md).

Os tipos são os do TypeScript. `readonly T[]` é uma lista que não pode ser alterada, e `?` indica um campo opcional. As bibliotecas externas aparecem fora do pacote Aplicativo, com o estereótipo `<<biblioteca>>`, e só com os métodos que o R4 usa.

## Diagrama

O diagrama está dividido em três partes para que nenhuma linha passe por cima de uma classe. A primeira mostra a estrutura do conteúdo da Ajuda, como está declarada em `src/constants/ajuda.ts`. A segunda mostra a tela Ajuda e os componentes que leem esse conteúdo e o apresentam. A terceira mostra a tela do manual e o visualizador do PDF. Uma classe já detalhada em uma parte aparece só com o nome nas outras.

### 1. Conteúdo da Ajuda

```mermaid
classDiagram
    direction LR

    namespace Aplicativo {
        class ajuda {
            <<módulo>>
            +TOPICOS_AJUDA: readonly TopicoAjuda[]
            +SELO_PERFIL: Record‹PerfilAjuda, string | null›
        }
        class TopicoAjuda {
            <<type>>
            +id: string
            +titulo: string
            +icone?: IoniconsIconName
            +perguntas: readonly PerguntaAjuda[]
        }
        class PerguntaAjuda {
            <<type>>
            +id: string
            +pergunta: string
            +resposta: string | readonly string[]
            +perfil: PerfilAjuda
        }
        class PerfilAjuda {
            <<enumeration>>
            responsavel
            todos
        }
    }

    ajuda "1" *-- "0..*" TopicoAjuda : TOPICOS_AJUDA
    TopicoAjuda "1" *-- "0..*" PerguntaAjuda : perguntas
    PerguntaAjuda ..> PerfilAjuda : perfil
    ajuda ..> PerfilAjuda : SELO_PERFIL
```

### 2. Tela Ajuda e componentes

```mermaid
classDiagram
    direction LR

    namespace Aplicativo {
        class TelaAjuda {
            <<tela>>
            -topico?: string
            -busca: string
            -abertas: ReadonlySet‹string›
            -topicosVisiveis: readonly TopicoAjuda[]
            -encontradas: number
            +TelaAjuda() JSX.Element
            +primeiraPerguntaDo(topicoId: string | undefined) string | undefined
            +definirBusca(texto: string) void
            +rolarParaTopico() void
            +alternar(perguntaId: string, aberta: boolean) void
            +voltar() void
        }
        class Sanfona {
            <<componente>>
            +pergunta: string
            +resposta: string | readonly string[]
            +aberta?: boolean
            +abertaInicial?: boolean
            +selo?: string | null
            +primeira?: boolean
            +aoAlternar(aberta: boolean) void
            +alternar() void
        }
        class CampoTexto {
            <<componente>>
            +rotulo: string
            +icone?: IoniconsIconName
            +compacto?: boolean
            +value?: string
            +onChangeText(texto: string) void
        }
        class EstadoVazio {
            <<componente>>
            +icone: IoniconsIconName
            +mensagem: string
        }
        class Botao {
            <<componente>>
            +titulo: string
            +icone?: IoniconsIconName
            +variante?: primario | secundario | perigo
            +pilula?: boolean
            +aoTocar() void
        }
        class buscaAjuda {
            <<módulo>>
            +normalizarBusca(texto: string) string
            +filtrarTopicosAjuda(topicos: readonly TopicoAjuda[], busca: string) readonly TopicoAjuda[]
        }
        class ajuda {
            <<módulo>>
        }
        class TopicoAjuda {
            <<type>>
        }
    }

    class Router {
        <<biblioteca>>
        +push(href: string) void
        +back() void
        +canGoBack() boolean
        +replace(href: string) void
    }

    TelaAjuda ..> ajuda : lê as constantes
    TelaAjuda ..> buscaAjuda : filtra pela busca
    TelaAjuda ..> Sanfona : monta uma por pergunta
    TelaAjuda ..> CampoTexto : campo Buscar
    TelaAjuda ..> EstadoVazio : busca sem resultado
    TelaAjuda ..> Botao : Ver manual completo (PDF)
    TelaAjuda ..> Router : push('/manual') e voltar
    buscaAjuda ..> TopicoAjuda : recebe e devolve
```

### 3. Tela do manual e visualizador do PDF

```mermaid
classDiagram
    direction TB

    namespace Aplicativo {
        class TelaManual {
            <<tela>>
            +TelaManual() JSX.Element
        }
        class TelaComCabecalho {
            <<componente>>
            +titulo: string
            +divisor?: boolean
            +rolagem?: boolean
            +children: ReactNode
            +aoVoltar() void
            +voltar() void
        }
        class ManualPdf {
            <<componente>>
            -tentativa: number
            +ManualPdf() JSX.Element
        }
        class Visualizador {
            <<componente>>
            -conteudo: Conteudo | null
            -estado: Estado
            +Visualizador(aoTentarDeNovo: VoidFunction) JSX.Element
            +aoReceberMensagem(evento: WebViewMessageEvent) void
            +falhar(detalhe: unknown) void
        }
        class manualPdf {
            <<módulo>>
            +CARREGAR_POR_ARQUIVO: boolean
            +MENSAGEM_ERRO: string
            -cacheConteudo: Map‹string, Promise‹Conteudo››
            +lerAsset(modulo: number) Promise‹File›
            +escaparScript(codigo: string) string
            +montarHtml(pdfBase64: string, pdfJs: string, worker: string, corFundo: string) string
            +prepararConteudo(corFundo: string) Promise‹Conteudo›
            +obterConteudo(corFundo: string) Promise‹Conteudo›
        }
        class scriptVisualizador {
            <<script>>
            -terminou: boolean
            -vigia: number | null
            -CODIGO_WORKER: string
            +enviar(mensagem: MensagemWebView) void
            +falhar(erro: unknown) void
            +renovarVigia() void
            +rodarWorkerNaThreadPrincipal() void
            +criarWorker() Worker | null
            +esperarWorkerPronto(worker: Worker, ms: number) Promise‹boolean›
            +abrirDocumento(bytes: Uint8Array) Promise‹PDFDocumentProxy›
            +base64ParaBytes(base64: string) Uint8Array
            +canvasParaUrl(canvas: HTMLCanvasElement) Promise‹string›
            +renderizar() Promise‹void›
        }
        class Conteudo {
            <<type>>
            +html: string
            +uri: string
        }
        class MensagemWebView {
            <<type>>
            +tipo: progresso | pronto | erro
            +pagina?: number
            +total?: number
            +detalhe?: string
        }
        class Estado {
            <<type>>
            +fase: preparando | renderizando | pronto | erro
            +pagina?: number
            +total?: number
        }
    }

    class WebView {
        <<biblioteca>>
        +source: html ou uri
        +onMessage(evento: WebViewMessageEvent) void
        +onError(evento) void
        +onContentProcessDidTerminate() void
        +onRenderProcessGone() void
    }
    class Asset {
        <<biblioteca>>
        +localUri: string | null
        +fromModule(modulo: number) Asset
        +downloadAsync() Promise‹Asset›
    }
    class File {
        <<biblioteca>>
        +uri: string
        +base64() Promise‹string›
        +text() Promise‹string›
        +create(opcoes) void
        +write(conteudo: string) Promise‹void›
    }
    class pdfjsLib {
        <<biblioteca>>
        +GlobalWorkerOptions.workerPort: Worker | null
        +getDocument(parametros) PDFDocumentLoadingTask
    }

    TelaManual ..> TelaComCabecalho : título e rolagem false
    TelaManual ..> ManualPdf : monta
    ManualPdf "1" *-- "1" Visualizador : key = fundo e tentativa
    Visualizador ..> manualPdf : obterConteudo
    Visualizador ..> WebView : exibe o HTML
    Visualizador ..> MensagemWebView : recebe
    Visualizador ..> Estado : guarda
    manualPdf ..> Asset : lê os três arquivos
    manualPdf ..> File : base64, text e write
    manualPdf ..> Conteudo : devolve
    manualPdf ..> scriptVisualizador : embute em montarHtml
    WebView ..> scriptVisualizador : executa
    scriptVisualizador ..> pdfjsLib : abre e desenha o PDF
    scriptVisualizador ..> MensagemWebView : envia
```

## Como ler as ligações

| Ligação | Significado | Exemplo no diagrama |
| --- | --- | --- |
| Losango cheio (composição) | A parte só existe dentro do todo e some junto com ele. | `TopicoAjuda *-- PerguntaAjuda`: a pergunta é escrita dentro da lista `perguntas` do seu tópico e não existe fora dela. `ManualPdf *-- Visualizador`: o visualizador só existe dentro do `ManualPdf`, que o recria quando a chave muda. |
| Seta tracejada (dependência) | Uma classe usa a outra em uma chamada, sem guardá-la como atributo. | `TelaAjuda ..> buscaAjuda`: a tela chama `filtrarTopicosAjuda` a cada mudança da busca. `manualPdf ..> Asset`: `lerAsset` usa `Asset.fromModule` e `downloadAsync` a cada preparação. |

Os números nas pontas das composições são as multiplicidades: o módulo `ajuda` tem uma lista com qualquer quantidade de tópicos (hoje, 7), e cada tópico tem qualquer quantidade de perguntas (hoje, de 4 a 7). A multiplicidade `0..*` é a que os tipos `readonly TopicoAjuda[]` e `readonly PerguntaAjuda[]` permitem. O `ManualPdf` monta exatamente um `Visualizador` por vez.

`Conteudo` é declarado como `{ html: string } | { uri: string }`: tem **um** dos dois campos, nunca os dois. `MensagemWebView` e `Estado` também são uniões; o diagrama mostra os campos de todas as variantes, e `?` marca os que só existem em algumas (`pagina` e `total` em `progresso` e `renderizando`, `detalhe` em `erro`).

Este módulo não tem associação (seta cheia) nem herança (triângulo vazio): nenhuma classe guarda o identificador de um documento de outra coleção, e nenhuma classe estende outra.

## Onde está cada classe

| Classe no diagrama | Arquivo | Observação |
| --- | --- | --- |
| `TelaAjuda` | `src/app/ajuda.tsx` | `primeiraPerguntaDo` é uma função do mesmo arquivo, declarada fora do componente. `definirBusca` é a função que altera o estado `busca`. |
| `Sanfona` | `src/components/Sanfona.tsx` | Criada para o R4. |
| `CampoTexto`, `EstadoVazio`, `Botao` | `src/components/CampoTexto.tsx`, `EstadoVazio.tsx`, `Botao.tsx` | Compartilhados com outras telas. O diagrama mostra só as propriedades usadas pelo R4. `EstadoVazio` e `Botao` também são usados pelo `Visualizador`, na tela de erro. |
| `buscaAjuda` | `src/utils/busca-ajuda.ts` | O hífen foi retirado do nome porque o diagrama não o aceita. |
| `ajuda`, `TopicoAjuda`, `PerguntaAjuda`, `PerfilAjuda` | `src/constants/ajuda.ts` | `PerfilAjuda` é declarado como o tipo `'responsavel' \| 'todos'`; aparece como enumeração porque só aceita esses dois valores. |
| `TelaManual` | `src/app/manual.tsx` | Criada para o R4. |
| `TelaComCabecalho` | `src/components/TelaComCabecalho.tsx` | Compartilhado com as outras telas. A propriedade `rolagem` foi criada para o manual; o diagrama mostra só as propriedades usadas pela `TelaManual`. `voltar` é a função interna usada quando `aoVoltar` não é informada. |
| `ManualPdf`, `Visualizador`, `manualPdf`, `Conteudo`, `MensagemWebView`, `Estado` | `src/components/ManualPdf.tsx` | `ManualPdf` é o único componente exportado. `Visualizador`, as funções e os tipos são internos ao arquivo. `manualPdf` (com inicial minúscula) representa as constantes e funções declaradas no nível do arquivo, fora dos componentes. |
| `scriptVisualizador` | Dentro de `montarHtml`, em `src/components/ManualPdf.tsx` | É o código JavaScript escrito no HTML montado e executado dentro da `WebView`, não no React Native. O nome é só para o diagrama; no código ele é uma função anônima executada imediatamente. |
| `Router` | biblioteca `expo-router` (`useRouter()`) | Fica fora do pacote Aplicativo porque não é código do projeto. |
| `WebView` | biblioteca `react-native-webview` 14.0.1 | Idem. |
| `Asset` | biblioteca `expo-asset` ~58.0.21 | Idem. |
| `File` | biblioteca `expo-file-system` ~58.0.14 (API `File`/`Paths`) | Idem. `create` e `write` só são usados com `CARREGAR_POR_ARQUIVO` igual a `true`. |
| `pdfjsLib` | PDF.js 3.11.174, embutido em `assets/pdfjs/pdf.min.js.txt` | Não é dependência do `package.json`: o arquivo foi copiado do pacote `pdfjs-dist` e vai junto com o aplicativo. |

## Rastreabilidade

| Requisito | Classes envolvidas |
| --- | --- |
| RF-MAN-01 — Consultar Ajuda | `TelaAjuda`, `ajuda`, `TopicoAjuda`, `PerguntaAjuda`, `PerfilAjuda`, `buscaAjuda`, `Sanfona`, `CampoTexto`, `EstadoVazio`, `Router` |
| RF-MAN-02 — Abrir manual em PDF | `TelaAjuda`, `Botao`, `Router`, `TelaManual`, `TelaComCabecalho`, `ManualPdf`, `Visualizador`, `manualPdf`, `scriptVisualizador`, `Conteudo`, `MensagemWebView`, `Estado`, `EstadoVazio`, `WebView`, `Asset`, `File`, `pdfjsLib` |
