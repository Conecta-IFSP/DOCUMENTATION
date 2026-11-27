# Dicionário de classes — Cadastro de usuário

> Padrão da disciplina · Módulo USR · Complementa o diagrama de classes · Revisado em 27/11/2026

## Resumo

| Classe                   | Camada               | O que representa                                                                                                   |
| ------------------------ | -------------------- | ------------------------------------------------------------------------------------------------------------------ |
| `TelaCadastro`           | Aplicativo           | A tela "Criar conta": formulário de nome, e-mail, senha e confirmação.                                             |
| `TelaVerificarCadastro`  | Aplicativo           | A tela "Confirmar e-mail": recebe o código de 7 caracteres.                                                         |
| `TelaLogin`              | Aplicativo           | A tela "Entrar": login com senha e botão do Google.                                                                 |
| `GoogleCallbackWeb`      | Aplicativo           | A página da web que recebe a volta do Google e conclui o login.                                                     |
| `SessaoProvider`         | Aplicativo (contexto)| A sessão do aplicativo: guarda quem está conectado e revalida a conta.                                              |
| `TemaProvider`           | Aplicativo (contexto)| A aparência do aplicativo: guarda a preferência de tema e a aplica.                                                 |
| `ToastProvider`          | Aplicativo (contexto)| O aviso rápido (toast) mostrado por alguns segundos no alto da tela.                                                |
| `api`                    | Aplicativo (serviço) | O único ponto de saída para a API: monta a chamada com chave e token e guarda o token no aparelho.                  |
| `google`                 | Aplicativo (serviço) | A conversa com o Google: SDK nativo no celular e troca do código na web.                                            |
| `validacao`              | Aplicativo (serviço) | As regras de preenchimento conferidas na tela antes de enviar.                                                      |
| `UsuariosController`     | API                  | Recebe as rotas `/usuarios` de cadastro e de perfil e repassa ao serviço.                                           |
| `AuthController`         | API                  | Recebe as rotas `/auth` de login, confirmação e Google.                                                             |
| `UsuariosService`        | Negócio              | Cria a conta, consulta o perfil e cria o administrador do sistema quando o servidor inicia.                         |
| `AuthService`            | Negócio              | Confere senha e código, conversa com o Google, emite o token e envia e-mails.                                       |
| `ApiKeyGuard`            | API (proteção)       | Barra toda chamada sem a chave do aplicativo.                                                                       |
| `JwtAuthGuard`           | API (proteção)       | Confere o token e se a conta continua ativa.                                                                        |
| `UsuarioModel`           | Dados (model)        | O model do Mongoose da coleção `usuarios`: busca, cria, grava e apaga documentos `Usuario`.                         |
| `Usuario`                | Documento            | Uma conta de pessoa no Conecta+.                                                                                    |
| `Router`                 | Biblioteca           | O roteador do `expo-router`, que troca a tela aberta.                                                               |
| `Location`               | API do navegador     | O endereço da aba do navegador (`window.location`), usado para abrir a página do Google na web.                     |
| `GoogleSignin`           | Biblioteca           | O SDK nativo de login do Google, de `@react-native-google-signin/google-signin`.                                    |
| `bcrypt`                 | Biblioteca           | Gera e confere o hash das senhas.                                                                                   |
| `JwtService`             | Biblioteca           | O serviço do `@nestjs/jwt` que assina e confere o token de acesso.                                                  |
| `Transporter`            | Biblioteca           | O transportador do Nodemailer que envia o e-mail pelo servidor SMTP.                                                |
| `fetch`                  | Biblioteca           | A função global `fetch` do Node.js, usada pela API para falar com o Google.                                         |

As classes dos vínculos do usuário com organizações (`Organizacao`, `MembroOrganizacao`, `OrganizacoesController`, `OrganizacoesService`, os guards `UsuarioAtivoGuard` e `UsuarioComumGuard` e as telas de organização) estão documentadas no módulo [Cadastro de organização](../cadastro-organizacao/README.md).

## `Usuario`

Documento da coleção `usuarios`, definido em `BACKEND/src/usuarios/schemas/usuario.schema.ts`. Representa uma pessoa que usa o sistema. O mesmo documento serve para quem entra com senha, com Google ou com os dois.

