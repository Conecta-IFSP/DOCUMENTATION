# Dicionário de classes — Manual de utilização

> Padrão da disciplina · Módulo MAN · Complementa o diagrama de classes · Revisado em 26/11/2026

## Resumo

Descreve cada classe do [Diagrama de classes](3%20-%20diagrama-classes-manual-de-utilizacao.md): atributos, tipos, obrigatoriedade, regras e métodos. O conteúdo da Ajuda é declarado no próprio aplicativo, em `src/constants/ajuda.ts`, e o manual em PDF é o recurso `assets/manual/manual.pdf`. Nenhum dos dois é armazenado no banco de dados nem enviado ou recebido pela API. Para alterar uma pergunta ou o manual, é preciso editar o arquivo correspondente e publicar uma nova versão do aplicativo. Os tipos abaixo são os do TypeScript, como estão no código.

| Classe              | Camada                 | Responsabilidade                                                                                   |
| ------------------- | ---------------------- | -------------------------------------------------------------------------------------------------- |
| `TopicoAjuda`       | Aplicativo (tipo)      | Agrupar as perguntas de um assunto.                                                                |
| `PerguntaAjuda`     | Aplicativo (tipo)      | Guardar uma pergunta, a resposta e o perfil.                                                       |
| `PerfilAjuda`       | Aplicativo (tipo)      | Indicar se a pergunta trata de ação exclusiva do responsável.                                      |
| `ajuda`             | Aplicativo (módulo)    | Declarar o conteúdo da Ajuda e o texto do selo.                                                    |
| `TelaAjuda`         | Aplicativo (tela)      | Mostrar os tópicos, a busca e o botão do manual.                                                   |
| `Sanfona`           | Aplicativo (componente)| Mostrar uma pergunta que abre e fecha.                                                             |
| `buscaAjuda`        | Aplicativo (módulo)    | Normalizar o texto e filtrar as perguntas.                                                         |
| `TelaManual`        | Aplicativo (tela)      | Mostrar o cabeçalho "Manual de utilização" e o leitor.                                             |
| `ManualPdf`         | Aplicativo (componente)| Montar o leitor de novo ao trocar o tema ou ao tentar de novo.                                     |
| `Visualizador`      | Aplicativo (componente)| Preparar a página do manual, mostrá-la na `WebView` e tratar o progresso e as falhas.              |
| `leitorManual`      | Aplicativo (módulo)    | Ler os recursos, montar a página HTML e guardá-la em memória.                                      |
| `paginaDoManual`    | Aplicativo (script)    | Desenhar as páginas do PDF dentro da `WebView` e avisar a tela.                                    |

---

## 1. Tipo `TopicoAjuda`

Representa um assunto da tela Ajuda, como Conta e acesso ou Votação. Agrupa as perguntas sobre esse assunto e é exibido como um título com ícone, seguido da lista de perguntas.

| Campo       | Tipo                       | Obrigatório | Descrição                                                                                                                                                       |
| ----------- | -------------------------- | ----------: | --------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `id`        | `string`                   |         Sim | Identificador do tópico, usado no endereço `/ajuda?topico=<id>`, que rola a tela até o tópico e abre a primeira pergunta dele. Criação: ver o [dicionário relacional](4%20-%20dicionario-dados-relacional-manual-de-utilizacao.md). |
| `titulo`    | `string`                   |         Sim | Nome do assunto exibido na tela. Também é considerado pela busca.                                                                                              |
| `icone`     | `IoniconsIconName`         |         Não | Nome de um ícone da biblioteca Ionicons, exibido ao lado do título. Sem ele, o título aparece sozinho.                                                         |
| `perguntas` | `readonly PerguntaAjuda[]` |         Sim | Perguntas do tópico, na ordem em que são exibidas.                                                                                                             |

**Regras.** O `id` usa letras minúsculas, sem acento nem espaço. O `id` `conta` não pode mudar: o link "Precisa de ajuda?" da tela Entrar abre a Ajuda com `topico=conta`. Os tópicos são exibidos na ordem da lista `TOPICOS_AJUDA`. O padrão de escrita do arquivo limita cada tópico a 7 perguntas; o código não verifica esse limite. Durante uma busca, o tópico sem nenhuma pergunta encontrada deixa de ser exibido.

## 2. Tipo `PerguntaAjuda`

Representa uma pergunta frequente e a sua resposta. Existe apenas dentro da lista `perguntas` de um `TopicoAjuda`.

| Campo      | Tipo                          | Obrigatório | Descrição                                                                                                       |
| ---------- | ----------------------------- | ----------: | --------------------------------------------------------------------------------------------------------------- |
| `id`       | `string`                      |         Sim | Identificador da pergunta; controla quais perguntas estão abertas na tela.                                      |
| `pergunta` | `string`                      |         Sim | Texto da pergunta exibido na sanfona. Também é considerado pela busca.                                         |
| `resposta` | `string \| readonly string[]` |         Sim | Um texto é exibido como parágrafo; uma lista é exibida como passos numerados. Também é considerada pela busca. |
| `perfil`   | `PerfilAjuda`                 |         Sim | Indica se a pergunta trata de uma ação exclusiva do responsável da comissão.                                   |

