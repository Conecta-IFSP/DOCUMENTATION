# Requisitos não funcionais — Manual de utilização

> Padrão da disciplina · Módulo MAN · Revisado em 25/11/2026

## Resumo

Os requisitos não funcionais aparecem em dois níveis, como os funcionais. Os **de usuário** dizem, em uma frase, a qualidade esperada do manual de utilização: a tela Ajuda e o manual completo em PDF, lido dentro do aplicativo. Os **de sistema** completam cada um com a medida que o aprova, a condição em que vale, como é verificado e onde está a evidência, de modo que qualquer pessoa consiga conferir. Onde ainda não houve medição ou execução no aparelho, isso está escrito no campo Situação.

A classificação segue a árvore de Sommerville usada na disciplina: requisitos de **produto** (facilidade de uso, eficiência, confiabilidade, portabilidade), **organizacionais** (entrega, implementação, padrões) e **externos** (interoperabilidade, éticos, legais).

| Código | Classificação | Requisito | Aplica-se a |
| --- | --- | --- | --- |
| RNF-MAN-01 | Produto › Facilidade de uso | Acesso sem login | RF-MAN-01 e RF-MAN-02 |
| RNF-MAN-02 | Produto › Confiabilidade | Funcionamento sem internet | RF-MAN-01 e RF-MAN-02 |
| RNF-MAN-03 | Organizacional › Entrega | Manual empacotado no aplicativo | RF-MAN-02 |
| RNF-MAN-04 | Organizacional › Padrões | Manual em formato A4 | RF-MAN-02 |
| RNF-MAN-05 | Produto › Facilidade de uso | Linguagem simples | RF-MAN-01 e RF-MAN-02 |
| RNF-MAN-06 | Produto › Facilidade de uso | Organização por funcionalidade e por perfil | RF-MAN-01 e RF-MAN-02 |
| RNF-MAN-07 | Produto › Portabilidade | Disponibilidade em Android e iOS | RF-MAN-01 e RF-MAN-02 |
| RNF-MAN-08 | Organizacional › Implementação | Versão do Expo e sistemas suportados | RF-MAN-01 e RF-MAN-02 |
| RNF-MAN-09 | Organizacional › Implementação | Manutenção do conteúdo da Ajuda | RF-MAN-01 |
| RNF-MAN-10 | Produto › Eficiência › Desempenho | Tempo de abertura do manual | RF-MAN-02 |
| RNF-MAN-11 | Produto › Eficiência › Espaço | Tamanho do arquivo do manual | RF-MAN-02 |
| RNF-MAN-12 | Produto › Eficiência › Espaço | Uso de memória ao exibir o manual | RF-MAN-02 |

## Requisitos não funcionais de usuário

**RNF-MAN-01 — Acesso sem login.** A Ajuda e o manual devem abrir para quem ainda não entrou na conta, para que a pessoa consiga tirar dúvidas sobre como entrar ou recuperar a senha.

**RNF-MAN-02 — Funcionamento sem internet.** As perguntas e respostas da Ajuda e o manual completo devem funcionar com o aparelho sem conexão.

**RNF-MAN-03 — Manual empacotado no aplicativo.** O manual em PDF deve ir junto com o aplicativo, sem depender de nenhum endereço da internet. Em troca, atualizar o manual passa a exigir uma nova versão do aplicativo.

**RNF-MAN-04 — Manual em formato A4.** O manual em PDF deve poder ser lido na tela e impresso em folha A4 sem cortes.

**RNF-MAN-05 — Linguagem simples.** A Ajuda e o manual devem usar respostas curtas, uma ação por passo e os mesmos nomes de botões e telas que aparecem no aplicativo.

**RNF-MAN-06 — Organização por funcionalidade e por perfil.** O conteúdo deve ser agrupado por funcionalidade e avisar quando uma orientação vale só para um perfil.

**RNF-MAN-07 — Disponibilidade em Android e iOS.** A Ajuda e o manual devem funcionar em celulares Android e em iPhones, tanto pelo Expo Go quanto no aplicativo instalado.

**RNF-MAN-08 — Versão do Expo e sistemas suportados.** O aplicativo deve usar o Expo SDK 58 e funcionar a partir do Android 7.0 e do iOS 16.4.

**RNF-MAN-09 — Manutenção do conteúdo da Ajuda.** Incluir, alterar ou remover uma pergunta deve exigir mudança em um único arquivo.

**RNF-MAN-10 — Tempo de abertura do manual.** A primeira página do manual deve aparecer em poucos segundos depois do toque no botão, e o manual inteiro logo em seguida.

