# Arquitetura e tecnologias

## 1. Arquitetura

```mermaid
flowchart LR
    U[Usuário] --> APP[Aplicativo Expo / React Native]
    APP -->|HTTP + X-API-Key + JWT quando autenticado| API[API NestJS]
    API -->|Mongoose| DB[(MongoDB)]
    API -->|SMTP| MAIL[Servidor de e-mail]
    APP -->|OAuth 2.0 / OpenID Connect| GOOGLE[Google]
    API -->|token e tokeninfo| GOOGLE
```

O frontend é uma aplicação React Native com Expo Router. O backend é uma API REST NestJS. O MongoDB persiste `usuarios`, `organizacoes`, `comissoes`, `reunioes`, `notificacoes` e `contadores`; vínculos, participantes, presenças, atas, avaliações das atas, votações, votos e os dados dos anexos são subdocumentos embutidos. Os arquivos anexados às pautas ficam no GridFS, nas coleções `anexos.files` e `anexos.chunks`. A coleção `contadores` guarda a numeração das atas por comissão e ano, no documento `atas:<comissão>:<ano>`. A coleção `notificacoes` guarda os avisos exibidos no sino do aplicativo; não há notificação push.

O `ReunioesService` executa uma rotina ao iniciar a API e a cada 60 segundos: registra a saída automática de quem marcou entrada e não marcou saída em reunião já encerrada e aprova as atas cujo prazo terminou em nome de quem não respondeu.

## 2. Camadas do frontend

| Camada      | Responsabilidade                      | Exemplos                                                         |
| ----------- | ------------------------------------- | ---------------------------------------------------------------- |
| Rotas/telas | interação, formulários e navegação    | `src/app/login`, `src/app/(abas)`, `src/app/reunioes`            |
| Componentes | elementos reutilizáveis e responsivos | `CampoTexto`, `Confirmacao`, `TelaComCabecalho`, `FolhaInferior` |
| Contextos   | sessão, tema e mensagens              | `SessaoContext`, `TemaContext`, `ToastContext`                   |
| Serviços    | HTTP, OAuth, autenticação local, anexos, PDF e CSV | `api.ts`, `google.ts`, `autenticacao-local.ts`, `anexos.ts`, `pdf.ts`, `pdfAta.ts`, `pdfListaPresenca.ts`, `exportar-relatorio-individual.ts` |
| Utilitários | validações e regras de apresentação   | `validacao.ts`, `reunioes.ts`, `sugestoes-horario.ts`, `diferenca.ts` |

Os formulários usam `react-hook-form`. O layout raiz envolve o aplicativo no `KeyboardProvider` de `react-native-keyboard-controller`, e `TelaComCabecalho` e `FundoGradiente` rolam com `KeyboardAwareScrollView`, para que o teclado não cubra o campo em foco no Android. As folhas inferiores (`FolhaInferior`) usam o `KeyboardAvoidingView` da mesma biblioteca, e não o do React Native, porque ele acompanha o teclado dentro do `Modal` com o Android em tela cheia (edge-to-edge), sem a folha tremer nem ficar atrás do teclado (FRONTEND `db5b2fe`; CT-REU-45 e CT-VOT-30). Em formulários de mais de um campo, `returnKeyType` com `setFocus` leva a tecla de avanço ao campo seguinte e a última tecla envia o formulário. Telas com dados mutáveis usam `useFocusEffect` para recarregar ao receber foco.

## 3. Camadas do backend

| Camada         | Responsabilidade                                                           |
| -------------- | -------------------------------------------------------------------------- |
| Controllers    | mapear rotas, DTOs, parâmetros e guards                                    |
| DTOs           | validar e transformar entradas com `class-validator` e `class-transformer` |
| Services       | aplicar regras de negócio e coordenar persistência                         |
| Schemas/Models | definir documentos e operações Mongoose                                    |
| Guards         | validar chave da API, JWT, conta ativa e tipo de usuário                   |

