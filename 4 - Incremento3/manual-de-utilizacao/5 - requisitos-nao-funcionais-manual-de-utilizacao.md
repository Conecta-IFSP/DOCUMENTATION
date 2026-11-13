# Requisitos Não Funcionais - Manual de Utilização (R4)

---

## Visão Geral

Os requisitos não funcionais abaixo definem critérios de disponibilidade, manutenibilidade, portabilidade e usabilidade para o manual de utilização: a tela Ajuda e o manual completo em PDF. Cada requisito tem um critério de aceitação que pode ser verificado.

| Código | Categoria | Requisito | Prioridade |
|---|---|---|---|
| RNF-MAN-01 | Disponibilidade | A Ajuda deve estar acessível sem login. | Alta |
| RNF-MAN-02 | Disponibilidade | A Ajuda deve funcionar sem conexão com a internet. | Alta |
| RNF-MAN-03 | Manutenibilidade | O endereço do manual em PDF não deve mudar quando o arquivo for atualizado. | Alta |
| RNF-MAN-04 | Portabilidade | O manual em PDF deve usar o tamanho de página A4. | Média |
| RNF-MAN-05 | Usabilidade | O conteúdo deve usar linguagem simples, com respostas curtas e os nomes que aparecem na tela. | Alta |
| RNF-MAN-06 | Usabilidade | O conteúdo deve ser organizado por funcionalidade e indicar o perfil a que cada orientação se aplica. | Média |
| RNF-MAN-07 | Portabilidade | A Ajuda e o manual devem estar disponíveis em Android e iOS. | Alta |
| RNF-MAN-08 | Manutenibilidade | O aplicativo deve usar o Expo SDK 58 e funcionar a partir do Android 7.0 e do iOS 16.4. | Alta |
| RNF-MAN-09 | Manutenibilidade | O conteúdo da Ajuda deve poder ser alterado apenas em `src/constants/ajuda.ts`. | Média |

---

# RNF-MAN-01 - Acesso Sem Login

| Campo | Descrição |
|---|---|
| **Categoria** | Disponibilidade |
| **Descrição** | A tela Ajuda deve poder ser aberta por quem ainda não fez login, para que a pessoa consiga tirar dúvidas sobre como entrar ou recuperar a conta. A rota `ajuda` é declarada fora dos grupos protegidos por sessão (`src/app/_layout.tsx`), e a tela Entrar tem o link "Precisa de ajuda?", que a abre no tópico Conta e acesso. |
| **Critério de Aceitação** | Sem sessão ativa, o toque em "Precisa de ajuda?" abre a tela Ajuda, sem redirecionar para a tela Entrar. A tela exibe todos os 7 tópicos, abre a pergunta "Esqueci minha senha. O que faço?" e mostra o botão "Ver manual completo (PDF)". Com sessão ativa, a mesma tela é aberta por Perfil → Suporte → Ajuda. |
| **Forma de Verificação** | Sair da conta, tocar em "Precisa de ajuda?" na tela Entrar e conferir a tela apresentada. Em seguida, fazer login e abrir a Ajuda pelo Perfil. |
| **Aplicação** | RF-MAN-01 e RF-MAN-02 |
| **Prioridade** | Alta |

---

# RNF-MAN-02 - Funcionamento Sem Internet

| Campo | Descrição |
|---|---|
| **Categoria** | Disponibilidade |
| **Descrição** | As perguntas e respostas da Ajuda devem fazer parte do aplicativo instalado (`TOPICOS_AJUDA`, em `src/constants/ajuda.ts`), sem depender do servidor. A tela não deve enviar requisições à API. A única função que precisa de internet é a abertura do manual em PDF. |
| **Critério de Aceitação** | Com o aparelho em modo avião, a tela Ajuda abre e exibe os 7 tópicos e as 42 perguntas. As respostas abrem e fecham, e a busca filtra as perguntas sem nenhuma mensagem de erro de conexão. |
| **Forma de Verificação** | Ativar o modo avião, abrir a Ajuda, abrir algumas respostas e buscar uma palavra, por exemplo "ata". Por inspeção, confirmar que `src/app/ajuda.tsx` e `src/utils/busca-ajuda.ts` não chamam `api()`. |
| **Aplicação** | RF-MAN-01 |
| **Prioridade** | Alta |