**RNF-MAN-11 — Tamanho do arquivo do manual.** O PDF empacotado no aplicativo deve ser pequeno, para não aumentar muito o tamanho do aplicativo nem o tempo de abertura.

**RNF-MAN-12 — Uso de memória ao exibir o manual.** Abrir e rolar o manual várias vezes não pode fazer o aplicativo travar nem fechar sozinho.

## Requisitos não funcionais de sistema

### RNF-MAN-01 — Acesso sem login

| Campo | Conteúdo |
| --- | --- |
| **Classificação** | Produto › Facilidade de uso |
| **Critério** | A tela Ajuda e a tela "Manual de utilização" abrem sem sessão ativa e mostram o mesmo conteúdo que mostram a quem fez login. |
| **Medida** | Sem sessão ativa, o toque em "Precisa de ajuda?" abre a tela Ajuda sem redirecionar para a tela Entrar. A tela exibe os 7 tópicos, a pergunta "Esqueci minha senha. O que faço?" já aberta e o botão "Ver manual completo (PDF)". O toque nesse botão abre a tela "Manual de utilização", também sem redirecionar para a tela Entrar. Com sessão ativa, a mesma tela Ajuda abre por Perfil → Suporte → Ajuda, com todas as perguntas fechadas. |
| **Condição** | Aplicativo instalado ou aberto pelo Expo Go, nas duas situações: sem sessão ativa e com sessão ativa. |
| **Verificação** | Inspeção de `src/app/_layout.tsx` e roteiros CT-MAN-01, CT-MAN-02 e CT-MAN-09. |
| **Evidência** | As rotas `ajuda` e `manual` são declaradas fora dos grupos `Stack.Protected` em `src/app/_layout.tsx`. O link "Precisa de ajuda?" está em `src/app/login/index.tsx`, e a linha "Ajuda", em `src/app/perfil.tsx`. |
| **Situação** | Atendido. Ajuda: em 13/11/2026, por inspeção e pelos roteiros CT-MAN-01 e CT-MAN-02, aprovados em iPhone 14 (iOS 27.2), via Expo Go. Manual: em 25/11/2026, por inspeção (rota `manual` fora do `Stack.Protected` no commit `fa1ba7a`) e pelo roteiro CT-MAN-09, aprovado sem e com sessão ativa em iPhone 14, via Expo Go. |

### RNF-MAN-02 — Funcionamento sem internet

| Campo | Conteúdo |
| --- | --- |
| **Classificação** | Produto › Confiabilidade |
| **Critério** | As perguntas e respostas, o manual em PDF e o leitor PDF.js fazem parte do aplicativo instalado. A tela Ajuda e a tela do manual não enviam nenhuma requisição à API, e o visualizador do manual não acessa nenhum endereço da internet. |
| **Medida** | Com o aparelho em modo avião, a tela Ajuda abre e exibe os 7 tópicos e as 42 perguntas, as respostas abrem e fecham, e a busca filtra as perguntas sem nenhuma mensagem de erro de conexão. Ainda em modo avião, o botão "Ver manual completo (PDF)" abre a tela "Manual de utilização" e exibe as 27 páginas, sem a mensagem "Não foi possível abrir o manual". |
| **Condição** | Aparelho em modo avião, com ou sem login. Para o manual, a prova definitiva é o aplicativo instalado (build), porque pelo Expo Go os arquivos do aplicativo vêm do Metro, pela rede local. |
| **Verificação** | Roteiros CT-MAN-07, CT-MAN-10, CT-MAN-19 e CT-MAN-20. Inspeção: `src/app/ajuda.tsx`, `src/utils/busca-ajuda.ts`, `src/app/manual.tsx` e `src/components/ManualPdf.tsx` não importam o módulo `api`, e os dois últimos não contêm nenhum endereço `http`. |
| **Evidência** | Constante `TOPICOS_AJUDA`, em `src/constants/ajuda.ts`; arquivos `assets/manual/manual.pdf` e `assets/pdfjs/`; importações de `src/app/ajuda.tsx` e `src/components/ManualPdf.tsx`. |
| **Situação** | Ajuda: atendido em 13/11/2026, por inspeção e pelo roteiro CT-MAN-07, aprovado em iPhone em modo avião, via Expo Go. Manual: atendido em 25/11/2026. Por inspeção, no commit `fa1ba7a`, `manual.tsx` e `ManualPdf.tsx` não têm nenhum endereço `http` nem importam `api`. No Android, o CT-MAN-19 foi aprovado no emulador Pixel 6 (Android 15, API 35) com o aplicativo instalado por build release e em modo avião desde o início. No iOS, a verificação foi feita pelo Expo Go, no iPhone 14 (iOS 27.2), em 25/11/2026 (CT-MAN-09 a CT-MAN-17); só o build iOS (aplicativo instalado no iPhone, CT-MAN-20) não foi executado, porque não há Mac disponível, e o build para iPhone exige conta paga de desenvolvedor Apple. Pelo Expo Go, o CT-MAN-10 foi feito ligando o modo avião depois de abrir o manual, porque os arquivos vêm do Metro; a abertura sem rede desde o início foi comprovada só no Android instalado. |

