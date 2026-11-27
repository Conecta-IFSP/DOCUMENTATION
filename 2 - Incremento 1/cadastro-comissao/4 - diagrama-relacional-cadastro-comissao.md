# Diagrama relacional — Cadastro de comissão

> Padrão do Incremento 1 · Visão lógica relacional de documentos MongoDB.

## Resumo

No MongoDB, `membros` é um array embutido em `comissoes`. O diagrama separa esse array apenas para explicitar cardinalidade, chaves e restrições.

## Diagrama

```mermaid
erDiagram
    ORGANIZACAO ||--o{ COMISSAO : possui
    COMISSAO ||--|{ COMISSAO_MEMBRO : contem
    USUARIO ||--o{ COMISSAO_MEMBRO : participa

    ORGANIZACAO {
        ObjectId _id PK
        String status
    }
    COMISSAO {
        ObjectId _id PK
        String nome
        String descricao
        ObjectId organizacao_id FK
        Boolean ativo
        Date criado_em
        Date atualizado_em
    }
    COMISSAO_MEMBRO {
        ObjectId comissao_id PK,FK
        ObjectId usuario_id PK,FK
        String papel
    }
    USUARIO {
        ObjectId _id PK
        Boolean ativo
    }
```

## Mapeamento físico

| Visão lógica        | MongoDB                                 |
| ------------------- | --------------------------------------- |
| `COMISSAO`          | coleção `comissoes`                     |
| `COMISSAO_MEMBRO`   | subdocumentos `comissoes.membros[]`     |
| FK `organizacao_id` | `ObjectId` referenciando `organizacoes` |
| FK `usuario_id`     | `ObjectId` referenciando `usuarios`     |

## Rastreabilidade

RF-COM-01 a RF-COM-09 e dicionário relacional do módulo.