**Regras.** O `id` é único em todo o arquivo, e não apenas no tópico. O padrão de escrita limita a resposta em lista a 4 passos, com uma ação curta por passo, e a resposta em texto a 2 ou 3 frases; o código não verifica; na contagem de 26/11/2026, sobre `8fc0a5a`, nenhuma lista passa de 4 passos. A resposta descreve só o que o aplicativo faz, com os nomes de botões e telas que aparecem para o usuário.

## 3. Enumeração `PerfilAjuda`

Declarada como o tipo `'responsavel' | 'todos'`.

| Valor         | Descrição                                                                                   |
| ------------- | ------------------------------------------------------------------------------------------- |
| `responsavel` | A pergunta trata de uma ação exclusiva do responsável da comissão e exibe o selo "Responsável". |
| `todos`       | A pergunta vale para qualquer usuário. Nenhum selo é exibido.                               |

O perfil não restringe quem vê a pergunta: todas são exibidas para todos, com ou sem login.

## 4. Módulo `ajuda`

Representa o arquivo `src/constants/ajuda.ts`, que declara os tipos acima e as constantes lidas pela tela Ajuda.

| Constante       | Tipo                                  | Descrição                                                                                         |
| --------------- | ------------------------------------- | ------------------------------------------------------------------------------------------------- |
| `TOPICOS_AJUDA` | `readonly TopicoAjuda[]`              | Todo o conteúdo da tela Ajuda, na ordem de exibição.                                              |
| `SELO_PERFIL`   | `Record<PerfilAjuda, string \| null>` | Texto do selo de cada perfil: `responsavel` → `'Responsável'`; `todos` → `null`, sem selo.        |

Conteúdo de `TOPICOS_AJUDA` conferido em 26/11/2026:

| `id`           | `titulo`                 | `icone`                  | Perguntas | Com perfil `responsavel` | Respostas em lista |
| -------------- | ------------------------ | ------------------------ | --------: | -----------------------: | -----------------: |
| `conta`        | Conta e acesso           | `person-circle-outline`  |         4 |                        0 |                  2 |
| `organizacoes` | Organizações e comissões | `business-outline`       |         6 |                        0 |                  3 |
| `reunioes`     | Reuniões                 | `calendar-outline`       |         6 |                        3 |                  3 |
| `presenca`     | Presença                 | `checkmark-done-outline` |         6 |                        3 |                  6 |
| `ata`          | Ata                      | `document-text-outline`  |         7 |                        2 |                  3 |
| `votacao`      | Votação                  | `checkbox-outline`       |         7 |                        1 |                  2 |
| `relatorio`    | Relatório individual     | `bar-chart-outline`      |         6 |                        0 |                  2 |
| **Total**      |                          |                          |    **42** |                    **9** |             **21** |

## 5. Classe `TelaAjuda`

Representa a tela Ajuda (`src/app/ajuda.tsx`), que abre com ou sem login. Os campos abaixo são o estado da tela e são descartados quando o usuário sai dela.

| Campo             | Tipo                     | Obrigatório | Descrição                                                                                         |
| ----------------- | ------------------------ | ----------: | ------------------------------------------------------------------------------------------------- |
| `topico`          | `string`                 |         Não | Parâmetro do endereço que indica o tópico a abrir, por exemplo `conta`.                          |
| `busca`           | `string`                 |         Sim | Texto digitado no campo Buscar. Começa vazio.                                                     |
| `abertas`         | `ReadonlySet<string>`    |         Sim | Identificadores das perguntas abertas. Começa vazio ou com a primeira pergunta do tópico recebido. |
| `topicosVisiveis` | `readonly TopicoAjuda[]` |         Sim | Resultado de `filtrarTopicosAjuda` aplicado a `TOPICOS_AJUDA` com o texto de `busca`.             |
| `encontradas`     | `number`                 |         Sim | Quantidade de perguntas em `topicosVisiveis`, exibida durante a busca.                            |