### RNF-MAN-03 — Manual empacotado no aplicativo

| Campo | Conteúdo |
| --- | --- |
| **Classificação** | Organizacional › Entrega |
| **Critério** | O manual em PDF e o leitor que o exibe são entregues dentro do pacote do aplicativo, e não por um endereço da internet. O arquivo do aplicativo é a mesma versão publicada neste repositório. |
| **Medida** | (1) `assets/manual/manual.pdf` está no commit do FRONTEND e é exibido pela tela "Manual de utilização"; (2) o código do aplicativo não guarda nenhum endereço do manual (não existe mais `URL_MANUAL_PDF`) e não chama `Linking.openURL` para ele; (3) `FRONTEND/assets/manual/manual.pdf` e `DOCUMENTATION/4 - Incremento3/manual-de-utilizacao/manual-usuario/manual.pdf` têm o mesmo hash SHA-256. |
| **Condição** | FRONTEND na branch `homolog` e DOCUMENTATION na branch `main`. |
| **Verificação** | Procurar `URL_MANUAL_PDF` e `Linking` em `src/` do FRONTEND. Calcular o SHA-256 dos dois arquivos (`sha256sum` ou `Get-FileHash`) e comparar. Abrir o manual com o aparelho sem internet (CT-MAN-10). |
| **Evidência** | Commit `fa1ba7a` do FRONTEND, que acrescentou `assets/manual/manual.pdf`, `assets/pdfjs/`, `src/app/manual.tsx` e `src/components/ManualPdf.tsx` e retirou `URL_MANUAL_PDF`; commit `93fa767`, que trocou o `manual.pdf` pela versão corrigida (SHA-256 `35a838f8…7b963097`). |
| **Situação** | Atendido em 25/11/2026: por inspeção, `URL_MANUAL_PDF` não existe mais em `src/`, o único `Linking.openURL` restante é de outras telas (`sobre.tsx` e `presenca.tsx`), e os dois arquivos têm o mesmo SHA-256. Conferido de novo em 25/11/2026, depois da troca pela versão corrigida do manual: SHA-256 `35a838f8…7b963097`, com 1.220.899 bytes, nos dois repositórios (a versão corrigida do PDF está no commit `93fa767` do FRONTEND e entra no commit desta revisão no DOCUMENTATION). O manual abriu sem internet no Android instalado (CT-MAN-19) e, via Expo Go, no iPhone com o modo avião ligado depois da abertura (CT-MAN-10). |

**Contrapartida.** Antes, o manual era atualizado apenas substituindo o arquivo no GitHub, sem nova versão do aplicativo. Agora, **atualizar o manual exige publicar uma nova versão do aplicativo**: quem já instalou continua com o manual da sua versão até atualizar. Além disso, o arquivo precisa ser trocado nos dois repositórios (FRONTEND e DOCUMENTATION), com o mesmo nome, para que a cópia de documentação continue igual à do aplicativo. A troca foi aceita porque o professor apontou que o manual dependia de internet e porque a abertura pelo link mostrou "Invalid PDF".

### RNF-MAN-04 — Manual em formato A4

