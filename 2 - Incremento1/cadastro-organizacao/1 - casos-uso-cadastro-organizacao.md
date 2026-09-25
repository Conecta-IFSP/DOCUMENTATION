# Diagrama de Casos de Uso - Cadastro de Organização

```mermaid
flowchart LR
    Usuario["Usuário Comum"]
    AdminSis["Administrador do Sistema"]

    UC1["UC1 - Cadastrar organização"]
    UC2["UC2 - Consultar organizações cadastradas"]
    UC3["UC3 - Autorizar cadastro de organização"]
    UC4["UC4 - Revogar organização"]
    UC5["UC5 - Editar dados da organização"]
    UC6["UC6 - Excluir organização revogada"]

    Servidor["Servidor"]

    Usuario --> UC1
    AdminSis --> UC2
    AdminSis --> UC3
    AdminSis --> UC4
    AdminSis --> UC5
    AdminSis --> UC6
    UC1 --> Servidor
    UC2 --> Servidor
    UC3 --> Servidor
    UC4 --> Servidor
    UC5 --> Servidor
    UC6 --> Servidor

    classDef actor stroke:#1976d2,stroke-width:2px;
    classDef uc stroke:#7b1fa2,stroke-width:2px;

    class Usuario,AdminSis,Servidor actor;
    class UC1,UC2,UC3,UC4,UC5,UC6 uc;
```

## Casos de uso

- **UC1 - Cadastrar organização:** o usuário cria uma organização, que fica aguardando autorização.
- **UC2 - Consultar organizações cadastradas:** o administrador vê as organizações cadastradas e a situação de cada uma.
- **UC3 - Autorizar cadastro de organização:** o administrador aprova uma organização e a libera para uso na plataforma.
- **UC4 - Revogar organização:** o administrador recusa uma organização pendente ou suspende uma já aprovada.
- **UC5 - Editar dados da organização:** o administrador corrige o nome e a descrição de uma organização.
- **UC6 - Excluir organização revogada:** o administrador apaga em definitivo uma organização revogada e as comissões dela.
