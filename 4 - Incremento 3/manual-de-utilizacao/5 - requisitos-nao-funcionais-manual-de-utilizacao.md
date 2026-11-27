# Requisitos não funcionais — Manual de utilização

> Padrão da disciplina · Módulo MAN · Revisado em 27/11/2026

## Resumo

Os requisitos não funcionais aparecem em dois níveis, como os funcionais. Os **de usuário** dizem, em uma frase, a qualidade esperada do manual de utilização: a tela Ajuda e o manual completo em PDF. Os **de sistema** completam cada um com a medida que o aprova, a condição em que vale, como é verificado e onde está a evidência, de modo que qualquer pessoa consiga conferir. Onde ainda não houve medição ou execução no aparelho, isso está escrito no campo Situação.

A classificação segue a árvore de Sommerville usada na disciplina: requisitos de **produto** (facilidade de uso, eficiência, com desempenho e espaço, confiabilidade e portabilidade), **organizacionais** (entrega, implementação e padrões) e **externos** (interoperabilidade, éticos e legais, com privacidade e segurança).

| Código     | Classificação                     | Requisito                                     | Aplica-se a            |
| ---------- | --------------------------------- | --------------------------------------------- | ---------------------- |
| RNF-MAN-01 | Produto › Facilidade de uso       | Acesso sem login                              | RF-MAN-01 e RF-MAN-02  |
| RNF-MAN-02 | Produto › Confiabilidade          | Funcionamento sem internet                    | RF-MAN-01 e RF-MAN-02  |
| RNF-MAN-03 | Organizacional › Entrega          | Um único arquivo do manual                    | RF-MAN-02              |
| RNF-MAN-04 | Organizacional › Padrões          | Manual em formato A4                          | RF-MAN-02              |
| RNF-MAN-05 | Produto › Facilidade de uso       | Linguagem simples                             | RF-MAN-01 e RF-MAN-02  |
| RNF-MAN-06 | Produto › Facilidade de uso       | Organização por funcionalidade e por perfil   | RF-MAN-01 e RF-MAN-02  |
| RNF-MAN-07 | Produto › Portabilidade           | Disponibilidade em Android e iOS              | RF-MAN-01 e RF-MAN-02  |
| RNF-MAN-08 | Organizacional › Implementação    | Versão do Expo e sistemas suportados          | RF-MAN-01 e RF-MAN-02  |
| RNF-MAN-09 | Organizacional › Implementação    | Manutenção do conteúdo da Ajuda               | RF-MAN-01              |
| RNF-MAN-10 | Produto › Eficiência › Desempenho | Exibição do manual sem travar o aparelho      | RF-MAN-02              |

## Requisitos não funcionais de usuário

**RNF-MAN-01 — Acesso sem login.** A Ajuda e o manual devem abrir para quem ainda não entrou na conta, para que a pessoa consiga tirar dúvidas sobre como entrar ou recuperar a senha.

**RNF-MAN-02 — Funcionamento sem internet.** As perguntas da Ajuda e o manual em PDF devem funcionar com o aparelho sem conexão.

**RNF-MAN-03 — Um único arquivo do manual.** O manual mostrado no aplicativo e o publicado na documentação devem ser o mesmo arquivo.

**RNF-MAN-04 — Manual em formato A4.** O manual em PDF deve poder ser lido na tela e impresso em folha A4 sem cortes.

**RNF-MAN-05 — Linguagem simples.** A Ajuda e o manual devem usar respostas curtas, uma ação por passo e os mesmos nomes de botões e telas que aparecem no aplicativo.

**RNF-MAN-06 — Organização por funcionalidade e por perfil.** O conteúdo deve ser agrupado por funcionalidade e avisar quando uma orientação vale só para um perfil.

**RNF-MAN-07 — Disponibilidade em Android e iOS.** A Ajuda e o manual devem funcionar em celulares Android e em iPhones.