| Atributo                        | Tipo TypeScript/Mongoose             | Tipo BSON  | Obrigatório | Regra                                                                                                                              |
| ------------------------------- | ------------------------------------ | ---------- | ----------: | ---------------------------------------------------------------------------------------------------------------------------------- |
| `_id`                           | `Types.ObjectId`                     | `objectId` |         Sim | Identificador da conta, criado na inserção. É o `sub` do token de acesso.                                                          |
| `nome`                          | `string`                             | `string`   |         Sim | Guardado sem os espaços das pontas. No cadastro, de 2 caracteres em diante (3 na tela). Na conta Google, é o nome da conta Google. |
| `email`                         | `string`                             | `string`   |         Sim | Guardado em minúsculas e sem espaços nas pontas. Único: duas contas não podem ter o mesmo e-mail.                                  |
| `senha_hash`                    | `string`                             | `string`   |         Não | Oculto nas consultas. Resultado do bcrypt com custo 10, nunca a senha. Ausente na conta criada pelo Google.                        |
| `tipo`                          | `TipoUsuario` (enum string)          | `string`   |         Sim | Padrão `USUARIO`. A conta criada pelo servidor ao iniciar (`onApplicationBootstrap`) é `ADMIN_SISTEMA`; o cadastro nunca cria esse tipo. A rota `PATCH /usuarios/:id`, reservada ao administrador do sistema, também aceita mudar o tipo, mas o aplicativo não tem tela que a use.                                     |
| `tema`                          | `TemaUsuario` (enum string)          | `string`   |         Sim | Padrão `sistema`. Alterado nas configurações (RF-CFG-05).                                                                          |
| `verificacao_email_codigo_hash` | `string`                             | `string`   |         Não | Oculto. SHA-256, em 64 caracteres hexadecimais, do código de confirmação do cadastro. Apagado na confirmação pelo código; o login Google confirma a conta sem apagá-lo. |
| `verificacao_email_expira_em`   | `Date`                               | `date`     |         Não | Oculto. Instante da gravação do código mais 30 minutos, em UTC. Apagado junto com o código.                                        |
| `email_verificado`              | `boolean`                            | `bool`     |         Sim | Padrão `true` no schema; o cadastro local grava `false` e a confirmação ou o login Google mudam para `true`. Sem ele, não há login com senha. |
| `ativo`                         | `boolean`                            | `bool`     |         Sim | Padrão `true`. Conta com `false` não entra e perde a sessão aberta.                                                                |
| `google_id`                     | `string`                             | `string`   |         Não | Oculto. Identificador da conta Google (`sub` do token do Google). Único entre as contas que o têm. Pode ser substituído quando outra conta Google com o mesmo e-mail entra (limitação do RF-USR-03). |
| `provedor_autenticacao`         | `ProvedorAutenticacao` (enum string) | `string`   |         Sim | Padrão `LOCAL`. `GOOGLE` na conta criada pelo Google; `AMBOS` quando uma conta local passa a entrar também pelo Google.            |
| `avatar_url`                    | `string`                             | `string`   |         Não | Oculto. Endereço da foto da conta Google.                                                                                          |
| `senha_alterada_em`             | `Date`                               | `date`     |         Não | Instante da última troca ou redefinição de senha (módulo CFG). Tokens emitidos antes dele são recusados.                          |
| `criado_em`                     | `Date`                               | `date`     |         Sim | Preenchido pelo Mongoose na inserção.                                                                                              |
| `atualizado_em`                 | `Date`                               | `date`     |         Sim | Atualizado pelo Mongoose a cada gravação.                                                                                          |

"Oculto" quer dizer que o campo tem `select: false`: ele não vem nas consultas, nem nas respostas da API, a menos que o código peça por ele. A classe também tem os campos ocultos e opcionais `reset_senha_token_hash`, `reset_senha_codigo_hash`, `reset_senha_expira_em`, `email_pendente`, `email_pendente_codigo_hash` e `email_pendente_expira_em`, usados pela recuperação de conta e pela troca de e-mail e descritos no módulo de configurações (CFG).

