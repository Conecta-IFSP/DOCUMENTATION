# Manual de utilização

> Padrão da disciplina · Módulo MAN · Requisito R4 do Incremento 3 · Revisado em 25/11/2026

## Resumo

O requisito R4 do Incremento 3 pede um manual digital que oriente os usuários sobre como utilizar o sistema. Ele foi atendido em duas partes. A primeira é a tela Ajuda do aplicativo, com 42 perguntas frequentes separadas em 7 assuntos, busca por palavra e o selo "Responsável" nas perguntas que tratam de ações exclusivas do responsável pela comissão. A segunda é o manual completo em PDF, com 27 páginas, que vai junto com o aplicativo e é aberto, sem internet, na tela "Manual de utilização", pelo botão "Ver manual completo (PDF)" no fim da tela Ajuda. O R4 não usa a API nem o banco de dados: o conteúdo da Ajuda, o PDF e o leitor de PDF (PDF.js) fazem parte do aplicativo instalado.

## Escopo

- consultar a tela Ajuda, com ou sem login, com as perguntas organizadas por assunto;
- abrir e fechar a resposta de cada pergunta;
- buscar perguntas por palavra;
- abrir o manual completo em PDF a partir da tela Ajuda e lê-lo dentro do aplicativo, com ou sem login e com ou sem internet, rolando as páginas e ampliando com o gesto de pinça.

Ficam fora deste módulo a edição das perguntas pelo próprio aplicativo, a troca do manual sem uma nova versão do aplicativo e as opções de baixar, compartilhar, imprimir ou pesquisar texto no PDF: o conteúdo da Ajuda e o PDF só mudam com uma nova versão do aplicativo.

## Atores

| Ator | Quem é no sistema | O que faz neste módulo |
| --- | --- | --- |
| Usuário | Qualquer pessoa que usa o aplicativo, com ou sem login. | Lê as perguntas frequentes, busca por palavra e lê o manual em PDF dentro do aplicativo. Não altera o conteúdo. |

## Índice padronizado

| Nº | Artefato | Finalidade |
| --: | --- | --- |
| 1 | [Casos de uso](1%20-%20casos-uso-manual-de-utilizacao.md) | Diagrama, ator e requisitos de usuário. |
| 2 | [Requisitos funcionais](2%20-%20requisitos-funcionais-manual-de-utilizacao.md) | RF-MAN-01 e RF-MAN-02 no formulário padrão. |
| 3 | [Diagrama de classes](3%20-%20diagrama-classes-manual-de-utilizacao.md) | Conteúdo da Ajuda, telas, componentes e visualizador do manual. |
| 3 | [Dicionário de classes](3%20-%20dicionario-dados-classes-manual-de-utilizacao.md) | Atributos, tipos, métodos, bibliotecas e criação dos identificadores. |
| 4 | [Diagrama relacional](4%20-%20diagrama-relacional-manual-de-utilizacao.md) | Não se aplica: o R4 não usa banco de dados. Contém a justificativa. |
| 4 | [Dicionário relacional](4%20-%20dicionario-dados-relacional-manual-de-utilizacao.md) | Não se aplica, pelo mesmo motivo. Contém a justificativa. |
| 5 | [Requisitos não funcionais](5%20-%20requisitos-nao-funcionais-manual-de-utilizacao.md) | RNF-MAN-01 a RNF-MAN-12, com medida, verificação e situação. |
| 6 | [Diagramas de sequência](6%20-%20diagramas-sequencia-manual-de-utilizacao.md) | SD-MAN-01 e SD-MAN-02. |
| 7 | [Testes](7%20-%20testes-manual-de-utilizacao.md) | CT-MAN-01 a CT-MAN-20 e seus resultados. |
| — | [Manual do Usuário (PDF)](manual-usuario/manual.pdf) | Manual completo do aplicativo. É o mesmo arquivo empacotado no aplicativo em `FRONTEND/assets/manual/manual.pdf` e exibido na tela "Manual de utilização". |

## Rastreabilidade

| Caso de uso | Requisito | Sequência | Requisitos não funcionais | Testes |
| --- | --- | --- | --- | --- |
| UC-MAN-01 — Consultar Ajuda | RF-MAN-01 | SD-MAN-01 | RNF-MAN-01, 02, 05, 06, 07, 08, 09 | CT-MAN-01 a CT-MAN-08, CT-MAN-12, CT-MAN-13 |
| UC-MAN-02 — Abrir manual em PDF | RF-MAN-02 | SD-MAN-02 | RNF-MAN-01 a 08, RNF-MAN-10, 11, 12 | CT-MAN-09 a CT-MAN-12, CT-MAN-14 a CT-MAN-20 |