**RNF-MAN-08 — Versão do Expo e sistemas suportados.** O aplicativo deve usar o Expo SDK 58 e funcionar a partir do Android 7.0 e do iOS 16.4.

**RNF-MAN-09 — Manutenção do conteúdo da Ajuda.** Incluir, alterar ou remover uma pergunta deve exigir mudança em um único arquivo.

**RNF-MAN-10 — Exibição do manual sem travar o aparelho.** Abrir as 27 páginas do manual não pode travar nem fechar o aplicativo, mesmo em aparelho com pouca memória.

## Requisitos não funcionais de sistema

### RNF-MAN-01 — Acesso sem login

| Campo             | Conteúdo |
| ----------------- | -------- |
| **Classificação** | Produto › Facilidade de uso |
| **Critério**      | As telas Ajuda e "Manual de utilização" abrem sem sessão ativa e mostram o mesmo conteúdo que mostram a quem fez login. |
| **Medida**        | Sem sessão ativa, o toque em "Precisa de ajuda?" abre a tela Ajuda sem redirecionar para a tela Entrar, com os 7 tópicos, a pergunta "Esqueci minha senha. O que faço?" já aberta e o botão "Ver manual completo (PDF)", que abre o manual. Com sessão ativa, a mesma tela abre por Perfil → Suporte → Ajuda, com todas as perguntas fechadas. |
| **Condição**      | Aplicativo instalado, nas duas situações: sem sessão ativa e com sessão ativa. |
| **Verificação**   | Inspeção de `src/app/_layout.tsx` e roteiros CT-MAN-01, CT-MAN-02 e CT-MAN-17. |
| **Evidência**     | As rotas `ajuda` e `manual` são declaradas fora dos grupos `Stack.Protected` em `src/app/_layout.tsx`. O link "Precisa de ajuda?" está em `src/app/login/index.tsx`, e a linha "Ajuda", em `src/app/perfil.tsx`. |
| **Situação**      | Ajuda: atendido em 13/11/2026, por inspeção e pelos roteiros CT-MAN-01 e CT-MAN-02, aprovados em iPhone 14 (iOS 27.2), via Expo Go. Manual: atendido por inspeção em 26/11/2026 e pelo roteiro CT-MAN-17, aprovado em 27/11/2026 no emulador Android 14 (API 34) com o APK release; no mesmo dia, CT-MAN-01 foi aprovado de novo nesse emulador. |

### RNF-MAN-02 — Funcionamento sem internet

| Campo             | Conteúdo |
| ----------------- | -------- |
| **Classificação** | Produto › Confiabilidade |
| **Critério**      | As perguntas, as respostas, o arquivo do manual e a biblioteca que o desenha fazem parte do aplicativo instalado; as telas Ajuda e "Manual de utilização" não enviam nenhuma requisição à rede. |
| **Medida**        | Com o aparelho em modo avião, a tela Ajuda abre e exibe os 7 tópicos e as 42 perguntas, as respostas abrem e fecham e a busca filtra as perguntas sem mensagem de erro de conexão; o manual abre e mostra as 27 páginas. |
| **Condição**      | Aparelho em modo avião, com ou sem login. |
| **Verificação**   | Roteiros CT-MAN-07 e CT-MAN-15. Inspeção: `src/app/ajuda.tsx`, `src/utils/busca-ajuda.ts`, `src/app/manual.tsx` e `src/components/ManualPdf.tsx` não importam o módulo `api`, e `ManualPdf` só lê recursos locais com `expo-asset` e `expo-file-system`. |
| **Evidência**     | Constante `TOPICOS_AJUDA`, em `src/constants/ajuda.ts`; recursos `assets/manual/manual.pdf` e `assets/pdfjs/`; `metro.config.js`. Fora das duas telas, com sessão ativa, o layout raiz (`src/app/_layout.tsx`) chama `atualizar()` da sessão a cada troca de rota, inclusive ao abrir a Ajuda e o manual, o que envia `GET /usuarios/me`; sem internet, essa falha fica só no estado da sessão e não aparece nas duas telas. |
| **Situação**      | Ajuda: atendido em 13/11/2026, por inspeção e pelo roteiro CT-MAN-07, aprovado em iPhone em modo avião, via Expo Go. Manual: atendido por inspeção em 26/11/2026. Em 27/11/2026, no emulador Android 14 (API 34) em modo avião, com o APK release, o manual mostrou "Abrindo o manual…", a pílula "Carregando página N de 27" e as 27 páginas até o "Glossário", sem mensagem de erro (CT-MAN-15 aprovado). |

