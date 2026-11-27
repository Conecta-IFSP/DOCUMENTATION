# Diagramas de sequência — Configurações de usuário

> Padrão da disciplina · Módulo CFG · Revisado em 26/11/2026

## Resumo

Há um diagrama para cada requisito de [Requisitos funcionais](2%20-%20requisitos-funcionais-configuracoes-usuario.md), com o mesmo número. As linhas de vida são classes e módulos que existem no código: as telas do aplicativo, os contextos, o módulo `api`, os controllers, os services, o guard, o model `UsuarioModel` e o documento `usuario` lido por ele. Toda mensagem é a chamada de um método existente, com seus parâmetros, e as mensagens são numeradas na ordem em que acontecem. Regras que não são métodos, como comparar um código com o hash guardado, aparecem em notas.

Em todos os diagramas, a chamada do módulo `api` ao controller é uma requisição HTTP que passa antes pelo `ApiKeyGuard`; nas rotas `/usuarios/me…`, passa também pelo `JwtAuthGuard`, desenhado só no SD-CFG-03, onde ele decide o resultado. A rota está indicada na nota.

## SD-CFG-01 — Alterar nome do perfil

```mermaid
sequenceDiagram
    autonumber
    actor U as Usuário autenticado
    participant T as TelaPerfil
    participant L as autenticacaoLocal
    participant SC as SessaoContext
    participant A as api
    participant C as UsuariosController
    participant S as UsuariosService
    participant M as UsuarioModel
    participant D as usuario (UsuarioDocument)
    Note over U,T: A tela ganha foco e useFocusEffect chama carregarPerfil
    U->>T: carregarPerfil(ativa)
    T->>A: api("/usuarios/me")
    Note over A,C: GET /usuarios/me
    A->>C: buscarMeuPerfil(requisicao)
    C->>S: buscarPorId(sub)
    S->>S: validarId(id)
    S->>M: findById(id)
    M-->>S: usuario
    S-->>C: usuario
    C-->>A: 200 usuario
    A-->>T: Resultado com o perfil
    T->>T: definirPerfil(dados)
    U->>T: abrirEdicaoNome()
    U->>T: salvarNome()
    Note over T: validarNome exige 3 caracteres ou mais, sem os espaços das pontas
    T->>L: autenticarComDispositivo()
    L->>L: dispositivoProtegido()
    Note over U,L: Com bloqueio de tela, o aparelho pede biometria, Face ID ou PIN.<br/>Na web e sem bloqueio, devolve true sem pedir
    alt identidade não confirmada
        L-->>T: false
        Note over T: aviso = tom "erro" e mensagem "Não foi possível confirmar sua identidade."
        T->>T: definirAvisoNome(aviso)
        Note over T: A folha continua aberta e nada é enviado
    else identidade confirmada
        L-->>T: true
        Note over T,A: corpo = { nome } com o nome aparado
        T->>A: api("/usuarios/me", "PATCH", corpo)
        Note over A,C: PATCH /usuarios/me
        A->>C: atualizarMeuPerfil(requisicao, atualizarPerfilDto)
        C->>S: atualizarPerfil(sub, atualizarPerfilDto)
        S->>S: validarId(id)
        S->>M: findById(id)
        M-->>S: usuario
        Note over S: Com dto.email preenchido, lança BadRequestException (400)<br/>e a tela mostra o motivo com definirAvisoNome. Senão, apara dto.nome
        S->>D: save()
        D-->>S: usuario atualizado
        S-->>C: usuario
        C-->>A: 200 usuario
        A-->>T: Resultado com o perfil
        T->>T: definirPerfil(dados)
        T->>T: definirEditandoNome(false)
        T->>T: mostrarToast("Nome atualizado.", "sucesso")
        T->>SC: atualizar()
        SC->>A: api("/usuarios/me")
        A-->>SC: Resultado com o perfil
    end
```

## SD-CFG-02 — Trocar e-mail

