# Diagrama relacional — Reuniões

> Padrão da disciplina · Módulo REU · Visão lógica dos documentos MongoDB · Revisado em 27/11/2026

## Resumo

O banco é o MongoDB, que não tem tabelas: participantes, presenças, anexos, link de presença e correções ficam dentro do documento da reunião, na coleção `reunioes`. O diagrama abaixo é uma **visão lógica**: ele separa esses dados em entidades para deixar claras as chaves, as cardinalidades e as restrições. Além de `reunioes`, o módulo usa três coleções: as duas do GridFS, que guardam o conteúdo dos anexos, e `notificacoes`, que recebe os avisos das reuniões: pedido de presença, convite, mudança de horário, cancelamento e ata a avaliar.

O documento da reunião também guarda a ata (`ata` e `atas_canceladas`) e as votações (`votacoes`). Essas partes estão nos diagramas relacionais das [atas](../../4%20-%20Incremento%203/atas/4%20-%20diagrama-relacional-atas.md) e da [votação](../../4%20-%20Incremento%203/votacao/4%20-%20diagrama-relacional-votacao.md).

## Diagrama

```mermaid
erDiagram
    ORGANIZACAO ||--o{ COMISSAO : possui
    COMISSAO ||--o{ REUNIAO : realiza
    USUARIO ||--o{ REUNIAO : agenda
    REUNIAO ||--|{ REUNIAO_PARTICIPANTE : convida
    USUARIO ||--o{ REUNIAO_PARTICIPANTE : participa
    REUNIAO_PARTICIPANTE ||--o| REUNIAO_PRESENCA : registra
    REUNIAO ||--o{ REUNIAO_ANEXO : "tem na pauta"
    REUNIAO_ANEXO ||--|| ARQUIVO_GRIDFS : armazena
    ARQUIVO_GRIDFS ||--|{ PEDACO_GRIDFS : "dividido em"
    REUNIAO ||--o| LINK_PRESENCA : "tem ativo"
    USUARIO ||--o{ LINK_PRESENCA : gera
    REUNIAO ||--o{ CORRECAO_PRESENCA : "guarda no histórico"
    USUARIO ||--o{ CORRECAO_PRESENCA : corrige
    USUARIO ||--o{ NOTIFICACAO : recebe

    REUNIAO {
        ObjectId _id PK
        ObjectId comissao_id FK
        string titulo
        string local
        date inicio
        date fim
        boolean dia_inteiro
        date horario_alterado_em
        string pauta
        ObjectId criada_por FK
        string situacao
        int presencas_revisao
        date criado_em
        date atualizado_em
    }
    REUNIAO_PARTICIPANTE {
        ObjectId reuniao_id PK,FK
        ObjectId usuario_id PK,FK
        string convite
    }
    REUNIAO_PRESENCA {
        ObjectId reuniao_id PK,FK
        ObjectId usuario_id PK,FK
        date entrada
        date saida
        boolean saida_automatica
        string situacao
        string marcada_por
    }
    REUNIAO_ANEXO {
        ObjectId reuniao_id FK
        ObjectId _id PK,FK
        string nome
        string tipo
        int tamanho
        date enviado_em
    }
    ARQUIVO_GRIDFS {
        ObjectId _id PK
        string filename
        long length
        int chunkSize
        date uploadDate
        object metadata
    }
    PEDACO_GRIDFS {
        ObjectId _id PK
        ObjectId files_id FK
        int n
        binary data
    }
    LINK_PRESENCA {
        ObjectId reuniao_id PK,FK
        string hash
        date inicio
        date fim
        string responsavel_id FK
        date criado_em
    }
    CORRECAO_PRESENCA {
        ObjectId reuniao_id PK,FK
        int posicao PK
        string participante_id FK
        string responsavel_id FK
        date alterada_em
        string motivo
        object antes
        object depois
    }
    NOTIFICACAO {
        ObjectId _id PK
        ObjectId usuario_id FK
        string chave
        string tipo
        string titulo
        string mensagem
        string destino
        date expira_em
        date lida_em
        date criada_em
        date atualizada_em
    }
    COMISSAO {
        ObjectId _id PK
        ObjectId organizacao_id FK
        boolean ativo
    }
    ORGANIZACAO {
        ObjectId _id PK
        string status
    }
    USUARIO {
        ObjectId _id PK
        string nome
        string email
        boolean ativo
    }
```