### RNF-MAN-03 — Um único arquivo do manual

| Campo             | Conteúdo |
| ----------------- | -------- |
| **Classificação** | Organizacional › Entrega |
| **Critério**      | O PDF que vai no pacote do aplicativo e a cópia publicada na documentação são o mesmo arquivo. Uma nova versão do manual é entregue substituindo `assets/manual/manual.pdf` e publicando nova versão do aplicativo; a cópia da documentação é substituída na mesma entrega. |
| **Medida**        | O resumo SHA-1 de `FRONTEND/assets/manual/manual.pdf` é igual ao de `manual-usuario/manual.pdf`. |
| **Condição**      | FRONTEND e DOCUMENTATION na branch `homolog`. |
| **Verificação**   | `sha1sum` dos dois arquivos (CT-MAN-14). |
| **Evidência**     | Os dois arquivos, com o resumo `bc080ba7c6e2c3cb30a49a58ebbb9f91642b758e` em 26/11/2026 e em 27/11/2026. |
| **Situação**      | Atendido em 26/11/2026 e conferido de novo em 27/11/2026. |

### RNF-MAN-04 — Manual em formato A4

| Campo             | Conteúdo |
| ----------------- | -------- |
| **Classificação** | Organizacional › Padrões |
| **Critério**      | O manual em PDF é gerado em páginas A4 (210 × 297 mm), para ser lido na tela e impresso sem cortes. |
| **Medida**        | Todas as páginas medem 595 × 842 pontos (A4). Exceto na capa, que ocupa a página inteira, o corpo do texto, as tabelas e as imagens ficam a 20 mm do topo e da base e a 18 mm das laterais. |
| **Condição**      | Arquivo do manual entregue com o aplicativo. |
| **Verificação**   | Ler o tamanho de cada página com `pdfinfo` ou com a biblioteca pypdf (CT-MAN-11 e CT-MAN-14). Para as margens, extrair a posição de cada palavra com `pdftotext -bbox` ou imprimir uma página e medir. |
| **Evidência**     | `FRONTEND/assets/manual/manual.pdf` e a cópia em `manual-usuario/`. |
| **Situação**      | Atendido no tamanho das páginas: em 13/11/2026, `pdfinfo` informou 27 páginas A4 no arquivo daquela data, e a posição do texto, extraída com `pdftotext -bbox`, mostrou margens laterais de 18 mm e corpo entre 19 mm do topo e 20 mm da base; em 26/11/2026, o arquivo entregue com o aplicativo tem 27 páginas de 594,96 × 841,92 pt (CT-MAN-14). Medição das margens no arquivo de 26/11/2026 e em página impressa pendente. |

### RNF-MAN-05 — Linguagem simples