```mermaid
sequenceDiagram
    autonumber
    actor U as Usuário autenticado
    participant P as TelaPerfil
    participant L as autenticacaoLocal
    participant T as TelaTrocarEmail
    participant SC as SessaoContext
    participant A as api
    participant C as UsuariosController
    participant S as AuthService
    participant M as UsuarioModel
    participant D as usuario (UsuarioDocument)
    participant E as Transporter (Nodemailer)
    U->>P: abrirProtegida("/login/trocar-email")
    P->>L: autenticarComDispositivo()
    L->>L: dispositivoProtegido()
    Note over U,L: Com bloqueio de tela, o aparelho pede biometria, Face ID ou PIN.<br/>Cancelado, devolve false e a tela não abre
    L-->>P: true
    P->>P: router.push("/login/trocar-email")
    Note over T,A: Ao abrir, a tela consulta o e-mail atual
    T->>A: api("/usuarios/me")
    A-->>T: Resultado com o e-mail atual
    U->>T: enviarCodigo()
    Note over T: validarEmail e e-mail normalizado diferente do atual
    T->>T: solicitar(email)
    T->>A: api("/usuarios/me/email/solicitar", "POST", corpo)
    Note over A,C: POST /usuarios/me/email/solicitar
    A->>C: solicitarAlteracaoEmail(requisicao, dto)
    C->>S: solicitarAlteracaoEmail(sub, dto.email)
    S->>M: findById(id)
    M-->>S: usuario
    Note over S: Recusa e-mail igual ao atual
    S->>M: findOne({ email, _id: { $ne: id } })
    M-->>S: null
    S->>S: gerarCodigoVerificacao()
    S->>S: hashToken(codigo)
    Note over S: email_pendente, email_pendente_codigo_hash<br/>e email_pendente_expira_em igual a agora mais 30 minutos
    S->>D: save()
    D-->>S: usuario
    S->>S: enviarEmail(mensagem)
    S->>S: criarTransportador()
    S-)E: sendMail(mensagem)
    Note over S,E: A resposta não espera o envio.<br/>Uma falha só gera um aviso no log da API
    S-->>C: mensagem
    C-->>A: 201 mensagem
    A-->>T: Resultado
    T->>T: definirEtapa("codigo")
    U->>T: confirmar()
    Note over T: Exige os 7 caracteres e envia no formato XXX-XXXX
    T->>A: api("/usuarios/me/email/confirmar", "POST", corpo)
    Note over A,C: POST /usuarios/me/email/confirmar
    A->>C: confirmarAlteracaoEmail(requisicao, dto)
    C->>S: confirmarAlteracaoEmail(sub, dto.email, dto.codigo)
    S->>M: findById(id).select("+email_pendente +email_pendente_codigo_hash +email_pendente_expira_em")
    M-->>S: usuario
    S->>S: hashToken(codigo)
    Note over S: Exige email_pendente igual ao informado, hash igual<br/>e email_pendente_expira_em maior que agora
    S->>M: findOne({ email, _id: { $ne: id } })
    M-->>S: null
    Note over S: Grava email e email_verificado igual a true<br/>e apaga os três campos pendentes
    S->>D: save()
    D-->>S: usuario
    S-->>C: mensagem e email
    C-->>A: 201 mensagem e email
    A-->>T: Resultado
    T->>T: definirEtapa("concluido")
    T->>SC: atualizar()
```

## SD-CFG-03 — Alterar senha

```mermaid
sequenceDiagram
    autonumber
    actor U as Usuário autenticado
    participant P as TelaPerfil
    participant L as autenticacaoLocal
    participant T as TelaAlterarSenha
    participant SC as SessaoContext
    participant A as api
    participant G as JwtAuthGuard
    participant C as UsuariosController
    participant S as UsuariosService
    participant M as UsuarioModel
    participant D as usuario (UsuarioDocument)
    participant B as bcrypt
    U->>P: abrirProtegida("/login/alterar-senha")
    P->>L: autenticarComDispositivo()
    L-->>P: true
    P->>P: router.push("/login/alterar-senha")
    U->>T: salvar()
    Note over T: handleSubmit aplica validarSenha, validarNovaSenha<br/>e validarConfirmacaoSenha
    T->>A: api("/usuarios/me/senha", "PATCH", corpo)
    Note over A,G: PATCH /usuarios/me/senha com { senha_atual, nova_senha }
    A->>G: canActivate(context)
    G->>M: findOne({ _id: sub, ativo: true })
    M-->>G: usuario
    Note over G: Recusa iat menor que senha_alterada_em em segundos
    G-->>A: true
    A->>C: alterarMinhaSenha(requisicao, alterarSenhaDto)
    C->>S: alterarSenha(sub, alterarSenhaDto)
    S->>S: validarId(id)
    S->>M: findById(id).select("+senha_hash")
    M-->>S: usuario
    Note over S: Sem senha_hash, recusa com "Use a recuperação de conta para definir sua senha"
    S->>B: compare(senha_atual, senha_hash)
    B-->>S: true
    S->>B: compare(nova_senha, senha_hash)
    B-->>S: false
    Note over T,S: Se a primeira comparação der false ou a segunda der true, lança BadRequestException (400)<br/>e a tela mostra o motivo com definirErro, sem sair da conta
    S->>B: hash(nova_senha, 10)
    B-->>S: novo hash
    Note over S: senha_hash recebe o novo hash e senha_alterada_em o horário atual
    S->>D: save()
    D-->>S: usuario
    S-->>C: mensagem
    C-->>A: 200 mensagem
    A-->>T: Resultado
    T->>SC: sair("Senha alterada. Entre novamente com a nova senha.")
    SC->>A: removerToken()
    A-->>SC: concluído
    SC->>SC: definirUsuario(null)
    SC->>SC: definirAviso(aviso)
    Note over U,SC: Stack.Protected retira as telas autenticadas e o login mostra o aviso
```

