# Requisitos não funcionais — Relatório individual

> Padrão da disciplina · Módulo REL · Revisado em 27/11/2026

## Resumo

Os requisitos não funcionais aparecem em dois níveis, como os funcionais. Os **de usuário** dizem, em uma frase, a qualidade esperada. Os **de sistema** completam cada um com a medida que o aprova, a condição em que vale, como foi verificado e onde está a evidência, de modo que qualquer pessoa consiga conferir. Onde ainda não houve medição ou execução, isso está escrito.

A classificação segue a árvore de Sommerville usada na disciplina: requisitos de **produto** (facilidade de uso, eficiência, confiabilidade, portabilidade), **organizacionais** (entrega, implementação, padrões) e **externos** (interoperabilidade, éticos, legais).

## Requisitos não funcionais de usuário

**RNF-REL-01 — Acesso restrito.** Só uma pessoa com conta comum ativa e conectada pode consultar um relatório individual.

**RNF-REL-02 — Privacidade do relatório.** Cada pessoa vê apenas o próprio relatório, sem dados de outros participantes.

**RNF-REL-03 — Exportação sem conteúdo executável.** Títulos e nomes cadastrados por outras pessoas não podem virar código no PDF nem fórmula na planilha.

**RNF-REL-04 — CSV compatível com planilhas.** O CSV deve abrir em planilhas em português com acentos corretos e cada informação na sua coluna.

**RNF-REL-05 — PDF em A4.** O PDF deve sair em folha A4, pronto para imprimir.

**RNF-REL-06 — Data, hora e período.** As datas devem aparecer no horário de quem consulta, e "este mês" e "este ano" devem seguir o calendário do aparelho.

**RNF-REL-07 — Coerência da tela.** A tela nunca deve mostrar um relatório que não corresponde ao filtro escolhido ou à última consulta.

**RNF-REL-08 — Telas de vários tamanhos.** O relatório deve caber na tela, sem rolagem lateral, do celular pequeno ao computador.

**RNF-REL-09 — Legibilidade nos dois temas.** O relatório deve ser legível no tema claro e no tema escuro.

**RNF-REL-10 — Acessibilidade.** O relatório deve poder ser usado pelo teclado e por leitores de tela.

**RNF-REL-11 — Portabilidade.** A consulta e as exportações devem funcionar no Android a partir da versão 7.0, no iOS, no Expo Go e na web.

**RNF-REL-12 — Desempenho com histórico longo.** O relatório deve abrir em poucos segundos mesmo para quem participou de muitas reuniões.

**RNF-REL-13 — Testabilidade.** Os testes automatizados do relatório devem passar antes de cada entrega.

## Requisitos não funcionais de sistema

### RNF-REL-01 — Acesso restrito

| Campo             | Conteúdo                                                                                                                                                                                                                   |
| ----------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Classificação** | Externo › Legal › Segurança                                                                                                                                                                                                |
| **Critério**      | A rota `GET /relatorios/individual` só responde com dados a requisições com a chave do aplicativo, um token válido de conta ativa e conta do tipo `USUARIO`.                                                               |
| **Medida**        | Sem `X-API-Key` ou com chave errada: 401. Sem token, com token inválido, expirado ou de conta inexistente: 401. Conta desativada: 401. Conta `ADMIN_SISTEMA`: 403. Em nenhum desses casos a resposta traz dados do relatório. |
| **Condição**      | Qualquer chamada à rota, pelo aplicativo ou direta.                                                                                                                                                                        |
| **Verificação**   | Teste de integração CT-REL-01, com MongoDB local e requisições HTTP reais.                                                                                                                                                 |
| **Evidência**     | `ApiKeyGuard`, `JwtAuthGuard` e `UsuarioComumGuard` (`BACKEND/src/auth/`), aplicados em `RelatoriosController`; `RelatoriosService.individual`; `BACKEND/test/relatorio-individual-mongo.mjs`. Desde o backend `543e03f`, o `JwtAuthGuard` também recusa com 401 o token emitido antes de `senha_alterada_em`, caso que o CT-REL-01 não cobre.                             |
| **Situação**      | Atendido em 26/11/2026.                                                                                                                                                                                                    |

