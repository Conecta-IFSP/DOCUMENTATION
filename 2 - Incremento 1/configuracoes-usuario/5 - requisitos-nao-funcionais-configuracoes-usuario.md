# Requisitos não funcionais — Configurações de usuário

> Padrão da disciplina · Módulo CFG · Revisado em 27/11/2026

## Resumo

Os requisitos não funcionais aparecem em dois níveis, como os funcionais. Os **de usuário** dizem, em uma frase, a qualidade esperada. Os **de sistema** completam cada um com a medida que o aprova, a condição em que vale, como foi verificado e onde está a evidência, de modo que qualquer pessoa consiga conferir. Onde ainda não houve medição, isso está escrito.

A classificação segue a árvore de Sommerville usada na disciplina: requisitos de **produto** (facilidade de uso, eficiência, confiabilidade, portabilidade), **organizacionais** (entrega, implementação, padrões) e **externos** (interoperabilidade, éticos, legais).

## Requisitos não funcionais de usuário

**RNF-CFG-01 — Senhas e códigos protegidos.** Nenhuma senha, código ou token de recuperação pode ficar guardado de forma legível no servidor.

**RNF-CFG-02 — Códigos de uso único e com validade.** Os códigos enviados por e-mail devem valer por pouco tempo e só uma vez.

**RNF-CFG-03 — Sigilo sobre a existência da conta.** A recuperação de conta não pode revelar se um e-mail está cadastrado.

**RNF-CFG-04 — Fim das sessões depois da troca de senha.** Depois de uma troca de senha, nenhum aparelho com a sessão anterior pode continuar usando a conta.

**RNF-CFG-05 — Confirmação de identidade no aparelho.** A alteração do nome, a troca de e-mail, a troca de senha e a gravação da senha recuperada devem pedir a biometria ou o PIN do aparelho.

**RNF-CFG-06 — Resposta sem esperar o e-mail.** O aplicativo não pode ficar esperando o servidor de e-mail para continuar.

**RNF-CFG-07 — Mensagens em português.** Toda mensagem mostrada nas telas deste módulo deve estar em português.

**RNF-CFG-08 — Navegação pelo teclado.** Os formulários de senha devem ser preenchidos e enviados só pelo teclado do celular.

**RNF-CFG-09 — Legibilidade nos dois temas.** As telas deste módulo devem ser legíveis no tema claro e no tema escuro.

**RNF-CFG-10 — Portabilidade.** As configurações devem funcionar em celulares Android a partir da versão 7.0, em iOS e na web.

**RNF-CFG-11 — Testabilidade.** Os testes automatizados devem passar antes de cada entrega.

## Requisitos não funcionais de sistema

### RNF-CFG-01 — Senhas e códigos protegidos

| Campo             | Conteúdo                                                                                                                                                                                                                                                          |
| ----------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Classificação** | Externo › Legal › Segurança                                                                                                                                                                                                                                       |
| **Critério**      | A senha, os códigos e o token de redefinição só são gravados depois de passar por uma função de hash, e os hashes não saem do servidor.                                                                                                                          |
| **Medida**        | `senha_hash` contém um hash bcrypt de custo 10; `reset_senha_codigo_hash`, `reset_senha_token_hash` e `email_pendente_codigo_hash` contêm SHA-256 de 64 caracteres hexadecimais; os quatro têm `select: false` e não aparecem nas respostas de `GET /usuarios/me` e `PATCH /usuarios/me`. |
| **Condição**      | Troca de senha, troca de e-mail e recuperação de conta.                                                                                                                                                                                                           |
| **Verificação**   | Inspeção de `usuario.schema.ts`, `UsuariosService.alterarSenha` e `AuthService.hashToken`, em 27/11/2026; teste CT-CFG-09, que confere a gravação depois da troca.                                                                                                |
| **Evidência**     | `BACKEND/src/usuarios/schemas/usuario.schema.ts`, `BACKEND/src/usuarios/usuarios.service.ts` e `BACKEND/src/auth/auth.service.ts`.                                                                                                                                |
| **Situação**      | Atendido por inspeção em 27/11/2026. Não há teste automatizado que leia o documento gravado no MongoDB.                                                                                                                                                           |

### RNF-CFG-02 — Códigos de uso único e com validade

