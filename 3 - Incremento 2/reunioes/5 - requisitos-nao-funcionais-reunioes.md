# Requisitos não funcionais — Reuniões

> Padrão da disciplina · Módulo REU · Revisado em 27/11/2026

## Resumo

Os requisitos não funcionais aparecem em dois níveis, como os funcionais. Os **de usuário** dizem, em uma frase, a qualidade esperada. Os **de sistema** completam cada um com a medida que o aprova, a condição em que vale, como foi verificado e onde está a evidência, de modo que qualquer pessoa consiga conferir. Onde ainda não houve medição, isso está escrito no campo Situação.

A classificação segue a árvore de Sommerville usada na disciplina: requisitos de **produto** (facilidade de uso, eficiência, com desempenho e espaço, confiabilidade e portabilidade), **organizacionais** (entrega, implementação e padrões) e **externos** (interoperabilidade, éticos e legais, com privacidade e segurança).

## Requisitos não funcionais de usuário

**RNF-REU-01 — Acesso identificado.** Só o aplicativo, com um usuário identificado, consegue usar as funções de reunião.

**RNF-REU-02 — Privacidade das reuniões.** Cada pessoa só vê as reuniões para as quais foi convidada, e só o responsável vê o link de presença e o histórico de correções.

**RNF-REU-03 — Acesso pela web.** A versão web só funciona a partir dos endereços autorizados.

**RNF-REU-04 — Proteção contra gravação simultânea.** Duas pessoas agindo ao mesmo tempo não podem apagar o registro uma da outra.

**RNF-REU-05 — Data e hora.** Datas e horários devem aparecer no horário do aparelho de quem consulta.

**RNF-REU-06 — Desempenho da agenda.** A agenda deve abrir em até 2 segundos.

**RNF-REU-07 — Portabilidade.** As reuniões devem funcionar em celulares Android a partir da versão 7.0 e em iOS.

**RNF-REU-08 — Atualização da tela.** O que outra pessoa altera deve aparecer na tela da reunião aberta em até 30 segundos, sem sair dela.

**RNF-REU-09 — Acessibilidade.** Os botões principais devem funcionar com leitor de tela.

**RNF-REU-10 — Testabilidade.** Os testes automatizados devem passar antes de cada entrega.

**RNF-REU-11 — Limite dos anexos.** Cada reunião aceita até 10 anexos de até 10 MB, dos tipos mais comuns de documento e imagem.

**RNF-REU-12 — Edição segura.** Editar a reunião não pode apagar respostas de convite nem presenças.

**RNF-REU-13 — Seletor de hora conhecido.** A hora deve ser escolhida no mesmo seletor que o aparelho usa nos outros aplicativos.

**RNF-REU-14 — Identidade na entrada e na saída.** Só quem consegue desbloquear o próprio aparelho pode marcar a entrada, a saída e a presença pelo link, e só quem agendou e desbloqueou o próprio aparelho pode desmarcar a presença de outra pessoa.

**RNF-REU-15 — Sigilo e validade do link de presença.** O link de presença não pode ser adivinhado, vale só durante a reunião e só para convidados identificados.

**RNF-REU-16 — Rastreabilidade das correções.** Toda correção de horário deve guardar quem corrigiu, quando, por quê e os valores anteriores.

## Requisitos não funcionais de sistema

### RNF-REU-01 — Acesso identificado

| Campo             | Conteúdo |
| ----------------- | -------- |
| **Classificação** | Externo › Legal › Segurança |
| **Critério**      | Toda rota de `/reunioes` e de `/presencas` exige a chave `X-API-Key`, um token JWT válido de conta ativa e uma conta de usuário comum. |
| **Medida**        | Sem chave ou sem token, a resposta é 401 e nada é lido nem gravado; com conta de administrador do sistema, 403 "O administrador do sistema não pode realizar esta operação". |
| **Condição**      | Qualquer rota do módulo. |
| **Verificação**   | CT-REU-13, com o roteiro `test/presencas-notificacoes-mongo.mjs` contra a API montada com os guards reais e um MongoDB local. |
| **Evidência**     | `ApiKeyGuard` (aplicado a toda a API em `app.module.ts`), `JwtAuthGuard` e `UsuarioComumGuard` em `ReunioesController` e `PresencasController`. |
| **Situação**      | Atendido em 26/11/2026. A chave fica no pacote do aplicativo e pode ser extraída dele; a autorização de cada usuário é feita pelo JWT. |

