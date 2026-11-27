# Requisitos não funcionais — Atas de reunião

> Padrão da disciplina · Módulo ATA · Revisado em 27/11/2026

## Resumo

Os requisitos não funcionais aparecem em dois níveis, como os funcionais. Os **de usuário** dizem, em uma frase, a qualidade esperada. Os **de sistema** completam cada um com a medida que o aprova, a condição em que vale, como foi verificado e onde está a evidência, de modo que qualquer pessoa consiga conferir. Onde ainda não houve medição, isso está escrito.

A classificação segue a árvore de Sommerville usada na disciplina: requisitos de **produto** (facilidade de uso, eficiência, confiabilidade, portabilidade), **organizacionais** (entrega, implementação, padrões) e **externos** (interoperabilidade, éticos, legais).

## Requisitos não funcionais de usuário

**RNF-ATA-01 — Autorização para escrever a ata.** Só o presidente da comissão pode escrever, enviar e refazer a ata.

**RNF-ATA-02 — Autorização para avaliar a ata.** Só quem esteve presente na reunião pode aprovar ou reprovar a ata, e uma única vez por versão.

**RNF-ATA-03 — Confirmação de identidade na avaliação.** A aprovação e a reprovação da ata devem ser confirmadas pela biometria ou senha do aparelho.

**RNF-ATA-04 — Proteção contra gravação simultânea.** Duas pessoas agindo ao mesmo tempo não podem apagar a decisão uma da outra.

**RNF-ATA-05 — Numeração única por comissão e ano.** Duas atas da mesma comissão não podem receber o mesmo número no mesmo ano.

**RNF-ATA-06 — Data e hora.** Datas e horários devem aparecer no horário do aparelho de quem consulta.

**RNF-ATA-07 — Desempenho da comparação de versões.** Mostrar o que mudou entre duas versões não pode travar a tela.

**RNF-ATA-08 — Portabilidade.** As atas devem funcionar em celulares Android a partir da versão 7.0 e em iOS.

**RNF-ATA-09 — Legibilidade nos dois temas.** As telas das atas devem ser legíveis no tema claro e no tema escuro.

**RNF-ATA-10 — Testabilidade.** Os testes automatizados devem passar antes de cada entrega.

## Requisitos não funcionais de sistema

### RNF-ATA-01 — Autorização para escrever a ata

| Campo           | Conteúdo                                                                                                                                     |
| --------------- | --------------------------------------------------------------------------------------------------------------------------------------------- |
| **Classificação** | Externo › Legal › Segurança |
| **Critério**    | Só o membro da comissão com papel `RESPONSAVEL` grava, envia ou refaz a ata.                                                                  |
| **Medida**      | Outro participante da reunião recebe HTTP 403 com a mensagem "Apenas o responsável da comissão pode escrever a ata"; quem não participa recebe 403 "Você não participa desta reunião", antes dessa conferência. Nos dois casos nada é gravado. |
| **Condição**    | Rotas `PATCH /reunioes/:id/ata`, `POST /reunioes/:id/ata/publicar` e `POST /reunioes/:id/ata/nova`.                                           |
| **Verificação** | Testes automatizados CT-ATA-02 e CT-ATA-08.                                                                                                   |
| **Evidência**   | `ReunioesService.autorizarResponsavel` e `BACKEND/src/reunioes/atas-presencas.spec.ts`.                                                       |
| **Situação**    | Atendido em 09/11/2026.                                                                                                                       |

### RNF-ATA-02 — Autorização para avaliar a ata

| Campo           | Conteúdo                                                                                                                            |
| --------------- | ------------------------------------------------------------------------------------------------------------------------------------- |
| **Classificação** | Externo › Legal › Segurança |
| **Critério**    | Só aprova ou reprova quem está em `ata.aprovadores`, uma única vez por versão.                                                      |
| **Medida**      | Quem não está na lista recebe 403; quem já avaliou recebe a mesma reunião, sem gravação nova (repetir a mesma decisão) ou 400 (trocar a decisão). |
| **Condição**    | Rotas `POST /reunioes/:id/ata/aprovar` e `POST /reunioes/:id/ata/reprovar`.                                                         |
| **Verificação** | Testes CT-ATA-05, CT-ATA-06 e CT-ATA-07.                                                                                            |
| **Evidência**   | `ReunioesService.avaliarAta` e o teste "não deixa trocar a decisão nem avaliar quem não esteve presente".                            |
| **Situação**    | Atendido em 09/11/2026.                                                                                                             |

### RNF-ATA-03 — Confirmação de identidade na avaliação