| Campo             | Conteúdo |
| ----------------- | -------- |
| **Classificação** | Produto › Facilidade de uso |
| **Critério**      | A Ajuda e o manual são escritos para pessoas sem conhecimento técnico, com respostas curtas, uma ação por passo e os mesmos nomes de botões e telas que aparecem no aplicativo. |
| **Medida**        | Na Ajuda, cada resposta em lista tem no máximo 4 passos, com uma frase por passo; cada resposta em texto tem no máximo 3 frases; cada tópico tem no máximo 7 perguntas. Todo nome de botão ou tela citado nas respostas e no manual existe no aplicativo, com a mesma grafia. Nenhuma resposta usa termos internos do sistema, como nomes de rotas, coleções ou campos. |
| **Condição**      | Conteúdo atual de `TOPICOS_AJUDA` (42 perguntas) e versão 1.0 do manual em PDF. |
| **Verificação**   | Contar os passos, as frases e as perguntas por tópico em `src/constants/ajuda.ts`. Procurar no código das telas cada nome de botão ou tela citado. Pedir a uma pessoa que não conhece o aplicativo que siga três respostas (recuperar a senha, registrar presença e votar) sem ajuda externa; o requisito é aprovado se ela concluir as três. |
| **Evidência**     | Regras de escrita no comentário inicial de `src/constants/ajuda.ts`; revisão do texto do manual no commit `93fa767` do FRONTEND ("corrige textos do manual (revisão RNF-MAN-05)"). |
| **Situação**      | Parcialmente atendido. Na contagem de 26/11/2026, sobre `8fc0a5a`, nenhuma resposta em texto passa de 3 frases, nenhuma lista passa de 4 passos, cada passo tem uma frase e os tópicos têm de 4 a 7 perguntas. A pergunta "conta-email-senha" chegou a ter 5 passos em `6ff2d50` e voltou a 4 em `8fc0a5a`. Na contagem de 27/11/2026, sobre `10abe49`, que mudou o texto de `votacao-abrir`, `votacao-votar`, `votacao-resultado` e `votacao-sigilo`, os limites continuam valendo: 21 respostas em lista, com 3 ou 4 passos e uma frase por passo; 21 respostas em texto, com 2 ou 3 frases; e tópicos de 4 a 7 perguntas. Os 58 nomes de botões, abas e telas citados nas respostas foram conferidos no código em 13/11/2026; os nomes acrescentados em `10abe49`, "Votar em branco" e "Votar nulo", existem com a mesma grafia em `src/components/VotacoesReuniao.tsx`. No manual em PDF, a seção da lista de presença orienta a tocar em "Compartilhar", e o aplicativo mostra o botão "PDF da lista". A seção 11 do manual, "Votação", não acompanhou `10abe49`: diz que o voto é secreto e que os outros veem só o total de cada opção, não cita o voto em branco, o voto nulo nem a escolha entre votação fechada e aberta, e as figuras 27 a 30 são anteriores a essa mudança; na votação do tipo aberta, o aplicativo mostra a todos, depois do encerramento, quem votou em quê. Teste com usuário pendente. |

### RNF-MAN-06 — Organização por funcionalidade e por perfil

| Campo             | Conteúdo |
| ----------------- | -------- |
| **Classificação** | Produto › Facilidade de uso |
| **Critério**      | O conteúdo é agrupado pela funcionalidade do aplicativo e avisa quando uma orientação vale só para um perfil, sem esconder nada dos demais usuários. |
| **Medida**        | Na Ajuda, cada pergunta pertence a um único tópico, e são 7 tópicos: Conta e acesso, Organizações e comissões, Reuniões, Presença, Ata, Votação e Relatório individual. Toda pergunta sobre ação que só o responsável da comissão pode fazer tem `perfil` igual a `responsavel` e exibe o selo "Responsável"; no conteúdo atual, são 9 perguntas. O leitor de tela anuncia o selo como "Exclusivo do Responsável". No manual, as seções seguem as funcionalidades, a seção 2, "Perfis de usuário", explica cada papel, e as seções exclusivas levam as etiquetas RESPONSÁVEL, ADMINISTRADOR ou MEMBRO. |
| **Condição**      | Tela Ajuda aberta com qualquer conta ou sem login, e versão 1.0 do manual em PDF. |
| **Verificação**   | Inspeção de `src/constants/ajuda.ts`, conferindo o tópico e o perfil de cada pergunta. Roteiro CT-MAN-08. Com o TalkBack ou o VoiceOver ativado, ouvir o anúncio do selo. No manual, conferir o sumário e as etiquetas de perfil. |
| **Evidência**     | Constante `SELO_PERFIL`, em `src/constants/ajuda.ts`; propriedade `accessibilityLabel` em `src/components/Sanfona.tsx`; seção 2 e etiquetas do manual em PDF. |
| **Situação**      | Atendido por inspeção em 13/11/2026 e conferido de novo em 26/11/2026: 9 perguntas com `responsavel` (3 em Reuniões, 3 em Presença, 2 em Ata e 1 em Votação). O selo foi conferido em iPhone (CT-MAN-08). Anúncio do leitor de tela pendente. |