### RNF-REU-02 — Privacidade das reuniões

| Campo             | Conteúdo |
| ----------------- | -------- |
| **Classificação** | Externo › Legal › Privacidade |
| **Critério**      | A agenda só devolve reuniões em que o usuário está em `participantes`; a reunião, a lista de presença e os anexos só são entregues a participantes; o link e o histórico, só ao responsável da comissão. |
| **Medida**        | Pessoa de fora recebe 403 "Você não participa desta reunião"; membro comum recebe 403 no histórico de correções; `acesso_presenca` e `historico_presencas` nunca aparecem em `GET /reunioes` nem em `GET /reunioes/:id`. |
| **Condição**      | Consultas da agenda, da reunião, da lista, dos anexos e da gestão de presença. |
| **Verificação**   | CT-REU-02, CT-REU-03, CT-REU-09, CT-REU-22, CT-REU-33 e CT-REU-36. |
| **Evidência**     | `ReunioesService.listar`, `ReunioesService.carregar`, `ReunioesService.apresentar`, `PresencasService.carregar` e `select: false` nos dois campos do schema. |
| **Situação**      | Atendido em 26/11/2026. |

### RNF-REU-03 — Acesso pela web

| Campo             | Conteúdo |
| ----------------- | -------- |
| **Classificação** | Externo › Interoperabilidade |
| **Critério**      | O navegador só recebe permissão de CORS para as origens autorizadas. |
| **Medida**        | A resposta traz `Access-Control-Allow-Origin` apenas para `localhost`, `127.0.0.1`, o esquema `conectamais://` e o endereço da variável `FRONTEND_URL`; para outra origem, o navegador bloqueia a resposta. |
| **Condição**      | Requisição com cabeçalho `Origin`. |
| **Verificação**   | CT-REU-20. |
| **Evidência**     | `BACKEND/src/main.ts`, `app.enableCors`. |
| **Situação**      | Atendido segundo o registro da equipe de 23/10/2026 (CT-REU-20). |

### RNF-REU-04 — Proteção contra gravação simultânea

| Campo             | Conteúdo |
| ----------------- | -------- |
| **Classificação** | Produto › Confiabilidade |
| **Critério**      | Uma gravação feita a partir de um estado desatualizado não sobrescreve o dado novo. |
| **Medida**        | Toda gravação repete no filtro do `findOneAndUpdate` as condições conferidas na leitura; se o documento mudou, a resposta é 409 e nada é alterado. Duas entradas simultâneas pelo link resultam em uma gravada (201) e uma recusada (409). |
| **Condição**      | Convite, entrada, saída, confirmação de presença, edição, anexos, link e correção feitos ao mesmo tempo. |
| **Verificação**   | CT-REU-13, CT-REU-21 ("só grava se a reunião não mudou desde a leitura"), CT-REU-22 ("apaga o arquivo quando a reunião muda durante o envio") e CT-REU-35. |
| **Evidência**     | Filtros de gravação em `ReunioesService` e `PresencasService`. |
| **Situação**      | Atendido em 26/11/2026. |

### RNF-REU-05 — Data e hora