| Campo           | Conteúdo                                                                                                                                                |
| --------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Classificação** | Externo › Legal › Segurança |
| **Critério**    | A aprovação e a reprovação exigem uma confirmação deliberada: biometria, Face ID ou PIN do aparelho; onde não houver bloqueio de tela, a aprovação mostra um diálogo de confirmação e a reprovação é confirmada pelo botão "Reprovar ata" da folha de reprovação. |
| **Medida**      | Em aparelho com bloqueio, cancelar a confirmação não envia nada à API. Em aparelho sem bloqueio, o diálogo aparece antes do envio.                      |
| **Condição**    | Botão "Aprovar" da aba Ata, botão "Reprovar ata" da folha de reprovação e botão "Enviar" quando o presidente esteve presente.                            |
| **Verificação** | Testes manuais CT-ATA-17 e CT-ATA-15 no emulador Android 16, com PIN cadastrado, CT-ATA-18 no Android 7.0 e CT-ATA-24 na web, sem bloqueio de tela, e CT-ATA-25 num Galaxy S24 com digital. |
| **Evidência**   | `FRONTEND/src/services/autenticacao-local.ts`, `TelaRegistrosReuniao.aprovar` e `TelaRegistrosReuniao.reprovar`.                                        |
| **Situação**    | Atendido no Android: aprovação com PIN no emulador e reprovação com a digital num Galaxy S24 (CT-ATA-25). No iOS o mesmo módulo é usado para Face ID, mas não houve execução em aparelho iOS; validação pendente. |

### RNF-ATA-04 — Proteção contra gravação simultânea

| Campo           | Conteúdo                                                                                                                                      |
| --------------- | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| **Classificação** | Produto › Confiabilidade |
| **Critério**    | Duas pessoas agindo ao mesmo tempo nunca sobrescrevem a decisão uma da outra.                                                                  |
| **Medida**      | Toda gravação usa `_id`, `situacao = AGENDADA` e a ata lida como filtro do `findOneAndUpdate`; salvar, enviar, aprovar e reprovar passam por `gravarAta`, que filtra também pelo `presencas_revisao` lido. Se o documento já mudou, a resposta é 409 "A ata mudou. Atualize antes de continuar." e nada é alterado. A aprovação pelo fim do prazo usa `updateOne` com o mesmo tipo de filtro e, sem documento encontrado, apenas não grava. |
| **Condição**    | Salvar, enviar, aprovar, reprovar e criar nova versão.                                                                                        |
| **Verificação** | Teste CT-ATA-11, que simula o `findOneAndUpdate` devolvendo `null`.                                                                           |
| **Evidência**   | `ReunioesService.gravarAta` e o teste "informa conflito quando outra escrita alterou a ata".                                                   |
| **Situação**    | Atendido em 09/11/2026.                                                                                                                       |

### RNF-ATA-05 — Numeração única por comissão e ano

| Campo           | Conteúdo                                                                                                                                        |
| --------------- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Classificação** | Produto › Confiabilidade |
| **Critério**    | Duas atas da mesma comissão, no mesmo ano, nunca recebem o mesmo número.                                                                        |
| **Medida**      | O número vem de `findOneAndUpdate` com `$inc` na coleção `contadores`, operação atômica do MongoDB. A sequência começa em 1 e sobe de 1 em 1.   |
| **Condição**    | Momento em que a última aprovação da ata chega.                                                                                                 |
| **Verificação** | Teste CT-ATA-05 confere o filtro, o `$inc` e o `upsert`; o roteiro CT-ATA-17 mostrou a sequência 2/2026, 3/2026 e 4/2026 no emulador.            |
| **Evidência**   | `ReunioesService.proximoNumeroAta` e `BACKEND/src/reunioes/schemas/contador.schema.ts`.                                                          |
| **Situação**    | Atendido em 09/11/2026. Limitação: o número é reservado antes da gravação da ata; se ela falhar por conflito (409) ou se a aprovação pelo fim do prazo não encontrar a ata como foi lida, o número fica sem ata e a sequência visível pula um valor. A unicidade não é afetada. |

### RNF-ATA-06 — Data e hora

| Campo           | Conteúdo                                                                                                                                       |
| --------------- | ------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Classificação** | Organizacional › Padrões |
| **Critério**    | A API guarda instantes em UTC e o aplicativo mostra no fuso do aparelho. O ano do número da ata usa sempre o fuso `America/Sao_Paulo`.          |
| **Medida**      | Uma aprovação feita em 31/12 às 22h no horário de Brasília recebe o ano daquele ano, não do seguinte, mesmo com o servidor em UTC.              |
| **Condição**    | Envio, aprovação, reprovação e exibição de prazos.                                                                                              |
| **Verificação** | Inspeção de `anoEmSaoPaulo` e teste CT-ATA-05, que calcula o ano esperado com o mesmo fuso. Virada de ano em aparelho real: medição pendente.   |
| **Evidência**   | `ReunioesService` (função `anoEmSaoPaulo`) e `FRONTEND/src/utils/reunioes.ts` (`formatarDataHora`).                                             |
| **Situação**    | Critério atendido no código; teste na virada do ano pendente.                                                                                    |

### RNF-ATA-07 — Desempenho da comparação de versões

