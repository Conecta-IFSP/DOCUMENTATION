# Diagrama Relacional

Este incremento não cria tabelas novas — usa as mesmas entidades já modeladas no cadastro de usuário e comissões.

```mermaid
erDiagram
    USUARIO ||--o{ ORGANIZACAO : "cria"
    ORGANIZACAO ||--o{ COMISSAO : "possui"

    USUARIO {
        ObjectId _id PK
        String email UK
        Enum tipo
        Boolean ativo
    }
    ORGANIZACAO {
        ObjectId _id PK
        Enum status
    }
    COMISSAO {
        ObjectId _id PK
        ObjectId organizacao_id FK
        Boolean ativo
    }
```

| Ação do módulo | Tabelas envolvidas | Operação |
|-----------------|----------------------|----------|
| Status / Relatórios | usuario, organizacao, comissao | `COUNT` |
| Popular dados de teste | usuario, organizacao, comissao | `INSERT` |
| Reset | usuario (exceto admin), organizacao, comissao | `DELETE` |
