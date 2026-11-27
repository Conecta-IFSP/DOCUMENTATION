# Testes — Atas de reunião

> Padrão da disciplina · Módulo ATA · Revisado em 26/11/2026

## Resumo

Registra o que foi executado, quando, em qual ambiente e qual foi o resultado. Os testes automatizados cobrem as regras do servidor; os roteiros manuais cobrem as telas, em três emuladores Android, no Expo Go e na versão web. Nada aqui está marcado como aprovado sem ter sido executado.

## Ambiente

| Item                | Valor                                                                                     |
| ------------------- | ------------------------------------------------------------------------------------------- |
| Data da execução    | 09/11/2026; CT-ATA-21 a CT-ATA-23 em 24/11/2026; CT-ATA-22 e CT-ATA-24 a CT-ATA-26 em 25/11/2026; CT-ATA-27 a CT-ATA-29 em 26/11/2026 |
| Node.js / npm       | 24.19.0 / 11.17.0                                                                           |
| Backend             | branch `homolog` com as alterações do Incremento 3, banco de testes local                  |
| Aplicativo          | branch `homolog`, APK de release gerado para `x86_64` (emuladores) e `arm64-v8a` (Aparelho 4), apontando para a API local |
| Aparelho 1          | Emulador Android 16 (API 36), 1080 × 2400, com PIN cadastrado                               |
| Aparelho 2          | Emulador Android 7.0 (API 24, versão mínima do aplicativo), 720 × 1280, sem bloqueio de tela |
| Aparelho 3          | Emulador Android 14 (API 34), 1080 × 2400, sem bloqueio de tela, com o APK de release e com o Expo Go 58.0.6 |
| Aparelho 4          | Galaxy S24, Android 16, com digital e PIN cadastrados, com o APK de release e com o Expo Go 58.0.5 |
| Web                 | `npx expo start`, aberto no navegador                                                       |
| iOS                 | Não executado: a equipe não dispõe de computador Apple nem de aparelho iOS                 |

## Evidência automatizada

| Data       | Projeto  | Comando                   | Resultado                                                  |
| ---------- | -------- | ------------------------- | ---------------------------------------------------------- |
| 09/11/2026 | BACKEND  | `npm test`                | 14 arquivos e 204 testes aprovados, nenhuma falha          |
| 09/11/2026 | BACKEND  | `npm run build`           | Aprovado                                                   |
| 09/11/2026 | BACKEND  | `npm run lint`            | Sem diagnóstico                                            |
| 09/11/2026 | FRONTEND | `npx tsc --noEmit`        | Sem erro                                                   |
| 09/11/2026 | FRONTEND | `npx eslint src`          | Sem diagnóstico                                            |
| 26/11/2026 | BACKEND  | `npx vitest run`          | 15 arquivos e 217 testes aprovados, nenhuma falha          |
| 26/11/2026 | BACKEND  | `npm run build` e `npm run lint` | Aprovado, sem diagnóstico                           |
| 26/11/2026 | FRONTEND | `npx tsc --noEmit` e `npm run lint` | Sem erro e sem diagnóstico                       |
| 26/11/2026 (revisão da documentação, backend `caca2d2`) | BACKEND | `npx vitest run src/reunioes/atas-presencas.spec.ts` e `npx vitest run` | 16 testes aprovados no arquivo das atas; 17 arquivos e 234 testes aprovados na suíte, nenhuma falha |

Em 26/11/2026, o arquivo `BACKEND/src/reunioes/atas-presencas.spec.ts` tem 16 testes. Os das atas estão descritos abaixo (CT-ATA-01 a CT-ATA-11, CT-ATA-27 e CT-ATA-28). CT-ATA-12 é o teste da lista de presença, que também aparece nos testes das reuniões como CT-REU-09; os dois da saída automática estão só nos testes das reuniões (CT-REU-29 e CT-REU-30).

## Matriz de testes automatizados