| Campo           | Conteúdo                                                                                                                                                        |
| --------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Classificação** | Produto › Eficiência › Desempenho |
| **Critério**    | Mostrar o que mudou entre duas versões não pode travar a tela em aparelho antigo.                                                                                |
| **Medida**      | A comparação palavra a palavra só é feita quando o produto do número de pedaços dos dois parágrafos (palavras, espaços e sinais de pontuação, separados por `separar`) é de até 250.000; acima disso o parágrafo é mostrado inteiro como trocado. A comparação entre os parágrafos das duas versões, feita antes, não tem esse limite. |
| **Condição**    | Ata em aprovação com uma versão cancelada anterior, texto de até 20.000 caracteres.                                                                              |
| **Verificação** | Teste manual CT-ATA-16 no emulador Android 7.0 (720 × 1280, 2 GB de RAM): a tela abriu sem travar.                                                               |
| **Evidência**   | `FRONTEND/src/utils/diferenca.ts`, constante `LIMITE_COMPARACAO`.                                                                                                |
| **Situação**    | Atendido no caso testado; sem medição de tempo em milissegundos.                                                                                                 |

### RNF-ATA-08 — Portabilidade

| Campo           | Conteúdo                                                                                                                                         |
| --------------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Classificação** | Produto › Portabilidade |
| **Critério**    | O fluxo completo funciona do Android 7.0 (API 24, o mínimo do aplicativo) ao Android 16, e no iOS.                                                |
| **Medida**      | Escrever, enviar, aprovar, reprovar, criar nova versão e gerar o PDF, sem erro e sem texto cortado.                                              |
| **Condição**    | Emuladores Android 16 (1080 × 2400), Android 14 e Android 7.0 (720 × 1280), e Galaxy S24 com Android 16.                                         |
| **Verificação** | Roteiros CT-ATA-14 a CT-ATA-19; para o PDF no aplicativo instalado, CT-ATA-26, CT-ATA-29 e CT-ATA-30.                                            |
| **Evidência**   | Capturas de tela da execução de 09/11/2026; registros de 25/11, 26/11 e 27/11/2026 em [Testes](7%20-%20testes-atas.md).                          |
| **Situação**    | Atendido no Android. iOS não executado por falta de equipamento; validação pendente. O PDF no aplicativo instalado foi conferido em CT-ATA-26 e CT-ATA-29, antes do frontend `795d49b`, e em CT-ATA-30, em 27/11/2026 no Galaxy S24, nos frontends `10abe49` e `8cf4839`; neste último, a versão atual, `nomearPdf` espera o `move` antes do compartilhamento, e nos dois o menu abriu com "Ata nº 1-2026.pdf", sem erro no log. |

### RNF-ATA-09 — Legibilidade nos dois temas

| Campo           | Conteúdo                                                                                                                                  |
| --------------- | --------------------------------------------------------------------------------------------------------------------------------------------- |
| **Classificação** | Produto › Facilidade de uso |
| **Critério**    | Todas as telas das atas funcionam no tema claro e no escuro, sem cor fixa no código.                                                       |
| **Medida**      | Nenhuma cor literal nos componentes das atas: todas vêm de `useTema().tema.cores`. O texto novo e o removido da comparação continuam legíveis nos dois temas. |
| **Condição**    | Tema do aplicativo em Claro e em Escuro.                                                                                                   |
| **Verificação** | Inspeção de `registros.tsx` e roteiro CT-ATA-19 no Android 7.0, nos dois temas.                                                            |
| **Evidência**   | Capturas de tela nos dois temas, 09/11/2026. O PDF é exceção proposital: usa papel branco e texto escuro, por ser documento para impressão. |
| **Situação**    | Atendido em 09/11/2026.                                                                                                                    |

### RNF-ATA-10 — Testabilidade

| Campo           | Conteúdo                                                                                                       |
| --------------- | ---------------------------------------------------------------------------------------------------------------- |
| **Classificação** | Organizacional › Entrega |
| **Critério**    | A suíte do backend precisa passar inteira antes de cada entrega.                                                |
| **Medida**      | `npm test` sem nenhuma falha, incluindo os testes das atas.                                                     |
| **Condição**    | Código da branch `homolog` com as alterações do Incremento 3.                                                   |
| **Verificação** | Execução registrada em [Testes](7%20-%20testes-atas.md).                                                        |
| **Evidência**   | 14 arquivos e 204 testes aprovados em 09/11/2026; na revisão de 26/11/2026, 17 arquivos e 234 testes aprovados com `npx vitest run`; na revisão de 27/11/2026 (backend `d1f1aeb`), 17 arquivos e 246 testes aprovados. |
| **Situação**    | Atendido em 09/11/2026; última execução sem falha em 27/11/2026.                                                |

## Rastreabilidade

RNF-ATA-01 a RNF-ATA-10 cobrem RF-ATA-01 a RF-ATA-08 e são verificados pela matriz CT-ATA.
