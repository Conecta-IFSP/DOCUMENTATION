# Diagrama Relacional - Reuniões

O banco do sistema é o MongoDB, que guarda os participantes e as presenças dentro do documento da reunião. Este diagrama representa esses dados como tabelas relacionais, para mostrar as chaves e os relacionamentos entre eles.

```mermaid
erDiagram
    USUARIO ||--o{ REUNIAO : "organiza"
    COMISSAO ||--o{ REUNIAO : "realiza"
    ORGANIZACAO ||--o{ COMISSAO : "possui"
    ORGANIZACAO ||--|{ MEMBRO_ORGANIZACAO : "tem"
    USUARIO ||--o{ MEMBRO_ORGANIZACAO : "participa"
    COMISSAO ||--|{ MEMBRO_COMISSAO : "tem"
    USUARIO ||--o{ MEMBRO_COMISSAO : "integra"
    REUNIAO ||--|{ PARTICIPANTE_REUNIAO : "convida"
    USUARIO ||--o{ PARTICIPANTE_REUNIAO : "é convidado"

    REUNIAO {
        ObjectId _id PK
        ObjectId comissao_id FK
        String titulo
        String local
        Date inicio
        Date fim
        String pauta
        ObjectId criada_por FK
        String situacao
        Date criado_em
        Date atualizado_em
    }

    PARTICIPANTE_REUNIAO {
        ObjectId reuniao_id PK,FK
        ObjectId usuario_id PK,FK
        String convite
        Date presenca_entrada
        Date presenca_saida
        String presenca_situacao
        String presenca_marcada_por
    }

    COMISSAO {
        ObjectId _id PK
        String nome
        ObjectId organizacao_id FK
        Boolean ativo
    }

    MEMBRO_COMISSAO {
        ObjectId comissao_id PK,FK
        ObjectId usuario_id PK,FK
        String papel
    }

    ORGANIZACAO {
        ObjectId _id PK
        String nome
        String status
    }

    MEMBRO_ORGANIZACAO {
        ObjectId organizacao_id PK,FK
        ObjectId usuario_id PK,FK
        String papel
        String status
    }

    USUARIO {
        ObjectId _id PK
        String nome
        String email UK
        String tipo
        Boolean ativo
    }
```

As tabelas `COMISSAO`, `MEMBRO_COMISSAO`, `ORGANIZACAO`, `MEMBRO_ORGANIZACAO` e `USUARIO` pertencem ao Incremento 1 e aparecem apenas com as colunas usadas pelas reuniões.