### RNF-MAN-07 — Disponibilidade em Android e iOS

| Campo             | Conteúdo |
| ----------------- | -------- |
| **Classificação** | Produto › Portabilidade |
| **Critério**      | A tela Ajuda e o leitor do manual são desenvolvidos uma única vez, em React Native com Expo, e funcionam em Android e iOS. O leitor usa a `WebView` de cada sistema e a PDF.js na versão compatível com os dois (`legacy`). |
| **Medida**        | Num Android (aparelho ou emulador) e num iPhone, a Ajuda abre com e sem login, as respostas abrem e fecham, a busca filtra as perguntas e o botão "Ver manual completo (PDF)" abre o manual com as 27 páginas. |
| **Condição**      | Um Android e um iPhone com o aplicativo aberto (instalado ou pelo Expo Go). Na web, o leitor do manual não é exibido, porque a `WebView` não tem implementação para navegador. |
| **Verificação**   | Roteiro CT-MAN-12, que repete CT-MAN-01 a CT-MAN-04 em cada aparelho, e roteiro CT-MAN-16 para o leitor do manual. |
| **Evidência**     | `src/app/ajuda.tsx` não tem código específico de plataforma; `src/components/ManualPdf.tsx` trata o encerramento da `WebView` no iOS (`onContentProcessDidTerminate`) e no Android (`onRenderProcessGone`). |
| **Situação**      | Ajuda: atendido em 13/11/2026, em iPhone 14 (iOS 27.2) e no emulador Pixel 6 do Android Studio (Android 15, API 35, x86_64), via Expo Go (CT-MAN-12). Leitor do manual: execução em aparelho pendente (CT-MAN-16). |

### RNF-MAN-08 — Versão do Expo e sistemas suportados

| Campo             | Conteúdo |
| ----------------- | -------- |
| **Classificação** | Organizacional › Implementação |
| **Critério**      | O aplicativo usa o Expo SDK 58, com React Native 0.88.1 e Expo Router 58. As versões mínimas de sistema são as padrão do SDK: Android 7.0 (API 24) e iOS 16.4. |
| **Medida**        | O `package.json` declara `"expo": "~58.1.0"`, `"react-native": "0.88.1"`, `"expo-router": "~58.0.33"` e `"react-native-webview": "14.0.1"`, e o `package-lock.json` fixa o Expo em 58.1.0. O `app.json` não altera as versões mínimas padrão do SDK. Os testes do aplicativo terminam sem falhas, incluindo os 5 testes da busca da Ajuda. |
| **Condição**      | FRONTEND na branch `homolog`. |
| **Verificação**   | Conferir o `package.json`, o `package-lock.json` e o `app.json`; executar `node --test tests/*.test.cjs` (CT-MAN-13). |
| **Evidência**     | `package.json`, `package-lock.json` e `app.json` do FRONTEND. |
| **Situação**      | Atendido em 26/11/2026: versões conferidas em `6ff2d50` e 47 testes aprovados em 5 arquivos (CT-MAN-13). Atendido de novo em 27/11/2026, sobre `10abe49`: o commit `42fd66c` atualizou as bibliotecas do Expo SDK 58 (Expo de 58.0.26 para 58.1.0 e Expo Router de 58.0.32 para 58.0.33), sem mudar o React Native, a `react-native-webview` nem as versões mínimas, e os 47 testes em 5 arquivos foram aprovados de novo. Em 13/11/2026, o aplicativo foi executado em iPhone 14 (iOS 27.2) e no emulador Pixel 6 (Android 15, API 35), via Expo Go. |