O `ValidationPipe` global usa `whitelist: true` e `transform: true`: propriedades sem decoradores são removidas e valores compatíveis podem ser transformados antes do controller.

## 4. Proteção das requisições

1. `ApiKeyGuard` global exige `X-API-Key` em todas as requisições, exceto `OPTIONS` e os dois callbacks GET do Google (`/auth/google/callback` e `/auth/google/callback-web`). A chave recebida é comparada com `API_ACCESS_TOKEN` por `timingSafeEqual`; sem a variável no servidor, a API responde 503.
2. Rotas privadas aplicam `JwtAuthGuard`, que valida o Bearer JWT, o `sub`, a existência e o estado ativo da conta, e recusa o token emitido antes de `senha_alterada_em`.
3. `AdminSistemaGuard` e `UsuarioComumGuard` restringem o tipo de conta. O `OrganizacoesController` aplica também `UsuarioAtivoGuard`, que repete a verificação de conta ativa.
4. Os services validam autorização de domínio, como ser administrador aprovado da organização ou responsável da comissão.

O CORS aceita as origens `http(s)://localhost`, `http(s)://127.0.0.1` (qualquer porta), `conectamais://` e `FRONTEND_URL`, com os métodos `GET`, `POST`, `PATCH`, `DELETE` e `OPTIONS` e os cabeçalhos `Content-Type`, `Authorization` e `X-API-Key`.

O JWT expira em **1 hora**. No aplicativo nativo ele é salvo por `expo-secure-store`, na chave `conecta_mais_token`; na web, por `localStorage`, com a mesma chave. A preferência de tema fica na chave `conecta_mais_tema`. A sessão é revalidada ao abrir o aplicativo, ao voltar ao primeiro plano e a cada 30 segundos.

As senhas são guardadas como hash `bcrypt` com custo 10. A API não limita o número de tentativas de login, de código de confirmação ou de senha no reset do sistema.

> Limite conhecido: a confirmação de identidade do aparelho (`autenticarComDispositivo`) só é pedida quando o aparelho tem biometria, PIN ou senha de bloqueio. Na web e em aparelho sem bloqueio, a ação segue sem confirmação.

> Limite conhecido: `EXPO_PUBLIC_API_ACCESS_TOKEN` é incorporado ao aplicativo e não deve ser tratado como credencial secreta. Ele bloqueia clientes sem a chave configurada, mas não substitui JWT, autorização por papel nem TLS.

## 5. Bibliotecas do frontend

Versões resolvidas pelo `package-lock.json` da branch `main`, conferidas em 27/11/2026 (o lockfile mudou pela última vez em 26/11/2026), no Expo SDK 58, com todas as bibliotecas na versão esperada pelo `expo` 58.1.0 (`npx expo install --check` sem pendências em 27/11/2026):

