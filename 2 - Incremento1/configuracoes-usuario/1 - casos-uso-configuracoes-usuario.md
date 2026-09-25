# Diagrama de Casos de Uso - Configurações de Usuário

```mermaid
flowchart LR
    Visitante["Usuário não autenticado"]
    Usuario["Usuário autenticado"]

    UC1["UC1 - Trocar senha"]
    UC2["UC2 - Recuperar conta"]
    UC3["UC3 - Alterar dados pessoais"]
    UC4["UC4 - Definir tema preferido"]

    Email["Serviço de e-mail"]
    Expo["Expo Go"]
    Servidor["Servidor"]

    Usuario --> UC1
    Visitante --> UC2
    Usuario --> UC3
    Usuario --> UC4

    UC1 --> Servidor
    UC2 --> Servidor
    UC2 --> Email
    UC2 --> Expo
    UC3 --> Servidor
    UC4 --> Servidor

    classDef actor stroke:#1976d2,stroke-width:2px;
    classDef uc stroke:#7b1fa2,stroke-width:2px;

    class Visitante,Usuario,Email,Expo,Servidor actor;
    class UC1,UC2,UC3,UC4 uc;
```

## Casos de uso

- **UC1 - Trocar senha:** o usuário autenticado informa a senha atual e escolhe uma nova senha para os próximos acessos.
- **UC2 - Recuperar conta:** o usuário sem acesso à conta solicita um link temporário por e-mail e define uma nova senha no Expo Go.
- **UC3 - Alterar dados pessoais:** o usuário autenticado consulta e altera seu nome e seu e-mail.
- **UC4 - Definir tema preferido:** o usuário escolhe entre tema claro, escuro ou o tema do sistema; a escolha só é aplicada após o salvamento ser confirmado.