---

# RNF-MAN-03 - Endereço Fixo do Manual em PDF

| Campo | Descrição |
|---|---|
| **Categoria** | Manutenibilidade |
| **Descrição** | O manual deve ser versionado na pasta `4 - Incremento3/manual-de-utilizacao/manual-usuario/` do repositório `Conecta-IFSP/DOCUMENTATION`, branch `main`, sempre com o nome `Manual-do-Usuario-Conecta.pdf`. O endereço salvo no aplicativo (`URL_MANUAL_PDF`) é formado só pela branch, pelo caminho e pelo nome do arquivo. Assim, substituir o arquivo não muda o endereço e não exige uma nova versão do aplicativo. |
| **Critério de Aceitação** | Depois de substituir o arquivo no mesmo caminho e enviar o commit para a `main`, o endereço `https://github.com/Conecta-IFSP/DOCUMENTATION/blob/main/4%20-%20Incremento3/manual-de-utilizacao/manual-usuario/Manual-do-Usuario-Conecta.pdf` exibe o arquivo novo, com resposta final `200`, e a versão do aplicativo já instalada abre esse arquivo sem nenhuma alteração no código. |
| **Forma de Verificação** | Substituir o PDF na pasta, fazer commit e push na `main`, consultar o endereço com `curl -sIL <endereço>` e conferir a resposta `200`. Em seguida, tocar em "Ver manual completo (PDF)" no aplicativo e conferir que a versão nova é aberta. |
| **Aplicação** | RF-MAN-02 |
| **Prioridade** | Alta |

---

# RNF-MAN-04 - Manual em Formato A4

| Campo | Descrição |
|---|---|
| **Categoria** | Portabilidade |
| **Descrição** | O manual em PDF deve ser gerado em páginas A4 (210 × 297 mm), com margens de 20 mm em cima e embaixo e de 18 mm nas laterais, para que possa ser lido na tela e impresso sem cortes. O arquivo é mantido em `4 - Incremento3/manual-de-utilizacao/manual-usuario/Manual-do-Usuario-Conecta.pdf`, no repositório `Conecta-IFSP/DOCUMENTATION`. |
| **Critério de Aceitação** | Todas as páginas do arquivo publicado medem 595 × 842 pontos (A4). Exceto na capa, que ocupa a página inteira, as margens são de 20 mm em cima e embaixo e de 18 mm à esquerda e à direita, e nenhum texto, tabela ou imagem ultrapassa essas margens. |
| **Forma de Verificação** | Executar `pdfinfo Manual-do-Usuario-Conecta.pdf` e conferir "Page size: 595.28 x 841.89 pts (A4)", ou abrir as propriedades do documento em um leitor de PDF. Imprimir uma página de teste e medir as margens. |
| **Aplicação** | RF-MAN-02 |
| **Prioridade** | Média |

---

# RNF-MAN-05 - Linguagem Simples

| Campo | Descrição |
|---|---|
| **Categoria** | Usabilidade |
| **Descrição** | A Ajuda e o manual devem ser escritos para pessoas sem conhecimento técnico, com frases curtas, uma ação por passo e os mesmos nomes de botões e telas que aparecem no aplicativo. As regras de escrita estão no início de `src/constants/ajuda.ts`. |
| **Critério de Aceitação** | Na Ajuda, cada resposta em lista tem no máximo 4 passos, cada um com uma única ação. Cada resposta em texto tem no máximo 3 frases, e cada tópico tem no máximo 7 perguntas. Todo nome de botão ou tela citado nas respostas e no manual existe no aplicativo, com a mesma grafia. Nenhuma resposta usa termos internos do sistema, como nomes de rotas, coleções ou campos. |
| **Forma de Verificação** | Contar os passos, as frases e as perguntas por tópico em `src/constants/ajuda.ts`. Percorrer o aplicativo comparando os nomes citados com os exibidos na tela. Pedir a uma pessoa que não conhece o aplicativo que siga três respostas, por exemplo recuperar a senha, registrar presença e votar, sem ajuda externa. |
| **Aplicação** | RF-MAN-01 e RF-MAN-02 |
| **Prioridade** | Alta |