| ID        | Requisito            | Cenário e entrada                                                                                             | Resultado esperado                                                                                        | Teste                                                        |
| --------- | -------------------- | --------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------ |
| CT-ATA-01 | RF-ATA-01            | Salvar a primeira ata de uma reunião encerrada, com versão 0.                                                  | Grava com `ata: { $exists: false }` no filtro, versão 1, situação `RASCUNHO` e `avaliacoes` vazia.         | "salva ata versionada apenas pelo responsável…"              |
| CT-ATA-02 | RF-ATA-01, RNF-ATA-01 | Salvar como participante comum; consultar a lista de presença como pessoa de fora.                            | 403 nos dois casos.                                                                                        | "recusa participante comum como autor…"                      |
| CT-ATA-03 | RF-ATA-01            | Salvar antes do fim da reunião, com a reunião cancelada e com versão antiga.                                  | 400, 400 e 409, nessa ordem.                                                                               | "recusa edição antes do fim, cancelamento e versão obsoleta" |
| CT-ATA-04 | RF-ATA-02            | Enviar com prazo de 72 horas; depois tentar enviar sem nenhuma presença confirmada.                           | Situação `EM_APROVACAO`, aprovadores com os dois presentes, aprovação de quem enviou, prazo 72 h; depois 400. | "envia com prazo, congela os presentes…"                   |
| CT-ATA-05 | RF-ATA-03, RNF-ATA-05 | Aprovar com um dos dois presentes e depois com o segundo.                                                     | Primeiro segue `EM_APROVACAO` sem tocar no contador; o segundo gera `APROVADA`, número 7, ano de São Paulo e `$inc` com `upsert`. | "só aprova com todos e então numera a ata…"       |
| CT-ATA-06 | RF-ATA-04            | Reprovar apontando um trecho existente, sem justificativa; depois apontar um trecho que não está na ata.      | Situação `REPROVADA`, avaliação com trecho e sem justificativa; depois 400 "O trecho não faz parte da ata". | "reprova com justificativa opcional…"                        |
| CT-ATA-07 | RF-ATA-03, RF-ATA-04, RNF-ATA-02 | Repetir a mesma decisão; trocar a decisão; avaliar sem estar entre os aprovadores.                  | Repetição não grava nada; troca recebe 400; quem não é aprovador recebe 403.                              | "não deixa trocar a decisão nem avaliar…"                    |
| CT-ATA-08 | RF-ATA-05            | Editar a ata reprovada; refazer como participante comum; refazer como presidente; refazer uma ata em aprovação. | 400; 403; a versão vai para `atas_canceladas` com `CANCELADA` e nasce o rascunho na versão seguinte; 400.  | "cancela a ata reprovada e abre uma nova…"                   |
| CT-ATA-09 | RF-ATA-07            | Consultar as atas pendentes de um participante.                                                               | A consulta usa `$or` com aprovador sem avaliação e autor de ata reprovada.                                 | "lista como pendente a ata a avaliar…"                       |
| CT-ATA-10 | RF-ATA-02, RF-ATA-04 | Validar `EnviarAtaDto` com prazo de 5 horas; `ReprovarAtaDto` com justificativa em branco e com 501 caracteres. | Prazo inválido gera erro; justificativa em branco vira ausente e é aceita; 501 caracteres gera erro.       | "valida o prazo de envio e deixa a justificativa opcional"   |
| CT-ATA-11 | RNF-ATA-04           | Simular `findOneAndUpdate` devolvendo `null` ao salvar.                                                        | 409 "A ata mudou. Atualize antes de continuar."                                                            | "informa conflito quando outra escrita alterou a ata"        |
| CT-ATA-12 | RF-REU-09 (módulo Reuniões) | Gerar a lista de presença com um presente e um sem registro.                                                  | Nome, situação e minutos corretos; a ausência não cria presença.                                           | "gera lista com nomes, situações, duração…"                  |
| CT-ATA-13 | RF-ATA-01 a RF-ATA-05 | Percorrer o fluxo inteiro contra a API em execução, com MongoDB real: salvar, enviar, reprovar, refazer, corrigir, reenviar e aprovar. | Cada passo devolve a situação esperada; a ata termina `APROVADA`, numerada, com uma versão cancelada no histórico. | Roteiro executado pela API de teste (porta 3001).   |
| CT-ATA-27 | RF-ATA-03 | Rodar `aprovarAtasVencidas` um minuto depois do prazo de uma ata em aprovação em que só o presidente aprovou. | Busca reuniões `AGENDADA` com ata `EM_APROVACAO` e `prazo` até o horário informado e grava com `updateOne` filtrando pela ata lida. A ata fica `APROVADA`, com `aprovada_em` igual ao prazo, número 7 e ano 2026; o segundo presente recebe `APROVADA` com `em` igual ao prazo e `automatica` verdadeiro; a aprovação do presidente fica sem `automatica`; o contador é incrementado uma vez. | "aprova no fim do prazo a ata de quem não respondeu e numera" |
| CT-ATA-28 | RF-ATA-03, RF-ATA-04 | Aprovar e reprovar uma ata cujo prazo terminou há 1 segundo; depois aprovar com o prazo 1 minuto à frente. | 400 nas duas avaliações, sem gravar nada; com o prazo aberto, a aprovação é gravada. | "recusa avaliar a ata depois do prazo" |