## Enumerações

| Enumeração                | Valores                                       | Onde aparece                      | Significado                                                                                           |
| ------------------------- | --------------------------------------------- | --------------------------------- | ----------------------------------------------------------------------------------------------------- |
| `TipoUsuario`             | `USUARIO`, `ADMIN_SISTEMA`                    | `Usuario.tipo`                    | Conta comum ou administrador do sistema. Quem administra uma organização é definido pelo vínculo, no módulo ORG.                  |
| `TemaUsuario`             | `claro`, `escuro`, `sistema`                  | `Usuario.tema`                    | Aparência preferida; `sistema` segue o aparelho.                                                      |
| `ProvedorAutenticacao`    | `LOCAL`, `GOOGLE`, `AMBOS`                    | `Usuario.provedor_autenticacao`   | Por onde a conta entra: senha, Google ou os dois.                                                     |

## Classes do aplicativo

| Classe                  | Atributos (estado da tela)                                                       | Métodos e o que fazem                                                                                                                                         |
| ----------------------- | -------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `TelaCadastro`          | `senhaVisivel`, `confirmacaoVisivel` (`boolean`, começam `false`); `erro` (`string`). | `criarConta()`: valida os quatro campos e chama `POST /usuarios`; no sucesso, abre `TelaVerificarCadastro` com o e-mail.                                   |
| `TelaVerificarCadastro` | `codigo` (`string`, até 7 caracteres); `erro` (`string`); `carregando` (`boolean`). | `verificar()`: exige 7 caracteres e chama `POST /auth/verificar-cadastro`; no sucesso, volta para `TelaLogin`.                                             |
| `TelaLogin`             | `senhaVisivel` (`boolean`); `erro` (`string`); `carregandoGoogle` (`boolean`).    | `entrar()`: valida e-mail e senha, chama `POST /auth/login` e `SessaoProvider.entrar`. `entrarComGoogle()`: SDK nativo no celular, `Location.assign` com o endereço do Google na web. `definirErro(valor: string): void`: troca o texto da faixa de erro (atualizador do `useState`). |
| `GoogleCallbackWeb`     | —                                                                                | `handleCallback()`: lê `code` do endereço, chama `exchangeCodeForToken` e `SessaoProvider.entrar` e abre `/inicio` com `Router.replace`; em erro, volta para `/login`. |
| `SessaoProvider`        | `usuario` (perfil ou `null`); `carregando` (`boolean`); `erro`, `aviso` (`string`). | `entrar(token)`: guarda o token e busca `GET /usuarios/me`; devolve `true` no sucesso. `atualizar()`: revalida o perfil (ao abrir, ao voltar ao app, ao trocar de tela e a cada 30 s). `sair(aviso)`: apaga o token. `definirUsuario(valor: Usuario): void`: guarda o perfil recebido (atualizador do `useState`); com `null`, a sessão volta para o login. |
| `TemaProvider`          | `preferencia` (`PreferenciaTema`: `sistema`, `claro` ou `escuro`; começa `sistema`). | `definirPreferencia(nova: PreferenciaTema): void`: aplica a aparência escolhida e a grava no aparelho. No login, a tela a chama com o `tema` da conta. |
| `ToastProvider`         | `toast` (`ToastAtual`: `id`, `mensagem` e `tipo`, ou `null`).                    | `mostrarToast(mensagem: string, tipo: TipoToast, duracao: number): void`: mostra o aviso; `tipo` é `sucesso`, `erro` ou `informacao` (padrão), e `duracao` é 4000 ms quando não informada. |
| `api`                   | Constantes `CHAVE_TOKEN = 'conecta_mais_token'`, endereço da API e chave do aplicativo, lidos das variáveis `EXPO_PUBLIC_API_URL` e `EXPO_PUBLIC_API_ACCESS_TOKEN`. | `api(caminho, metodo, corpo)`: faz o `fetch` com `X-API-Key` e `Authorization: Bearer`; devolve `{ ok: true, dados }` ou `{ ok: false, erro }`; em 401 de rota privada, apaga o token e avisa a sessão. `salvarToken`, `obterToken`, `removerToken`: `SecureStore` no celular, `localStorage` na web. |
| `google`                | Identificadores públicos dos clientes Google, lidos de variáveis `EXPO_PUBLIC_GOOGLE_*`. | `signInWithGoogleNative()`: escolha de conta pelo SDK; devolve `{ cancelled, idToken }`. `loginComIdTokenGoogle(idToken)` e `exchangeCodeForToken(code, redirectUri)`: chamam `POST /auth/google`. Funções internas: `estaNoExpoGo(): boolean` diz se o app roda no Expo Go, onde o SDK nativo não existe; `configureNativeGoogle(GoogleSignin): void` chama `GoogleSignin.configure` uma única vez. |
| `validacao`             | Constantes `TAMANHO_MINIMO_SENHA = 8` e `REGRA_SENHA`.                           | Cada função devolve o texto do erro ou `null`. Ver a tabela de validações abaixo.                                                                             |