| Campo             | Conteúdo |
| ----------------- | -------- |
| **Classificação** | Organizacional › Padrões |
| **Critério**      | A API recebe e guarda instantes em UTC, no formato ISO 8601, e o aplicativo os mostra no fuso do aparelho. |
| **Medida**        | Uma reunião às 14:00 de Brasília aparece às 14:00 num aparelho em Brasília e às 13:00 num aparelho em Manaus. |
| **Condição**      | Agenda, reunião, presença e lista de presença. |
| **Verificação**   | Inspeção dos DTOs (`IsISO8601`) e de `formatarHora`, que usa as horas locais do aparelho; CT-REU-20. |
| **Evidência**     | `BACKEND/src/reunioes/dto/create-reuniao.dto.ts` e `FRONTEND/src/utils/reunioes.ts`. |
| **Situação**      | Atendido no código; teste em dois fusos registrado pela equipe em 23/10/2026 (CT-REU-20). |

### RNF-REU-06 — Desempenho da agenda

| Campo             | Conteúdo |
| ----------------- | -------- |
| **Classificação** | Produto › Eficiência › Desempenho |
| **Critério**      | A listagem da agenda responde rapidamente com o volume esperado de uma comissão. |
| **Medida**        | `GET /reunioes` responde em até 2 s com até 100 reuniões no período consultado. |
| **Condição**      | API e banco na mesma rede local, sem carga concorrente. |
| **Verificação**   | Medir o tempo da requisição com as ferramentas de rede do navegador ou com `curl -w "%{time_total}"`. |
| **Evidência**     | Índice `{ participantes.usuario_id: 1, inicio: 1 }` em `reuniao.schema.ts`. |
| **Situação**      | Critério definido; medição pendente. |

### RNF-REU-07 — Portabilidade

| Campo             | Conteúdo |
| ----------------- | -------- |
| **Classificação** | Produto › Portabilidade |
| **Critério**      | Os fluxos funcionam do Android 7.0 (API 24, o mínimo do aplicativo) ao Android 16, e no iOS. |
| **Medida**        | Agendar, editar, responder convite, marcar entrada e saída, anexar e gerar o PDF da lista sem erro e sem texto cortado. |
| **Condição**      | Emuladores Android 16, 14 e 7.0, Galaxy S24 com Android 16 e Expo Go. |
| **Verificação**   | Roteiros CT-REU-24 a CT-REU-28, CT-REU-31, CT-REU-32, CT-REU-42, CT-REU-44 e CT-REU-45. |
| **Evidência**     | Registros de execução de 09/11/2026 a 27/11/2026 em [Testes](7%20-%20testes-reunioes.md). |
| **Situação**      | Atendido no Android nos roteiros executados. O PDF da lista no aplicativo instalado foi conferido no Galaxy S24 em 26/11/2026 (CT-REU-32) e em 27/11/2026 (CT-REU-42), com versões anteriores ao frontend `db5b2fe`. A entrega atual do PDF, que confere se o aparelho compartilha arquivos e compartilha o arquivo gerado sem renomeá-lo, foi conferida em 27/11/2026 num emulador Android 14 com o APK do `4f53a25` (CT-REU-44, em parte: o conteúdo do PDF não foi conferido), e a folha "Convidados" com o teclado aberto foi aprovada no mesmo emulador (CT-REU-45). iOS não executado por falta de equipamento; validação pendente. |

### RNF-REU-08 — Atualização da tela

| Campo             | Conteúdo |
| ----------------- | -------- |
| **Classificação** | Produto › Facilidade de uso |
| **Critério**      | A tela da reunião busca os dados de novo sem ação do usuário. |
| **Medida**        | Enquanto a tela está em primeiro plano, `buscar` é chamada a cada 30.000 ms; uma alteração feita em outro aparelho aparece em até 30 s. Puxar a tela para baixo busca na hora e limpa as faixas de erro e de sucesso. |
| **Condição**      | Tela da reunião aberta, com conexão com a API. |
| **Verificação**   | Inspeção de `setInterval` em `[id].tsx`; roteiro CT-REU-17 em dois aparelhos; o roteiro CT-REU-43, executado em parte em 27/11/2026, conferiu que a faixa de sucesso some sozinha. |
| **Evidência**     | `FRONTEND/src/app/reunioes/[id].tsx`, `useFocusEffect`. |
| **Situação**      | Atendido na tela da reunião segundo o registro da equipe de 23/10/2026 (CT-REU-17). Limitação: a tela "Gestão de presença" (`FRONTEND/src/app/reunioes/acesso.tsx`) não se atualiza sozinha; a função `carregar` só roda quando a tela ganha o foco (`useFocusEffect`), quando é puxada para baixo e depois de uma correção de horário, sem consulta periódica. No CT-REU-38, essa tela, aberta antes do registro do convidado, só mostrou a saída dele depois de puxada para baixo. |