| Campo | Conteúdo |
| --- | --- |
| **Classificação** | Organizacional › Padrões |
| **Critério** | O manual em PDF é gerado em páginas A4 (210 × 297 mm), para ser lido na tela e impresso sem cortes. |
| **Medida** | Todas as páginas do arquivo medem 595 × 842 pontos (A4). Exceto na capa, que ocupa a página inteira, o corpo do texto, as tabelas e as imagens ficam a 20 mm do topo e da base e a 18 mm das laterais. O cabeçalho e o rodapé de cada página ficam dentro da margem, a cerca de 7 mm da borda. |
| **Condição** | Arquivo `manual.pdf` empacotado no aplicativo e publicado na branch `main` do DOCUMENTATION (os dois são iguais, ver RNF-MAN-03). |
| **Verificação** | Executar `pdfinfo manual.pdf` e conferir "Page size: 594.96 x 841.92 pts (A4)" (roteiro CT-MAN-11). Para as margens, extrair a posição de cada palavra com `pdftotext -bbox` ou imprimir uma página e medir. |
| **Evidência** | `4 - Incremento3/manual-de-utilizacao/manual-usuario/manual.pdf` e `FRONTEND/assets/manual/manual.pdf`. |
| **Situação** | Atendido em 25/11/2026, na versão corrigida do manual (SHA-256 `35a838f8…7b963097`, 1.220.899 bytes). Tamanho: `pdfinfo` informa 27 páginas de 594.96 × 841.92 pt (A4) (CT-MAN-11); o PDF.js 3.11.174 do projeto confirmou as 27 páginas com o mesmo tamanho. Margens, pela posição do texto extraída com `pdftotext -bbox` nas páginas 2 a 27: laterais de 18,0 mm, corpo a 19,1 mm do topo e a 21,5 mm da base. Medição em página impressa não executada (sem impressora disponível); a verificação foi feita pela posição do texto no arquivo. |

### RNF-MAN-05 — Linguagem simples

| Campo | Conteúdo |
| --- | --- |
| **Classificação** | Produto › Facilidade de uso |
| **Critério** | A Ajuda e o manual são escritos para pessoas sem conhecimento técnico, com respostas curtas, uma ação por passo e os mesmos nomes de botões e telas que aparecem no aplicativo. |
| **Medida** | Na Ajuda, cada resposta em lista tem no máximo 4 passos, com uma frase por passo. Cada resposta em texto tem no máximo 3 frases, e cada tópico tem no máximo 7 perguntas. No manual em PDF, cada passo numerado tem **um único objetivo**; preencher um campo e confirmar (por exemplo, "digite o código e toque em Continuar") conta como um só. Todo nome de botão ou tela citado nas respostas e no manual existe no aplicativo, com a mesma grafia. Nenhuma resposta usa termos internos do sistema, como nomes de rotas, coleções ou campos. |
| **Condição** | Conteúdo atual de `TOPICOS_AJUDA` (42 perguntas) e versão corrigida do manual em PDF (SHA-256 `35a838f8…7b963097`). |
| **Verificação** | Contar os passos, as frases e as perguntas por tópico em `src/constants/ajuda.ts`. Procurar no código das telas cada nome de botão ou tela citado. Pedir a uma pessoa que não conhece o aplicativo que siga três respostas (recuperar a senha, registrar presença e votar) sem ajuda externa; o requisito é aprovado se ela concluir as três. |
| **Evidência** | Regras de escrita no comentário inicial de `src/constants/ajuda.ts`. |
| **Situação** | **Parcialmente atendido.** Ajuda: atendido por contagem em 13/11/2026 — a maior lista tem 4 passos, nenhum passo tem mais de uma frase, nenhuma resposta em texto passa de 3 frases, os tópicos têm de 4 a 7 perguntas, e os 58 nomes de botões, abas e telas citados existem no código com a mesma grafia. Manual em PDF: revisão feita e corrigida em 25/11/2026. A revisão extraiu o texto das 27 páginas com o PDF.js e conferiu 143 nomes de botões, telas, abas e campos no código do FRONTEND: 2 estavam divergentes (p. 6, "Ir para hoje", que é só um ícone; p. 10, código de acesso na aba Comissões e botão "Gerar", quando o código fica na folha Convidar e o botão é "Gerar novo código"); havia passos com mais de um objetivo (4.3, 9.4, 10.3, 10.4, 10.5, 12.1 e 12.4) e uma frase de 29 palavras (11.2). Na versão corrigida, os dois nomes foram ajustados, esses passos foram divididos e a frase da 11.2 virou lista; o texto novo contém "Gerar novo código", "Para cancelar" e "Não é possível votar quando" e não contém "Ir para hoje". O teste com uma pessoa que não conhece o aplicativo (três tarefas), previsto na verificação, **não foi executado**. |

### RNF-MAN-06 — Organização por funcionalidade e por perfil

