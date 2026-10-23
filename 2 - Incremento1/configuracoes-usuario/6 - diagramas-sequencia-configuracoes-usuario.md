# Diagramas de sequência — Configurações de usuário

> Padrão do Incremento 1 · Módulo CFG

## Resumo

Mostra os fluxos entre telas Expo, contexts, controllers, services, MongoDB e e-mail.

Os participantes abaixo correspondem a componentes e métodos existentes.

## SD-CFG-01 — Alterar perfil

```mermaid
sequenceDiagram
    actor U as Usuário
    participant T as TelaPerfil
    participant A as api()
    participant K as ApiKeyGuard
    participant J as JwtAuthGuard
    participant C as UsuariosController
    participant S as UsuariosService
    participant M as UsuarioModel
    participant TC as TemaContext

    T->>A: api('/usuarios/me')
    A->>K: GET + X-API-Key + Bearer
    K->>J: canActivate()
    J->>M: findOne({_id: sub, ativo: true})
    J->>C: buscarMeuPerfil(req)
    C->>S: buscarPorId(sub)
    S->>M: findById(sub)
    M-->>T: perfil
    U->>T: altera nome e salva
    T->>A: PATCH /usuarios/me {nome}
    C->>S: atualizarPerfil(sub,dto)
    S->>M: usuario.save()
    M-->>T: perfil atualizado
    T->>T: sessao.atualizar()
    U->>T: escolhe tema
    T->>TC: definirPreferencia(novo)
    T->>A: PATCH /usuarios/me {tema}
    alt falha HTTP
        T->>TC: definirPreferencia(anterior)
        T-->>U: informa erro
    else sucesso
        T-->>U: mantém tema
    end
```

## SD-CFG-02 — Trocar e-mail

```mermaid
sequenceDiagram
    actor U as Usuário
    participant P as TelaPerfil
    participant L as autenticarComDispositivo()
    participant T as TelaTrocarEmail
    participant A as api()
    participant C as UsuariosController
    participant S as AuthService
    participant M as UsuarioModel
    participant E as Nodemailer

    U->>P: seleciona E-mail
    P->>L: autenticarComDispositivo()
    alt cancelado
        L-->>P: false
    else confirmado ou indisponível
        L-->>P: true
        P->>T: navega
        U->>T: informa novo e-mail
        T->>A: POST /usuarios/me/email/solicitar {email}
        A->>C: solicitarAlteracaoEmail(req,dto)
        C->>S: solicitarAlteracaoEmail(sub,email)
        S->>M: verificar unicidade; save(campos pendentes)
        S->>E: sendMail(código)
        U->>T: informa código
        T->>A: POST /usuarios/me/email/confirmar {email,codigo}
        A->>C: confirmarAlteracaoEmail(req,dto)
        C->>S: confirmarAlteracaoEmail(sub,email,codigo)
        S->>M: findById(+campos pendentes)
        S->>M: save(email; unset pendentes)
        T->>T: sessao.atualizar()
    end
```

## SD-CFG-03 — Trocar senha autenticada

```mermaid
sequenceDiagram
    actor U as Usuário
    participant T as TelaAlterarSenha
    participant A as api()
    participant C as UsuariosController
    participant S as UsuariosService
    participant M as UsuarioModel
    participant B as bcrypt

    U->>T: informa atual, nova e confirmação
    T->>T: handleSubmit() valida ≥8 e igualdade
    T->>A: PATCH /usuarios/me/senha {senha_atual,nova_senha}
    A->>C: alterarMinhaSenha(req,dto)
    C->>S: alterarSenha(sub,dto)
    S->>M: findById(sub).select('+senha_hash')
    S->>B: compare(senha_atual,hash)
    S->>B: compare(nova_senha,hash)
    alt atual inválida ou nova repetida
        S-->>T: 400 + mensagem
    else válida
        S->>B: hash(nova_senha,10)
        S->>M: usuario.save()
        S-->>T: Senha alterada com sucesso
    end
```

## SD-CFG-04 — Recuperar conta

```mermaid
sequenceDiagram
    actor V as Visitante
    participant T as TelaRecuperarConta
    participant A as api()
    participant C as AuthController
    participant S as AuthService
    participant M as UsuarioModel
    participant E as Nodemailer
    participant L as autenticarComDispositivo()
    participant B as bcrypt

    V->>T: informa e-mail
    T->>A: POST /auth/recuperar-conta {email}
    A->>C: recuperarConta(dto)
    C->>S: solicitarRecuperacao(dto)
    S->>M: findOne({email,ativo:true})
    opt conta existente
        S->>M: save(hash do código, expiração)
        S->>E: sendMail(código)
    end
    S-->>T: mensagem genérica
    V->>T: informa código
    T->>A: POST /auth/verificar-recuperacao {email,codigo}
    C->>S: verificarRecuperacao(dto)
    S->>M: findOne(+código + expiração)
    S->>M: save(hash do token; unset código)
    S-->>T: token temporário
    V->>T: informa nova senha e confirmação
    T->>L: autenticarComDispositivo()
    L-->>T: resultado
    T->>A: POST /auth/redefinir-senha {token,nova_senha}
    C->>S: redefinirSenha(dto)
    S->>M: findOne({token_hash,expiração,ativo:true})
    S->>B: hash(nova_senha,10)
    S->>M: save(senha; unset token/expiração)
    S-->>T: sucesso
```

## SD-CFG-05 — Escolher tema

```mermaid
sequenceDiagram
    actor U as Usuário
    participant T as TelaPerfil
    participant TC as TemaContext
    participant SS as SecureStore
    participant A as api()
    participant C as UsuariosController
    participant S as UsuariosService
    participant M as UsuarioModel
    U->>T: seleciona sistema, claro ou escuro
    T->>TC: definirPreferencia(novoTema)
    TC->>SS: setItemAsync('conecta_mais_tema', novoTema)
    T->>A: PATCH /usuarios/me {tema}
    A->>C: atualizarMeuPerfil(dto)
    C->>S: atualizarPerfil(usuarioId, dto)
    S->>M: findByIdAndUpdate({tema})
    alt API falha
        T->>TC: definirPreferencia(temaAnterior)
        TC->>SS: restaura armazenamento
    end
```

## SD-CFG-06 — Encerrar sessão

```mermaid
sequenceDiagram
    actor U as Usuário
    participant T as TelaPerfil
    participant S as SessaoContext
    participant SS as SecureStore
    participant R as Stack.Protected
    U->>T: seleciona Sair da conta
    T->>S: sair()
    S->>SS: deleteItemAsync('conecta_mais_token')
    S->>S: definirUsuario(null)
    S->>R: renderiza com guard=false
    R-->>U: exibe login
```
