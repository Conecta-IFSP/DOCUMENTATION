# Requisitos não funcionais — Cadastro de usuário

> Padrão da disciplina · Módulo USR · Revisado em 27/11/2026

## Resumo

Os requisitos não funcionais aparecem em dois níveis, como os funcionais. Os **de usuário** dizem, em uma frase, a qualidade esperada. Os **de sistema** completam cada um com a medida que o aprova, a condição em que vale, como foi verificado e onde está a evidência, de modo que qualquer pessoa consiga conferir. Onde ainda não houve medição, isso está escrito. Os requisitos não funcionais dos vínculos do usuário com organizações, como a proteção contra gravação simultânea, estão no módulo [Cadastro de organização](../cadastro-organizacao/README.md).

A classificação segue a árvore de Sommerville usada na disciplina: requisitos de **produto** (facilidade de uso, eficiência, confiabilidade, portabilidade), **organizacionais** (entrega, implementação, padrões) e **externos** (interoperabilidade, éticos, legais).

## Requisitos não funcionais de usuário

**RNF-USR-01 — Guarda da senha e do código.** A senha e o código de confirmação nunca podem ficar gravados como foram digitados.

**RNF-USR-02 — Força da senha.** A senha de uma conta nova deve ser difícil de adivinhar.

**RNF-USR-03 — Sigilo dos dados da conta.** Os dados internos da conta, como o resumo da senha e o vínculo com o Google, não podem aparecer para quem usa o aplicativo.

**RNF-USR-04 — Validade da sessão.** A sessão deve terminar sozinha depois de um tempo e logo que a conta for desativada ou a senha trocada.

**RNF-USR-05 — Chave do aplicativo.** A API só deve atender chamadas feitas pelo aplicativo Conecta+.

**RNF-USR-06 — Resposta sem esperar o e-mail.** O cadastro não pode ficar parado esperando o servidor de e-mail.

**RNF-USR-07 — Tempo de resposta do login.** Entrar com e-mail e senha deve levar no máximo 2 segundos.

**RNF-USR-08 — Formulários usáveis no celular.** Os formulários de cadastro e de login devem ser preenchidos no celular sem que o teclado ou textos longos escondam os campos.

**RNF-USR-09 — Portabilidade.** O cadastro e o login devem funcionar em Android a partir da versão 7.0, no iOS e no navegador.

**RNF-USR-10 — Testabilidade.** Os testes automatizados devem passar antes de cada entrega.

## Requisitos não funcionais de sistema

### RNF-USR-01 — Guarda da senha e do código

| Campo             | Conteúdo                                                                                                                                                                                       |
| ----------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Classificação** | Externo › Legal › Segurança                                                                                                                                                                    |
| **Critério**      | A senha é gravada só como hash bcrypt; o código de confirmação, só como hash SHA-256, com prazo.                                                                                              |
| **Medida**        | Todo `senha_hash` é bcrypt com custo 10; `verificacao_email_codigo_hash` tem 64 caracteres hexadecimais; o código vale 30 minutos e é apagado na confirmação. Nenhum campo guarda o texto digitado. |
| **Condição**      | Cadastro local (RF-USR-01) e criação do administrador do sistema.                                                                                                                             |
| **Verificação**   | Inspeção de `UsuariosService.criar`, `UsuariosService.onApplicationBootstrap` e `AuthService.enviarCodigoVerificacao`; teste CT-USR-14 ("autentica com bcrypt e não devolve o hash").           |
| **Evidência**     | `BACKEND/src/usuarios/usuarios.service.ts` (`bcrypt.hash(…, 10)`) e `BACKEND/src/auth/auth.service.ts` (`hashToken`).                                                                          |
| **Situação**      | Atendido em 26/11/2026.                                                                                                                                                                        |

### RNF-USR-02 — Força da senha