| Campo | Conteúdo |
| --- | --- |
| **Classificação** | Produto › Facilidade de uso |
| **Critério** | O conteúdo é agrupado pela funcionalidade do aplicativo e avisa quando uma orientação vale só para um perfil, sem esconder nada dos demais usuários. |
| **Medida** | Na Ajuda, cada pergunta pertence a um único tópico, e são 7 tópicos: Conta e acesso, Organizações e comissões, Reuniões, Presença, Ata, Votação e Relatório individual. Toda pergunta sobre uma ação que só o responsável da comissão pode fazer tem `perfil` igual a `responsavel` e exibe o selo "Responsável"; no conteúdo atual, são 9 perguntas. O leitor de tela anuncia o selo como "Exclusivo do Responsável". No manual, as seções seguem as funcionalidades, a seção 2, "Perfis de usuário", explica cada papel, e as seções exclusivas levam as etiquetas RESPONSÁVEL, ADMINISTRADOR ou MEMBRO. |
| **Condição** | Tela Ajuda aberta com qualquer conta ou sem login, e versão 1.0 do manual em PDF. |
| **Verificação** | Inspeção de `src/constants/ajuda.ts`, conferindo o tópico e o perfil de cada pergunta. Roteiro CT-MAN-08. Com o TalkBack ou o VoiceOver ativado, ouvir o anúncio do selo. No manual, conferir o sumário e as etiquetas de perfil. |
| **Evidência** | Constante `SELO_PERFIL`, em `src/constants/ajuda.ts`; propriedade `accessibilityLabel` em `src/components/Sanfona.tsx`; seção 2 e etiquetas do manual em PDF. |
| **Situação** | Atendido em 25/11/2026. Por inspeção (13/11/2026): 9 perguntas com `responsavel` (3 em Reuniões, 3 em Presença, 2 em Ata e 1 em Votação). O selo foi conferido em iPhone (CT-MAN-08, 13/11/2026). Leitor de tela: com o VoiceOver no iPhone 14 (iOS 27.2), via Expo Go, ao selecionar a pergunta "Como agendo uma reunião?", o selo "Responsável" foi lido (o rótulo de acessibilidade definido em `Sanfona.tsx` é "<pergunta>. Exclusivo do Responsável"). Anúncio com o TalkBack no Android não verificado. |

### RNF-MAN-07 — Disponibilidade em Android e iOS

| Campo | Conteúdo |
| --- | --- |
| **Classificação** | Produto › Portabilidade |
| **Critério** | A tela Ajuda e o manual são desenvolvidos uma única vez, em React Native com Expo, e funcionam em Android e iOS, pelo Expo Go e no aplicativo instalado. O manual é exibido em uma `WebView` (biblioteca `react-native-webview`, que já vem no Expo Go) com o leitor PDF.js em JavaScript, sem biblioteca nativa própria do projeto. A única diferença por sistema está no tratamento do encerramento da `WebView`: `onContentProcessDidTerminate` no iOS e `onRenderProcessGone` no Android. |
| **Medida** | Em um Android (aparelho ou emulador) e em um iPhone, a Ajuda abre com e sem login, as respostas abrem e fecham, a busca filtra as perguntas, e o botão "Ver manual completo (PDF)" abre a tela "Manual de utilização" com as 27 páginas, rolagem e zoom por pinça. O mesmo vale para o aplicativo instalado (build) em cada sistema. |
| **Condição** | Um Android (aparelho ou emulador) e um iPhone, com o aplicativo aberto pelo Expo Go e, depois, instalado por build (`npx expo run:android` e `npx expo run:ios`). |
| **Verificação** | Roteiro CT-MAN-12, que repete CT-MAN-01 a CT-MAN-04, CT-MAN-09 e CT-MAN-14 em cada aparelho, e roteiros CT-MAN-19 e CT-MAN-20 nos builds. |
| **Evidência** | `src/app/ajuda.tsx`, `src/app/manual.tsx` e `src/components/ManualPdf.tsx` não têm código separado por plataforma, exceto as duas propriedades de encerramento da `WebView` e a sombra da faixa de progresso (`Platform.select`). `react-native-webview` 14.0.1 no `package.json`. |
| **Situação** | Ajuda: atendido em 13/11/2026, aprovado em iPhone 14 (iOS 27.2) e no emulador Pixel 6 do Android Studio (Android 15, API 35, x86_64), via Expo Go. Manual: atendido em 25/11/2026 no Android, via Expo Go (CT-MAN-12) e com o aplicativo instalado por build release (CT-MAN-19). No iOS, a verificação foi feita pelo Expo Go, no iPhone 14 (iOS 27.2), em 25/11/2026 (CT-MAN-09 a CT-MAN-17); só o build iOS (aplicativo instalado no iPhone, CT-MAN-20) não foi executado, porque não há Mac disponível, e o build para iPhone exige conta paga de desenvolvedor Apple. |

### RNF-MAN-08 — Versão do Expo e sistemas suportados

