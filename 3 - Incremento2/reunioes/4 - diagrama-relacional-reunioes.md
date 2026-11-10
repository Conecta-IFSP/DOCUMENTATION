# Diagrama relacional — Reuniões

> Padrão da disciplina · Módulo REU · Visão lógica dos documentos MongoDB · Revisado em 10/11/2026

## Resumo

No MongoDB, participantes, presenças, anexos e ata ficam embutidos no documento de `reunioes`. O diagrama os separa logicamente para explicitar cardinalidades, chaves e restrições. O conteúdo dos arquivos anexados fica no GridFS, nas coleções `anexos.files` e `anexos.chunks`, e é ligado ao anexo pelo mesmo identificador.

A ata aparece aqui apenas como ligação; seus campos estão no [diagrama relacional das atas](../../4%20-%20Incremento3/atas/4%20-%20diagrama-relacional-atas.md).

## Diagrama

```mermaid
erDiagram
    ORGANIZACAO ||--o{ COMISSAO : possui
    COMISSAO ||--o{ REUNIAO : realiza
    USUARIO ||--o{ REUNIAO : organiza
    REUNIAO ||--|{ REUNIAO_PARTICIPANTE : contem
    USUARIO ||--o{ REUNIAO_PARTICIPANTE : participa
    REUNIAO_PARTICIPANTE ||--o| REUNIAO_PRESENCA : registra
    REUNIAO ||--o{ REUNIAO_ANEXO : "tem na pauta"
    REUNIAO_ANEXO ||--|| ARQUIVO_GRIDFS : armazena
    REUNIAO ||--o| REUNIAO_ATA : documenta
    USUARIO ||--o{ REUNIAO_ATA : redige

    REUNIAO {
        ObjectId _id PK
        ObjectId comissao_id FK
        String titulo
        String local
        Date inicio
        Date fim
        Boolean dia_inteiro
        Date horario_alterado_em
        String pauta
        ObjectId criada_por FK
        String situacao
        Date criado_em
        Date atualizado_em
    }
    REUNIAO_ANEXO {
        ObjectId reuniao_id PK,FK
        ObjectId _id PK,FK
        String nome
        String tipo
        Int tamanho
        Date enviado_em
    }
    ARQUIVO_GRIDFS {
        ObjectId _id PK
        String filename
        Int length
        Int chunkSize
        Date uploadDate
        Object metadata
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

| Visão lógica           | MongoDB                                                     |
| ---------------------- | ----------------------------------------------------------- |
| `REUNIAO`              | coleção `reunioes`                                          |
| `REUNIAO_PARTICIPANTE` | `reunioes.participantes[]`                                  |
| `REUNIAO_PRESENCA`     | `reunioes.participantes[].presenca`                         |
| `REUNIAO_ANEXO`        | `reunioes.anexos[]`                                         |
| `ARQUIVO_GRIDFS`       | coleções `anexos.files` e `anexos.chunks`, criadas pelo GridFS |
| `REUNIAO_ATA`          | `reunioes.ata` e `reunioes.atas_canceladas[]`               |

`REUNIAO_ANEXO._id` e `ARQUIVO_GRIDFS._id` são o mesmo valor: é assim que o sistema acha o conteúdo do arquivo a partir da reunião. O GridFS divide o arquivo em pedaços de 255 KB em `anexos.chunks`; o aplicativo nunca acessa essas coleções diretamente.

## Rastreabilidade

RF-REU-01, RF-REU-05 a RF-REU-09 e RF-REU-13 a RF-REU-16. Os campos da ata estão no [dicionário relacional das atas](../../4%20-%20Incremento3/atas/4%20-%20dicionario-dados-relacional-atas.md).
