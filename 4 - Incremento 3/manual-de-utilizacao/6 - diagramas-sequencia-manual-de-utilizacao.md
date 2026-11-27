# Diagramas de sequência — Manual de utilização

> Padrão da disciplina · Módulo MAN · Revisado em 25/11/2026

## Resumo

Há um diagrama para cada requisito de [Requisitos funcionais](2%20-%20requisitos-funcionais-manual-de-utilizacao.md), com o mesmo número. As linhas de vida são telas, componentes e módulos que existem no código do aplicativo. Toda mensagem é a chamada de uma função existente, com seus parâmetros, e as mensagens são numeradas na ordem em que acontecem. O que não é uma função, como a leitura de uma constante, uma regra de comparação ou uma condição do código, aparece em notas. Todas as funções chamadas estão no [Diagrama de classes](3%20-%20diagrama-classes-manual-de-utilizacao.md), com os tipos dos parâmetros e do retorno.

Nem a tela Ajuda nem a tela do manual enviam requisições à API do Conecta+. Por isso nenhum diagrama tem `api`, guards, controller, service ou model. No SD-MAN-02, `scriptVisualizador` é o código executado dentro da `WebView`, e as mensagens que ele envia ao aplicativo (`enviar`) chegam como chamadas de `aoReceberMensagem`.

| Linha de vida | Origem no código |
| --- | --- |
| `TelaAjuda` | `src/app/ajuda.tsx` (`TelaAjuda`, `primeiraPerguntaDo`, `definirBusca`, `rolarParaTopico`, `alternar` e `voltar`) |
| `buscaAjuda` | `src/utils/busca-ajuda.ts` (`filtrarTopicosAjuda` e `normalizarBusca`) |
| `Sanfona` | `src/components/Sanfona.tsx` (`alternar` e a propriedade `aoAlternar`) |
| `Botao` | `src/components/Botao.tsx` (propriedade `aoTocar`) |
| `Router` | biblioteca `expo-router`, obtido com `useRouter()` (`push`, `back` e `replace`) |
| `TelaManual` | `src/app/manual.tsx` (`TelaManual`) |
| `TelaComCabecalho` | `src/components/TelaComCabecalho.tsx` (o próprio componente, com a opção `rolagem`, e `voltar`) |
| `ManualPdf` | `src/components/ManualPdf.tsx` (componente `ManualPdf` e a função `aoTentarDeNovo` que ele entrega) |
| `Visualizador` | `src/components/ManualPdf.tsx` (componente interno `Visualizador`, `aoReceberMensagem` e `falhar`) |
| `manualPdf` | `src/components/ManualPdf.tsx`, nível do arquivo (`obterConteudo`, `prepararConteudo`, `lerAsset`, `montarHtml`, `escaparScript` e `cacheConteudo`) |
| `Asset` | biblioteca `expo-asset` (`fromModule` e `downloadAsync`) |
| `File` | biblioteca `expo-file-system` (`base64`, `text` e `write`) |
| `WebView` | biblioteca `react-native-webview` (componente `WebView` e `onMessage`) |
| `scriptVisualizador` | Script escrito por `montarHtml` e executado na `WebView` (`renderizar`, `renovarVigia`, `base64ParaBytes`, `abrirDocumento`, `criarWorker`, `esperarWorkerPronto`, `rodarWorkerNaThreadPrincipal`, `canvasParaUrl` e `enviar`) |
| `pdfjsLib` | PDF.js 3.11.174, embutido em `assets/pdfjs/` (`getDocument`, `getPage` e `render`) |

## SD-MAN-01 — Consultar Ajuda