### RNF-REU-09 — Acessibilidade

| Campo             | Conteúdo |
| ----------------- | -------- |
| **Classificação** | Produto › Facilidade de uso |
| **Critério**      | Os botões de ícone das telas de reunião têm descrição para leitor de tela. |
| **Medida**        | Com TalkBack ou VoiceOver, os botões são anunciados com o nome da ação, por exemplo "Mais opções", "Confirmar presença de &lt;nome&gt;", "Recusar presença de &lt;nome&gt;", "Marcar &lt;nome&gt; como presente" e "Desmarcar presença de &lt;nome&gt;". |
| **Condição**      | Tela da reunião, agenda e aba "Presença". |
| **Verificação**   | Inspeção das propriedades `accessibilityLabel`; roteiro CT-REU-20 com leitor de tela. |
| **Evidência**     | `FRONTEND/src/app/reunioes/[id].tsx`. |
| **Situação**      | Registro da equipe de 23/10/2026 (CT-REU-20); contraste e área de toque sem medição. O rótulo "Desmarcar presença de &lt;nome&gt;", do frontend `db5b2fe`, foi conferido só no código, sem leitor de tela. |

### RNF-REU-10 — Testabilidade

| Campo             | Conteúdo |
| ----------------- | -------- |
| **Classificação** | Organizacional › Entrega |
| **Critério**      | As suítes automáticas dos dois projetos passam inteiras antes de cada entrega. |
| **Medida**        | BACKEND: `npx vitest run`, `npm run build` e `npm run lint` sem falha. FRONTEND: `npx tsc --noEmit`, `npm run lint` e `node --test tests/*.test.cjs` sem falha. |
| **Condição**      | Branch `homolog` dos dois projetos. |
| **Verificação**   | Execução registrada em [Testes](7%20-%20testes-reunioes.md). |
| **Evidência**     | 26/11/2026: BACKEND com 16 arquivos e 227 testes aprovados; FRONTEND com 5 arquivos e 47 testes aprovados. 27/11/2026, backend `d1f1aeb` e frontend `10abe49`: BACKEND com 17 arquivos e 246 testes aprovados, `npm run lint` sem diagnóstico e `npx tsc -p tsconfig.build.json --noEmit` sem erro; FRONTEND com 5 arquivos e 47 testes aprovados, `npx tsc --noEmit` e `npm run lint` sem diagnóstico. |
| **Situação**      | Atendido em 26/11/2026. Em 27/11/2026, repetido sobre a nova base, com o `npm run build` substituído pela compilação sem gerar arquivos (`tsc --noEmit` com a configuração do build). |

### RNF-REU-11 — Limite dos anexos

| Campo             | Conteúdo |
| ----------------- | -------- |
| **Classificação** | Produto › Eficiência › Espaço |
| **Critério**      | O espaço ocupado pelos anexos de uma reunião é limitado. |
| **Medida**        | No máximo 10 anexos por reunião, de até 10.485.760 bytes cada (no máximo 100 MB por reunião), só nos 11 tipos de `TIPOS_ANEXO`. Acima disso, 400 ou 413 e nada é gravado. |
| **Condição**      | Envio pela aba "Pauta" ou pelo formulário da reunião. |
| **Verificação**   | CT-REU-22 (`anexos.spec.ts`) e CT-REU-25. |
| **Evidência**     | `BACKEND/src/reunioes/anexos.ts` e `FRONTEND/src/services/anexos.ts`. |
| **Situação**      | Atendido em 09/11/2026. |

