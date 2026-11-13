# Requisitos não funcionais — Manual de utilização

> Padrão da disciplina · Módulo MAN · Revisado em 13/11/2026

## Resumo

Os requisitos não funcionais aparecem em dois níveis, como os funcionais. Os **de usuário** dizem, em uma frase, a qualidade esperada do manual de utilização: a tela Ajuda e o manual completo em PDF. Os **de sistema** completam cada um com a medida que o aprova, a condição em que vale, como é verificado e onde está a evidência, de modo que qualquer pessoa consiga conferir. Onde ainda não houve medição ou execução no aparelho, isso está escrito no campo Situação.

A classificação segue a árvore de Sommerville usada na disciplina: requisitos de **produto** (facilidade de uso, eficiência, confiabilidade, portabilidade), **organizacionais** (entrega, implementação, padrões) e **externos** (interoperabilidade, éticos, legais).

| Código | Classificação | Requisito | Aplica-se a |
| --- | --- | --- | --- |
| RNF-MAN-01 | Produto › Facilidade de uso | Acesso sem login | RF-MAN-01 e RF-MAN-02 |
| RNF-MAN-02 | Produto › Confiabilidade | Funcionamento sem internet | RF-MAN-01 |
| RNF-MAN-03 | Organizacional › Entrega | Endereço fixo do manual em PDF | RF-MAN-02 |
| RNF-MAN-04 | Organizacional › Padrões | Manual em formato A4 | RF-MAN-02 |
| RNF-MAN-05 | Produto › Facilidade de uso | Linguagem simples | RF-MAN-01 e RF-MAN-02 |
| RNF-MAN-06 | Produto › Facilidade de uso | Organização por funcionalidade e por perfil | RF-MAN-01 e RF-MAN-02 |
| RNF-MAN-07 | Produto › Portabilidade | Disponibilidade em Android e iOS | RF-MAN-01 e RF-MAN-02 |
| RNF-MAN-08 | Organizacional › Implementação | Versão do Expo e sistemas suportados | RF-MAN-01 e RF-MAN-02 |
| RNF-MAN-09 | Organizacional › Implementação | Manutenção do conteúdo da Ajuda | RF-MAN-01 |

## Requisitos não funcionais de usuário

**RNF-MAN-01 — Acesso sem login.** A Ajuda deve abrir para quem ainda não entrou na conta, para que a pessoa consiga tirar dúvidas sobre como entrar ou recuperar a senha.

**RNF-MAN-02 — Funcionamento sem internet.** As perguntas e respostas da Ajuda devem funcionar com o aparelho sem conexão.

**RNF-MAN-03 — Endereço fixo do manual em PDF.** Atualizar o manual em PDF não pode exigir uma nova versão do aplicativo.

**RNF-MAN-04 — Manual em formato A4.** O manual em PDF deve poder ser lido na tela e impresso em folha A4 sem cortes.

**RNF-MAN-05 — Linguagem simples.** A Ajuda e o manual devem usar respostas curtas, uma ação por passo e os mesmos nomes de botões e telas que aparecem no aplicativo.

**RNF-MAN-06 — Organização por funcionalidade e por perfil.** O conteúdo deve ser agrupado por funcionalidade e avisar quando uma orientação vale só para um perfil.

**RNF-MAN-07 — Disponibilidade em Android e iOS.** A Ajuda e a abertura do manual devem funcionar em celulares Android e em iPhones.

**RNF-MAN-08 — Versão do Expo e sistemas suportados.** O aplicativo deve usar o Expo SDK 58 e funcionar a partir do Android 7.0 e do iOS 16.4.

**RNF-MAN-09 — Manutenção do conteúdo da Ajuda.** Incluir, alterar ou remover uma pergunta deve exigir mudança em um único arquivo.

## Requisitos não funcionais de sistema

### RNF-MAN-01 — Acesso sem login