## Operações dos services e guards

| Classe                | Método                    | Parâmetros                                          | Retorno                      | O que faz                                                                                                                     |
| --------------------- | ------------------------- | --------------------------------------------------- | ---------------------------- | ----------------------------------------------------------------------------------------------------------------------------- |
| `UsuariosService`     | `criar`                   | `CreateUsuarioDto`                                  | conta criada                 | Recusa e-mail repetido, gera o hash da senha e grava a conta não confirmada.                                                  |
| `UsuariosService`     | `buscarPorId`             | `id`                                                | conta                        | Valida o formato do id e busca a conta; 404 se não existe.                                                                    |
| `UsuariosService`     | `descartarNaoVerificado`  | `id`                                                | —                            | Apaga a conta só se ela ainda não foi confirmada.                                                                             |
| `UsuariosService`     | `onApplicationBootstrap`  | —                                                   | —                            | Na inicialização, cria o administrador do sistema com `ROOT_USER_NAME`, `ROOT_USER_EMAIL` e `ROOT_USER_PASSWORD`, se não existir nenhum administrador do sistema nem conta com esse e-mail. |
| `AuthService`         | `login`                   | `LoginDto`                                          | `{ access_token, usuario }`  | Confere conta, confirmação e senha e assina o token.                                                                          |
| `AuthService`         | `loginComGoogle`          | `GoogleLoginDto`                                    | `{ access_token, usuario }`  | Valida o token do Google, cria ou liga a conta e assina o token.                                                              |
| `AuthService`         | `enviarCodigoVerificacao` | `id`                                                | —                            | Sorteia e guarda o código e dispara o e-mail sem esperar.                                                                     |
| `AuthService`         | `verificarCadastro`       | `VerificarCadastroDto`                              | `{ mensagem }`               | Compara o código e o prazo e confirma a conta.                                                                                |
| `AuthService`         | `validarIdTokenGoogle`    | `idToken`, `clientID`                               | dados do token               | Consulta o Google e confere destinatário, emissor e validade.                                                                 |
| `AuthService`         | `gerarCodigoVerificacao`  | —                                                   | texto de 7 caracteres        | Sorteia 7 caracteres entre A–Z e 0–9.                                                                                         |
| `AuthService`         | `hashToken`               | `token`                                             | texto de 64 caracteres       | Calcula o SHA-256 em hexadecimal.                                                                                             |
| `AuthService`         | `enviarEmail`             | mensagem, ação em caso de falha                     | —                            | Envia sem esperar, pelo transportador de `criarTransportador`; na falha, registra o aviso no log com o `Logger` guardado em `registro` e executa a ação (no cadastro, apagar a conta). |
| `AuthService`         | `gerarRespostaLogin`      | conta                                               | `{ access_token, usuario }`  | Assina o token com `sub`, `email` e `tipo` e monta a resposta do login pelo Google.                                           |
| `AuthService`         | `smtpConfigurado`         | —                                                   | `boolean`                    | Diz se `MAIL_HOST`, `MAIL_USER` e `MAIL_PASS` estão configurados.                                                             |
| `AuthService`         | `criarTransportador`      | —                                                   | transportador do Nodemailer  | Cria o transportador uma vez e o reaproveita: com o servidor de e-mail configurado, usa `MAIL_HOST` e `MAIL_PORT` (padrão 587, conexão segura direta na 465), com 10 segundos para conectar e 20 segundos sem resposta; sem ele, usa um transportador que não envia a mensagem. |
| `ApiKeyGuard`         | `canActivate`             | contexto da requisição                              | `true` ou erro               | Libera `OPTIONS` e as duas rotas de volta do Google; nas demais, compara `X-API-Key` com `API_ACCESS_TOKEN` em tempo constante (`timingSafeEqual`). |
| `JwtAuthGuard`        | `canActivate`             | contexto da requisição                              | `true` ou erro               | Confere o token, a conta ativa e `senha_alterada_em`; atualiza `tipo` e `email` da requisição com os valores do banco.        |