### RNF-REL-02 — Privacidade do relatório

| Campo             | Conteúdo                                                                                                                                                                                                                                                                                                         |
| ----------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Classificação** | Externo › Legal › Segurança (proteção de dados pessoais)                                                                                                                                                                                                                                                         |
| **Critério**      | O relatório é sempre o da conta do token, e a resposta não traz dados de outros participantes.                                                                                                                                                                                                                   |
| **Medida**        | Com `usuario_id` de outra pessoa na URL, a resposta continua com o e-mail da conta do token; o e-mail de outro participante da mesma reunião não aparece em nenhum ponto da resposta; os itens de `reunioes` não trazem `participantes` nem `ata`; em 12 requisições simultâneas alternando duas contas, cada resposta traz o e-mail da própria conta. |
| **Condição**      | Reunião com mais de um participante confirmado.                                                                                                                                                                                                                                                                  |
| **Verificação**   | Teste de integração CT-REL-06.                                                                                                                                                                                                                                                                                   |
| **Evidência**     | `RelatoriosController.individual` usa `req.usuario.sub`; o `ValidationPipe` global com `whitelist` descarta parâmetros fora do `RelatorioIndividualDto`; `RelatoriosService.individual` monta a resposta só com os campos listados em RF-REL-01.                                                                 |
| **Situação**      | Atendido em 26/11/2026.                                                                                                                                                                                                                                                                                          |

### RNF-REL-03 — Exportação sem conteúdo executável

| Campo             | Conteúdo                                                                                                                                                                                                                      |
| ----------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Classificação** | Externo › Legal › Segurança                                                                                                                                                                                                   |
| **Critério**      | Todo texto vindo do banco é escapado no HTML do PDF, e nenhuma célula do CSV começa com um caractere que a planilha interpreta como fórmula.                                                                                   |
| **Medida**        | Textos como `<script>Ana</script>`, `<img src=x onerror=alert(1)>` e `<Org>` aparecem no HTML como texto, com `&lt;` e `&gt;`; valores que começam com `=`, `+`, `-` ou `@`, inclusive depois de espaços, e valores que começam com tabulação ou quebra de linha recebem um apóstrofo na frente. |
| **Condição**      | Qualquer título, nome, local, organização ou comissão com esses caracteres.                                                                                                                                                   |
| **Verificação**   | Testes CT-REL-08, CT-REL-10, CT-REL-12 e CT-REL-15.                                                                                                                                                                           |
| **Evidência**     | `escapar`, `htmlRelatorioIndividual` e `celulaCsv`, em `FRONTEND/src/utils/relatorio-individual.ts`.                                                                                                                          |
| **Situação**      | Atendido em 26/11/2026.                                                                                                                                                                                                       |

### RNF-REL-04 — CSV compatível com planilhas

| Campo             | Conteúdo                                                                                                                                                                                                                                                         |
| ----------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Classificação** | Externo › Interoperabilidade                                                                                                                                                                                                                                     |
| **Critério**      | Arquivo em UTF-8 com BOM, separador `;`, quebra de linha CR LF, todos os valores entre aspas e o mesmo número de colunas em todas as linhas.                                                                                                                      |
| **Medida**        | O primeiro caractere é o BOM (`U+FEFF`); toda linha tem 19 colunas (cabeçalho, reunião, vínculo e relatório vazio); aspas internas são duplicadas; `;`, aspas e quebras de linha dentro de um valor não criam coluna nem linha nova; acentos, ideogramas e emoji são preservados; as datas ficam em ISO 8601 UTC, como diz o cabeçalho. |
| **Condição**      | Relatórios com e sem reuniões e com e sem vínculos.                                                                                                                                                                                                              |
| **Verificação**   | Testes CT-REL-09, CT-REL-15 e CT-REL-16, e o download pelo navegador em CT-REL-25. A abertura do arquivo no Excel ou no LibreOffice não está registrada.                                                                                                          |
| **Evidência**     | `csvRelatorioIndividual` e `celulaCsv`.                                                                                                                                                                                                                          |
| **Situação**      | Atendido nos testes automatizados de 26/11/2026; abertura em programa de planilha pendente.                                                                                                                                                                      |