| Campo | Conteúdo |
| --- | --- |
| **Classificação** | Organizacional › Implementação |
| **Critério** | O aplicativo usa o Expo SDK 58, com React Native 0.88.1 e Expo Router 58, para receber as correções da versão atual do Expo. As bibliotecas acrescentadas para o manual foram instaladas com `npx expo install`, que escolhe as versões compatíveis com o SDK. As versões mínimas de sistema são as padrão do SDK: Android 7.0 (API 24) e iOS 16.4. |
| **Medida** | O `package.json` declara `"expo": "^58.0.23"`, `"react-native": "0.88.1"`, `"expo-router": "~58.0.30"`, `"react-native-webview": "14.0.1"`, `"expo-asset": "~58.0.21"` e `"expo-file-system": "~58.0.14"`. O `app.json` não altera as versões mínimas padrão do SDK. Os testes do aplicativo terminam sem falhas, incluindo os 5 testes da busca da Ajuda. |
| **Condição** | FRONTEND na branch `homolog`. |
| **Verificação** | Conferir o `package.json`, o `package-lock.json` e o `app.json`. Executar `npx expo-doctor`, que não deve apontar dependências incompatíveis com o SDK 58, e `node --test tests/*.test.cjs` (roteiro CT-MAN-13). Executar o aplicativo nos aparelhos de teste e registrar o modelo e a versão do sistema de cada um. |
| **Evidência** | `package.json`, `package-lock.json` e `app.json` do FRONTEND; `minSdk = "24"` no React Native e plataforma mínima 16.4 em `Expo.podspec`. |
| **Situação** | **Parcialmente atendido** em 25/11/2026, no commit `fa1ba7a`. Versões do `package.json` conferidas e testes automatizados aprovados (CT-MAN-13: 18 testes, 0 falhas). `npx expo-doctor` (versão 1.21.8): 19 de 20 verificações aprovadas. A verificação "Check that packages match versions required by installed Expo SDK" falhou: 14 pacotes estão com a versão de correção (patch) abaixo da esperada pelo SDK 58 instalado — `expo` (esperado ~58.0.26, instalado 58.0.23), `expo-auth-session`, `expo-dev-client`, `expo-document-picker`, `expo-font`, `expo-linear-gradient`, `expo-linking`, `expo-local-authentication`, `expo-print`, `expo-router` (esperado ~58.0.32, instalado 58.0.30), `expo-secure-store`, `expo-sharing`, `expo-splash-screen` e `expo-status-bar`. As bibliotecas usadas pelo manual (`react-native-webview`, `expo-asset` e `expo-file-system`) não aparecem na lista. A correção indicada pela ferramenta, `npx expo install --check`, **não foi aplicada por decisão**, e não por esquecimento: as versões das dependências são definidas pela equipe e não podem ser alteradas por um único integrante. Com as versões atuais (as do `package.json` dos commits `fa1ba7a` e `93fa767`, que não mudou as dependências), todos os casos de teste executados no módulo (CT-MAN-01 a CT-MAN-19) foram aprovados; o CT-MAN-20 não foi executado (sem Mac). O aplicativo foi executado em iPhone 14 (iOS 27.2) e no emulador Pixel 6 (Android 15, API 35), via Expo Go, em 13/11/2026, e de novo em 25/11/2026, já com a `react-native-webview`: no iPhone pelo Expo Go e no Pixel 6 pelo Expo Go e por build release (CT-MAN-09 a CT-MAN-19). |

### RNF-MAN-09 — Manutenção do conteúdo da Ajuda

| Campo | Conteúdo |
| --- | --- |
| **Classificação** | Organizacional › Implementação |
| **Critério** | O conteúdo da Ajuda é alterado apenas em `src/constants/ajuda.ts`, sem mudança em componentes. A tela apenas lê `TOPICOS_AJUDA` e exibe os tópicos na ordem da lista, e a busca considera automaticamente qualquer pergunta incluída. |
| **Medida** | Incluir, alterar ou remover um tópico ou uma pergunta exige mudança somente em `src/constants/ajuda.ts`. A tela Ajuda e a busca passam a refletir a mudança sem alteração em `ajuda.tsx`, `Sanfona.tsx` ou `busca-ajuda.ts`. O `id` do tópico `conta`, usado pelo link da tela Entrar, é mantido. |
| **Condição** | Qualquer alteração de conteúdo da Ajuda. |
| **Verificação** | Incluir uma pergunta de teste em `src/constants/ajuda.ts`, conferir que ela aparece na tela e é encontrada pela busca, e conferir com `git diff --stat` que apenas esse arquivo mudou. |
| **Evidência** | Commit `63d3bcb` do FRONTEND ("ajuste das perguntas"), que removeu três perguntas alterando apenas `src/constants/ajuda.ts`. |
| **Situação** | Atendido por inspeção e pelo commit `63d3bcb`, em 13/11/2026. |