Sessão aberta antes da troca, em outro aparelho:

```mermaid
sequenceDiagram
    autonumber
    participant SC as SessaoContext (outro aparelho)
    participant A as api
    participant G as JwtAuthGuard
    participant M as UsuarioModel
    Note over SC: A cada 30 segundos com o aplicativo aberto,<br/>ao voltar ao aplicativo e a cada troca de tela
    SC->>SC: atualizar()
    SC->>A: api("/usuarios/me")
    Note over A,G: GET /usuarios/me com o token antigo
    A->>G: canActivate(context)
    G->>M: findOne({ _id: sub, ativo: true })
    M-->>G: usuario
    Note over G: iat do token menor que senha_alterada_em em segundos
    G-->>A: 401 A senha foi alterada. Entre novamente.
    A->>A: removerToken()
    A->>SC: ouvinte()
    Note over A,SC: Ouvinte registrado por observarSessaoInvalida
    SC->>SC: definirUsuario(null)
    SC->>SC: definirAviso("Sua sessão expirou ou a conta está indisponível. Entre novamente.")
```

## SD-CFG-04 — Recuperar conta

```mermaid
sequenceDiagram
    autonumber
    actor V as Visitante
    participant T as TelaRecuperarConta
    participant L as autenticacaoLocal
    participant A as api
    participant C as AuthController
    participant S as AuthService
    participant M as UsuarioModel
    participant D as usuario (UsuarioDocument)
    participant B as bcrypt
    participant E as Transporter (Nodemailer)
    V->>T: solicitar()
    T->>A: api("/auth/recuperar-conta", "POST", corpo)
    Note over A,C: POST /auth/recuperar-conta, sem token
    A->>C: recuperarConta(recuperarContaDto)
    C->>S: solicitarRecuperacao(recuperarContaDto)
    S->>M: findOne({ email, ativo: true })
    M-->>S: usuario
    opt conta ativa encontrada
        S->>S: gerarCodigoVerificacao()
        S->>S: hashToken(codigo)
        Note over S: Apaga reset_senha_token_hash e grava reset_senha_codigo_hash<br/>e reset_senha_expira_em igual a agora mais 30 minutos
        S->>D: save()
        D-->>S: usuario
        S->>S: enviarEmail(mensagem)
        S-)E: sendMail(mensagem)
    end
    S-->>C: mensagem genérica
    C-->>A: 201 mensagem genérica
    A-->>T: Resultado
    T->>T: definirEtapa("codigo")
    V->>T: verificarCodigo()
    T->>A: api("/auth/verificar-recuperacao", "POST", corpo)
    Note over A,C: POST /auth/verificar-recuperacao
    A->>C: verificarRecuperacao(dto)
    C->>S: verificarRecuperacao(dto)
    S->>M: findOne({ email, ativo: true }).select("+reset_senha_codigo_hash +reset_senha_expira_em")
    M-->>S: usuario
    S->>S: hashToken(codigo)
    Note over S: Exige hash igual e reset_senha_expira_em maior que agora
    S->>S: randomBytes(32)
    S->>S: hashToken(token)
    Note over S: Grava reset_senha_token_hash e apaga o código
    S->>D: save()
    D-->>S: usuario
    S-->>C: token
    C-->>A: 201 token
    A-->>T: Resultado com o token
    T->>T: definirToken(token)
    T->>T: definirEtapa("senha")
    V->>T: redefinir()
    T->>L: autenticarComDispositivo()
    L-->>T: true
    T->>A: api("/auth/redefinir-senha", "POST", corpo)
    Note over A,C: POST /auth/redefinir-senha com { token, nova_senha }
    A->>C: redefinirSenha(redefinirSenhaDto)
    C->>S: redefinirSenha(redefinirSenhaDto)
    S->>S: hashToken(token)
    S->>M: findOne({ reset_senha_token_hash, reset_senha_expira_em: { $gt: agora }, ativo: true })
    M-->>S: usuario
    S->>B: hash(nova_senha, 10)
    B-->>S: novo hash
    Note over S: Grava senha_hash e senha_alterada_em<br/>e apaga reset_senha_token_hash e reset_senha_expira_em
    S->>D: save()
    D-->>S: usuario
    S-->>C: mensagem
    C-->>A: 201 mensagem
    A-->>T: Resultado
    T->>T: definirEtapa("concluido")
```