### RNF-REU-12 — Edição segura

| Campo             | Conteúdo |
| ----------------- | -------- |
| **Classificação** | Produto › Confiabilidade |
| **Critério**      | A edição nunca apaga uma resposta de convite ou uma presença registrada entre a leitura e a gravação. |
| **Medida**        | A gravação da edição filtra por `atualizado_em` igual ao lido; se outra gravação aconteceu, a resposta é 409 "A reunião mudou. Atualize a tela." e nada muda. |
| **Condição**      | Edição simultânea a uma resposta de convite ou a uma entrada. |
| **Verificação**   | CT-REU-21, teste "só grava se a reunião não mudou desde a leitura". |
| **Evidência**     | `ReunioesService.editar`. |
| **Situação**      | Atendido em 09/11/2026. |

### RNF-REU-13 — Seletor de hora conhecido

| Campo             | Conteúdo |
| ----------------- | -------- |
| **Classificação** | Produto › Facilidade de uso |
| **Critério**      | Os campos "Início" e "Término" abrem o seletor de hora do próprio sistema, em formato de 24 horas, já na hora sugerida. |
| **Medida**        | No Android, abre o relógio do sistema; no iOS, a roda, dentro de uma folha com o botão "Confirmar". O início sugerido é a próxima meia hora, quando a data é hoje, ou 08:00, em outro dia; o término, uma hora depois do início. Enquanto nada foi escolhido, o campo mostra "Escolher". |
| **Condição**      | Formulário de nova reunião e de edição. |
| **Verificação**   | CT-REU-14 (`horarios.test.cjs`) e roteiro CT-REU-26. |
| **Evidência**     | `FRONTEND/src/components/SeletorHora.tsx` e `FRONTEND/src/utils/sugestoes-horario.ts`. |
| **Situação**      | Atendido no Android em 10/11/2026. O diálogo é desenhado pelo sistema e segue a aparência e o idioma do aparelho, não o tema do aplicativo; no Android 7 ele não tem o modo de digitação. iOS pendente, sem equipamento. |

### RNF-REU-14 — Identidade na entrada e na saída

| Campo             | Conteúdo |
| ----------------- | -------- |
| **Classificação** | Externo › Legal › Segurança |
| **Critério**      | Marcar entrada ou saída na tela da reunião, marcar entrada pelo cartão "Agora" da tela inicial, registrar presença pelo link, corrigir horários e desmarcar a presença de quem está confirmado chamam `autenticarComDispositivo` antes da requisição; sem confirmação, nada é enviado. |
| **Medida**        | Em aparelho com biometria, Face ID ou PIN, o pedido do próprio aparelho aparece e, cancelado, a tela mostra o aviso de identidade sem gravar nada. O pedido não tem texto do aplicativo: o título vai com um espaço porque o `androidx.biometric` recusa título vazio. |
| **Condição**      | Aparelho com bloqueio de tela configurado. |
| **Verificação**   | Roteiro CT-REU-27 com PIN de tela, na tela da reunião, roteiro CT-REU-40, na tela inicial, roteiro CT-REU-38, no link e na correção, e roteiro CT-REU-43, na desmarcação de presença. |
| **Evidência**     | `FRONTEND/src/services/autenticacao-local.ts`, `TelaReuniao.registrar`, `TelaReuniao.desmarcarPresenca`, `TelaInicio.marcarEntrada`, `TelaPresencaLink.registrar` e `TelaGestaoPresenca.corrigir`. |
| **Situação**      | Atendido no Android em 10/11/2026 na tela da reunião. Na tela inicial, o roteiro CT-REU-40 foi executado em parte em 26/11/2026: o pedido do aparelho apareceu e, cancelado, a tela mostrou o aviso sem gravar a entrada; falta a confirmação com a digital. Limitações: em aparelho sem bloqueio e na web, a função devolve verdadeiro e a marcação segue sem confirmação; na tela da reunião, `registrar` e `desmarcarPresenca` não tratam um erro lançado pelo pedido do aparelho, que não envia nada mas também não mostra aviso, enquanto a tela inicial trata esse erro como identidade não confirmada. A desmarcação de presença, do frontend `db5b2fe`, foi executada em parte em 27/11/2026 num emulador Android 14 sem bloqueio de tela (CT-REU-43): o pedido do aparelho não aparece nesse emulador, então o cancelamento da biometria não foi conferido. O link e a correção foram executados em 27/11/2026 (CT-REU-38) num emulador Android 14 sem bloqueio de tela, em que o pedido do aparelho não aparece; nessas duas telas, o pedido de identidade não foi conferido em aparelho com bloqueio. |