## Alterações após a avaliação

Na avaliação do R4, o professor apontou dois problemas no manual em PDF:

1. **Dependia de internet.** O botão "Ver manual completo (PDF)" chamava `Linking.openURL(URL_MANUAL_PDF)`, que entregava ao navegador do aparelho o endereço do arquivo no GitHub. Sem conexão, o manual não abria, e quem mostrava a falha era o navegador.
2. **Apareceu a mensagem "Invalid PDF"** ao abrir o manual pelo link.

A solução foi tirar o manual da internet e colocá-lo dentro do aplicativo:

- o PDF passou a ser empacotado no aplicativo, em `FRONTEND/assets/manual/manual.pdf`, e foi comprimido para cerca de 1,2 MB (1.220.899 bytes na versão corrigida);
- o botão da Ajuda agora abre a tela "Manual de utilização" (`src/app/manual.tsx`), que, como a Ajuda, também abre sem login;
- a tela usa o componente `ManualPdf` (`src/components/ManualPdf.tsx`), que mostra o PDF em uma `WebView` (biblioteca `react-native-webview`) com o leitor PDF.js 3.11.174 embutido em `assets/pdfjs/`. Nenhum arquivo é buscado na internet, e não há biblioteca nativa própria, por isso funciona no Expo Go e nos builds Android e iOS;
- saíram do código a constante `URL_MANUAL_PDF` e a chamada a `Linking.openURL`;
- `TelaComCabecalho` ganhou a opção `rolagem`, e `metro.config.js` passou a tratar arquivos `.pdf` e `.txt` como assets.

Como o aplicativo não abre mais o arquivo pelo link do GitHub, a causa do "Invalid PDF" deixa de afetar o usuário. A contrapartida é que atualizar o manual agora exige uma nova versão do aplicativo; ela está registrada no [RNF-MAN-03](5%20-%20requisitos-nao-funcionais-manual-de-utilizacao.md#rnf-man-03--manual-empacotado-no-aplicativo). Os testes do manual dentro do aplicativo foram executados em 25/11/2026 no iPhone 14, via Expo Go, e no emulador Pixel 6, via Expo Go e com build release em modo avião, e foram aprovados. O CT-MAN-18 (erro e "Tentar de novo") também foi aprovado no iPhone, com um arquivo inválido no lugar do PDF, restaurado em seguida. Não foi executado o CT-MAN-20 (build iOS), porque não há Mac disponível, e o build para iPhone exige conta paga de desenvolvedor Apple; no iPhone, o manual foi verificado pelo Expo Go. O pico de memória do RNF-MAN-12 foi medido só no emulador Android: 711,1 MiB com o manual aberto, 280,5 MiB acima da tela Ajuda.

## Base de conferência

FRONTEND na branch `homolog`, commit `93fa767` ("corrige textos do manual (revisão RNF-MAN-05)"), e DOCUMENTATION na branch `main`, conferidos em 25/11/2026. Os arquivos principais são `FRONTEND/src/app/ajuda.tsx`, `FRONTEND/src/constants/ajuda.ts`, `FRONTEND/src/utils/busca-ajuda.ts`, `FRONTEND/src/components/Sanfona.tsx` e `FRONTEND/tests/busca-ajuda.test.cjs`, para a Ajuda, e os arquivos criados para o manual no commit `fa1ba7a` ("ajuste do pdf para abrir dentro do app"): `FRONTEND/src/app/manual.tsx`, `FRONTEND/src/components/ManualPdf.tsx`, `FRONTEND/assets/manual/manual.pdf` e `FRONTEND/assets/pdfjs/` (`pdf.min.js.txt`, `pdf.worker.min.js.txt` e `README.md`). O mesmo commit alterou `src/app/ajuda.tsx`, `src/constants/ajuda.ts`, `src/app/_layout.tsx`, `src/components/TelaComCabecalho.tsx`, `metro.config.js`, `package.json` e `app.json`. O commit seguinte, `93fa767`, substituiu apenas o `assets/manual/manual.pdf` pela versão corrigida na revisão de linguagem do RNF-MAN-05 (SHA-256 `35a838f8…7b963097`, 1.220.899 bytes, conferido no próprio commit). No DOCUMENTATION, o mesmo arquivo corrigido substitui `manual-usuario/manual.pdf` e entra no commit desta revisão. O BACKEND não é usado pelo R4.

## Repositórios

- [Backend](https://github.com/Conecta-IFSP/BACKEND)
- [Frontend](https://github.com/Conecta-IFSP/FRONTEND)
