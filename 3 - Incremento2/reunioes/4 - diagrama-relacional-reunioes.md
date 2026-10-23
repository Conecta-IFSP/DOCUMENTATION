# Diagrama relacional — Reuniões

> Padrão do Incremento 2 · Visão lógica relacional de documentos MongoDB

## Resumo

No MongoDB, participantes, presenças e ata ficam embutidos no documento de `reunioes`. O diagrama os separa logicamente para explicitar cardinalidades, chaves e restrições.

## Diagrama

```mermaid
erDiagram
    ORGANIZACAO ||--o{ COMISSAO : possui
    COMISSAO ||--o{ REUNIAO : realiza
    USUARIO ||--o{ REUNIAO : organiza
    REUNIAO ||--|{ REUNIAO_PARTICIPANTE : contem
    USUARIO ||--o{ REUNIAO_PARTICIPANTE : participa
    REUNIAO_PARTICIPANTE ||--o| REUNIAO_PRESENCA : registra
    REUNIAO ||--o| REUNIAO_ATA : documenta
    USUARIO ||--o{ REUNIAO_ATA : redige
    REUNIAO_ATA ||--o{ ATA_APROVADOR : define
    USUARIO ||--o{ ATA_APROVADOR : aprova

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
    REUNIAO_PARTICIPANTE {
        ObjectId reuniao_id PK,FK
        ObjectId usuario_id PK,FK
        String convite
    }
    REUNIAO_PRESENCA {
        ObjectId reuniao_id PK,FK
        ObjectId usuario_id PK,FK
        Date entrada
        Date saida
        String situacao
        String marcada_por
    }
    REUNIAO_ATA {
        ObjectId reuniao_id PK,FK
        String conteudo
        Int versao
        String situacao
        ObjectId autor_id FK
        Date atualizada_em
    }
    ATA_APROVADOR {
        ObjectId reuniao_id PK,FK
        ObjectId usuario_id PK,FK
        Boolean aprovou
    }
    COMISSAO {
        ObjectId _id PK
        ObjectId organizacao_id FK
        Boolean ativo
    }
    ORGANIZACAO {
        ObjectId _id PK
        String status
    }
    USUARIO {
        ObjectId _id PK
        String nome
        Boolean ativo
    }
```

## Mapeamento físico

| Visão lógica           | MongoDB                                       |
| ---------------------- | --------------------------------------------- |
| `REUNIAO`              | coleção `reunioes`                            |
| `REUNIAO_PARTICIPANTE` | `reunioes.participantes[]`                    |
| `REUNIAO_PRESENCA`     | `reunioes.participantes[].presenca`           |
| `REUNIAO_ATA`          | `reunioes.ata`                                |
| `ATA_APROVADOR`        | `reunioes.ata.aprovadores[]` e `aprovacoes[]` |

## Rastreabilidade

RF-REU-01, RF-REU-05 a RF-REU-12.
