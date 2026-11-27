# Diagramas de sequência — Manual de utilização

> Padrão da disciplina · Módulo MAN · Revisado em 26/11/2026

## Resumo

Há um diagrama para cada requisito de [Requisitos funcionais](2%20-%20requisitos-funcionais-manual-de-utilizacao.md), com o mesmo número. As linhas de vida são telas, componentes e módulos que existem no código do aplicativo e as bibliotecas que eles chamam. Toda mensagem é a chamada de uma função existente, com seus parâmetros, e as mensagens são numeradas na ordem em que acontecem. O que não é uma função, como a leitura de uma constante ou uma regra de comparação, aparece em notas. Todas as funções chamadas estão no [Diagrama de classes](3%20-%20diagrama-classes-manual-de-utilizacao.md), com os tipos dos parâmetros e do retorno.

As telas Ajuda e "Manual de utilização" não enviam requisições à API do Conecta+. Por isso nenhum diagrama tem `api`, guards, controller, service ou model.

| Linha de vida    | Origem no código                                                                                                     |
| ---------------- | -------------------------------------------------------------------------------------------------------------------- |
| `TelaAjuda`      | `src/app/ajuda.tsx` (`TelaAjuda`, `primeiraPerguntaDo`, `definirBusca`, `rolarParaTopico`, `alternar` e `voltar`)   |
| `buscaAjuda`     | `src/utils/busca-ajuda.ts` (`filtrarTopicosAjuda` e `normalizarBusca`)                                               |
| `Sanfona`        | `src/components/Sanfona.tsx` (`alternar` e a propriedade `aoAlternar`)                                               |
| `Router`         | biblioteca `expo-router` (`push`, `back`, `replace` e `canGoBack`)                                                   |
| `TelaManual`     | `src/app/manual.tsx`                                                                                                 |
| `ManualPdf`, `Visualizador`, `leitorManual` | `src/components/ManualPdf.tsx`                                                                         |
| `paginaDoManual` | funções da página HTML montada por `montarHtml`, executadas dentro da `WebView`                                     |
| `Asset`, `File`, `WebView`, `pdfjsLib` | bibliotecas `expo-asset`, `expo-file-system`, `react-native-webview` e PDF.js 3.11.174                        |

## SD-MAN-01 — Consultar Ajuda

```mermaid
sequenceDiagram
    autonumber
    actor U as Usuário
    participant T as TelaAjuda
    participant B as buscaAjuda
    participant S as Sanfona
    participant R as Router
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
    T->>R: canGoBack()
    alt há tela anterior
        R-->>T: true
        T->>R: back()
    else não há tela anterior
        R-->>T: false
        T->>R: replace(rota)
        Note over T,R: rota é /inicio com usuário na sessão ou /login sem usuário
    end
```

Para fechar a resposta, o usuário toca de novo na pergunta: as mensagens a partir de `alternar()` se repetem com `aberta` igual a `false`.

## SD-MAN-02 — Abrir manual em PDF

```mermaid
sequenceDiagram
    autonumber
    actor U as Usuário
    participant T as TelaAjuda
    participant R as Router
    participant M as TelaManual
    participant P as ManualPdf
    participant V as Visualizador
    participant L as leitorManual
    participant AT as Asset
    participant F as File
    participant W as WebView
    participant G as paginaDoManual
    participant J as pdfjsLib
    U->>T: aoTocar()
    T->>R: push("/manual")
    R->>M: TelaManual()
    M->>P: ManualPdf()
    P->>V: Visualizador(aoTentarDeNovo)
    Note over V: Enquanto prepara, a tela mostra "Abrindo o manual…"
    V->>L: obterConteudo(cores.fundo)
    alt página já preparada com esta cor de fundo
        Note over L: cacheConteudo tem a página da cor
    else primeira abertura com esta cor de fundo
        L->>L: prepararConteudo(corFundo)
        loop manual.pdf, pdf.min.js.txt e pdf.worker.min.js.txt
            L->>L: lerAsset(modulo)
            L->>AT: fromModule(modulo)
            AT-->>L: asset
            L->>AT: downloadAsync()
            AT-->>L: localUri
        end
        L->>F: base64()
        F-->>L: pdfBase64
        L->>F: text()
        F-->>L: codigo da PDF.js e do processo auxiliar
        L->>L: montarHtml(pdfBase64, pdfJs, worker, corFundo)
        L->>L: escaparScript(codigo)
    end
    L-->>V: conteudo
    V->>W: WebView(source, onMessage, onError)
    W->>G: renderizar()
    G->>G: renovarVigia()
    G->>G: base64ParaBytes(PDF_BASE64)
    G->>G: abrirDocumento(bytes)
    G->>G: criarWorker()
    G->>G: esperarWorkerPronto(worker, 3000)
    opt processo auxiliar não respondeu em 3 segundos
        G->>G: rodarWorkerNaThreadPrincipal()
    end
    G->>J: getDocument({ data, isEvalSupported: false })
    J-->>G: documento
    loop cada página, de 1 até o total
        G->>J: getPage(numero)
        J-->>G: pagina
        Note over G: escala: largura da tela menos 16 pixels, vezes a densidade limitada a 2
        G->>J: render({ canvasContext, viewport })
        J-->>G: concluído
        G->>G: canvasParaUrl(canvas)
        G->>G: renovarVigia()
        G->>G: enviar({ tipo: "progresso", pagina, total })
        G->>W: postMessage(mensagem)
        W->>V: aoReceberMensagem(evento)
        Note over V: Mostra "Carregando página N+1 de T" enquanto faltam páginas
    end
    G->>G: enviar({ tipo: "pronto", total })
    G->>W: postMessage(mensagem)
    W->>V: aoReceberMensagem(evento)
    opt falha ao abrir ou desenhar, ou 30 segundos sem página nova com a tela visível
        G->>G: falhar(erro)
        G->>W: postMessage(mensagem)
        W->>V: aoReceberMensagem(evento)
        V->>V: falhar(detalhe)
        Note over V: EstadoVazio com a mensagem de erro e o botão "Tentar de novo"
        U->>V: aoTentarDeNovo()
        Note over P,V: proxima = função que soma 1 ao valor anterior de tentativa
        V->>P: definirTentativa(proxima)
        Note over P,V: A chave do Visualizador muda e ele é montado de novo
    end
```

Na versão web, `WebView` não tem implementação para navegador, e a página do manual não é executada.

## Rastreabilidade

SD-MAN-01 e SD-MAN-02 correspondem, na mesma ordem, a UC-MAN-01 e UC-MAN-02 e a RF-MAN-01 e RF-MAN-02.