| Campo | Conteúdo |
| --- | --- |
| **Classificação** | Produto › Facilidade de uso |
| **Critério** | A tela Ajuda abre sem sessão ativa e mostra o mesmo conteúdo que mostra a quem fez login. |
| **Medida** | Sem sessão ativa, o toque em "Precisa de ajuda?" abre a tela Ajuda sem redirecionar para a tela Entrar. A tela exibe os 7 tópicos, a pergunta "Esqueci minha senha. O que faço?" já aberta e o botão "Ver manual completo (PDF)". Com sessão ativa, a mesma tela abre por Perfil → Suporte → Ajuda, com todas as perguntas fechadas. |
| **Condição** | Aplicativo instalado, nas duas situações: sem sessão ativa e com sessão ativa. |
| **Verificação** | Inspeção de `src/app/_layout.tsx` e roteiros CT-MAN-01 e CT-MAN-02. |
| **Evidência** | A rota `ajuda` é declarada fora dos grupos `Stack.Protected` em `src/app/_layout.tsx`. O link "Precisa de ajuda?" está em `src/app/login/index.tsx`, e a linha "Ajuda", em `src/app/perfil.tsx`. |
| **Situação** | Atendido em 13/11/2026: por inspeção e pelos roteiros CT-MAN-01 e CT-MAN-02, aprovados em iPhone 14 (iOS 27.2), via Expo Go. |

### RNF-MAN-02 — Funcionamento sem internet

| Campo | Conteúdo |
| --- | --- |
| **Classificação** | Produto › Confiabilidade |
| **Critério** | As perguntas e respostas fazem parte do aplicativo instalado, e a tela Ajuda não envia nenhuma requisição à API. A única função do R4 que precisa de internet é a abertura do manual em PDF. |
| **Medida** | Com o aparelho em modo avião, a tela Ajuda abre e exibe os 7 tópicos e as 42 perguntas. As respostas abrem e fecham, e a busca filtra as perguntas sem nenhuma mensagem de erro de conexão. |
| **Condição** | Aparelho em modo avião, com ou sem login. |
| **Verificação** | Roteiro CT-MAN-07. Inspeção: `src/app/ajuda.tsx` e `src/utils/busca-ajuda.ts` não importam o módulo `api`. |
| **Evidência** | Constante `TOPICOS_AJUDA`, em `src/constants/ajuda.ts`, e as importações de `src/app/ajuda.tsx`. |
| **Situação** | Atendido em 13/11/2026: por inspeção e pelo roteiro CT-MAN-07, aprovado em iPhone em modo avião, via Expo Go. |

### RNF-MAN-03 — Endereço fixo do manual em PDF

| Campo | Conteúdo |
| --- | --- |
| **Classificação** | Organizacional › Entrega |
| **Critério** | O endereço guardado no aplicativo (`URL_MANUAL_PDF`) é formado só pela branch, pelo caminho e pelo nome do arquivo, e nenhum deles muda quando o manual é atualizado. |
| **Medida** | Depois de substituir o arquivo no mesmo caminho e enviar o commit para a `main`, o endereço `https://github.com/Conecta-IFSP/DOCUMENTATION/blob/main/4%20-%20Incremento3/manual-de-utilizacao/manual-usuario/Manual-do-Usuario-Conecta.pdf` responde `200` com o arquivo novo, e a versão do aplicativo já instalada abre esse arquivo sem nenhuma alteração no código. |
| **Condição** | Arquivo `Manual-do-Usuario-Conecta.pdf` na pasta `4 - Incremento3/manual-de-utilizacao/manual-usuario/` do repositório `Conecta-IFSP/DOCUMENTATION`, branch `main`. |
| **Verificação** | Substituir o PDF, fazer commit e push na `main`, consultar o endereço com `curl -sIL <endereço>` e conferir a resposta final `200`. Em seguida, roteiro CT-MAN-09. |
| **Evidência** | Constante `URL_MANUAL_PDF`, em `src/constants/ajuda.ts`, e o commit `225ca39` do DOCUMENTATION, que publicou o arquivo. |
| **Situação** | Atendido em 13/11/2026: o endereço responde `200`, `URL_MANUAL_PDF` aponta para ele, e o botão abriu o manual em iPhone (CT-MAN-09). |

### RNF-MAN-04 — Manual em formato A4