### RNF-REU-15 — Sigilo e validade do link de presença

| Campo             | Conteúdo |
| ----------------- | -------- |
| **Classificação** | Externo › Legal › Segurança |
| **Critério**      | O código do link é imprevisível, não fica guardado em texto no banco e só é aceito durante a reunião, para participantes autenticados com convite aceito. |
| **Medida**        | Código de 32 bytes sorteados (64 caracteres hexadecimais); o banco guarda só o resumo SHA-256; o link é recusado com 400 depois do término, depois de revogado, depois de um novo link, quando o horário da reunião muda e quando a comissão troca de responsável; pessoa de fora da reunião recebe 403; abrir o link não registra presença. |
| **Condição**      | Rotas `GET` e `POST /presencas/:id/acesso` e `POST /presencas/:id/enviar`. |
| **Verificação**   | CT-REU-33, CT-REU-35 e CT-REU-37; no aparelho, CT-REU-38. |
| **Evidência**     | `PresencasService.gerar`, `PresencasService.validarLink` e `PresencasService.registrar`. |
| **Situação**      | Atendido em 26/11/2026 pelo roteiro automatizado. No aparelho, CT-REU-38 aprovado em 27/11/2026: depois de revogado, o link antigo, aberto no Galaxy S24, mostrou "Link inválido, revogado ou expirado. Solicite um novo ao responsável.", e a notificação do convidado deixou de aparecer. |

### RNF-REU-16 — Rastreabilidade das correções

| Campo             | Conteúdo |
| ----------------- | -------- |
| **Classificação** | Externo › Legal › Segurança |
| **Critério**      | Toda correção de horário deixa um registro permanente. |
| **Medida**        | Cada correção acrescenta a `historico_presencas` um item com participante, responsável, horário, motivo de 5 a 500 caracteres e as presenças antes e depois, no mesmo comando que altera a presença; nenhuma rota altera ou apaga esses itens. |
| **Condição**      | Rota `PATCH /presencas/:id/horarios/:usuarioId`. |
| **Verificação**   | CT-REU-36; no aparelho, CT-REU-38. |
| **Evidência**     | `PresencasService.corrigir` e `PresencasService.historico`. |
| **Situação**      | Atendido em 26/11/2026 pelo roteiro automatizado. No aparelho, CT-REU-38 aprovado em 27/11/2026: a correção da saída de 10:34 para 10:36, com o motivo "Saiu depois do registro", apareceu em "Histórico de correções" com os valores de antes e de depois. |

## Rastreabilidade

RNF-REU-01 a RNF-REU-16 cobrem RF-REU-01 a RF-REU-09 e RF-REU-13 a RF-REU-22 e são verificados pela matriz CT-REU. Os requisitos não funcionais da ata estão em [RNF-ATA](../../4%20-%20Incremento%203/atas/5%20-%20requisitos-nao-funcionais-atas.md).