### RNF-REL-05 — PDF em A4

| Campo             | Conteúdo                                                                                                                                                                                                                                          |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Classificação** | Organizacional › Padrões                                                                                                                                                                                                                          |
| **Critério**      | O documento sai em A4 retrato, e o bloco de cada reunião fica inteiro numa página.                                                                                                                                                                |
| **Medida**        | No celular, `Print.printAsync` e `Print.printToFileAsync` recebem 595 × 842 pontos (A4); a folha de estilo declara página A4 retrato com margens de 16 mm e 17 mm e proíbe quebra de página dentro do bloco de uma reunião; na web, o arquivo gerado pela impressão do navegador começa com `%PDF-` e usa o tamanho de página do CSS. |
| **Condição**      | Web (motor Chromium) e celular.                                                                                                                                                                                                                   |
| **Verificação**   | CT-REL-26, na web, e inspeção de `htmlRelatorioIndividual` e `exportarRelatorioIndividual`. No aparelho: CT-REL-34 e CT-REL-35, não executados.                                                                                                   |
| **Evidência**     | `FRONTEND/src/utils/relatorio-individual.ts` e `FRONTEND/src/services/exportar-relatorio-individual.ts`.                                                                                                                                         |
| **Situação**      | Atendido na web em 26/11/2026; pendente no Android e no iOS.                                                                                                                                                                                      |

### RNF-REL-06 — Data, hora e período

| Campo             | Conteúdo                                                                                                                                                                                                                                                                     |
| ----------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Classificação** | Organizacional › Padrões                                                                                                                                                                                                                                                     |
| **Critério**      | A API guarda e devolve instantes em UTC; a tela e o PDF mostram no fuso do aparelho; o CSV mantém UTC e diz isso no cabeçalho; os limites de mês e ano são calculados à meia-noite do fuso do aparelho, com o fim exclusivo.                                                    |
| **Medida**        | Em 11/11/2026, "Este mês" envia `de` igual a 01/11/2026 00:00 e `ate` igual a 12/11/2026 00:00, no horário do aparelho, convertidos para UTC; o mesmo cálculo vale em 1º de janeiro, 31 de dezembro e 29 de fevereiro de 2028; uma reunião com `inicio` igual a `ate` não entra; o rodapé do PDF informa o fuso usado. |
| **Condição**      | Aparelho em qualquer fuso horário.                                                                                                                                                                                                                                           |
| **Verificação**   | CT-REL-04 (limite exclusivo na API) e CT-REL-11 (limites calculados no aplicativo). A execução com o aparelho em fuso diferente do de Brasília não está registrada.                                                                                                           |
| **Evidência**     | `consultaRelatorio`, `dataRelatorio` e `periodoRelatorio` no aplicativo; filtro de `inicio` em `RelatoriosService.individual`.                                                                                                                                                 |
| **Situação**      | Atendido em 26/11/2026 nos testes automatizados; execução em outro fuso pendente.                                                                                                                                                                                            |

### RNF-REL-07 — Coerência da tela

| Campo             | Conteúdo                                                                                                                                                                                                                                                                                 |
| ----------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Classificação** | Produto › Confiabilidade                                                                                                                                                                                                                                                                 |
| **Critério**      | Uma resposta atrasada não substitui a mais recente; depois de um erro, o relatório anterior não continua na tela; uma exportação não começa enquanto outra está em curso.                                                                                                                 |
| **Medida**        | Com a consulta de "Este mês" atrasada em 800 ms e a troca imediata para "Todo período", a tela mostra só o resultado de "Todo período"; depois de um erro 500, os botões de exportação não aparecem; depois de um 401, a sessão é encerrada e o aplicativo vai para a tela de entrada; com a nova janela bloqueada, a orientação aparece e o botão volta a ficar ativo. |
| **Condição**      | Web, com a API simulada pelo teste.                                                                                                                                                                                                                                                      |
| **Verificação**   | CT-REL-27, CT-REL-29 e CT-REL-30.                                                                                                                                                                                                                                                        |
| **Evidência**     | Contador `pedido`, trava `travaExportacao` e funções `carregar` e `exportar` em `FRONTEND/src/app/(abas)/relatorio.tsx`.                                                                                                                                                                 |
| **Situação**      | Atendido na web em 26/11/2026; no aparelho, pendente.                                                                                                                                                                                                                                    |