## Matriz de testes manuais

| ID        | Requisito                | Aparelho            | Cenário                                                                                                | Resultado obtido                                                                                            |
| --------- | ------------------------ | ------------------- | -------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------- |
| CT-ATA-14 | RF-ATA-01, RF-ATA-02     | Android 16          | Abrir a ata de uma reunião encerrada, escrever o texto, salvar e enviar com prazo de 3 dias.           | Aprovado. A folha mostrou os presentes, o prazo calculado e o aviso de que enviar registra a própria aprovação. |
| CT-ATA-15 | RF-ATA-03, RF-ATA-04, RF-ATA-07 | Android 16   | Entrar com o segundo participante, ver a ata na tela inicial, apontar um parágrafo e reprovar com justificativa. | Aprovado. A ata ficou "Reprovada" e a faixa mostrou o nome, a data, a justificativa e o trecho.              |
| CT-ATA-16 | RF-ATA-05, RF-ATA-06     | Android 16          | Como presidente, criar a nova versão, corrigir o texto e reenviar; conferir, como aprovador, o quadro de mudanças. | Aprovado. A versão 1 ficou no histórico como cancelada e o quadro mostrou só o trecho alterado.             |
| CT-ATA-17 | RF-ATA-03, RNF-ATA-03, RNF-ATA-05 | Android 16 | Aprovar com o PIN do aparelho.                                                                         | Aprovado. A aprovação pediu a confirmação de identidade do aparelho e a ata virou "Ata nº 4/2026".          |
| CT-ATA-18 | RF-ATA-04, RNF-ATA-08    | Android 7.0, tela pequena | Abrir a ata, apontar um parágrafo, abrir a folha de reprovação com o teclado aberto e reprovar sem justificativa. | Aprovado. Nada ficou cortado e a ata registrou "Sem justificativa.".                                   |
| CT-ATA-19 | RNF-ATA-09               | Android 7.0         | Repetir consulta, comparação de versões, reprovação e aprovação com o aplicativo no tema escuro.       | Aprovado. Textos, faixas e o destaque verde e vermelho da comparação permaneceram legíveis.                 |
| CT-ATA-20 | RNF-ATA-03, RNF-ATA-08   | iOS                 | Repetir CT-ATA-14 a CT-ATA-17 em aparelho iOS.                                                         | Não executado: sem equipamento Apple disponível.                                                            |
| CT-ATA-21 | RF-ATA-06                | Web                 | Numa ata com uma versão reprovada, tocar em "Versão 1" em "Versões anteriores".                        | Aprovado. A folha mostrou o texto da versão 1, a faixa com quem reprovou e a justificativa, e o parágrafo apontado em vermelho. |
| CT-ATA-22 | RF-ATA-08                | Android 14, Expo Go | Tocar em "PDF da ata" numa ata aprovada.                                                              | Aprovado. Abriu a tela de impressão do sistema com a ata em uma página e a opção "Save as PDF".              |
| CT-ATA-23 | RF-ATA-08                | Web                 | Tocar em "PDF da ata" numa ata aprovada.                                                              | Aprovado. A nova janela recebeu só a ata, com o título "Ata nº 1/2026", e a impressão do navegador foi chamada. |
| CT-ATA-24 | RF-ATA-04, RNF-ATA-03    | Web                 | Apontar um parágrafo, escrever a justificativa e tocar em "Reprovar ata", sem bloqueio de tela.       | Aprovado. A reprovação foi enviada sem pedido de identidade e a faixa mostrou o nome, a data e a justificativa. |
| CT-ATA-25 | RF-ATA-04, RNF-ATA-03    | Aparelho 4, APK e Expo Go | Tocar em "Reprovar ata", cancelar o pedido de identidade e depois repetir confirmando com a digital. | Aprovado. Nos dois, o pedido de identidade do aparelho apareceu e, cancelado, a ata seguiu "Em aprovação" sem avaliação nova; no APK, confirmada a digital, a ata ficou "Reprovada" com a avaliação registrada. |
| CT-ATA-26 | RF-ATA-08                | Aparelho 3, APK     | Tocar em "PDF da ata" numa ata aprovada, fechar o menu de compartilhamento e repetir duas vezes.       | Aprovado. Nas três vezes abriu o menu de compartilhamento do Android com "Ata nº 1-2026.pdf" (40 KB), sem tela de escolha de pasta. Na primeira tentativa logo após instalar o APK, a tela ficou em "Gerando PDF..." até o aplicativo ser reaberto. |
| CT-ATA-29 | RF-ATA-03, RF-ATA-06, RF-ATA-08 | Aparelho 4, APK | Como presidente, abrir uma ata em aprovação com uma aprovação de 3; levar o prazo para 10 minutos antes no banco de testes; puxar a tela para baixo depois de 1 minuto e tocar em "PDF da ata". | Aprovado. Antes, a linha do prazo mostrou "Responder até 28/11/2026 às 16:41. Sem resposta, a aprovação é automática." em duas linhas. Depois, a ata virou "Ata nº 2/2026", "Aprovada", com os dois outros presentes em "Aprovada pelo fim do prazo em 26/11/2026 às 18:37"; o menu de compartilhamento abriu com "Ata nº 2-2026.pdf", e o documento termina com "Aprovada por Filipe Teste, Maria Teste (pelo fim do prazo) e Carlos Teste (pelo fim do prazo) em 26/11/2026.". |

## Rastreabilidade

CT-ATA-01 a CT-ATA-11 e CT-ATA-13 cobrem RF-ATA-01 a RF-ATA-05, RF-ATA-07 e RNF-ATA-04; CT-ATA-12 cobre a lista de presença (RF-REU-09); CT-ATA-14 a CT-ATA-19 cobrem as telas e os RNFs; CT-ATA-20 está pendente; CT-ATA-21 a CT-ATA-23 cobrem as versões anteriores e o PDF da ata; CT-ATA-24 e CT-ATA-25 cobrem a confirmação de identidade na reprovação; CT-ATA-26 cobre o PDF no aplicativo instalado; CT-ATA-27 e CT-ATA-28 cobrem a aprovação pelo fim do prazo e a recusa de avaliação depois dele; CT-ATA-29 cobre a mesma aprovação nas telas e no PDF.