---

# RNF-MAN-06 - Organização por Funcionalidade e por Perfil

| Campo | Descrição |
|---|---|
| **Categoria** | Usabilidade |
| **Descrição** | O conteúdo deve ser agrupado pela funcionalidade do aplicativo e deve indicar quando uma orientação vale só para um perfil. Na Ajuda, as perguntas ficam em 7 tópicos: Conta e acesso, Organizações e comissões, Reuniões, Presença, Ata, Votação e Relatório individual. As perguntas sobre ações exclusivas do responsável da comissão exibem o selo "Responsável", sem ficarem escondidas dos demais usuários. No manual, as seções seguem as funcionalidades, a seção "Perfis de usuário" explica cada papel, e as seções exclusivas são marcadas com [Responsável] ou [Administrador]. |
| **Critério de Aceitação** | Cada pergunta da Ajuda pertence a um único tópico, e cada tópico trata de uma única funcionalidade. Toda pergunta sobre uma ação que só o responsável da comissão pode fazer tem `perfil` igual a `responsavel` e exibe o selo; no conteúdo atual, são 9 perguntas. O leitor de tela anuncia o selo como "Exclusivo do Responsável". |
| **Forma de Verificação** | Por inspeção de `src/constants/ajuda.ts`, conferir o tópico e o perfil de cada pergunta com as permissões verificadas no servidor. Na tela, conferir o selo e, com o TalkBack ou o VoiceOver ativado, o anúncio. No manual, conferir o sumário e as marcações de perfil. |
| **Aplicação** | RF-MAN-01 e RF-MAN-02 |
| **Prioridade** | Média |

---

# RNF-MAN-07 - Disponibilidade em Android e iOS

| Campo | Descrição |
|---|---|
| **Categoria** | Portabilidade |
| **Descrição** | A tela Ajuda e a abertura do manual devem ser desenvolvidas uma única vez em React Native com Expo e funcionar em Android e iOS. A abertura do PDF usa `Linking.openURL`, que entrega o endereço ao navegador ou ao leitor de PDF de cada sistema. |
| **Critério de Aceitação** | Em um aparelho Android e em um iPhone, a Ajuda abre com e sem login. As respostas abrem e fecham, a busca filtra as perguntas, e o botão "Ver manual completo (PDF)" abre o manual no aplicativo padrão do sistema. |
| **Forma de Verificação** | Em um aparelho Android e em um iPhone: sem login, abrir a Ajuda por "Precisa de ajuda?"; com login, abrir por Perfil → Suporte → Ajuda; abrir e fechar uma resposta; buscar uma palavra, por exemplo "ata"; tocar em "Ver manual completo (PDF)" e conferir que o manual abre. |
| **Aplicação** | RF-MAN-01 e RF-MAN-02 |
| **Prioridade** | Alta |

---

# RNF-MAN-08 - Versão do Expo e Sistemas Suportados