| Biblioteca                       |  Versão | Uso no projeto                                                     |
| -------------------------------- | ------: | ------------------------------------------------------------------ |
| Expo                             | 58.1.0  | runtime, bundler e ferramentas do aplicativo                       |
| React                            |  19.3.0 | componentes e estado                                               |
| React DOM                        |  19.3.0 | renderização da versão web                                         |
| React Native                     |  0.88.1 | interface nativa Android/iOS                                       |
| Expo Router                      | 58.0.33 | rotas por arquivos e proteção de telas                             |
| React Hook Form                  |  7.90.0 | estado, validação e envio dos formulários                          |
| React Native Keyboard Controller |  1.22.7 | ajuste da tela ao teclado e janela edge-to-edge, inclusive no Expo Go |
| Expo Secure Store                |  58.0.3 | JWT e preferência de tema no dispositivo                           |
| Expo Local Authentication        |  58.0.3 | biometria, Face ID ou PIN ao abrir o aplicativo com a sessão salva (RF-USR-04), ao marcar entrada e saída, desmarcar e corrigir presenças, enviar, aprovar e reprovar a ata e em ações de conta (nome, e-mail, senha e recuperação) |
| Expo Print                       |  58.0.9 | PDF da ata aprovada, da lista de presença e do relatório individual: arquivo no aplicativo instalado e tela de impressão no Expo Go; na web o aplicativo abre o documento numa janela e chama a impressão do navegador |
| DateTimePicker da comunidade     |   9.2.1 | seletor de hora nativo do Android e do iOS                          |
| Expo Sharing                     | 58.0.24 | entrega do PDF (aplicativo instalado) e do CSV do relatório ao menu de compartilhamento |
| Expo Document Picker             |  58.0.5 | escolha dos arquivos anexados à pauta                              |
| Expo File System                 | 58.0.15 | anexos, arquivos do PDF e do CSV do relatório no cache e leitura do manual e do PDF.js embutidos |
| Expo Auth Session                | 58.0.20 | fluxo OAuth no navegador                                           |
| Google Sign-In                   |  16.1.5 | login Google nativo                                                |
| Expo Linking                     | 58.0.17 | deep links e callbacks                                             |
| Expo Constants                   | 58.0.16 | configuração Expo e descoberta do host local                       |
| Expo Clipboard                   |  58.0.4 | cópia do código de acesso da organização e do link de presença     |
| Expo Linear Gradient             |  58.0.4 | fundos visuais                                                     |
| Expo Font                        | 58.0.12 | carregamento das fontes                                            |
| Expo Build Properties            | 58.0.12 | ajustes da compilação nativa Android                               |
| Expo Dev Client                  | 58.0.22 | execução de versões de desenvolvimento                             |
| Expo Splash Screen               | 58.0.15 | tela de abertura até as fontes carregarem                          |
| Expo Status Bar                  |  58.0.5 | cor dos ícones da barra de status conforme o tema                  |
| Expo System UI                   |  58.0.7 | tema claro ou escuro do sistema na janela nativa do Android        |
| Expo Asset                       | 58.0.21 | leitura do manual em PDF e do PDF.js embutidos no aplicativo       |
| React Native WebView             |  14.0.1 | exibição do manual em PDF dentro do aplicativo, sem internet       |
| PDF.js (`pdfjs-dist`)            | 3.11.174 | desenho das páginas do manual na WebView; arquivos copiados para `assets/pdfjs`, fora do `package.json` |
| Poppins para Expo                |   0.4.1 | família tipográfica                                                |
| Ionicons para React Native       |  14.0.0 | ícones                                                             |
| Gesture Handler                  |   3.3.0 | gestos nativos                                                     |
| Reanimated                       |   4.7.1 | animações nativas                                                  |
| React Native Worklets            |  0.13.0 | execução das animações na thread de UI, exigida pelo Reanimated 4  |
| Safe Area Context                | 5.11.1  | respeito às áreas seguras do aparelho                              |
| React Native Screens             |  4.30.1 | telas nativas usadas pela navegação                                |
| React Native Web                 |  0.21.4 | execução no navegador                                              |
| TypeScript                       |   6.0.3 | tipagem e verificação estática                                     |
| ESLint                           |  9.39.5 | análise de estilo e problemas de código                            |

## 6. Bibliotecas do backend

Versões resolvidas pelo `package-lock.json` da branch `main`, conferidas em 27/11/2026 (as versões são as mesmas desde a entrega do Incremento 2, em 23/10/2026):

