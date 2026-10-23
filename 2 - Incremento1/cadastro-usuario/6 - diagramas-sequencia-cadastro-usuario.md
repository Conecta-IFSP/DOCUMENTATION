# Diagramas de sequência — Cadastro de usuário

> Padrão do Incremento 1 · Módulo USR

## Resumo

Mostra os fluxos entre telas, controllers, services, MongoDB, Google e serviço de e-mail.

Os nomes correspondem ao código auditado. Toda chamada passa pelo `ApiKeyGuard`; rotas privadas também usam `JwtAuthGuard` e `UsuarioComumGuard`.

## SD-USR-01 — Cadastro local e confirmação

```mermaid
sequenceDiagram
    actor U as Usuário
    participant F as TelaCadastro
    participant AC as AuthController
    participant US as UsuariosService
    participant AS as AuthService
    participant DB as MongoDB
    participant Mail as Nodemailer
    U->>F: nome, email e senha
    F->>AC: POST /usuarios
    AC->>US: criar(dto)
    US->>DB: grava usuário, bcrypt e email_verificado=false
    AC->>AS: enviarCodigoVerificacao(email)
    AS->>DB: grava hash SHA-256 e expiração de 30 min
    AS->>Mail: envia código
    U->>F: informa código
    F->>AC: POST /auth/verificar-cadastro
    AC->>AS: verificarCadastro(email, codigo)
    AS->>DB: valida hash/expiração e confirma e-mail
```

## SD-USR-02 — Login local

```mermaid
sequenceDiagram
    actor U as Usuário
    participant F as TelaLogin
    participant AC as AuthController
    participant AS as AuthService
    participant DB as MongoDB
    U->>F: email e senha
    F->>AC: POST /auth/login
    AC->>AS: login(dto)
    AS->>DB: busca conta ativa e campos protegidos
    AS->>AS: bcrypt.compare e verifica e-mail confirmado
    AS-->>F: JWT de 1 hora e usuário
    F->>AC: GET /usuarios/me com Bearer JWT
    AC-->>F: perfil validado
```

## SD-USR-03 — Login Google

```mermaid
sequenceDiagram
    actor U as Usuário
    participant F as Aplicativo Expo
    participant G as Google OAuth
    participant AC as AuthController
    participant AS as AuthService
    participant DB as MongoDB
    U->>F: Entrar com Google
    F->>G: autenticação nativa ou Auth Session
    G-->>F: id_token ou authorization code
    F->>AC: POST /auth/google
    AC->>AS: loginGoogle(dto)
    AS->>G: valida emissor, audiência e expiração
    AS->>DB: cria ou vincula por google_id/e-mail
    AS-->>F: JWT e usuário
```

## SD-USR-04 — Entrar por código

```mermaid
sequenceDiagram
    actor U as Usuário
    participant F as Aplicativo
    participant OC as OrganizacoesController
    participant OS as OrganizacoesService
    participant DB as MongoDB
    U->>F: código de 10 caracteres
    F->>OC: POST /organizacoes/entrar
    OC->>OS: entrarComCodigo(codigo, usuarioId)
    OS->>DB: busca organização APROVADA por codigo_acesso
    OS->>DB: adiciona MEMBRO/PENDENTE com filtro concorrente
    OS-->>F: organização atualizada
```

## SD-USR-05 — Convidar e responder

```mermaid
sequenceDiagram
    actor A as Administrador
    actor U as Convidado
    participant F as Aplicativo
    participant OC as OrganizacoesController
    participant OS as OrganizacoesService
    participant DB as MongoDB
    A->>F: informa e-mail
    F->>OC: POST /organizacoes/:id/convites
    OC->>OS: convidar(id, email, adminId)
    OS->>DB: grava MEMBRO/CONVIDADO
    U->>F: aceita ou recusa
    F->>OC: PATCH /organizacoes/:id/convite
    OC->>OS: responderConvite(id, aceitar, usuarioId)
    OS->>DB: aceita como APROVADO ou remove vínculo
```

## SD-USR-06 — Analisar solicitação

```mermaid
sequenceDiagram
    actor A as Administrador
    participant F as Aplicativo
    participant OC as OrganizacoesController
    participant OS as OrganizacoesService
    participant DB as MongoDB
    A->>F: aprova ou rejeita
    F->>OC: PATCH /organizacoes/:id/membros/:usuarioId
    OC->>OS: alterarStatusMembro(...)
    OS->>DB: findOneAndUpdate com estado/autorização originais
    alt estado mudou simultaneamente
        DB-->>F: 409 Conflict
    else alteração aplicada
        DB-->>F: vínculo atualizado
    end
```

## SD-USR-07 — Alterar papel ou remover membro

```mermaid
sequenceDiagram
    actor A as Administrador
    participant F as Aplicativo
    participant OC as OrganizacoesController
    participant OS as OrganizacoesService
    participant DB as MongoDB
    alt alterar papel
        F->>OC: PATCH /organizacoes/:id/membros/:usuarioId/papel
        OC->>OS: alterarPapelMembro(...)
        OS->>DB: atualiza e preserva ao menos um ADMIN
    else remover
        F->>OC: DELETE /organizacoes/:id/membros/:usuarioId
        OC->>OS: removerMembro(...)
        OS->>DB: remove vínculo MEMBRO
        OS->>DB: remove usuário das comissões da organização
    end
```