| Campo | Descrição |
|---|---|
| **Categoria** | Manutenibilidade |
| **Descrição** | O aplicativo deve usar o Expo SDK 58, com React Native 0.88.1 e Expo Router 58, para receber as correções da versão atual do Expo. As versões mínimas de sistema são as padrão do SDK, sem alteração no `app.json`: Android 7.0 (API 24), definido em `minSdk = "24"` do React Native e no plugin de autolinking do Expo, e iOS 16.4, definido na plataforma mínima de `Expo.podspec`. |
| **Critério de Aceitação** | O `package.json` declara `"expo": "^58.0.23"`, `"react-native": "0.88.1"` e `"expo-router": "~58.0.30"`, e a versão instalada do Expo é 58.0.23. O `app.json` não altera as versões mínimas padrão do SDK. O aplicativo é executado nos aparelhos de teste do grupo, Android e iPhone, e os fluxos do R4 funcionam neles. Os testes do aplicativo terminam sem falhas, incluindo os 5 testes da busca da Ajuda. |
| **Forma de Verificação** | Conferir o `package.json` e o `app.json` e executar `npx expo-doctor`, que não deve apontar dependências incompatíveis com o SDK 58. Executar `node --test tests/*.test.cjs` no aplicativo. Executar o aplicativo nos aparelhos de teste e registrar o modelo e a versão do sistema de cada um. |
| **Aplicação** | RF-MAN-01 e RF-MAN-02 |
| **Prioridade** | Alta |

---

# RNF-MAN-09 - Manutenção do Conteúdo da Ajuda

| Campo | Descrição |
|---|---|
| **Categoria** | Manutenibilidade |
| **Descrição** | O conteúdo da Ajuda deve poder ser alterado apenas em `src/constants/ajuda.ts`, sem mudança em componentes. A tela `src/app/ajuda.tsx` apenas lê `TOPICOS_AJUDA` e exibe os tópicos na ordem da lista, e a busca (`src/utils/busca-ajuda.ts`) considera automaticamente qualquer pergunta incluída. |
| **Critério de Aceitação** | Incluir, alterar ou remover um tópico ou uma pergunta exige mudança somente em `src/constants/ajuda.ts`. A tela Ajuda e a busca passam a refletir a mudança sem alteração em `ajuda.tsx`, `Sanfona.tsx` ou `busca-ajuda.ts`. O `id` do tópico `conta`, usado pelo link da tela Entrar, é mantido. |
| **Forma de Verificação** | Incluir uma pergunta de teste em `src/constants/ajuda.ts`, conferir que ela aparece na tela e é encontrada pela busca, e conferir com `git diff --stat` que apenas esse arquivo mudou. Como evidência, o commit `63d3bcb` ("ajuste das perguntas") removeu três perguntas alterando apenas `src/constants/ajuda.ts`. |
| **Aplicação** | RF-MAN-01 |
| **Prioridade** | Média |

---

## Situação da Verificação

Conferido por inspeção em 12/11/2026, no FRONTEND `homolog` (`d156cc6`): versões do Expo, do React Native e do Expo Router, `minSdk` 24, iOS 16.4, rota da Ajuda fora dos grupos protegidos, ausência de chamadas à API na tela Ajuda e limites de passos e perguntas do conteúdo atual.

| Requisito | Situação |
|---|---|
| RNF-MAN-01, RNF-MAN-02 | Atendido por inspeção; verificação no aparelho pendente. |
| RNF-MAN-03 | Atendido: o PDF foi publicado na `main` do DOCUMENTATION (`225ca39`, 13/11/2026), o endereço responde `200` e `URL_MANUAL_PDF` aponta para ele. Abertura pelo botão no aparelho pendente. |
| RNF-MAN-04 | Atendido: `pdfinfo` do arquivo publicado na `main` em 13/11/2026 informa 27 páginas de 594.96 × 841.92 pt (A4). Medição das margens em página impressa pendente. |
| RNF-MAN-05, RNF-MAN-06 | Atendido por inspeção na Ajuda; revisão do manual e teste com usuário pendentes. |
| RNF-MAN-07 | Pendente: o aplicativo ainda não foi executado em iPhone. |
| RNF-MAN-08 | Versões conferidas; execução nos aparelhos de teste pendente. |
| RNF-MAN-09 | Atendido por inspeção e pelo commit `63d3bcb`. |
