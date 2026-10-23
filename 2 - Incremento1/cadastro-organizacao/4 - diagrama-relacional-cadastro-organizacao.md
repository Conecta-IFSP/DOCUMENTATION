# Diagrama relacional — Cadastro de organização

> Padrão do Incremento 1 · Módulo ORG · Visão lógica sobre MongoDB

## Resumo

Representa organizações, vínculos embutidos, criador e comissões associadas em forma relacional para facilitar a leitura.

```mermaid
erDiagram
    USUARIO ||--o{ ORGANIZACAO : cadastra
    ORGANIZACAO ||--o{ ORGANIZACAO_MEMBRO : possui
    ORGANIZACAO ||--o{ COMISSAO : possui

    ORGANIZACAO {
        ObjectId _id PK
        String nome UK
        String cidade
        String descricao
        String status
        ObjectId criada_por FK
        String codigo_acesso UK
        Date criado_em
        Date atualizado_em
    }

    USUARIO {
        ObjectId _id PK
    }

    ORGANIZACAO_MEMBRO {
        ObjectId organizacao_id PK,FK
        ObjectId usuario_id PK,FK
        String papel
        String status
        Date solicitado_em
        Date aprovado_em
    }

    COMISSAO {
        ObjectId _id PK
        ObjectId organizacao_id FK
    }
```

## Relacionamentos

- `ORGANIZACAO.criada_por → USUARIO._id`: um usuário pode cadastrar várias organizações.
- `ORGANIZACAO_MEMBRO.organizacao_id → ORGANIZACAO._id`: uma organização tem vários vínculos de membro.
- `ORGANIZACAO_MEMBRO` é uma representação lógica: no MongoDB, esses campos ficam embutidos em `organizacoes.membros[]`.
- `COMISSAO.organizacao_id → ORGANIZACAO._id`: uma organização tem várias comissões, e a exclusão da organização revogada (RF-06) apaga as comissões ligadas a ela.