| Campo             | Conteúdo                                                                                                                                                                                                                                         |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Classificação** | Externo › Legal › Segurança                                                                                                                                                                                                                      |
| **Critério**      | O código da troca de e-mail e o da recuperação são sorteados, valem por tempo limitado e só podem ser usados uma vez.                                                                                                                            |
| **Medida**        | Código de 7 caracteres entre 36 possíveis (cerca de 78 bilhões de combinações), aceito até 30 minutos depois do pedido e apagado no primeiro uso certo; um novo pedido invalida o anterior. O token de redefinição tem 64 caracteres hexadecimais, é aceito uma vez e até o fim dos mesmos 30 minutos. |
| **Condição**      | Rotas `POST /usuarios/me/email/confirmar`, `POST /auth/verificar-recuperacao` e `POST /auth/redefinir-senha`.                                                                                                                                    |
| **Verificação**   | Inspeção das constantes `DURACAO_CODIGO_VERIFICACAO_MS` e `DURACAO_TOKEN_RECUPERACAO_MS` e dos métodos de `AuthService`, em 27/11/2026; roteiros CT-CFG-02 e CT-CFG-04.                                                                          |
| **Evidência**     | `BACKEND/src/auth/auth.service.ts`.                                                                                                                                                                                                              |
| **Situação**      | Atendido por inspeção. A recusa depois de 30 minutos não foi cronometrada em teste. Não há limite de tentativas erradas por conta: limitação conhecida, não implementada.                                                                        |

### RNF-CFG-03 — Sigilo sobre a existência da conta

| Campo             | Conteúdo                                                                                                                                                                                                  |
| ----------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Classificação** | Externo › Legal › Segurança                                                                                                                                                                               |
| **Critério**      | O pedido de recuperação responde da mesma forma para e-mail cadastrado, não cadastrado ou de conta desativada.                                                                                            |
| **Medida**        | `POST /auth/recuperar-conta` devolve sempre 201 com "Se existir uma conta ativa para este e-mail, enviaremos as instruções de recuperação.", e o e-mail só é enviado à conta ativa.                        |
| **Condição**      | Recuperação de conta. Na troca de e-mail, que exige sessão aberta, a API informa "Já existe um usuário com este email", porque a pessoa precisa saber por que o endereço foi recusado.                    |
| **Verificação**   | Inspeção de `AuthService.solicitarRecuperacao`, em 27/11/2026; CT-CFG-12 confere a mensagem com conta existente.                                                                                          |
| **Evidência**     | `BACKEND/src/auth/auth.service.ts`.                                                                                                                                                                       |
| **Situação**      | Atendido por inspeção em 27/11/2026. Não há teste automatizado com e-mail não cadastrado.                                                                                                                 |

### RNF-CFG-04 — Fim das sessões depois da troca de senha

| Campo             | Conteúdo                                                                                                                                                                                                                                                                  |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Classificação** | Externo › Legal › Segurança                                                                                                                                                                                                                                               |
| **Critério**      | Depois de uma troca ou redefinição de senha, nenhuma sessão aberta antes continua usando a conta.                                                                                                                                                                         |
| **Medida**        | Toda requisição autenticada com token de `iat` anterior a `senha_alterada_em` (em segundos) recebe 401 "A senha foi alterada. Entre novamente.". Com o aplicativo aberto em primeiro plano, o outro aparelho mostra o login em até 30 segundos, o intervalo da consulta periódica do `SessaoContext`. |
| **Condição**      | Tokens emitidos até o segundo anterior à troca; o aparelho que trocou a senha sai na hora.                                                                                                                                                                               |
| **Verificação**   | Teste automatizado CT-CFG-08 e roteiro CT-CFG-10 no Galaxy S24.                                                                                                                                                                                                           |
| **Evidência**     | `BACKEND/src/auth/jwt-auth.guard.ts`, `FRONTEND/src/contexts/SessaoContext.tsx` (intervalo de 30.000 ms) e `FRONTEND/src/services/api.ts`.                                                                                                                                 |
| **Situação**      | Recusa dos tokens atendida em 26/11/2026. O tempo até o outro aparelho mostrar o login não foi cronometrado.                                                                                                                                                              |

### RNF-CFG-05 — Confirmação de identidade no aparelho