| Campo | Conteúdo |
| --- | --- |
| **Classificação** | Organizacional › Padrões |
| **Critério** | O manual em PDF é gerado em páginas A4 (210 × 297 mm), para ser lido na tela e impresso sem cortes. |
| **Medida** | Todas as páginas do arquivo publicado medem 595 × 842 pontos (A4). Exceto na capa, que ocupa a página inteira, o corpo do texto, as tabelas e as imagens ficam a 20 mm do topo e da base e a 18 mm das laterais. O cabeçalho e o rodapé de cada página ficam dentro da margem, a cerca de 7 mm da borda. |
| **Condição** | Arquivo publicado na branch `main` do DOCUMENTATION. |
| **Verificação** | Executar `pdfinfo Manual-do-Usuario-Conecta.pdf` e conferir "Page size: 594.96 x 841.92 pts (A4)" (roteiro CT-MAN-11). Para as margens, extrair a posição de cada palavra com `pdftotext -bbox` ou imprimir uma página e medir. |
| **Evidência** | `4 - Incremento3/manual-de-utilizacao/manual-usuario/Manual-do-Usuario-Conecta.pdf`. |
| **Situação** | Atendido em 13/11/2026: `pdfinfo` informa 27 páginas de 594.96 × 841.92 pt (A4). Pela posição do texto extraída com `pdftotext -bbox` nas páginas 2 a 27, as margens laterais são de 18 mm e o corpo começa a 19 mm do topo e termina a 20 mm da base. Medição em página impressa pendente. |

### RNF-MAN-05 — Linguagem simples

| Campo | Conteúdo |
| --- | --- |
| **Classificação** | Produto › Facilidade de uso |
| **Critério** | A Ajuda e o manual são escritos para pessoas sem conhecimento técnico, com respostas curtas, uma ação por passo e os mesmos nomes de botões e telas que aparecem no aplicativo. |
| **Medida** | Na Ajuda, cada resposta em lista tem no máximo 4 passos, com uma frase por passo. Cada resposta em texto tem no máximo 3 frases, e cada tópico tem no máximo 7 perguntas. Todo nome de botão ou tela citado nas respostas e no manual existe no aplicativo, com a mesma grafia. Nenhuma resposta usa termos internos do sistema, como nomes de rotas, coleções ou campos. |
| **Condição** | Conteúdo atual de `TOPICOS_AJUDA` (42 perguntas) e versão 1.0 do manual em PDF. |
| **Verificação** | Contar os passos, as frases e as perguntas por tópico em `src/constants/ajuda.ts`. Procurar no código das telas cada nome de botão ou tela citado. Pedir a uma pessoa que não conhece o aplicativo que siga três respostas (recuperar a senha, registrar presença e votar) sem ajuda externa; o requisito é aprovado se ela concluir as três. |
| **Evidência** | Regras de escrita no comentário inicial de `src/constants/ajuda.ts`. |
| **Situação** | Atendido na Ajuda, por contagem, em 13/11/2026: a maior lista tem 4 passos, nenhum passo tem mais de uma frase, nenhuma resposta em texto passa de 3 frases, e os tópicos têm de 4 a 7 perguntas. Os 58 nomes de botões, abas e telas citados nas respostas foram procurados no código das telas, e todos existem com a mesma grafia. Revisão do manual em PDF e teste com usuário pendentes. |

### RNF-MAN-06 — Organização por funcionalidade e por perfil

| Campo | Conteúdo |
| --- | --- |
| **Classificação** | Produto › Facilidade de uso |
| **Critério** | O conteúdo é agrupado pela funcionalidade do aplicativo e avisa quando uma orientação vale só para um perfil, sem esconder nada dos demais usuários. |
| **Medida** | Na Ajuda, cada pergunta pertence a um único tópico, e são 7 tópicos: Conta e acesso, Organizações e comissões, Reuniões, Presença, Ata, Votação e Relatório individual. Toda pergunta sobre uma ação que só o responsável da comissão pode fazer tem `perfil` igual a `responsavel` e exibe o selo "Responsável"; no conteúdo atual, são 9 perguntas. O leitor de tela anuncia o selo como "Exclusivo do Responsável". No manual, as seções seguem as funcionalidades, a seção 2, "Perfis de usuário", explica cada papel, e as seções exclusivas levam as etiquetas RESPONSÁVEL, ADMINISTRADOR ou MEMBRO. |
| **Condição** | Tela Ajuda aberta com qualquer conta ou sem login, e versão 1.0 do manual em PDF. |
| **Verificação** | Inspeção de `src/constants/ajuda.ts`, conferindo o tópico e o perfil de cada pergunta. Roteiro CT-MAN-08. Com o TalkBack ou o VoiceOver ativado, ouvir o anúncio do selo. No manual, conferir o sumário e as etiquetas de perfil. |
| **Evidência** | Constante `SELO_PERFIL`, em `src/constants/ajuda.ts`; propriedade `accessibilityLabel` em `src/components/Sanfona.tsx`; seção 2 e etiquetas do manual em PDF. |
| **Situação** | Atendido por inspeção em 13/11/2026: 9 perguntas com `responsavel` (3 em Reuniões, 3 em Presença, 2 em Ata e 1 em Votação). O selo foi conferido em iPhone (CT-MAN-08). Anúncio do leitor de tela pendente. |

