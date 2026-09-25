# Diagramas de Sequência - Configurações de Usuário

## 1. Trocar senha (RF-01)

```mermaid
sequenceDiagram
    autonumber
    actor Usuario as Usuário autenticado
    participant Front as Frontend
    participant API as Backend
    participant DB as MongoDB

    Usuario->>Front: Informa senha atual, nova senha e confirmação
    Front->>Front: Valida tamanho e confirmação
    Front->>API: PATCH /usuarios/me/senha {senha_atual, nova_senha} + JWT
    API->>DB: Busca o usuário e seleciona o hash da senha
    DB-->>API: Usuário com senha_hash
    API->>API: bcrypt.compare(senha_atual, senha_hash)
    API->>API: Verifica que a nova senha é diferente
    API->>API: bcrypt.hash(nova_senha, 10)
    API->>DB: Grava o novo senha_hash
    DB-->>API: Usuário atualizado
    API-->>Front: 200 - Senha alterada
    Front->>Usuario: Limpa os campos e confirma a alteração
```

### Exceções principais

- Senha atual incorreta: o backend recusa a operação.
- Nova senha igual à atual: o backend recusa a operação.
- Senha ou confirmação inválida: o frontend não envia a requisição.

---

## 2A. Solicitar recuperação de conta (RF-02)

```mermaid
sequenceDiagram
    autonumber
    actor Usuario as Usuário não autenticado
    participant Front as Frontend Expo
    participant API as Backend
    participant DB as MongoDB
    participant Email as Serviço de e-mail

    Usuario->>Front: Informa o e-mail e toca em "Enviar link"
    Front->>Front: Valida e normaliza o e-mail
    Front->>Front: Linking.createURL("login/redefinir-senha")
    Front->>API: POST /auth/recuperar-conta {email, redirect_url}
    API->>DB: Busca conta ativa pelo e-mail
    alt Conta ativa encontrada
        DB-->>API: Usuário
        API->>API: Gera token aleatório de 32 bytes
        API->>API: Calcula SHA-256 do token
        API->>DB: Grava hash e expiração de 30 minutos
        DB-->>API: Recuperação registrada
        API->>API: Monta página HTTP local com o host do Expo
        API->>Email: Envia o link de recuperação
    else Conta inexistente ou inativa
        DB-->>API: Nenhum usuário
    end
    API-->>Front: 200 - Mensagem genérica
    Front->>Usuario: Orienta a conferir o e-mail
```

### Regra de privacidade

O resultado mostrado é o mesmo para e-mail existente, inexistente ou inativo. Assim, a tela não confirma quais pessoas possuem conta.

---

## 2B. Abrir o link e redefinir a senha (RF-02)

```mermaid
sequenceDiagram
    autonumber
    actor Usuario as Usuário
    participant Navegador as Navegador do celular
    participant API as Backend
    participant Expo as Expo Go
    participant Front as Tela de redefinição
    participant DB as MongoDB

    Usuario->>Navegador: Toca no botão do e-mail
    Navegador->>API: GET /auth/abrir-redefinicao?token&redirect_url
    API->>API: Valida token hexadecimal e URL exata do Expo Go
    API-->>Navegador: Página HTML com link exp://.../redefinir-senha?token
    Navegador->>Expo: Abre o projeto pelo protocolo exp://
    Expo->>Front: Navega para /login/redefinir-senha com token
    Usuario->>Front: Informa e confirma a nova senha
    Front->>API: POST /auth/redefinir-senha {token, nova_senha}
    API->>API: Calcula SHA-256 do token recebido
    API->>DB: Busca conta ativa com hash igual e prazo futuro
    DB-->>API: Usuário válido
    API->>API: Gera bcrypt da nova senha
    API->>DB: Grava senha e remove hash/expiração do reset
    DB-->>API: Usuário atualizado
    API-->>Front: 200 - Senha redefinida
    Front->>Usuario: Exibe sucesso e botão para voltar ao login
```

### Exceções principais

- Token malformado: a página intermediária devolve erro.
- URL do Expo alterada: o backend recusa o redirecionamento.
- Token inexistente, já usado ou expirado: a redefinição é recusada.
- Expo ou backend desligado, ou celular fora da rede: o fluxo local não consegue abrir a tela.

---

## 3. Alterar dados pessoais (RF-03)

```mermaid
sequenceDiagram
    autonumber
    actor Usuario as Usuário autenticado
    participant Front as Frontend
    participant API as Backend
    participant DB as MongoDB

    Usuario->>Front: Abre a tela "Perfil"
    Front->>API: GET /usuarios/me + JWT
    API->>DB: Busca usuário pelo sub do JWT
    DB-->>API: Nome, e-mail, tema e demais campos públicos
    API-->>Front: 200 - Perfil
    Front->>Usuario: Preenche os campos
    Usuario->>Front: Altera nome/e-mail e salva
    Front->>Front: Valida nome e e-mail
    Front->>API: PATCH /usuarios/me {nome, email, tema} + JWT
    API->>DB: Verifica e-mail em outra conta
    DB-->>API: E-mail disponível
    API->>DB: Salva nome e e-mail normalizado
    DB-->>API: Perfil atualizado
    API-->>Front: 200 - Perfil atualizado
    Front->>Usuario: Exibe confirmação
```

### Exceção principal

Se o e-mail já pertence a outra conta, o backend responde conflito e não salva nenhuma alteração.

---

## 4. Definir tema preferido (RF-04)

```mermaid
sequenceDiagram
    autonumber
    actor Usuario as Usuário autenticado
    participant Perfil as Tela Perfil
    participant API as Backend
    participant DB as MongoDB
    participant Tema as TemaContext
    participant Local as SecureStore

    Usuario->>Perfil: Seleciona claro, escuro ou sistema
    Perfil->>Perfil: Mantém a preferência como rascunho
    Usuario->>Perfil: Toca em "Salvar alterações"
    Perfil->>API: PATCH /usuarios/me {nome, email, tema} + JWT
    API->>DB: Valida enum e grava o tema
    DB-->>API: Perfil atualizado
    API-->>Perfil: 200 - Alteração confirmada
    Perfil->>Tema: definirPreferencia(tema)
    Tema->>Local: Grava conecta_mais_tema
    Tema-->>Perfil: Aplica a paleta correspondente
    Perfil->>Usuario: Exibe confirmação com o novo tema
```

### Comportamento da opção `sistema`

O `TemaContext` consulta `useColorScheme()`. Se o aparelho estiver em modo escuro, usa a paleta escura; nos demais casos, usa a clara.