| Biblioteca              |  Versão | Uso no projeto                                |
| ----------------------- | ------: | --------------------------------------------- |
| NestJS Common/Core      |  12.0.1 | framework e injeção de dependências           |
| NestJS Platform Express |  12.0.1 | servidor HTTP Express                         |
| NestJS Config           |  12.0.0 | variáveis de ambiente                         |
| NestJS JWT              |  12.0.1 | assinatura e validação do JWT                 |
| NestJS Mongoose         |  12.0.0 | integração com MongoDB                        |
| Mongoose                |   9.9.5 | schemas, consultas e persistência             |
| Driver MongoDB          |   7.5.0 | `GridFSBucket` dos anexos, instalado pelo Mongoose |
| bcrypt                  |   6.0.0 | hash e comparação de senhas, custo 10         |
| class-validator         |  0.15.1 | validação declarativa dos DTOs                |
| class-transformer       |   0.5.1 | transformação de entradas                     |
| Nodemailer              | 10.0.10 | envio de códigos de confirmação e recuperação e do link de presença |
| RxJS                    |   7.8.2 | infraestrutura reativa usada pelo NestJS      |
| reflect-metadata        |   0.2.2 | metadados dos decoradores do NestJS           |
| NestJS CLI              |  12.0.0 | compilação (`nest build`) e execução local    |
| NestJS Testing          |  12.0.1 | módulos de teste do NestJS                    |
| Vitest                  |  4.1.11 | testes automatizados                          |
| Supertest               |   7.2.2 | testes HTTP                                   |
| Oxlint                  |  1.82.0 | análise estática                              |
| Prettier                |   3.9.6 | formatação                                    |
| TypeScript              |   6.0.3 | compilação e tipagem                          |

O backend usa também os módulos nativos `node:crypto` para códigos, tokens e comparação temporal segura, `node:os` para métricas do servidor, `node:events` para esperar o fim da gravação dos anexos no GridFS e `fetch` do Node para endpoints OAuth do Google. Os anexos são lidos e gravados pelo `GridFSBucket` do driver MongoDB, exposto pelo Mongoose, e recebidos pelo `FileInterceptor` do `@nestjs/platform-express`.

## 7. Variáveis de ambiente

| Aplicação | Variável                              | Finalidade                                | Sensível?                                |
| --------- | ------------------------------------- | ----------------------------------------- | ---------------------------------------- |
| Frontend  | `EXPO_PUBLIC_API_URL`                 | endereço base da API                      | Não                                      |
| Frontend  | `EXPO_PUBLIC_API_ACCESS_TOKEN`        | valor enviado em `X-API-Key`              | Público no bundle; não usar como segredo |
| Frontend  | `EXPO_PUBLIC_GOOGLE_*`                | IDs e URIs públicas do OAuth              | Não são segredos                         |
| Backend   | `MONGODB_URI`                         | conexão MongoDB                           | Sim quando contém credenciais            |
| Backend   | `JWT_SECRET`                          | assinatura dos JWTs                       | Sim                                      |
| Backend   | `API_ACCESS_TOKEN`                    | valor esperado pelo `ApiKeyGuard`         | Sim no servidor                          |
| Backend   | `ROOT_USER_*`                         | administrador inicial                     | Sim para a senha                         |
| Backend   | `MAIL_*`                              | conexão SMTP e remetente                  | Sim para usuário/senha                   |
| Backend   | `GOOGLE_CLIENT_ID` e IDs móveis       | clientes OAuth permitidos                 | Não são segredo                          |
| Backend   | `GOOGLE_CLIENT_SECRET`                | segredo OAuth web                         | Sim                                      |
| Backend   | `FRONTEND_URL`                        | origem/callback web autorizado e base do link de presença | Não                      |
| Backend   | `PORT`                                | porta HTTP da API (padrão 3000)           | Não                                      |
| Backend   | `NODE_ENV`                            | ambiente exibido em `GET /sistema/sobre`  | Não                                      |
| Backend   | `TEST_USER_*` e `TEST_USERS_PASSWORD` | massa criada por `/sistema/popular-teste` | Senha é sensível                         |

## 8. Verificação da base

Em 09/11/2026, com Node.js 24.19.0 e npm 11.17.0, na branch `homolog` com as alterações do Incremento 3:

- backend: `npm test` com **14 arquivos e 204 testes aprovados**; `npm run build` concluído; `npm run lint` sem diagnóstico;
- frontend: `npx tsc --noEmit` sem erro e `npx eslint src` sem diagnóstico, com o arquivo de typed routes do Expo já regenerado;
- aplicativo: APK de release gerado para `x86_64` e executado nos emuladores Android 16 e Android 7.0, com o fluxo das atas conferido tela a tela.

