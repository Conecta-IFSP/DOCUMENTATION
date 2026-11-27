# Diagramas de sequência — Cadastro de usuário

> Padrão da disciplina · Módulo USR · Revisado em 26/11/2026

## Resumo

Há um diagrama para cada requisito de [Requisitos funcionais](2%20-%20requisitos-funcionais-cadastro-usuario.md), com o mesmo número. As linhas de vida são classes e módulos que existem no código: as telas do aplicativo, o `SessaoProvider`, os módulos `api` e `google`, os controllers, os services, o `JwtAuthGuard` e o model do Mongoose, além dos sistemas externos (servidor de e-mail e Google). Toda mensagem é a chamada de um método existente, com seus parâmetros, e as mensagens são numeradas na ordem em que acontecem. Regras que não são métodos, como comparações de situação, aparecem em notas, assim como as rotas HTTP.

Em todos os diagramas, a chamada do módulo `api` ao controller é uma requisição HTTP que passa antes pelo `ApiKeyGuard`; o SD-USR-02 mostra em detalhe o `JwtAuthGuard`, que protege `GET /usuarios/me`. Os diagramas dos vínculos do usuário com organizações estão no módulo [Cadastro de organização](../cadastro-organizacao/README.md).

## SD-USR-01 — Cadastrar e confirmar conta local

```mermaid
sequenceDiagram
    autonumber
    actor V as Visitante
    participant TC as TelaCadastro
    participant TV as TelaVerificarCadastro
    participant A as api
    participant UC as UsuariosController
    participant US as UsuariosService
    participant AC as AuthController
    participant AS as AuthService
    participant M as UsuarioModel
    participant E as Servidor de e-mail
    V->>TC: criarConta()
    Note over TC: validarNome, validarEmail, validarNovaSenha<br/>e validarConfirmacaoSenha
    TC->>A: api("/usuarios", "POST", corpo)
    Note over A,UC: POST /usuarios, sem token<br/>ValidationPipe aplica CreateUsuarioDto
    A->>UC: criar(createUsuarioDto)
    UC->>US: criar(createUsuarioDto)
    US->>M: findOne(email)
    M-->>US: null
    US->>US: bcrypt.hash(senha, 10)
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
    AS-)E: sendMail(mensagem)
    AS-->>UC: concluído
    UC-->>A: 201 mensagem
    A-->>TC: Resultado ok
    TC->>TC: mostrarToast(mensagem, "sucesso")
    Note over TC,TV: destino = pathname /login/verificar-cadastro e params com o email normalizado
    TC->>TV: router.replace(destino)
    opt servidor de e-mail recusa ou não responde
        E--)AS: erro
        AS->>M: deleteOne({ _id: usuario._id, email_verificado: false })
    end
    V->>TV: verificar()
    Note over TV: exige 7 caracteres e envia no formato XXX-XXXX
    TV->>A: api("/auth/verificar-cadastro", "POST", corpo)
    Note over A,AC: POST /auth/verificar-cadastro
    A->>AC: verificarCadastro(dto)
    AC->>AS: verificarCadastro(dto)
    AS->>M: findOne(email)
    M-->>AS: usuario com o hash e o prazo do código
    AS->>AS: hashToken(codigo)
    Note over AS: exige conta não confirmada, hash igual ao gravado<br/>e verificacao_email_expira_em depois do horário atual
    AS->>M: save()
    M-->>AS: usuario confirmado
    AS-->>AC: mensagem
    AC-->>A: 201 mensagem
    A-->>TV: Resultado ok
    TV->>TV: mostrarToast(mensagem, "sucesso", 5000)
    TV->>TV: router.replace("/login")
```

## SD-USR-02 — Autenticar com senha