```mermaid
sequenceDiagram
    autonumber
    actor U as Usuário
    participant T as TelaAjuda
    participant B as buscaAjuda
    participant S as Sanfona
    Note over U,T: O usuário toca em "Precisa de ajuda?" na tela Entrar (topico igual a conta)<br/>ou em Ajuda, na seção Suporte do Perfil (sem topico)
    U->>T: TelaAjuda()
    T->>T: primeiraPerguntaDo(topico)
    Note over T: Compara topico com o id de cada tópico de TOPICOS_AJUDA, por igualdade.<br/>Se encontra, o id da primeira pergunta entra no conjunto abertas
    T->>B: filtrarTopicosAjuda(TOPICOS_AJUDA, busca)
    B->>B: normalizarBusca(busca)
    Note over B: Busca vazia: devolve a lista inteira, com 7 tópicos e 42 perguntas
    B-->>T: topicosVisiveis
    loop para cada pergunta de cada tópico visível
        T->>S: Sanfona(pergunta, resposta, selo, aberta, aoAlternar, primeira)
    end
    Note over T,S: selo vem de SELO_PERFIL: responsavel mostra "Responsável", todos não mostra selo
    opt topico recebido no endereço
        T->>T: rolarParaTopico()
    end
    opt busca por palavra
        Note over U,T: O usuário digita no campo Buscar, e o onChangeText do CampoTexto chama definirBusca
        U->>T: definirBusca(texto)
        T->>B: filtrarTopicosAjuda(TOPICOS_AJUDA, busca)
        B->>B: normalizarBusca(busca)
        Note over B: Mantém a pergunta quando o título do tópico, a pergunta e a resposta,<br/>juntos e sem acentos, contêm todas as palavras digitadas
        B-->>T: topicosVisiveis
        Note over U,T: A tela mostra "N perguntas encontradas" ou, sem resultado,<br/>"Nenhuma pergunta encontrada." e o EstadoVazio
    end
    U->>S: alternar()
    S->>T: aoAlternar(aberta)
    T->>T: alternar(perguntaId, aberta)
    Note over T: aberta true acrescenta perguntaId ao conjunto abertas, false retira
    T->>S: Sanfona(pergunta, resposta, selo, aberta, aoAlternar, primeira)
    Note over U,S: A resposta aparece em parágrafo ou em passos numerados
    U->>T: voltar()
    Note over T: router.back() se há tela anterior, senão router.replace para /inicio (com login) ou /login (sem login)
```

Para fechar a resposta, o usuário toca de novo na pergunta: as mensagens a partir de `alternar()` se repetem com `aberta` igual a `false`.

## SD-MAN-02 — Abrir manual em PDF