### RNF-MAN-09 — Manutenção do conteúdo da Ajuda

| Campo             | Conteúdo |
| ----------------- | -------- |
| **Classificação** | Organizacional › Implementação |
| **Critério**      | O conteúdo da Ajuda é alterado apenas em `src/constants/ajuda.ts`, sem mudança em componentes. A tela apenas lê `TOPICOS_AJUDA` e exibe os tópicos na ordem da lista, e a busca considera automaticamente qualquer pergunta incluída. |
| **Medida**        | Incluir, alterar ou remover um tópico ou uma pergunta exige mudança somente em `src/constants/ajuda.ts`. A tela Ajuda e a busca passam a refletir a mudança sem alteração em `ajuda.tsx`, `Sanfona.tsx` ou `busca-ajuda.ts`. O `id` do tópico `conta`, usado pelo link da tela Entrar, é mantido. |
| **Condição**      | Qualquer alteração de conteúdo da Ajuda. |
| **Verificação**   | Incluir uma pergunta de teste em `src/constants/ajuda.ts`, conferir que ela aparece na tela e é encontrada pela busca, e conferir com `git diff --stat` que apenas esse arquivo mudou. |
| **Evidência**     | Commit `63d3bcb` do FRONTEND ("ajuste das perguntas"), que removeu três perguntas alterando apenas `src/constants/ajuda.ts`. Os commits `0778723`, `066d347` e `8fd815f` mudam o texto de três perguntas junto com as funcionalidades que elas descrevem; `4f03c20` muda o texto de `relatorio-o-que-e` e `6ff2d50` divide um passo de `conta-email-senha`, os dois alterando só `src/constants/ajuda.ts`. O commit `10abe49` muda o texto de quatro perguntas do tópico Votação junto com a tela de votação (`VotacoesReuniao.tsx` e `services/api.ts`). Nenhum deles altera `ajuda.tsx`, `Sanfona.tsx` nem `busca-ajuda.ts`. |
| **Situação**      | Atendido por inspeção e pelo commit `63d3bcb`, em 13/11/2026. Conferido de novo em 26/11/2026 pelos commits `4f03c20` e `6ff2d50` e em 27/11/2026 pelo commit `10abe49`. |

### RNF-MAN-10 — Exibição do manual sem travar o aparelho

| Campo             | Conteúdo |
| ----------------- | -------- |
| **Classificação** | Produto › Eficiência › Desempenho |
| **Critério**      | O leitor desenha uma página por vez, num único `canvas` reaproveitado, com densidade limitada a 2 e imagem JPEG de qualidade 0,85, e libera cada página depois de desenhá-la; uma página que não fica pronta em 30 segundos com a tela visível vira erro, com o botão "Tentar de novo". |
| **Medida**        | No emulador Android 7.0 (720 × 1280, 2 GB de RAM), as 27 páginas aparecem sem a tela de erro e sem o aplicativo fechar, e a primeira página aparece em até 10 segundos depois do toque no botão. |
| **Condição**      | APK de release, sem internet, primeira abertura do manual depois de abrir o aplicativo. |
| **Verificação**   | Roteiro CT-MAN-18, cronometrando do toque à primeira página. |
| **Evidência**     | `src/components/ManualPdf.tsx`: `renderizar`, `renovarVigia` e `canvasParaUrl`. |
| **Situação**      | Critério implementado, conferido por inspeção em 26/11/2026; medição pendente. |

## Base de conferência

FRONTEND na branch `homolog`, em `10abe49`, e DOCUMENTATION na branch `homolog`, conferidos em 27/11/2026. A revisão anterior, de 26/11/2026, conferiu o FRONTEND em `6ff2d50` e, para o conteúdo da Ajuda, em `8fc0a5a`.