| Campo             | Conteúdo                                                                                                                                                       |
| ----------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Classificação** | Externo › Legal › Segurança                                                                                                                                    |
| **Critério**      | Senha nova com pelo menos 8 caracteres e ao menos uma letra minúscula, uma maiúscula, um número e um símbolo, conferida na tela e na API.                       |
| **Medida**        | "senhafraca1", "SENHAFRACA1", "SenhaFraca", "SenhaFraca1" e "Se@1" são recusadas com 400; "Senha@Forte1" é aceita. A tela mostra a regra antes de a pessoa digitar. |
| **Condição**      | `POST /usuarios` e tela "Criar conta". A mesma regra vale na troca e na redefinição de senha (módulo CFG); o login não a aplica.                                |
| **Verificação**   | Teste automatizado CT-USR-11 e roteiros CT-USR-12 (APK no Galaxy S24 e chamada à API local) e CT-USR-26 (tela "Criar conta" no emulador Android 14, APK x86_64). |
| **Evidência**     | `BACKEND/src/usuarios/dto/senha-forte.ts`, `BACKEND/src/usuarios/dto/senha-forte.spec.ts` e `FRONTEND/src/utils/validacao.ts` (`validarNovaSenha`).            |
| **Situação**      | Atendido em 26/11/2026.                                                                                                                                        |

### RNF-USR-03 — Sigilo dos dados da conta

| Campo             | Conteúdo                                                                                                                                                                                                                         |
| ----------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Classificação** | Externo › Legal › Privacidade                                                                                                                                                                                                    |
| **Critério**      | As respostas da API não trazem `senha_hash`, o código de confirmação e seu prazo, `google_id` nem `avatar_url`.                                                                                                                  |
| **Medida**        | Esses campos têm `select: false` no schema; a resposta do login traz só `id`, `nome`, `email`, `tipo` e `tema`. E-mail sem conta e senha errada recebem a mesma mensagem, "Email ou senha inválidos".                           |
| **Condição**      | Toda resposta de `/auth` e de `/usuarios`.                                                                                                                                                                                       |
| **Verificação**   | Inspeção dos schemas; teste CT-USR-14 ("autentica com bcrypt e não devolve o hash").                                                                                                                                             |
| **Evidência**     | `BACKEND/src/usuarios/schemas/usuario.schema.ts` e `AuthService.login`.                                                                                                                                                          |
| **Situação**      | Atendido para os campos ocultos. Limitação conhecida: o login responde "Usuário desativado" e "Confirme seu e-mail antes de entrar" antes de conferir a senha, e o cadastro responde "Já existe um usuário com este email"; essas respostas revelam que o e-mail tem conta. |

### RNF-USR-04 — Validade da sessão

| Campo             | Conteúdo                                                                                                                                                                                         |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Classificação** | Externo › Legal › Segurança                                                                                                                                                                      |
| **Critério**      | O token vale 1 hora; a cada chamada privada a API confere se a conta ainda existe, está ativa e não trocou a senha depois da emissão do token.                                                    |
| **Medida**        | Token vencido, inválido, de conta desativada ou excluída, ou emitido antes de `senha_alterada_em`, recebe 401 e não chega ao service. No aplicativo, um 401 de rota privada apaga o token e volta à tela "Entrar"; um 401 de `/auth/login`, um 403 ou um 500 não apagam. |
| **Condição**      | Qualquer rota com `JwtAuthGuard`; revalidação do perfil a cada 30 segundos com o aplicativo aberto.                                                                                              |
| **Verificação**   | Testes CT-USR-15 (API) e CT-USR-16 (aplicativo).                                                                                                                                                 |
| **Evidência**     | `BACKEND/src/auth/auth.module.ts` (`expiresIn: '1h'`), `BACKEND/src/auth/jwt-auth.guard.ts`, `FRONTEND/src/services/api.ts` e `FRONTEND/src/contexts/SessaoContext.tsx`.                        |
| **Situação**      | Atendido em 26/11/2026.                                                                                                                                                                          |