### RNF-REL-08 — Telas de vários tamanhos

| Campo             | Conteúdo                                                                                                                                                                                                                                       |
| ----------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Classificação** | Produto › Facilidade de uso                                                                                                                                                                                                                    |
| **Critério**      | Nenhuma rolagem horizontal; textos longos quebram linha; o conteúdo não passa de 560 px de largura.                                                                                                                                            |
| **Medida**        | A largura rolável do documento não passa da largura da janela em 320, 360, 390, 768 e 1440 px; o mesmo vale em 320 px com nome de usuário de 250 caracteres e nome de organização de 360 caracteres, e com a fonte ampliada em 130%, sem cortar o título. |
| **Condição**      | Web, navegador Microsoft Edge (Chromium), com a API simulada.                                                                                                                                                                                  |
| **Verificação**   | CT-REL-24 e CT-REL-33.                                                                                                                                                                                                                         |
| **Evidência**     | Estilo `conteudo` (`maxWidth: 560`) em `relatorio.tsx`.                                                                                                                                                                                        |
| **Situação**      | Atendido na web em 26/11/2026; no aparelho, pendente.                                                                                                                                                                                          |

### RNF-REL-09 — Legibilidade nos dois temas

| Campo             | Conteúdo                                                                                                                                         |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Classificação** | Produto › Facilidade de uso                                                                                                                      |
| **Critério**      | Todas as cores da tela vêm do tema ativo, sem cor fixa no código. O PDF é exceção proposital: usa papel branco e texto escuro, por ser documento para impressão. |
| **Medida**        | Nenhuma cor literal em `relatorio.tsx`; com o tema escuro, a página usa o esquema de cores escuro e mantém dados e botões visíveis.                |
| **Condição**      | Tema do aplicativo em Claro e em Escuro.                                                                                                         |
| **Verificação**   | Inspeção de `relatorio.tsx` e CT-REL-32, na web.                                                                                                 |
| **Evidência**     | Cores de `useTema().tema.cores` em `FRONTEND/src/app/(abas)/relatorio.tsx`.                                                                      |
| **Situação**      | Atendido na web em 26/11/2026; no aparelho, pendente.                                                                                            |

### RNF-REL-10 — Acessibilidade

| Campo             | Conteúdo                                                                                                                                                                                                                         |
| ----------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Classificação** | Produto › Facilidade de uso                                                                                                                                                                                                      |
| **Critério**      | Os controles são alcançáveis pelo teclado; o título é marcado como cabeçalho; o período é anunciado como grupo de opções, com a opção marcada.                                                                                     |
| **Medida**        | A partir do controle de período, no máximo 12 toques em Tab levam o foco a "Exportar CSV"; o título "Relatório individual" tem papel de cabeçalho; o controle tem o papel `radiogroup` com o rótulo "Período do relatório", e cada opção, o papel `radio` com o estado de marcada. |
| **Condição**      | Web; leitor de tela no Android (TalkBack) e no iOS (VoiceOver).                                                                                                                                                                  |
| **Verificação**   | CT-REL-31. Leitor de tela: CT-REL-37, não executado.                                                                                                                                                                                        |
| **Evidência**     | Em `relatorio.tsx`, o título "Relatório individual" com `accessibilityRole="header"`, escrito dentro do `CabecalhoAba`, e o `ControleSegmentado`, que dá os papéis `radiogroup` e `radio` e o estado de marcada.                  |
| **Situação**      | Parcial. A navegação por teclado foi aprovada em 26/11/2026. Na web, a opção marcada não é exposta como `aria-checked`, pendência do componente compartilhado `ControleSegmentado` registrada pela equipe. Leitor de tela pendente. |

### RNF-REL-11 — Portabilidade