### RNF-MAN-10 — Tempo de abertura do manual

| Campo | Conteúdo |
| --- | --- |
| **Classificação** | Produto › Eficiência › Desempenho |
| **Critério** | O manual começa a ser exibido logo depois do toque e termina de carregar pouco depois. Para isso, as páginas aparecem uma a uma, à medida que são desenhadas, com a faixa "Carregando página X de 27", e a página HTML montada fica em cache para as aberturas seguintes. |
| **Medida** | Na primeira abertura depois de iniciar o aplicativo: a primeira página do manual fica visível em até **3 s** depois do toque em "Ver manual completo (PDF)", e as **27 páginas** terminam de carregar (a faixa "Carregando página X de 27" some) em até **10 s** depois do toque. Na segunda abertura, sem fechar o aplicativo, a primeira página aparece em tempo igual ou menor que na primeira. |
| **Condição** | Primeira abertura do manual depois de iniciar o aplicativo, e segunda abertura sem fechá-lo, no iPhone 14 (iOS 27.2) com o aplicativo aberto pelo Expo Go. |
| **Verificação** | Medir com cronômetro, no iPhone pelo Expo Go, o tempo entre o toque em "Ver manual completo (PDF)" e o aparecimento da primeira página e entre o toque e o desaparecimento da faixa "Carregando página X de 27"; repetir na segunda abertura (roteiros CT-MAN-14 e CT-MAN-17). O cronômetro manual tem a precisão do tempo de reação de quem mede, suficiente diante dos limites de 3 s e 10 s. O Android não foi medido. |
| **Evidência** | `src/components/ManualPdf.tsx`: `cacheConteudo`, `obterConteudo` e o envio da mensagem `progresso` a cada página. |
| **Situação** | Atendido em 25/11/2026 no iPhone 14 (iOS 27.2), via Expo Go, medido com cronômetro: primeira página em 0,70 s (limite 3 s), 27 páginas em 1,96 s (limite 10 s) e, na segunda abertura, primeira página em 0,46 s, menor que na primeira (CT-MAN-14 e CT-MAN-17). O Android não foi medido. Como referência, sem valor de aprovação: em 25/11/2026, a mesma página HTML montada pelo `ManualPdf`, aberta no Microsoft Edge sem janela (headless) em um computador Windows, com tela de 390 px de largura, desenhou as 27 páginas em cerca de 2,6 s. |

### RNF-MAN-11 — Tamanho do arquivo do manual

| Campo | Conteúdo |
| --- | --- |
| **Classificação** | Produto › Eficiência › Espaço |
| **Critério** | O PDF empacotado no aplicativo é comprimido, porque ele aumenta o tamanho do aplicativo instalado e é lido por inteiro (em base64) na primeira abertura do manual. |
| **Medida** | `assets/manual/manual.pdf` tem no máximo **1,5 MB** (1.500.000 bytes), mantendo as 27 páginas legíveis na tela. |
| **Condição** | Arquivo do commit atual do FRONTEND, branch `homolog`. |
| **Verificação** | Conferir o tamanho com `ls -l assets/manual/manual.pdf`, `Get-Item` ou `pdfinfo` ("File size"). Conferir a leitura no roteiro CT-MAN-14. |
| **Evidência** | `FRONTEND/assets/manual/manual.pdf`, commit `93fa767`. |
| **Situação** | Atendido em 25/11/2026: a versão corrigida do manual tem 1.220.899 bytes (cerca de 1,22 MB), abaixo do limite de 1,5 MB, com 27 páginas; conferido com `ls -l` e `pdfinfo`. A versão anterior tinha 1.214.143 bytes. Páginas legíveis no iPhone 14 (CT-MAN-14), conferido com a versão anterior e de novo com a versão corrigida, que abriu dentro do aplicativo até a página 27. Observação: a versão corrigida do PDF está no commit `93fa767` do FRONTEND e entra no commit desta revisão no DOCUMENTATION. |

### RNF-MAN-12 — Uso de memória ao exibir o manual