## Model do Mongoose e bibliotecas

O `UsuarioModel` é o `Model<UsuarioDocument>` que o NestJS injeta com `@InjectModel(Usuario.name)` nos atributos `usuarioModel` (`UsuariosService` e `AuthService`) e `usuarios` (`JwtAuthGuard`). As consultas devolvem uma `Query`, concluída com `.exec()`; quando a consulta usa `.select`, os diagramas de sequência mostram isso em nota. As bibliotecas abaixo aparecem só com os métodos chamados nos diagramas de sequência.

| Classe          | Estereótipo            | Método                                          | Retorno                         | O que faz                                                                                                                   |
| --------------- | ---------------------- | ----------------------------------------------- | ------------------------------- | --------------------------------------------------------------------------------------------------------------------------- |
| `UsuarioModel`  | `<<Model Mongoose>>`   | `findOne(filtro: object)`                       | `Query<UsuarioDocument>`        | Busca a primeira conta que atende ao filtro (`{ email }`, `{ google_id }` ou `_id` com `ativo: true`); `null` se não há.     |
|                 |                        | `findById(id: string)`                          | `Query<UsuarioDocument>`        | Busca a conta pelo `_id`.                                                                                                   |
|                 |                        | `create(dados: object)`                         | `Promise<UsuarioDocument>`      | Insere a conta nova; o Mongoose preenche `_id`, `criado_em` e `atualizado_em`.                                             |
|                 |                        | `deleteOne(filtro: object)`                     | `Query<DeleteResult>`           | Apaga a conta ainda não confirmada quando o e-mail de confirmação falha.                                                   |
|                 |                        | `save()`                                        | `Promise<UsuarioDocument>`      | Chamado no documento devolvido por `findOne` ou `findById`: grava os campos alterados e atualiza `atualizado_em`.            |
| `Router`        | `<<biblioteca>>`       | `replace(rota: Href)`                           | `void`                          | Troca a tela aberta sem deixar a anterior na pilha (`expo-router` 58.0.33).                                                 |
| `Location`      | `<<API do navegador>>` | `assign(url: string)`                           | `void`                          | Abre o endereço na mesma aba do navegador (`window.location.assign`).                                                       |
| `GoogleSignin`  | `<<biblioteca>>`       | `configure(opcoes: ConfigureParams)`            | `void`                          | Informa o `webClientId` e, no iOS, o `iosClientId`.                                                                         |
|                 |                        | `hasPlayServices(opcoes: object)`               | `Promise<boolean>`              | No Android, confere o Google Play Services e oferece a atualização.                                                         |
|                 |                        | `signOut()`                                     | `Promise<null>`                 | Esquece a conta escolhida antes, para que a pessoa escolha de novo.                                                         |
|                 |                        | `signIn()`                                      | `Promise<SignInResponse>`       | Abre a escolha de conta do Google; a resposta diz se a pessoa escolheu ou cancelou.                                         |
|                 |                        | `getTokens()`                                   | `Promise<GetTokensResponse>`    | Devolve o `idToken` da conta escolhida.                                                                                     |
| `bcrypt`        | `<<biblioteca>>`       | `hash(dado: string, custo: number)`             | `Promise<string>`               | Gera o hash da senha; o cadastro usa custo 10.                                                                              |
|                 |                        | `compare(dado: string, hash: string)`           | `Promise<boolean>`              | Diz se a senha informada corresponde ao hash gravado.                                                                       |
| `JwtService`    | `<<biblioteca>>`       | `signAsync(payload: object)`                    | `Promise<string>`               | Assina o token de acesso com `sub`, `email` e `tipo`, com a chave `JWT_SECRET`.                                             |
|                 |                        | `verifyAsync(token: string)`                    | `Promise<UsuarioAutenticado>`   | Confere a assinatura e a validade do token e devolve `sub`, `email`, `tipo` e `iat`.                                        |
| `Transporter`   | `<<biblioteca>>`       | `sendMail(mensagem: SendMailOptions)`           | `Promise<SentMessageInfo>`      | Envia o e-mail pelo servidor SMTP; a promessa falha quando o servidor recusa ou não responde.                                |
| `fetch`         | `<<biblioteca>>`       | `fetch(url: string, opcoes: RequestInit)`       | `Promise<Response>`             | Faz a requisição HTTP ao Google (troca do `code` e conferência do `id_token`). A `Response` é a da Fetch API do Node.js.     |

