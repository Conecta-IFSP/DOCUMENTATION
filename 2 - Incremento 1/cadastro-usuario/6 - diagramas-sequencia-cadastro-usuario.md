# Diagramas de sequência — Cadastro de usuário

> Padrão da disciplina · Módulo USR · Revisado em 27/11/2026

## Resumo

Há um diagrama para cada requisito de [Requisitos funcionais](2%20-%20requisitos-funcionais-cadastro-usuario.md), com o mesmo número. As linhas de vida são classes e módulos que existem no código e estão no [Diagrama de classes](3%20-%20diagrama-classes-cadastro-usuario.md): as telas do aplicativo, os contextos `SessaoProvider`, `TemaProvider` e `ToastProvider`, os módulos `api` e `google`, os controllers, os services, o `JwtAuthGuard`, o model do Mongoose `UsuarioModel` e as bibliotecas chamadas diretamente (`Router`, `Location`, `GoogleSignin`, `bcrypt`, `JwtService`, `Transporter` e `fetch`). O Google, no login pela web, é um sistema externo e aparece como ator. Toda mensagem é a chamada de um método existente, com seus parâmetros, e as mensagens são numeradas na ordem em que acontecem. Regras que não são métodos, como comparações de situação, aparecem em notas, assim como as rotas HTTP. Os retornos são identificadores; uma nota diz o que cada um traz quando isso não é evidente.

Em todos os diagramas, a chamada do módulo `api` ao controller é uma requisição HTTP que passa antes pelo `ApiKeyGuard`; o SD-USR-02 mostra em detalhe o `JwtAuthGuard`, que protege `GET /usuarios/me`. O `resultado` devolvido pelo módulo `api` é o tipo `Resultado`: `ok` verdadeiro com `dados`, ou `ok` falso com `erro`. Os diagramas dos vínculos do usuário com organizações estão no módulo [Cadastro de organização](../cadastro-organizacao/README.md).

## SD-USR-01 — Cadastrar e confirmar conta local

```mermaid
sequenceDiagram
    autonumber
    actor V as Visitante
    participant TC as TelaCadastro
    participant TO as ToastProvider
    participant R as Router
    participant TV as TelaVerificarCadastro
    participant A as api
    participant UC as UsuariosController
    participant US as UsuariosService
    participant B as bcrypt
    participant AC as AuthController
    participant AS as AuthService
    participant M as UsuarioModel
    participant TR as Transporter
    V->>TC: criarConta()
    Note over TC: validarNome, validarEmail, validarNovaSenha<br/>e validarConfirmacaoSenha
    TC->>A: api("/usuarios", "POST", corpo)
    Note over A,UC: POST /usuarios, sem token<br/>ValidationPipe aplica CreateUsuarioDto
    A->>UC: criar(createUsuarioDto)
    UC->>US: criar(createUsuarioDto)
    US->>M: findOne({ email })
    M-->>US: null
    US->>B: hash(senha, 10)
    B-->>US: senhaHash
    US->>M: create(dados)
    M-->>US: usuario
    US->>M: findById(usuario._id)
    M-->>US: usuario
    US-->>UC: usuario
    UC->>AS: enviarCodigoVerificacao(id)
    AS->>M: findById(id)
    M-->>AS: usuario
    AS->>AS: gerarCodigoVerificacao()
    AS->>AS: hashToken(codigo)
    AS->>M: save()
    M-->>AS: usuario
    AS->>AS: enviarEmail(mensagem, aoFalhar)
    AS->>AS: criarTransportador()
    AS-)TR: sendMail(mensagem)
    Note over TR: entrega a mensagem ao servidor SMTP de MAIL_HOST, sem a API esperar
    AS-->>UC: concluído
    UC-->>A: 201 mensagem
    A-->>TC: resultado
    Note over TC,A: resultado.ok verdadeiro
    TC->>TO: mostrarToast(mensagem, "sucesso")
    Note over TC,R: destino = pathname /login/verificar-cadastro e params com o email normalizado
    TC->>R: replace(destino)
    Note over R,TV: o Router abre TelaVerificarCadastro
    opt servidor de e-mail recusa ou não responde
        TR--)AS: erro
        AS->>M: deleteOne(filtro)
        Note over AS,M: filtro = _id da conta e email_verificado false
    end
    V->>TV: verificar()
    Note over TV: exige 7 caracteres e envia no formato XXX-XXXX
    TV->>A: api("/auth/verificar-cadastro", "POST", corpo)
    Note over A,AC: POST /auth/verificar-cadastro
    A->>AC: verificarCadastro(dto)
    AC->>AS: verificarCadastro(dto)
    AS->>M: findOne({ email })
    Note over AS,M: select inclui os campos ocultos<br/>verificacao_email_codigo_hash e verificacao_email_expira_em
    M-->>AS: usuario
    AS->>AS: hashToken(codigo)
    Note over AS: exige conta não confirmada, hash igual ao gravado<br/>e verificacao_email_expira_em depois do horário atual
    AS->>M: save()
    Note over AS,M: grava email_verificado true e apaga o hash e o prazo do código
    M-->>AS: usuario
    AS-->>AC: mensagem
    AC-->>A: 201 mensagem
    A-->>TV: resultado
    Note over TV,A: resultado.ok verdadeiro
    TV->>TO: mostrarToast(mensagem, "sucesso", 5000)
    TV->>R: replace("/login")
```

