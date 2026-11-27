# Diagrama de classes — Manual de utilização

> Padrão da disciplina · Módulo MAN · Revisado em 26/11/2026

## Resumo

O conteúdo da Ajuda e o manual em PDF são estáticos: fazem parte do pacote do aplicativo e não vêm do servidor. Por isso o R4 só tem classes no pacote **Aplicativo**; os pacotes **API** e **Dados**, usados nos outros módulos, ficam vazios, porque não há controller, service, rota nem documento no banco. As bibliotecas externas usadas aparecem fora do pacote, marcadas como `<<biblioteca>>`. Cada atributo traz o seu tipo, e cada método traz o tipo dos parâmetros e do retorno. Toda classe e todo método do diagrama existem no código e são os mesmos chamados nos [Diagramas de sequência](6%20-%20diagramas-sequencia-manual-de-utilizacao.md).

Os tipos são os do TypeScript. `readonly T[]` é uma lista que não pode ser alterada, e `?` indica um campo opcional.

## Diagrama

O diagrama está dividido em três partes, para que nenhuma linha passe por cima de uma classe. A primeira mostra a estrutura do conteúdo da Ajuda, como está declarada em `src/constants/ajuda.ts`. A segunda mostra a tela Ajuda e os componentes que leem esse conteúdo. A terceira mostra a tela do manual e o leitor de PDF. Nas partes 2 e 3, as classes já detalhadas antes aparecem só com o que é usado ali.

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

Figura 2 — Diagrama de classes do conteúdo da Ajuda. Fonte: equipe Conecta+.

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
            +TOPICOS_AJUDA: readonly TopicoAjuda[]
        }
    }

    class Router {
        <<biblioteca>>
        +push(rota: string) void
        +back() void
        +replace(rota: string) void
        +canGoBack() boolean
    }

    TelaAjuda ..> ajuda : lê as constantes
    TelaAjuda ..> buscaAjuda : filtra pela busca
    TelaAjuda ..> Sanfona : monta uma por pergunta
    TelaAjuda ..> CampoTexto : campo Buscar
    TelaAjuda ..> EstadoVazio : busca sem resultado
    TelaAjuda ..> Botao : Ver manual completo (PDF)
    TelaAjuda ..> Router : abre /manual e volta
```

Figura 3 — Diagrama de classes da tela Ajuda. Fonte: equipe Conecta+.

### 3. Manual em PDF

```mermaid
classDiagram
    direction LR

    namespace Aplicativo {
        class TelaManual {
            <<tela>>
            +TelaManual() JSX.Element
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
            +Visualizador(aoTentarDeNovo: Function) JSX.Element
            +aoReceberMensagem(evento: WebViewMessageEvent) void
            +falhar(detalhe: unknown) void
        }
        class leitorManual {
            <<módulo>>
            -cacheConteudo: Map‹string, Promise‹Conteudo››
            -lerAsset(modulo: number) Promise‹File›
            -escaparScript(codigo: string) string
            -montarHtml(pdfBase64: string, pdfJs: string, worker: string, corFundo: string) string
            -prepararConteudo(corFundo: string) Promise‹Conteudo›
            -obterConteudo(corFundo: string) Promise‹Conteudo›
        }
        class paginaDoManual {
            <<script>>
            -terminou: boolean
            -vigia: number
            +renderizar() Promise‹void›
            +abrirDocumento(bytes: Uint8Array) Promise‹PDFDocumentProxy›
            +criarWorker() Worker | null
            +esperarWorkerPronto(worker: Worker, ms: number) Promise‹boolean›
            +rodarWorkerNaThreadPrincipal() void
            +base64ParaBytes(base64: string) Uint8Array
            +canvasParaUrl(canvas: HTMLCanvasElement) Promise‹string›
            +renovarVigia() void
            +enviar(mensagem: MensagemWebView) void
            +falhar(erro: unknown) void
        }
        class TelaComCabecalho {
            <<componente>>
            +titulo: string
            +rolagem?: boolean
            +divisor?: boolean
        }
    }

    class WebView {
        <<biblioteca>>
        +source: Conteudo
        +onMessage(evento: WebViewMessageEvent) void
        +onError(evento: WebViewErrorEvent) void
        +onContentProcessDidTerminate() void
        +onRenderProcessGone() void
    }
    class Asset {
        <<biblioteca>>
        +fromModule(modulo: number) Asset
        +downloadAsync() Promise‹Asset›
        +localUri: string | null
    }
    class File {
        <<biblioteca>>
        +File(uri: string)
        +base64() Promise‹string›
        +text() Promise‹string›
    }
    class pdfjsLib {
        <<biblioteca>>
        +getDocument(fonte: object) PDFDocumentLoadingTask
    }

    TelaManual ..> TelaComCabecalho : cabeçalho Manual de utilização
    TelaManual ..> ManualPdf : mostra
    ManualPdf ..> Visualizador : monta com a chave cor e tentativa
    Visualizador ..> leitorManual : obterConteudo
    Visualizador ..> WebView : mostra a página
    leitorManual ..> Asset : lê os recursos
    leitorManual ..> File : lê base64 e texto
    WebView ..> paginaDoManual : executa
    paginaDoManual ..> pdfjsLib : abre e desenha o PDF