## DTOs e validação

As regras abaixo são aplicadas pelo `ValidationPipe` da API, que também descarta campos não declarados. A coluna "Na tela" mostra a regra conferida antes pelo aplicativo.

| DTO                     | Campo          | Validação na API                                                  | Mensagem da API                                                       | Na tela                                                              |
| ----------------------- | -------------- | ----------------------------------------------------------------- | --------------------------------------------------------------------- | -------------------------------------------------------------------- |
| `CreateUsuarioDto`      | `nome`         | texto, ao menos 2 caracteres                                      | "Informe o nome", "O nome deve ter pelo menos 2 caracteres"           | `validarNome`: ao menos 3, sem espaços nas pontas                    |
|                         | `email`        | formato de e-mail                                                 | "Informe um e-mail válido"                                            | `validarEmail`                                                       |
|                         | `senha`        | texto, ao menos 8 caracteres, minúscula, maiúscula, número e símbolo (`SenhaForte`) | "Informe a senha", "A senha deve ter pelo menos 8 caracteres", "Use letra maiúscula, letra minúscula, número e símbolo na senha" | `validarNovaSenha` e `validarConfirmacaoSenha` |
|                         | `tema`         | opcional; `claro`, `escuro` ou `sistema`                          | "Tema deve ser claro, escuro ou sistema"                              | não enviado                                                          |
| `VerificarCadastroDto`  | `email`        | formato de e-mail                                                 | "Informe um e-mail válido"                                            | levado da tela de cadastro                                           |
|                         | `codigo`       | texto de exatamente 8 caracteres                                  | "Informe o código", "Código inválido"                                 | 7 letras ou algarismos, enviados como `XXX-XXXX`                     |
| `LoginDto`              | `email`        | formato de e-mail                                                 | "Informe um e-mail válido"                                            | `validarEmail`                                                       |
|                         | `senha`        | texto, ao menos 6 caracteres                                      | "Informe a senha", "A senha deve ter pelo menos 6 caracteres"         | `validarSenha`: ao menos 8                                           |
| `GoogleLoginDto`        | `code`, `redirect_uri`, `id_token` | opcionais, texto                              | "Código de autorização inválido", "Endereço de retorno inválido", "ID Token inválido" | montados pelo módulo `google`                      |
|                         | `plataforma`   | opcional; texto `web`, `ios` ou `android`                         | "Plataforma inválida", "Plataforma deve ser web, ios ou android"      | `Platform.OS`                                                        |

A senha forte usa a expressão `^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[^A-Za-z\d])`, a mesma na API (`BACKEND/src/usuarios/dto/senha-forte.ts`) e na tela (`FRONTEND/src/utils/validacao.ts`). O login não a aplica, para que contas criadas antes dela continuem entrando.

## Rastreabilidade

RF-USR-01 a RF-USR-03; [Diagrama de classes](3%20-%20diagrama-classes-cadastro-usuario.md) e [Dicionário relacional](4%20-%20dicionario-dados-relacional-cadastro-usuario.md).