```mermaid
sequenceDiagram
    autonumber
    actor U as Usuário
    participant T as TelaAjuda
    participant Bt as Botao
    participant R as Router
    participant TM as TelaManual
    participant TC as TelaComCabecalho
    participant MP as ManualPdf
    participant V as Visualizador
    participant M as manualPdf
    participant A as Asset
    participant F as File
    participant W as WebView
    participant S as scriptVisualizador
    participant P as pdfjsLib

    T->>Bt: Botao(titulo, icone, variante, pilula, aoTocar)
    Note over T,Bt: O botão é sempre exibido no fim da tela Ajuda
    U->>Bt: aoTocar()
    Bt->>R: push('/manual')
    Note over R: A rota manual fica fora dos grupos Stack.Protected: abre com ou sem login
    R->>TM: TelaManual()
    TM->>TC: TelaComCabecalho(titulo, divisor, rolagem)
    Note over TC: titulo "Manual de utilização", rolagem false: sem a rolagem da tela
    TM->>MP: ManualPdf()
    MP->>V: Visualizador(aoTentarDeNovo)
    Note over MP,V: key = cores.fundo e tentativa. O Visualizador mostra "Abrindo o manual…"
    V->>M: obterConteudo(cores.fundo)
    alt cacheConteudo já tem a chave cores.fundo
        M-->>V: Conteudo guardado
    else primeira abertura com este tema
        M->>M: prepararConteudo(corFundo)
        loop manual.pdf, pdf.min.js.txt e pdf.worker.min.js.txt, ao mesmo tempo
            M->>M: lerAsset(modulo)
            M->>A: fromModule(modulo)
            A-->>M: asset
            M->>A: downloadAsync()
            Note over A: Copia o arquivo empacotado para a pasta de cache do aplicativo
            A-->>M: asset com localUri
        end
        M->>F: base64()
        F-->>M: pdfBase64
        M->>F: text()
        F-->>M: pdfJs e worker
        M->>M: montarHtml(pdfBase64, pdfJs, worker, corFundo)
        M->>M: escaparScript(codigo)
        Note over M: Troca cada fechamento de script por uma forma escapada, no PDF.js e no worker
        opt CARREGAR_POR_ARQUIVO igual a true
            M->>F: write(html)
            F-->>M: arquivo manual-conecta-cor.html em Paths.cache
        end
        M-->>V: Conteudo com html, ou com uri quando a flag é true
    end
    V->>W: WebView(source, onMessage, onError, onContentProcessDidTerminate, onRenderProcessGone)
    W->>S: renderizar()
    S->>S: renovarVigia()
    Note over S: Tempo limite de 30 s, renovado a cada página e pausado com o aplicativo em segundo plano
    S->>S: base64ParaBytes(PDF_BASE64)
    S->>S: abrirDocumento(bytes)
    S->>S: criarWorker()
    S->>S: esperarWorkerPronto(worker, 3000)
    alt o worker respondeu em até 3 s
        Note over S,P: O worker é entregue em GlobalWorkerOptions.workerPort
    else sem resposta, com erro ou sem worker
        S->>S: rodarWorkerNaThreadPrincipal()
        Note over S,P: O PDF.js passa a funcionar sem worker, na própria página
    end
    S->>P: getDocument(data, isEvalSupported)
    P-->>S: documento com numPages igual a 27
    loop numero de 1 até numPages
        S->>P: getPage(numero)
        P-->>S: pagina
        Note over S: escala = largura útil vezes min(devicePixelRatio, 2) dividida pela largura da página
        S->>P: render(canvasContext, viewport)
        P-->>S: página desenhada no canvas único
        S->>S: canvasParaUrl(canvas)
        Note over S: JPEG 0,85 acrescentado como imagem "Página N de 27"
        S->>S: renovarVigia()
        S->>W: enviar(progresso, pagina, total)
        W->>V: aoReceberMensagem(evento)
        Note over V: Some o "Abrindo o manual…". Faixa "Carregando página X de 27" enquanto pagina for menor que total
    end
    alt as 27 páginas foram desenhadas
        S->>W: enviar(pronto, total)
        W->>V: aoReceberMensagem(evento)
        Note over U,V: A faixa some. O usuário rola as páginas e amplia até 4 vezes com a pinça
    else falha em qualquer ponto
        Note over V,S: Leitura dos arquivos rejeitada, erro do PDF.js, tempo limite, onError ou processo da WebView encerrado
        S->>W: enviar(erro, detalhe)
        W->>V: aoReceberMensagem(evento)
        V->>V: falhar(detalhe)
        Note over U,V: "Não foi possível abrir o manual. Tente de novo..." e o botão "Tentar de novo"
        U->>MP: aoTentarDeNovo()
        Note over MP,V: tentativa mais 1 muda a key: o Visualizador é montado de novo, a partir da mensagem 7
    end
    U->>TC: voltar()
    TC->>R: back()
    Note over R,T: Sem tela anterior, replace('/inicio')
    R-->>T: tela Ajuda no mesmo ponto
```

O manual é lido só de arquivos empacotados no aplicativo: não há chamada à API nem a nenhum endereço da internet. Na primeira abertura com um tema, os passos do bloco "primeira abertura" leem os três arquivos e montam a página HTML; nas aberturas seguintes, enquanto o aplicativo estiver aberto, `obterConteudo` devolve a página guardada em `cacheConteudo`, e o fluxo segue direto para a `WebView`. Quando a falha acontece já na leitura dos arquivos, a promessa de `obterConteudo` é rejeitada, a chave sai do cache e o `Visualizador` passa para o erro sem chegar a montar a `WebView`.

## Rastreabilidade

SD-MAN-01 e SD-MAN-02 correspondem, na mesma ordem, a UC-MAN-01 e UC-MAN-02 e a RF-MAN-01 e RF-MAN-02.