## SD-CFG-05 — Escolher tema

```mermaid
sequenceDiagram
    autonumber
    actor U as Usuário autenticado
    participant T as TelaPerfil
    participant TC as TemaContext
    participant SS as SecureStore
    participant A as api
    participant C as UsuariosController
    participant S as UsuariosService
    participant M as UsuarioModel
    participant D as usuario (UsuarioDocument)
    U->>T: definirEscolhendoTema(true)
    U->>T: selecionarTema(novaPreferencia)
    Note over T: Ignora a opção atual e o toque durante outra gravação
    T->>TC: definirPreferencia(novaPreferencia)
    TC->>TC: gravarPreferencia(preferencia)
    TC->>SS: setItemAsync("conecta_mais_tema", preferencia)
    Note over TC,SS: Na web, window.localStorage.setItem com a mesma chave
    T->>A: api("/usuarios/me", "PATCH", corpo)
    alt API grava o tema
        Note over A,C: PATCH /usuarios/me com { tema }
        A->>C: atualizarMeuPerfil(requisicao, atualizarPerfilDto)
        C->>S: atualizarPerfil(sub, atualizarPerfilDto)
        S->>M: findById(id)
        M-->>S: usuario
        S->>D: save()
        D-->>S: usuario
        S-->>C: usuario
        C-->>A: 200 usuario
        A-->>T: Resultado com o perfil
    else API recusa ou não responde
        A-->>T: Resultado com o erro
        T->>T: definirPreferenciaSelecionada(preferenciaAnterior)
        T->>TC: definirPreferencia(preferenciaAnterior)
        TC->>TC: gravarPreferencia(preferenciaAnterior)
        TC->>SS: setItemAsync("conecta_mais_tema", preferenciaAnterior)
        T->>T: definirAviso(erro)
    end
```

## SD-CFG-06 — Encerrar sessão

```mermaid
sequenceDiagram
    autonumber
    actor U as Usuário autenticado
    participant T as TelaPerfil
    participant SC as SessaoContext
    participant A as api
    participant SS as SecureStore
    U->>T: definirConfirmandoSaida(true)
    Note over U,T: Diálogo "Deseja realmente sair da sua conta?"
    U->>T: sair()
    T->>SC: sair()
    Note over SC: Soma 1 a geracao, para descartar respostas de consultas antigas
    SC->>A: removerToken()
    A->>SS: deleteItemAsync("conecta_mais_token")
    Note over A,SS: Na web, localStorage.removeItem com a mesma chave
    SS-->>A: concluído
    A-->>SC: concluído
    SC->>SC: definirUsuario(null)
    SC->>SC: definirAviso(aviso)
    SC-->>T: concluído
    T->>T: definirConfirmandoSaida(false)
    Note over U,SC: Stack.Protected retira as telas autenticadas e o aplicativo mostra "Entrar".<br/>A API não é chamada e o token continua válido no servidor até expirar
```

## Rastreabilidade

SD-CFG-01 a SD-CFG-06 correspondem, na mesma ordem, a UC-CFG-01 a UC-CFG-06 e a RF-CFG-01 a RF-CFG-06.