```mermaid
sequenceDiagram
    autonumber
    actor V as Visitante
    participant T as TelaLogin
    participant S as SessaoProvider
    participant A as api
    participant AC as AuthController
    participant AS as AuthService
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
    AS->>M: findOne(email)
    Note over AS,M: select("+senha_hash")
    M-->>AS: usuario
    Note over AS: recusa conta inexistente, inativa,<br/>não confirmada ou sem senha local
    AS->>AS: bcrypt.compare(senha, hash)
    alt senha correta
        Note over AS,J: payload = sub, email e tipo do usuário
        AS->>J: signAsync(payload)
        J-->>AS: access_token
        AS-->>AC: access_token e usuario
        AC-->>A: 201 access_token e usuario
        A-->>T: Resultado ok
        T->>S: entrar(token)
        S->>A: salvarToken(token)
        Note over A: SecureStore ou localStorage, chave conecta_mais_token
        S->>A: api("/usuarios/me")
        Note over A,G: GET /usuarios/me com Authorization Bearer
        A->>G: canActivate(contexto)
        G->>J: verifyAsync(token)
        J-->>G: sub, email, tipo e iat
        G->>M: findOne({ _id: requisicao.usuario.sub, ativo: true })
        M-->>G: usuario
        Note over G: recusa iat anterior a senha_alterada_em
        G-->>A: true
        A->>UC: buscarMeuPerfil(requisicao)
        UC->>US: buscarPorId(sub)
        US->>M: findById(id)
        M-->>US: usuario
        US-->>UC: usuario
        UC-->>A: 200 usuario
        A-->>S: Resultado ok
        S->>S: definirUsuario(usuario)
        S-->>T: true
        T->>T: definirPreferencia(usuario.tema)
        Note over T: Stack.Protected deixa de mostrar o login e abre /inicio
    else senha errada
        AS-->>AC: UnauthorizedException
        AC-->>A: 401 Email ou senha inválidos
        A-->>T: Resultado com o erro
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
    participant GO as Google
    participant M as UsuarioModel
    participant S as SessaoProvider
    V->>T: entrarComGoogle()
    T->>GM: signInWithGoogleNative()
    GM->>SDK: hasPlayServices()
    GM->>SDK: signOut()
    GM->>SDK: signIn()
    SDK-->>GM: conta escolhida
    GM->>SDK: getTokens()
    SDK-->>GM: idToken
    GM-->>T: idToken
    T->>GM: loginComIdTokenGoogle(idToken)
    GM->>A: api("/auth/google", "POST", corpo)
    Note over A,AC: POST /auth/google com id_token e plataforma
    A->>AC: loginComGoogle(dto)
    AC->>AS: loginComGoogle(dto)
    AS->>AS: validarIdTokenGoogle(idToken, clientID)
    Note over AS,GO: url = https://oauth2.googleapis.com/tokeninfo com o id_token na consulta
    AS->>GO: fetch(url)
    GO-->>AS: sub, email, email_verified, name, picture, aud, iss e exp
    Note over AS: exige aud igual ao cliente, iss do Google,<br/>exp no futuro e email_verified verdadeiro
    AS->>M: findOne(google_id)
    alt conta já ligada ao Google
        M-->>AS: usuario
        AS->>M: save()
        M-->>AS: usuario
    else conta local com o mesmo e-mail
        M-->>AS: null
        AS->>M: findOne(email)
        M-->>AS: usuario
        Note over AS: grava google_id, nome, avatar_url e<br/>email_verificado true, e LOCAL vira AMBOS
        AS->>M: save()
        M-->>AS: usuario
    else nenhuma conta
        M-->>AS: null
        AS->>M: findOne(email)
        M-->>AS: null
        AS->>M: create(dados)
        M-->>AS: usuario GOOGLE
    end
    AS->>AS: gerarRespostaLogin(usuario)
    AS-->>AC: access_token e usuario
    AC-->>A: 201 access_token e usuario
    A-->>GM: Resultado ok
    GM-->>T: Resultado ok
    T->>S: entrar(access_token)
    Note over S: segue como no SD-USR-02, a partir de salvarToken
```

No navegador:

```mermaid
sequenceDiagram
    autonumber
    actor V as Visitante
    participant T as TelaLogin
    participant GO as Google
    participant AC as AuthController
    participant W as GoogleCallbackWeb
    participant GM as google
    participant A as api
    participant AS as AuthService
    participant S as SessaoProvider
    V->>T: entrarComGoogle()
    T->>GO: window.location.assign(request.url)
    Note over V,GO: a pessoa escolhe a conta na página do Google
    GO->>AC: callbackGoogleWeb(code, state, res)
    Note over GO,AC: GET /auth/google/callback-web, liberada no ApiKeyGuard
    Note over AC,W: webRedirect = FRONTEND_URL mais /login/google-callback, com code e state na consulta
    AC->>W: res.redirect(webRedirect)
    W->>W: handleCallback()
    W->>GM: exchangeCodeForToken(code, redirectUri)
    GM->>A: api("/auth/google", "POST", corpo)
    Note over A,AC: POST /auth/google com code, redirect_uri e plataforma web
    A->>AC: loginComGoogle(dto)
    AC->>AS: loginComGoogle(dto)
    Note over AS,GO: opcoes = method POST e body com code, client_id, redirect_uri e grant_type
    AS->>GO: fetch("https://oauth2.googleapis.com/token", opcoes)
    GO-->>AS: id_token
    AS->>AS: validarIdTokenGoogle(idToken, clientID)
    Note over AS,GO: url = https://oauth2.googleapis.com/tokeninfo com o id_token na consulta
    AS->>GO: fetch(url)
    GO-->>AS: dados do token
    Note over AS: procura, liga ou cria a conta como no diagrama anterior
    AS->>AS: gerarRespostaLogin(usuario)
    AS-->>AC: access_token e usuario
    AC-->>A: 201 access_token e usuario
    A-->>GM: Resultado ok
    GM-->>W: Resultado ok
    W->>S: entrar(access_token)
    S-->>W: true
    W->>W: router.replace("/inicio")
```

## Rastreabilidade

SD-USR-01 a SD-USR-03 correspondem, na mesma ordem, a UC-USR-01 a UC-USR-03 e a RF-USR-01 a RF-USR-03.