### RNF-USR-05 — Chave do aplicativo

| Campo             | Conteúdo                                                                                                                                                                                         |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Classificação** | Externo › Legal › Segurança                                                                                                                                                                      |
| **Critério**      | Toda chamada à API, com exceção das duas rotas de volta do Google e das consultas `OPTIONS` do navegador, traz o cabeçalho `X-API-Key` igual à chave do servidor.                               |
| **Medida**        | Sem a chave, ou com chave diferente, a resposta é 401 "Chave de acesso da API inválida ou ausente"; a comparação é feita em tempo constante (`timingSafeEqual`). Sem chave configurada no servidor, 503. |
| **Condição**      | Todas as rotas, pelo guard global (`APP_GUARD`).                                                                                                                                                 |
| **Verificação**   | Inspeção do `ApiKeyGuard`. Não há teste automatizado que chame a API sem a chave: verificação pendente.                                                                                          |
| **Evidência**     | `BACKEND/src/auth/api-key.guard.ts` e `BACKEND/src/app.module.ts`.                                                                                                                               |
| **Situação**      | Critério implementado; teste pendente. A chave fica numa variável `EXPO_PUBLIC_*` e pode ser extraída do aplicativo, por isso não substitui o token de cada usuário.                             |

### RNF-USR-06 — Resposta sem esperar o e-mail

| Campo             | Conteúdo                                                                                                                                                                                                 |
| ----------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Classificação** | Produto › Eficiência › Desempenho                                                                                                                                                                        |
| **Critério**      | O envio do código não faz parte do tempo de resposta do cadastro; uma falha no envio não deixa conta provisória.                                                                                          |
| **Medida**        | Com um envio que nunca termina, `enviarCodigoVerificacao` conclui; com o envio falhando, a conta não confirmada é apagada. Na mesma técnica aplicada à recuperação de conta, com servidor de e-mail inalcançável, a resposta caiu de 10,3 s para 0,26 s (CT-USR-10). |
| **Condição**      | `POST /usuarios` com servidor de e-mail lento, fora do ar ou recusando a mensagem.                                                                                                                       |
| **Verificação**   | Testes CT-USR-08, CT-USR-09 e CT-USR-13; medição CT-USR-10, feita em `POST /auth/recuperar-conta`. O tempo de `POST /usuarios` com servidor de e-mail inalcançável não foi medido.                     |
| **Evidência**     | `AuthService.enviarEmail` (envio sem `await`) e `BACKEND/src/auth/auth.service.spec.ts` ("E-mails assíncronos").                                                                                        |
| **Situação**      | Atendido em 26/11/2026 pelos testes automatizados; medição de tempo do cadastro pendente.                                                                                                                |

### RNF-USR-07 — Tempo de resposta do login

| Campo             | Conteúdo                                                                                                                                                                               |
| ----------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Classificação** | Produto › Eficiência › Desempenho                                                                                                                                                      |
| **Critério**      | `POST /auth/login` com credenciais corretas responde rápido o bastante para não fazer a pessoa esperar.                                                                                |
| **Medida**        | Média de até 2 segundos em 10 chamadas seguidas, medida do envio da requisição até o fim da resposta.                                                                                  |
| **Condição**      | API e MongoDB no mesmo computador de desenvolvimento, sem outra carga, chamada feita na mesma rede local; conta comum já confirmada.                                                    |
| **Verificação**   | CT-USR-25: 10 chamadas seguidas medidas com `fetch` do Node.js 24.19.0, do envio da requisição ao fim da resposta.                                                                                                                       |
| **Evidência**     | Resultado de CT-USR-25 em [Testes](7%20-%20testes-cadastro-usuario.md): 10 chamadas com 201, média de 63,0 ms, maior 88,1 ms e menor 54,4 ms, com a API local `6e0d3f5` e o MongoDB 8.3 no mesmo computador.                                                                                                                                                                          |
| **Situação**      | Atendido em 27/11/2026: média de 63,0 ms nas 10 chamadas, abaixo do limite de 2 segundos.                                                                                                                                     |