```

Figura 4 — Diagrama de classes do leitor do manual. Fonte: equipe Conecta+.

## Como ler as ligações

| Ligação                      | Significado                                                         | Exemplo no diagrama                                                                                                     |
| ---------------------------- | ------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------- |
| Losango cheio (composição)   | A parte só existe dentro do todo e some junto com ele.              | `TopicoAjuda *-- PerguntaAjuda`: a pergunta é escrita dentro da lista `perguntas` do seu tópico e não existe fora dela. |
| Seta tracejada (dependência) | Uma classe usa a outra numa chamada, sem guardá-la como atributo.   | `TelaAjuda ..> buscaAjuda`: a tela chama `filtrarTopicosAjuda` a cada mudança da busca.                                 |

Os números nas pontas das composições são as multiplicidades: o módulo `ajuda` tem uma lista com qualquer quantidade de tópicos (hoje, 7), e cada tópico tem qualquer quantidade de perguntas (hoje, de 4 a 7). A multiplicidade `0..*` é a que os tipos `readonly TopicoAjuda[]` e `readonly PerguntaAjuda[]` permitem.

Este módulo não tem associação (seta cheia) nem herança (triângulo vazio): nenhuma classe guarda o identificador de um documento de outra coleção, e nenhuma classe estende outra.

## Onde está cada classe

| Classe no diagrama                                      | Arquivo                                                                 | Observação                                                                                                                                         |
| ------------------------------------------------------- | ----------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| `TelaAjuda`                                             | `src/app/ajuda.tsx`                                                     | `primeiraPerguntaDo` é uma função do mesmo arquivo, declarada fora do componente. `definirBusca` é a função que altera o estado `busca`.          |
| `Sanfona`                                               | `src/components/Sanfona.tsx`                                            | Criada para o R4.                                                                                                                                  |
| `CampoTexto`, `EstadoVazio`, `Botao`, `TelaComCabecalho` | `src/components/`                                                      | Compartilhados com outras telas. O diagrama mostra só as propriedades usadas aqui.                                                                 |
| `buscaAjuda`                                            | `src/utils/busca-ajuda.ts`                                              | O hífen foi retirado do nome porque o diagrama não o aceita.                                                                                       |
| `ajuda`, `TopicoAjuda`, `PerguntaAjuda`, `PerfilAjuda`  | `src/constants/ajuda.ts`                                                | `PerfilAjuda` é declarado como o tipo `'responsavel' \| 'todos'`; aparece como enumeração porque só aceita esses dois valores.                    |
| `TelaManual`                                            | `src/app/manual.tsx`                                                    | Rota `/manual`.                                                                                                                                    |
| `ManualPdf`, `Visualizador`                             | `src/components/ManualPdf.tsx`                                          | `Visualizador` é um componente interno do arquivo; só `ManualPdf` é exportado.                                                                    |
| `leitorManual`                                          | `src/components/ManualPdf.tsx`                                          | Nome dado no diagrama às funções e ao cache declarados no nível do arquivo, fora dos componentes.                                                 |
| `paginaDoManual`                                        | `src/components/ManualPdf.tsx`, dentro do texto montado por `montarHtml` | Funções JavaScript da página HTML que roda dentro da `WebView`.                                                                                   |
| `Router`, `WebView`, `Asset`, `File`, `pdfjsLib`        | bibliotecas `expo-router`, `react-native-webview`, `expo-asset`, `expo-file-system` e PDF.js 3.11.174 (`assets/pdfjs/`) | Ficam fora do pacote Aplicativo porque não são código do projeto.                                                       |

## Rastreabilidade

| Requisito                         | Classes envolvidas                                                                                                                           |
| --------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------- |
| RF-MAN-01 — Consultar Ajuda       | `TelaAjuda`, `ajuda`, `TopicoAjuda`, `PerguntaAjuda`, `PerfilAjuda`, `buscaAjuda`, `Sanfona`, `CampoTexto`, `EstadoVazio`, `Router`          |
| RF-MAN-02 — Abrir manual em PDF   | `TelaAjuda`, `Botao`, `Router`, `TelaManual`, `TelaComCabecalho`, `ManualPdf`, `Visualizador`, `leitorManual`, `paginaDoManual`, `WebView`, `Asset`, `File`, `pdfjsLib` |