| Campo             | Conteúdo                                                                                                                                                                                                                                                         |
| ----------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Classificação** | Externo › Legal › Segurança                                                                                                                                                                                                                                      |
| **Critério**      | Salvar o novo nome, abrir "Trocar e-mail", abrir "Alterar senha" e gravar a senha recuperada exigem a biometria, o Face ID ou o PIN do aparelho, quando ele tem bloqueio de tela.                                                                                                    |
| **Medida**        | Em aparelho com bloqueio, cancelar o pedido do aparelho não grava o nome (a folha mostra "Não foi possível confirmar sua identidade."), não abre a tela e não envia a senha. Em aparelho sem bloqueio e na web, o fluxo segue sem pedido.                                                                                                         |
| **Condição**      | Botão "Salvar" da folha "Editar nome", linhas "E-mail" e "Senha" da tela "Perfil" e botão "Salvar senha" da recuperação. Escolher o tema não pede confirmação (não implementado; ver RF-CFG-05).                                                                                                  |
| **Verificação**   | Inspeção de `autenticacao-local.ts`, `TelaPerfil.salvarNome`, `TelaPerfil.abrirProtegida` e `TelaRecuperarConta.redefinir`; roteiro CT-CFG-17 para o nome. A mesma função `autenticarComDispositivo` foi verificada nas atas, no emulador Android 16 com PIN (CT-ATA-17) e no Galaxy S24 com digital (CT-ATA-25).        |
| **Evidência**     | `FRONTEND/src/services/autenticacao-local.ts`, `FRONTEND/src/app/perfil.tsx` e `FRONTEND/src/app/login/recuperar-conta.tsx`.                                                                                                                                      |
| **Situação**      | Implementado. O pedido ao salvar o nome foi incluído em 26/11/2026; o roteiro CT-CFG-17 foi executado em parte no mesmo dia, no Galaxy S24: o cancelamento do pedido não gravou o nome, e a confirmação com a digital ou o PIN não foi executada. Para e-mail e senha, roteiro próprio deste módulo com cancelamento do pedido não registrado; iOS não executado.                                                                                                                                                       |

### RNF-CFG-06 — Resposta sem esperar o e-mail

| Campo             | Conteúdo                                                                                                                                                                                                  |
| ----------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Classificação** | Produto › Eficiência › Desempenho                                                                                                                                                                         |
| **Critério**      | As rotas que mandam código por e-mail respondem sem esperar o servidor de e-mail terminar o envio.                                                                                                        |
| **Medida**        | `solicitarAlteracaoEmail` e `solicitarRecuperacao` devolvem a resposta mesmo quando o `sendMail` nunca termina. A conexão SMTP tem limites de 10 s para conectar, 10 s para a saudação e 20 s sem dados. |
| **Condição**      | Servidor de e-mail lento, fora do ar ou recusando a autenticação.                                                                                                                                         |
| **Verificação**   | Teste automatizado CT-CFG-12.                                                                                                                                                                             |
| **Evidência**     | `AuthService.enviarEmail` e `AuthService.criarTransportador`, em `BACKEND/src/auth/auth.service.ts`.                                                                                                      |
| **Situação**      | Atendido em 27/11/2026. O tempo de resposta em milissegundos não foi medido.                                                                                                                              |

### RNF-CFG-07 — Mensagens em português

| Campo             | Conteúdo                                                                                                                                                                                                  |
| ----------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Classificação** | Produto › Facilidade de uso                                                                                                                                                                               |
| **Critério**      | Todo texto mostrado ao usuário neste módulo, inclusive os erros de validação vindos da API, está em português do Brasil.                                                                                 |
| **Medida**        | Todos os decoradores de validação dos 7 DTOs do módulo têm mensagem própria em português; nenhuma mensagem padrão do class-validator, que é em inglês, chega à tela. Toda exceção lançada pelos serviços e pelo `JwtAuthGuard` tem texto em português. |
| **Condição**      | Entradas inválidas nas telas "Perfil", "Trocar e-mail", "Alterar senha" e "Recuperar conta", e chamadas com JSON válido feitas direto à API.                                                             |
| **Verificação**   | Inspeção dos DTOs, de `UsuariosService`, `AuthService` e `JwtAuthGuard`, em 27/11/2026; CT-CFG-11 confere a recusa da senha fraca.                                                                       |
| **Evidência**     | `BACKEND/src/usuarios/dto/*.ts`, `BACKEND/src/auth/dto/*.ts` e `FRONTEND/src/utils/validacao.ts`.                                                                                                         |
| **Situação**      | Atendido por inspeção em 27/11/2026.                                                                                                                                                                      |

### RNF-CFG-08 — Navegação pelo teclado