### RNF-USR-08 — Formulários usáveis no celular

| Campo             | Conteúdo                                                                                                                                                                                                  |
| ----------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Classificação** | Produto › Facilidade de uso                                                                                                                                                                               |
| **Critério**      | Os formulários de cadastro e de login usam `react-hook-form`, mostram o erro de cada campo em português abaixo dele, mantêm o campo em foco acima do teclado e não deixam o texto de exemplo passar da largura do campo.          |
| **Medida**        | Ao tocar no campo "Senha" da tela "Entrar" com o teclado fechado, o campo fica inteiro acima do teclado, com "Entrar" logo abaixo. O texto de exemplo é cortado com reticências em uma linha (`numberOfLines={1}`). |
| **Condição**      | Telas "Entrar", "Criar conta" e "Confirmar e-mail", com rolagem `KeyboardAwareScrollView`.                                                                                                               |
| **Verificação**   | Roteiro CT-USR-07 (Expo Go no Galaxy S24 e emulador Android 14) e CT-USR-12 (mesmo componente de campo, na troca de senha).                                                                                |
| **Evidência**     | `FRONTEND/src/components/CampoTexto.tsx`, `FRONTEND/src/components/FundoGradiente.tsx` e `FRONTEND/src/components/TelaComCabecalho.tsx`.                                                                |
| **Situação**      | Atendido no Android em 25/11/2026. iOS não verificado.                                                                                                                                                    |

### RNF-USR-09 — Portabilidade

| Campo             | Conteúdo                                                                                                                                                                       |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Classificação** | Produto › Portabilidade                                                                                                                                                        |
| **Critério**      | Cadastro, confirmação e login com senha funcionam no Android 7.0 (API 24, mínimo do aplicativo) ao 16, no iOS e na web. O login Google funciona no aplicativo instalado e na web. |
| **Medida**        | Os roteiros CT-USR-01 a CT-USR-03 concluem sem erro em cada plataforma.                                                                                                       |
| **Condição**      | APK de release, Expo Go e navegador.                                                                                                                                           |
| **Verificação**   | Roteiros CT-USR-01 a CT-USR-03 e CT-USR-07; CT-USR-24 para iOS.                                                                                                                |
| **Evidência**     | CT-USR-07 no Galaxy S24 (Android 16) e no emulador Android 14. Os roteiros CT-USR-01 a CT-USR-03 foram aprovados pela equipe sem registro de aparelho.                         |
| **Situação**      | Parcial. No Expo Go o login Google não funciona ("O login Google não funciona no Expo."). Android 7.0 sem execução registrada para este módulo. iOS não executado por falta de equipamento. |

### RNF-USR-10 — Testabilidade

| Campo             | Conteúdo                                                                                                       |
| ----------------- | -------------------------------------------------------------------------------------------------------------- |
| **Classificação** | Organizacional › Entrega                                                                                       |
| **Critério**      | As suítes do backend e do aplicativo passam inteiras antes de cada entrega.                                    |
| **Medida**        | `npx vitest run` no BACKEND e `node --test tests/*.test.cjs` no FRONTEND sem nenhuma falha.                    |
| **Condição**      | Código da branch `homolog`: backend `d1f1aeb` e frontend `10abe49`.                                            |
| **Verificação**   | Execução registrada em [Testes](7%20-%20testes-cadastro-usuario.md).                                           |
| **Evidência**     | 27/11/2026: BACKEND, 17 arquivos e 246 testes aprovados; FRONTEND, 5 arquivos e 47 testes aprovados.           |
| **Situação**      | Atendido em 27/11/2026.                                                                                        |

## Rastreabilidade

RNF-USR-01 a RNF-USR-10 cobrem RF-USR-01 a RF-USR-03 e são verificados pela matriz CT-USR.