| Campo | Conteúdo |
| --- | --- |
| **Classificação** | Produto › Eficiência › Espaço |
| **Critério** | O visualizador limita a memória usada para desenhar o manual: um único `canvas` é reaproveitado para todas as páginas, a densidade do desenho é limitada a 2 vezes a da tela (`Math.min(devicePixelRatio, 2)`), cada página desenhada vira uma imagem JPEG (qualidade 0,85), e os dados da página e do documento são liberados com `cleanup()` depois de usados. |
| **Medida** | Abrir o manual, rolar até a página 27 e voltar à Ajuda, **3 vezes seguidas**, sem a mensagem "Não foi possível abrir o manual", sem o aplicativo fechar sozinho e sem o sistema encerrar o processo da `WebView` (que chamaria `onContentProcessDidTerminate` ou `onRenderProcessGone` e mostraria a mesma mensagem). O pico de memória do aplicativo durante as 3 aberturas é registrado, sem limite em MB definido. |
| **Condição** | Memória: emulador Pixel 6 (Android 15, API 35, x86_64), com o aplicativo instalado por build release e o `adb` conectado. Aberturas sem erro: o mesmo emulador e, no iOS, o iPhone 14 (iOS 27.2) pelo Expo Go. |
| **Verificação** | Com o `adb shell dumpsys meminfo` (campo TOTAL PSS): 1) encerrar o aplicativo (`adb shell am force-stop br.com.conectamais.app`), abri-lo de novo e ir à tela Ajuda sem abrir o manual; medir o PSS do aplicativo (`dumpsys meminfo br.com.conectamais.app`) como referência. 2) Abrir o manual, rolar até a página 27 e medir o PSS do aplicativo e o do processo de desenho da `WebView` (processo `sandboxed_process` do Chrome, localizado com `adb shell ps -A` e medido com `dumpsys meminfo <pid>`), porque no Android esse processo não entra no PSS do aplicativo; somar os dois. 3) Voltar à Ajuda e repetir o passo 2 mais 2 vezes. O pico é a maior soma das 3 aberturas; o acréscimo do manual é o pico menos a referência. Roteiro CT-MAN-19. |
| **Evidência** | Função `renderizar` do script montado por `montarHtml`, em `src/components/ManualPdf.tsx`. |
| **Situação** | **Atendido no Android** em 25/11/2026: 3 aberturas seguidas sem erro e sem encerramento da `WebView` (CT-MAN-19), e pico de memória registrado. Medição de memória em 25/11/2026, no emulador Pixel 6 (Android 15, API 35, x86_64), com o aplicativo instalado por build release, fora do modo avião, pelo `adb shell dumpsys meminfo` (TOTAL PSS). No Android, a `WebView` desenha as páginas em um processo separado (`sandboxed_process` do Chrome), que não entra no PSS do aplicativo; por isso os dois foram medidos e somados. Referência: aplicativo recém-iniciado (`am force-stop` e nova abertura) na tela Ajuda, sem ter aberto o manual: 430,7 MiB, sem processo da `WebView`. Com o manual aberto na página 27 (aplicativo + `WebView` = soma): abertura 1, 529,8 + 163,6 = 693,3 MiB; abertura 2, 548,5 + 140,2 = 688,7 MiB; abertura 3, 544,8 + 166,4 = 711,1 MiB. Pico: 711,1 MiB (abertura 3), 280,5 MiB acima da referência. Sem sinal de vazamento nas 3 aberturas: a soma variou 22,4 MiB (cerca de 3%), sem crescer a cada abertura, e o processo da `WebView` é encerrado ao sair do manual e recriado na abertura seguinte. O aplicativo ficou cerca de 15 a 19 MiB acima da abertura 1 nas aberturas 2 e 3, compatível com a página HTML mantida em `cacheConteudo`; três medições não bastam para descartar um crescimento lento em uso prolongado, e a memória com o manual fechado entre as aberturas não foi medida. No iOS, a verificação foi feita pelo Expo Go, no iPhone 14 (iOS 27.2): as 3 aberturas seguidas foram feitas sem erro e sem encerramento da `WebView`, mas o pico de memória não foi medido, porque a medição exige o Xcode. Só o build iOS (CT-MAN-20) não foi executado, porque não há Mac disponível, e o build para iPhone exige conta paga de desenvolvedor Apple. Os valores são do emulador e podem ser diferentes em um aparelho físico. |

## Base de conferência

FRONTEND na branch `homolog`, commit `93fa767`, e DOCUMENTATION na branch `main`, conferidos em 25/11/2026. O código do manual é o do commit `fa1ba7a`; o `93fa767` mudou só o `manual.pdf`. Os resultados de 13/11/2026 citados acima foram obtidos no commit `66806e4` do FRONTEND.