| Método                                          | Retorno               | Descrição                                                                                                                    |
| ----------------------------------------------- | --------------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| `TelaAjuda()`                                   | `JSX.Element`         | Monta a tela com o texto de orientação, o campo Buscar, os tópicos e o botão do manual.                                      |
| `primeiraPerguntaDo(topicoId: string \| undefined)` | `string \| undefined` | Procura em `TOPICOS_AJUDA` o tópico cujo `id` é igual a `topicoId` e devolve o `id` da primeira pergunta dele.           |
| `definirBusca(texto: string)`                   | `void`                | Guarda em `busca` o texto digitado; com texto vazio, é chamada pelo ícone "Limpar busca".                                   |
| `rolarParaTopico()`                             | `void`                | Rola a tela até o tópico recebido, uma única vez por tópico.                                                                 |
| `alternar(perguntaId: string, aberta: boolean)` | `void`                | Com `aberta` verdadeiro, acrescenta `perguntaId` a `abertas`; com falso, retira.                                             |
| `voltar()`                                      | `void`                | Volta à tela anterior ou, sem uma, abre o Início (com login) ou a tela Entrar (sem login).                                    |

O botão "Ver manual completo (PDF)" recebe em `aoTocar` uma função que executa `router.push('/manual')`.

## 6. Classe `Sanfona`

Representa uma pergunta que se abre e se fecha (`src/components/Sanfona.tsx`).

| Campo           | Tipo                          | Obrigatório | Descrição                                                                                              |
| --------------- | ----------------------------- | ----------: | ------------------------------------------------------------------------------------------------------ |
| `pergunta`      | `string`                      |         Sim | Texto do cabeçalho.                                                                                    |
| `resposta`      | `string \| readonly string[]` |         Sim | Texto corrido ou lista de passos, exibida numerada.                                                    |
| `aberta`        | `boolean`                     |         Não | Indica se a resposta está visível. A tela Ajuda sempre informa; sem ele, o componente guarda o estado. |
| `abertaInicial` | `boolean`                     |         Não | Estado inicial quando `aberta` não é informado. Padrão: falso.                                         |
| `aoAlternar`    | `(aberta: boolean) => void`   |         Não | Função chamada no toque.                                                                               |
| `selo`          | `string \| null`              |         Não | Texto do selo; com valor, o leitor de tela anuncia "Exclusivo do Responsável".                         |
| `primeira`      | `boolean`                     |         Não | Indica a primeira pergunta do tópico, sem linha divisória acima. Padrão: falso.                        |

O método `alternar` inverte o estado e chama `aoAlternar`. A abertura e o fechamento são animados em 220 ms; com a opção de reduzir movimento do aparelho ativada, a animação não é feita.

## 7. Módulo `buscaAjuda`

Arquivo `src/utils/busca-ajuda.ts`.

| Função                                                                | Retorno                  | Descrição                                                                                                                                                                                   |
| --------------------------------------------------------------------- | ------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `normalizarBusca(texto: string)`                                      | `string`                 | Remove acentos e espaços nas pontas e converte para minúsculas, para que "Votação", "VOTACAO" e "votacao" sejam iguais.                                                                    |
| `filtrarTopicosAjuda(topicos: readonly TopicoAjuda[], busca: string)` | `readonly TopicoAjuda[]` | Mantém só as perguntas que contêm todas as palavras digitadas no título do tópico, na pergunta ou na resposta, e retira os tópicos sem perguntas. A comparação é por trecho, não por palavra inteira. Com a busca vazia, devolve a própria lista. |

## 8. Classe `TelaManual`

Representa a tela "Manual de utilização" (`src/app/manual.tsx`, rota `/manual`), que abre com ou sem login. Não tem estado próprio: mostra o `TelaComCabecalho` com o título "Manual de utilização", sem rolagem própria (`rolagem={false}`) e com divisor, e dentro dele o `ManualPdf`.

## 9. Classes `ManualPdf` e `Visualizador`

Definidas em `src/components/ManualPdf.tsx`.

| Classe         | Campo ou método                                        | Tipo                     | Descrição                                                                                                                                                                 |
| -------------- | ------------------------------------------------------ | ------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `ManualPdf`    | `tentativa`                                            | `number`                 | Começa em 0 e soma 1 a cada "Tentar de novo".                                                                                                                             |
| `ManualPdf`    | `ManualPdf()`                                          | `JSX.Element`            | Monta `Visualizador` com a chave `<cor de fundo>-<tentativa>`, o que o monta do zero quando o tema ou a tentativa mudam.                                                 |
| `Visualizador` | `conteudo`                                             | `Conteudo \| null`       | Página HTML preparada; nulo enquanto prepara.                                                                                                                             |
| `Visualizador` | `estado`                                               | `Estado`                 | Fase da exibição: `preparando`, `renderizando` (com `pagina` e `total`), `pronto` ou `erro`. Começa em `preparando`.                                                     |
| `Visualizador` | `Visualizador(aoTentarDeNovo)`                         | `JSX.Element`            | Chama `obterConteudo` ao ser montado e mostra a `WebView`, o aviso "Abrindo o manual…", a pílula de progresso ou a tela de erro.                                         |
| `Visualizador` | `aoReceberMensagem(evento: WebViewMessageEvent)`       | `void`                   | Lê a mensagem enviada pela página: `progresso` atualiza a pílula, `pronto` a esconde e `erro` chama `falhar`. Mensagem que não é JSON é ignorada.                        |
| `Visualizador` | `falhar(detalhe: unknown)`                             | `void`                   | Registra o detalhe no console de depuração e muda o estado para `erro`.                                                                                                   |