Figura 3 — Visão lógica relacional das reuniões. Fonte: equipe Conecta+.

## Como as entidades lógicas viram documentos

| Entidade lógica        | Onde fica de verdade                                  | Observação                                                                                         |
| ---------------------- | ----------------------------------------------------- | -------------------------------------------------------------------------------------------------- |
| `REUNIAO`              | coleção `reunioes`                                    | Documento principal.                                                                               |
| `REUNIAO_PARTICIPANTE` | `reunioes.participantes[]`                            | Subdocumento; a chave lógica é a reunião mais o usuário.                                           |
| `REUNIAO_PRESENCA`     | `reunioes.participantes[].presenca`                   | Subdocumento de um participante; só existe depois da entrada.                                      |
| `REUNIAO_ANEXO`        | `reunioes.anexos[]`                                   | Subdocumento; o `_id` é o mesmo do arquivo no GridFS.                                              |
| `ARQUIVO_GRIDFS`       | coleção `anexos.files`                                | Criada e mantida pelo GridFS; uma linha por arquivo.                                               |
| `PEDACO_GRIDFS`        | coleção `anexos.chunks`                               | Criada e mantida pelo GridFS; o conteúdo dividido em pedaços de 255 KB.                            |
| `LINK_PRESENCA`        | `reunioes.acesso_presenca`                            | Objeto embutido; no máximo um por reunião. Não aparece nas consultas comuns.                       |
| `CORRECAO_PRESENCA`    | `reunioes.historico_presencas[]`                      | Item de lista; `posicao` não é gravada: é a posição do item na lista. Não aparece nas consultas comuns. |
| `NOTIFICACAO`          | coleção `notificacoes`                                | O aviso `PRESENCA` é gravado no envio do link; os de `CONVITE`, `HORARIO`, `CANCELAMENTO` e `ATA`, quando o usuário consulta as notificações. A reunião só aparece no texto de `chave` e de `destino`, sem chave estrangeira. |
| `COMISSAO`, `ORGANIZACAO`, `USUARIO` | coleções `comissoes`, `organizacoes` e `usuarios` | Descritas nos módulos do Incremento 1; aqui só os campos usados.                       |

Nas entidades guardadas dentro da reunião, `reuniao_id` aparece no diagrama como parte da chave, mas não existe como campo gravado: ele vem do documento que as contém. O aplicativo nunca acessa as coleções do GridFS diretamente: o servidor lê e grava por meio do `GridFSBucket`.

## Regras de integridade

- um usuário aparece uma única vez em `REUNIAO_PARTICIPANTE` de cada reunião, e quem agendou sempre está na lista;
- todo participante é integrante da comissão da reunião no momento em que é convidado;
- `REUNIAO_PRESENCA` tem no máximo uma linha por participante, e a saída nunca passa do término da reunião;
- uma reunião tem no máximo 10 `REUNIAO_ANEXO`, e cada um corresponde a exatamente um `ARQUIVO_GRIDFS` com o mesmo identificador;
- `LINK_PRESENCA` só vale enquanto o início e o término da reunião forem os mesmos da geração, enquanto `responsavel_id` for o responsável da comissão e antes do término;
- `CORRECAO_PRESENCA` só cresce: nenhum registro é alterado ou apagado;
- `NOTIFICACAO` não se repete para o mesmo usuário e a mesma `chave`;
- depois que a reunião é cancelada ou a ata é enviada, nenhuma presença muda, nenhuma correção é registrada e nenhum link é gerado; o link existente ainda pode ser revogado.

## Rastreabilidade

RF-REU-01 a RF-REU-09 e RF-REU-13 a RF-REU-22; detalhado em [Dicionário relacional](4%20-%20dicionario-dados-relacional-reunioes.md).