Em 10/11/2026, com o seletor de hora nativo já incluído, um novo APK de release foi gerado e o formulário de reunião foi percorrido nos emuladores Android 16 e Android 7.0: escolha de início e término pelo relógio do sistema, "Dia inteiro", aviso de término anterior ao início e agendamento completo. No mesmo dia, com os dois emuladores bloqueados por PIN, foram conferidos a desistência do convite já aceito e o pedido de identidade antes de marcar entrada e saída.

Ainda em 10/11/2026, com um servidor SMTP local capturando as mensagens, foram percorridas no Android 16 as funções de entrada: criar conta e confirmar por código, recusa de código inválido, alterar senha, trocar e-mail, recuperar conta e recusa da senha antiga depois da redefinição. O botão "Continuar com Google" abre o fluxo da Google, que exige uma conta real e não foi concluído. O encadeamento do teclado da tela Alterar senha foi corrigido e reconferido (CT-CFG-07).

Em 12/11/2026 a base foi reconferida: backend com **15 arquivos e 213 testes aprovados**, `npm run build` concluído e `npm run lint` sem diagnóstico; frontend com `npm run typecheck` sem erro. O gancho `pretypecheck` do npm executa `FRONTEND/scripts/gerar-tipos-rotas.cjs`, que gera o arquivo de rotas tipadas do expo-router antes do `tsc`. Esse arquivo não entra no Git.

Em 25/11/2026, com as bibliotecas do Expo SDK 58 nas versões esperadas pelo `expo` 58.0.26: `npx expo install --check` sem pendências; `npx expo-doctor` com as 20 verificações aprovadas; `npm run typecheck` sem erro; `npx eslint src` sem diagnóstico; 18 testes Node do diretório `FRONTEND/tests` aprovados; `npx expo export` gerou os pacotes Android, iOS e web sem erro. No Expo Go 58.0.6, no emulador Android 14, foram abertos o login, a tela inicial, o manual dentro do aplicativo e o PDF da ata, sem erro no log.

Em 26/11/2026, o `npx expo install --check` voltou a apontar sete bibliotecas atrás da versão esperada (`expo` 58.0.26 → 58.1.0, `expo-auth-session`, `expo-dev-client`, `expo-file-system`, `expo-router`, `expo-sharing` e `expo-splash-screen`). Depois de `npx expo install --fix`: `npx expo install --check` sem pendências; `npx expo-doctor` com as 20 verificações aprovadas; `npm run typecheck` e `npm run lint` sem erro; 47 testes Node de `FRONTEND/tests/*.test.cjs` aprovados.

No mesmo dia, num Galaxy S24 com Android 16, foram percorridos no Expo Go 58.0.5 e num APK de release `arm64-v8a`: login com o teclado aberto, manual dentro do aplicativo, PDF da ata, PDF do relatório individual e pedido de identidade do aparelho ao reprovar a ata.

Em 27/11/2026, com Node.js 24.19.0 e npm 11.17.0, na branch `homolog`, na versão com o voto em branco e nulo (backend e frontend): backend com **17 arquivos e 246 testes aprovados** (`npx vitest run`), `npx tsc --noEmit -p tsconfig.build.json` sem erro e `npx oxlint src/ test/` sem diagnóstico; frontend com `npx tsc --noEmit` sem erro, `npx eslint src` sem diagnóstico, 47 testes Node de `FRONTEND/tests/*.test.cjs` aprovados e `npx expo install --check` sem pendências.

Em 23/10/2026, na entrega do Incremento 2, a base era Node.js 24.11.0 e npm 11.6.1, com 13 arquivos e 182 testes aprovados.

Nenhum resultado de desempenho foi medido nestas revisões. Metas de tempo presentes nos RNFs são critérios de aceitação, não evidência de medição. O aplicativo foi executado no iPhone pelo Expo Go: num iPhone 14 em 13/11/2026 e num iPhone 14 com iOS 27.2.1 em 27/11/2026. O build nativo para iOS não foi executado, porque a equipe não tem Mac.