Tipos usados: `Conteudo` é `{ html: string }` ou `{ uri: string }` (a segunda forma só é usada se a constante `CARREGAR_POR_ARQUIVO` for verdadeira; na base conferida, ela é falsa); `MensagemWebView` é `{ tipo: 'progresso', pagina, total }`, `{ tipo: 'pronto', total }` ou `{ tipo: 'erro', detalhe }`.

## 10. Módulo `leitorManual`

Nome dado no diagrama às funções e ao cache declarados no nível do arquivo `src/components/ManualPdf.tsx`.

| Campo ou função                                                | Tipo ou retorno                    | Descrição                                                                                                                                                                             |
| -------------------------------------------------------------- | ---------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `cacheConteudo`                                                | `Map<string, Promise<Conteudo>>`   | Uma página preparada por cor de fundo, guardada na memória até o aplicativo ser fechado.                                                                                             |
| `lerAsset(modulo: number)`                                     | `Promise<File>`                    | Obtém o recurso com `Asset.fromModule(modulo).downloadAsync()` e o abre como `File` pelo endereço local.                                                                             |
| `escaparScript(codigo: string)`                                | `string`                           | Troca `</script` por `<\/script`, sem diferenciar maiúsculas, para o código embutido não fechar a marcação antes da hora.                                                             |
| `montarHtml(pdfBase64, pdfJs, worker, corFundo)`               | `string`                           | Escreve a página HTML com a cor de fundo do tema, o PDF em base64, o código da PDF.js e o do processo auxiliar embutidos, e o script `paginaDoManual`.                               |
| `prepararConteudo(corFundo: string)`                           | `Promise<Conteudo>`                | Lê os três recursos ao mesmo tempo, o PDF em base64 e os scripts como texto, e chama `montarHtml`.                                                                                   |
| `obterConteudo(corFundo: string)`                              | `Promise<Conteudo>`                | Devolve a página do cache ou chama `prepararConteudo`; se a preparação falhar, retira a entrada do cache.                                                                            |

## 11. Script `paginaDoManual`

Funções JavaScript escritas dentro do texto montado por `montarHtml`, executadas pela `WebView`.

| Função                                   | Descrição                                                                                                                                                                       |
| ---------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `renderizar()`                           | Liga o vigia, converte o PDF em bytes, abre o documento e desenha as páginas uma a uma, enviando o progresso; ao terminar, envia `pronto`.                                      |
| `abrirDocumento(bytes)`                  | Tenta o processo auxiliar da PDF.js; se ele não responde em 3 segundos, usa o processo principal; abre o PDF com `pdfjsLib.getDocument({ data, isEvalSupported: false })`.   |
| `criarWorker()`                          | Cria o processo auxiliar a partir de um `Blob` com o código embutido; devolve nulo se não conseguir.                                                                           |
| `esperarWorkerPronto(worker, ms)`        | Espera a primeira mensagem do processo auxiliar por até `ms` milissegundos.                                                                                                   |
| `rodarWorkerNaThreadPrincipal()`         | Executa o código do processo auxiliar na própria página, mais lento, quando o processo auxiliar não funciona.                                                                 |
| `base64ParaBytes(base64)`                | Converte o PDF em base64 numa lista de bytes.                                                                                                                                   |
| `canvasParaUrl(canvas)`                  | Converte o desenho da página em imagem JPEG com qualidade 0,85.                                                                                                                |
| `renovarVigia()`                         | Reinicia o limite de 30 segundos; com a página oculta, só renova; vencido com a página visível, chama `falhar`.                                                                |
| `enviar(mensagem)`                       | Envia a mensagem à tela com `window.ReactNativeWebView.postMessage`.                                                                                                            |
| `falhar(erro)`                           | Envia uma única mensagem `erro` e desliga o vigia.                                                                                                                              |

## Como os identificadores são criados

Os únicos identificadores do R4 são o `id` do tópico e o `id` da pergunta, textos escritos à mão no arquivo `src/constants/ajuda.ts`. Quem os cria, o formato, o valor inicial e o incremento estão no [dicionário relacional](4%20-%20dicionario-dados-relacional-manual-de-utilizacao.md).

## Rastreabilidade

`TopicoAjuda`, `PerguntaAjuda`, `PerfilAjuda`, `ajuda`, `TelaAjuda`, `Sanfona` e `buscaAjuda` atendem ao RF-MAN-01. `TelaManual`, `ManualPdf`, `Visualizador`, `leitorManual` e `paginaDoManual` atendem ao RF-MAN-02.