| Campo             | Conteúdo                                                                                                                                                           |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Classificação** | Produto › Facilidade de uso                                                                                                                                        |
| **Critério**      | Nos formulários de senha, a tecla de avanço do teclado leva ao campo seguinte e a tecla "ir" do último campo envia o formulário.                                   |
| **Medida**        | Na tela "Alterar senha", os três campos são preenchidos e enviados sem tocar fora do teclado. Na etapa da senha de "Recuperar conta", o mesmo vale para os dois campos. |
| **Condição**      | Teclado virtual do Android.                                                                                                                                        |
| **Verificação**   | Roteiro CT-CFG-07, no emulador Android 16, em 10/11/2026.                                                                                                          |
| **Evidência**     | `returnKeyType` e `setFocus` em `FRONTEND/src/app/login/alterar-senha.tsx` e `FRONTEND/src/app/login/recuperar-conta.tsx`.                                         |
| **Situação**      | Atendido em "Alterar senha" em 10/11/2026. Em "Recuperar conta", implementado e não verificado em aparelho.                                                         |

### RNF-CFG-09 — Legibilidade nos dois temas

| Campo             | Conteúdo                                                                                                                                                                  |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Classificação** | Produto › Facilidade de uso                                                                                                                                               |
| **Critério**      | As telas deste módulo funcionam no tema claro e no escuro, sem cor fixa no código.                                                                                       |
| **Medida**        | Nenhuma cor literal (hexadecimal ou `rgb`) nas telas e componentes do módulo: todas vêm de `useTema().tema.cores`.                                                       |
| **Condição**      | Tema do aplicativo em Claro, em Escuro e em Sistema.                                                                                                                      |
| **Verificação**   | Busca por cores literais em `perfil.tsx`, `alterar-senha.tsx`, `trocar-email.tsx`, `recuperar-conta.tsx`, `CampoTexto.tsx` e `FaixaAviso.tsx`, sem ocorrência, em 27/11/2026; roteiro CT-CFG-05. |
| **Evidência**     | `FRONTEND/src/contexts/TemaContext.tsx` e as telas citadas.                                                                                                               |
| **Situação**      | Atendido por inspeção em 27/11/2026. A conferência visual nos dois temas está registrada só no CT-CFG-05, sem data nem aparelho.                                         |

### RNF-CFG-10 — Portabilidade

| Campo             | Conteúdo                                                                                                                                                         |
| ----------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Classificação** | Produto › Portabilidade                                                                                                                                          |
| **Critério**      | Os seis fluxos do módulo funcionam do Android 7.0 (API 24, o mínimo do aplicativo) ao Android 16, no iOS e na web.                                               |
| **Medida**        | Alterar nome, trocar e-mail, alterar senha, recuperar conta, escolher tema e sair, sem erro e sem texto cortado.                                                  |
| **Condição**      | Emulador Android 16, Galaxy S24 com Android 16, emulador Android 7.0, aparelho iOS e navegador.                                                                  |
| **Verificação**   | Roteiros CT-CFG-02 a CT-CFG-04 no Android 16, CT-CFG-07 no emulador Android 16, CT-CFG-10 no Galaxy S24 e CT-CFG-15 no iOS.                                                   |
| **Evidência**     | Registros de 10/11/2026 e 26/11/2026 em [Testes](7%20-%20testes-configuracoes-usuario.md).                                                                       |
| **Situação**      | Atendido no Android 16. Android 7.0 e web sem registro de execução neste módulo; iOS não executado por falta de equipamento.                                    |

### RNF-CFG-11 — Testabilidade

| Campo             | Conteúdo                                                                                                                                         |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Classificação** | Organizacional › Entrega                                                                                                                         |
| **Critério**      | As suítes automatizadas do backend e do aplicativo passam inteiras antes de cada entrega.                                                        |
| **Medida**        | `npx vitest run` no BACKEND e `node --test tests/*.test.cjs` no FRONTEND sem nenhuma falha; `npm run lint`, `npm run build` e `npx tsc --noEmit` sem erro. |
| **Condição**      | Branch `homolog`: backend `d1f1aeb` e frontend `10abe49`.                                                                                        |
| **Verificação**   | Execução registrada em [Testes](7%20-%20testes-configuracoes-usuario.md).                                                                        |
| **Evidência**     | Na execução de 27/11/2026: BACKEND com 17 arquivos e 246 testes aprovados e `npm run lint` sem erro; FRONTEND com 5 arquivos e 47 testes aprovados e `npx tsc --noEmit` e `npm run lint` sem erro. |
| **Situação**      | Atendido em 27/11/2026 nos testes, no lint e na checagem de tipos. O `npm run build` do backend foi executado por último em 26/11/2026, sem erro. |

## Rastreabilidade

RNF-CFG-01 a RNF-CFG-11 cobrem RF-CFG-01 a RF-CFG-06 e são verificados pela matriz CT-CFG.