## SD-USR-02 — Autenticar com senha

```mermaid
sequenceDiagram
    autonumber
    actor V as Visitante
    participant T as TelaLogin
    participant TP as TemaProvider
    participant S as SessaoProvider
    participant A as api
    participant AC as AuthController
    participant AS as AuthService
    participant B as bcrypt
    participant J as JwtService
    participant G as JwtAuthGuard
    participant UC as UsuariosController
    participant US as UsuariosService
    participant M as UsuarioModel
    V->>T: entrar()
    Note over T: validarEmail e validarSenha
    T->>A: api("/auth/login", "POST", corpo)
    Note over A,AC: POST /auth/login, sem token
    A->>AC: login(loginDto)
    AC->>AS: login(loginDto)
    AS->>M: findOne({ email })
    Note over AS,M: select("+senha_hash")
    M-->>AS: usuario
    Note over AS: recusa conta inexistente, inativa,<br/>não confirmada ou sem senha local
    AS->>B: compare(senha, hash)
    B-->>AS: senhaValida
    alt senha correta
        Note over AS,J: payload = sub, email e tipo do usuário
        AS->>J: signAsync(payload)
        J-->>AS: accessToken
        AS-->>AC: resposta
        Note over AC,AS: resposta = access_token e usuario
        AC-->>A: 201 resposta
        A-->>T: resultado
        Note over T,A: resultado.dados = resposta
        T->>S: entrar(token)
        S->>A: salvarToken(token)
        Note over A: SecureStore ou localStorage, chave conecta_mais_token
        S->>A: api("/usuarios/me")
        Note over A,G: GET /usuarios/me com Authorization Bearer
        A->>G: canActivate(contexto)
        G->>J: verifyAsync(token)
        J-->>G: payload
        Note over G,J: payload = sub, email, tipo e iat
        G->>M: findOne(filtro)
        Note over G,M: filtro = _id igual a payload.sub e ativo true
        M-->>G: usuario
        Note over G: recusa iat anterior a senha_alterada_em
        G-->>A: true
        A->>UC: buscarMeuPerfil(requisicao)
        UC->>US: buscarPorId(sub)
        US->>M: findById(id)
        M-->>US: usuario
        US-->>UC: usuario
        UC-->>A: 200 usuario
        A-->>S: resultado
        S->>S: definirUsuario(usuario)
        S-->>T: true
        T->>TP: definirPreferencia(usuario.tema)
        Note over T,TP: só quando usuario.tema é claro, escuro ou sistema
        Note over T: Stack.Protected deixa de mostrar o login e abre /inicio
    else senha errada
        AS-->>AC: UnauthorizedException
        AC-->>A: 401 "Email ou senha inválidos"
        A-->>T: resultado
        Note over T,A: resultado.ok falso, com a mensagem em resultado.erro
        T->>T: definirErro(resultado.erro)
    end
```

## SD-USR-03 — Autenticar com Google

No aplicativo instalado (Android e iOS):