### RNF-MAN-07 — Disponibilidade em Android e iOS

| Campo | Conteúdo |
| --- | --- |
| **Classificação** | Produto › Portabilidade |
| **Critério** | A tela Ajuda e a abertura do manual são desenvolvidas uma única vez, em React Native com Expo, e funcionam em Android e iOS. A abertura do PDF usa `Linking.openURL`, que entrega o endereço ao navegador de cada sistema. |
| **Medida** | Em um Android (aparelho ou emulador) e em um iPhone, a Ajuda abre com e sem login, as respostas abrem e fecham, a busca filtra as perguntas, e o botão "Ver manual completo (PDF)" abre o manual no navegador do sistema. |
| **Condição** | Um Android (aparelho ou emulador) e um iPhone com o aplicativo aberto (instalado ou pelo Expo Go) e com internet. |
| **Verificação** | Roteiro CT-MAN-12, que repete CT-MAN-01 a CT-MAN-04 e CT-MAN-09 em cada aparelho. |
| **Evidência** | `src/app/ajuda.tsx` não tem código específico de plataforma. |
| **Situação** | Atendido em 13/11/2026: aprovado em iPhone 14 (iOS 27.2) e no emulador Pixel 6 do Android Studio (Android 15, API 35, x86_64), via Expo Go (CT-MAN-12). |

### RNF-MAN-08 — Versão do Expo e sistemas suportados

| Campo | Conteúdo |
| --- | --- |
| **Classificação** | Organizacional › Implementação |
| **Critério** | O aplicativo usa o Expo SDK 58, com React Native 0.88.1 e Expo Router 58, para receber as correções da versão atual do Expo. As versões mínimas de sistema são as padrão do SDK: Android 7.0 (API 24) e iOS 16.4. |
| **Medida** | O `package.json` declara `"expo": "^58.0.23"`, `"react-native": "0.88.1"` e `"expo-router": "~58.0.30"`, e a versão instalada do Expo é 58.0.23. O `app.json` não altera as versões mínimas padrão do SDK. Os testes do aplicativo terminam sem falhas, incluindo os 5 testes da busca da Ajuda. |
| **Condição** | FRONTEND na branch `homolog`. |
| **Verificação** | Conferir o `package.json`, o `package-lock.json` e o `app.json`. Executar `npx expo-doctor`, que não deve apontar dependências incompatíveis com o SDK 58, e `node --test tests/*.test.cjs` (roteiro CT-MAN-13). Executar o aplicativo nos aparelhos de teste e registrar o modelo e a versão do sistema de cada um. |
| **Evidência** | `package.json`, `package-lock.json` e `app.json` do FRONTEND; `minSdk = "24"` no React Native e plataforma mínima 16.4 em `Expo.podspec`. |
| **Situação** | Atendido em 13/11/2026: versões conferidas no commit `66806e4`, e testes automatizados aprovados (CT-MAN-13: 18 testes, 0 falhas). O aplicativo foi executado em iPhone 14 (iOS 27.2) e no emulador Pixel 6 (Android 15, API 35), via Expo Go. |

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

## Base de conferência

FRONTEND na branch `homolog`, commit `66806e4`, e DOCUMENTATION na branch `main`, conferidos em 13/11/2026.