| Campo             | Conteúdo                                                                                                                                                                                         |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Classificação** | Produto › Portabilidade                                                                                                                                                                          |
| **Critério**      | Consulta, PDF e CSV funcionam no aplicativo instalado (Android 7.0 a 16 e iOS), no Expo Go e na web.                                                                                             |
| **Medida**        | Em cada forma de uso, a consulta mostra o relatório e cada exportação abre o menu de compartilhamento, a tela de impressão ou o download, sem erro.                                                |
| **Condição**      | Aparelhos Android e iOS com o aplicativo instalado e com o Expo Go; navegador.                                                                                                                   |
| **Verificação**   | Web: CT-REL-23 a CT-REL-33. Android e iOS: com as bibliotecas simuladas (CT-REL-18 a CT-REL-22); no aparelho, só a consulta e o PDF no Android com o aplicativo instalado (CT-REL-38); o restante está registrado como pendente (CT-REL-34 a CT-REL-36). |
| **Evidência**     | Ramos por `Platform.OS` e `isRunningInExpoGo` em `exportarRelatorioIndividual`.                                                                                                                  |
| **Situação**      | Atendido na web em 26/11/2026. No Android com o aplicativo instalado, a consulta e o PDF foram confirmados em 26/11/2026 (CT-REL-38); o CSV no Android, o Expo Go e o iOS seguem pendentes, porque os testes com bibliotecas simuladas não comprovam as permissões nem o comportamento nativo. |

### RNF-REL-12 — Desempenho com histórico longo

| Campo             | Conteúdo                                                                                                                                                                                                            |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Classificação** | Produto › Eficiência › Desempenho                                                                                                                                                                                   |
| **Critério**      | Tempo de resposta de `GET /relatorios/individual` para um usuário com histórico longo.                                                                                                                              |
| **Medida**        | Meta proposta, ainda não medida: resposta em até 2 segundos, medida no servidor, para um usuário com 1.000 reuniões com presença confirmada.                                                                                         |
| **Condição**      | API e MongoDB locais, sem outra carga.                                                                                                                                                                              |
| **Verificação**   | Não executada: não há teste de carga.                                                                                                                                                                               |
| **Evidência**     | A busca das reuniões usa o índice `{ "participantes.usuario_id": 1, "inicio": 1 }`; as buscas de vínculos não têm índice em `membros.usuario_id`; a rota devolve a lista inteira, sem paginação.                     |
| **Situação**      | Não medido.                                                                                                                                                                                                         |

### RNF-REL-13 — Testabilidade

| Campo             | Conteúdo                                                                                                                                                                                                                      |
| ----------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Classificação** | Organizacional › Entrega                                                                                                                                                                                                      |
| **Critério**      | Os testes automatizados do relatório e as suítes dos dois projetos passam antes de cada entrega.                                                                                                                              |
| **Medida**        | `node test/relatorio-individual-mongo.mjs` termina com a linha "PASSOU"; `node --test tests/*.test.cjs` sem falha; `npx vitest run` sem falha. O teste de interface `tests/relatorio-browser.e2e.cjs` exige Playwright instalado à parte e é executado separadamente. |
| **Condição**      | Branch `homolog` (backend `d1f1aeb`, frontend `10abe49`), MongoDB local na porta 27019.                                                                                                                                        |
| **Verificação**   | Execuções registradas em [Testes](7%20-%20testes-relatorio-individual.md).                                                                                                                                                    |
| **Evidência**     | Em 27/11/2026, sobre backend `d1f1aeb` e frontend `10abe49`: script com MongoDB aprovado; `node --test` com 47 testes aprovados, 33 deles do relatório; `npx vitest run` com 17 arquivos e 246 testes aprovados. Em 26/11/2026, sobre backend `caca2d2` e frontend `6ff2d50`, as mesmas suítes também passaram (com 16 arquivos e 227 testes no `vitest`). |
| **Situação**      | Atendido em 27/11/2026.                                                                                                                                                                                                       |

## Rastreabilidade

RNF-REL-01 a RNF-REL-13 cobrem RF-REL-01 a RF-REL-05 e são verificados pela matriz CT-REL de [Testes](7%20-%20testes-relatorio-individual.md).