```mermaid
sequenceDiagram
    autonumber
    actor V as Visitante
    participant T as TelaLogin
    participant GM as google
    participant SDK as GoogleSignin
    participant A as api
    participant AC as AuthController
    participant AS as AuthService
    participant F as fetch
    participant M as UsuarioModel
    participant S as SessaoProvider
    V->>T: entrarComGoogle()
    T->>GM: signInWithGoogleNative()
    GM->>GM: estaNoExpoGo()
    Note over GM: no Expo Go, interrompe com O login Google não funciona no Expo.
    GM->>GM: configureNativeGoogle(GoogleSignin)
    Note over GM,SDK: opcoes = webClientId e, no iOS, iosClientId. Só na primeira chamada
    GM->>SDK: configure(opcoes)
    GM->>SDK: hasPlayServices({ showPlayServicesUpdateDialog: true })
    Note over GM,SDK: só no Android
    GM->>SDK: signOut()
    GM->>SDK: signIn()
    SDK-->>GM: response
    Note over GM,SDK: isSuccessResponse(response) confirma que a pessoa escolheu uma conta
    GM->>SDK: getTokens()
    SDK-->>GM: tokens
    Note over GM,SDK: tokens.idToken é o token de identidade da conta
    GM-->>T: nativeResult
    Note over T,GM: nativeResult = cancelled false e idToken
    T->>GM: loginComIdTokenGoogle(nativeResult.idToken)
    GM->>A: api("/auth/google", "POST", corpo)
    Note over A,AC: POST /auth/google com id_token e plataforma
    A->>AC: loginComGoogle(dto)
    AC->>AS: loginComGoogle(dto)
    AS->>AS: validarIdTokenGoogle(idToken, clientID)
    Note over AS,F: url = https://oauth2.googleapis.com/tokeninfo com o id_token na consulta
    AS->>F: fetch(url)
    F-->>AS: response
    Note over AS,F: o JSON de response traz sub, email, email_verified,<br/>name, picture, aud, iss e exp
    Note over AS: exige aud igual ao cliente, iss do Google,<br/>exp no futuro e email_verified verdadeiro
    AS->>M: findOne({ google_id })
    alt conta já ligada ao Google
        M-->>AS: usuario
        AS->>M: save()
        M-->>AS: usuario
    else conta local com o mesmo e-mail
        M-->>AS: null
        AS->>M: findOne({ email })
        M-->>AS: usuario
        Note over AS,M: conta ainda sem google_id, que é campo oculto
        Note over AS: grava google_id, nome, avatar_url e<br/>email_verificado true, e LOCAL vira AMBOS
        AS->>M: save()
        M-->>AS: usuario
    else nenhuma conta
        M-->>AS: null
        AS->>M: findOne({ email })
        M-->>AS: null
        AS->>M: create(dados)
        M-->>AS: usuario
        Note over AS,M: conta nova com provedor_autenticacao GOOGLE
    end
    AS->>AS: gerarRespostaLogin(usuario)
    AS-->>AC: resposta
    Note over AC,AS: resposta = access_token e usuario
    AC-->>A: 201 resposta
    A-->>GM: resultado
    GM-->>T: resultado
    T->>S: entrar(resultado.dados.access_token)
    Note over S: segue como no SD-USR-02, a partir de salvarToken
```

No navegador:

```mermaid
sequenceDiagram
    autonumber
    actor V as Visitante
    participant T as TelaLogin
    participant L as Location
    actor GO as Google
    participant AC as AuthController
    participant W as GoogleCallbackWeb
    participant GM as google
    participant A as api
    participant AS as AuthService
    participant F as fetch
    participant S as SessaoProvider
    participant R as Router
    V->>T: entrarComGoogle()
    T->>L: assign(request.url)
    Note over T,L: request vem de useAuthRequest, do expo-auth-session
    Note over L,GO: o navegador abre a página do Google e a pessoa escolhe a conta
    GO->>AC: callbackGoogleWeb(code, state, res)
    Note over GO,AC: GET /auth/google/callback-web, liberada no ApiKeyGuard
    Note over AC: webRedirect = FRONTEND_URL mais /login/google-callback,<br/>com code e state na consulta
    AC-->>GO: 302 webRedirect
    Note over GO,W: o navegador segue o redirecionamento e abre GoogleCallbackWeb
    W->>W: handleCallback()
    W->>GM: getWebCallbackRedirectUri()
    GM-->>W: redirectUri
    W->>GM: exchangeCodeForToken(code, redirectUri)
    GM->>A: api("/auth/google", "POST", corpo)
    Note over A,AC: POST /auth/google com code, redirect_uri e plataforma web
    A->>AC: loginComGoogle(dto)
    AC->>AS: loginComGoogle(dto)
    Note over AS,F: opcoes = method POST e body com code, client_id,<br/>redirect_uri, grant_type e client_secret
    AS->>F: fetch("https://oauth2.googleapis.com/token", opcoes)
    F-->>AS: tokenResponse
    Note over AS,F: o JSON de tokenResponse traz o id_token
    AS->>AS: validarIdTokenGoogle(idToken, clientID)
    Note over AS,F: url = https://oauth2.googleapis.com/tokeninfo com o id_token na consulta
    AS->>F: fetch(url)
    F-->>AS: response
    Note over AS: confere o token, procura, liga ou cria a conta como no diagrama anterior
    AS->>AS: gerarRespostaLogin(usuario)
    AS-->>AC: resposta
    Note over AC,AS: resposta = access_token e usuario
    AC-->>A: 201 resposta
    A-->>GM: resultado
    GM-->>W: resultado
    W->>S: entrar(resultado.dados.access_token)
    S-->>W: true
    W->>R: replace("/inicio")
```

## Rastreabilidade

SD-USR-01 a SD-USR-03 correspondem, na mesma ordem, a UC-USR-01 a UC-USR-03 e a RF-USR-01 a RF-USR-03.
