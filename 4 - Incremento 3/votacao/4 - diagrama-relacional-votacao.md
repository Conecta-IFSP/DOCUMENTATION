# Diagrama relacional — Votação na reunião

> Padrão da disciplina · Módulo VOT · Visão lógica dos documentos MongoDB · Revisado em 13/11/2026

## Resumo

O banco é o MongoDB, que não tem tabelas: as votações, suas opções e os votos ficam dentro do documento da reunião, na coleção `reunioes`. O diagrama abaixo é uma **visão lógica**: ele separa esses dados em entidades para deixar claras as chaves, as cardinalidades e as restrições. A votação não cria nenhuma coleção nova.

## Diagrama

```mermaid
erDiagram
  COMISSAO ||--o{ REUNIAO : realiza
  REUNIAO ||--|{ REUNIAO_PARTICIPANTE : convida
  USUARIO ||--o{ REUNIAO_PARTICIPANTE : participa
  REUNIAO_PARTICIPANTE ||--o| REUNIAO_PRESENCA : registra
  REUNIAO ||--o{ VOTACAO : "abre durante"
  USUARIO ||--o{ VOTACAO : "abre (presidente)"
  VOTACAO ||--|{ VOTACAO_OPCAO : oferece
  VOTACAO ||--o{ VOTO : recebe
  VOTACAO_OPCAO ||--o{ VOTO : "escolhida em"
  USUARIO ||--o{ VOTO : vota

  REUNIAO {
    ObjectId _id PK
    ObjectId comissao_id FK
    date inicio
    date fim
    string situacao
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
    string situacao
  }
  VOTACAO {
    ObjectId _id PK
    ObjectId reuniao_id FK
    string titulo
    string descricao
    string situacao
    ObjectId aberta_por FK
    date aberta_em
    date encerrada_em
  }
  VOTACAO_OPCAO {
    ObjectId votacao_id PK,FK
    int indice PK
    string texto
  }
  VOTO {
    ObjectId votacao_id PK,FK
    ObjectId usuario_id PK,FK
    int opcao FK
    date votado_em
  }
  COMISSAO {
    ObjectId _id PK
    boolean ativo
  }
  USUARIO {
    ObjectId _id PK
    string nome
    boolean ativo
  }
```

Figura 3 — Visão lógica relacional da votação. Fonte: equipe Conecta+.

## Como as entidades lógicas viram documentos

| Entidade lógica | Onde fica de verdade | Observação |
| --- | --- | --- |
| `REUNIAO` | coleção `reunioes` | Documento principal. |
| `REUNIAO_PARTICIPANTE` | `reunioes.participantes[]` | Subdocumento; a chave lógica é a reunião mais o usuário. |
| `REUNIAO_PRESENCA` | `reunioes.participantes[].presenca` | Subdocumento de um participante; define quem pode votar. |
| `VOTACAO` | `reunioes.votacoes[]` | Subdocumento com `_id` próprio. `reuniao_id` não é gravado: vem do documento que contém a votação. |
| `VOTACAO_OPCAO` | `reunioes.votacoes[].opcoes[]` | Lista de textos. `indice` não é gravado: é a posição do texto na lista, começando em 0. |
| `VOTO` | `reunioes.votacoes[].votos[]` | Subdocumento sem `_id`. `votacao_id` vem da votação que o contém; `opcao` é o `indice` da opção escolhida. |

## Regras de integridade

- um usuário aparece no máximo uma vez em `VOTO` para cada `VOTACAO`; a gravação só acontece se o filtro `votos.usuario_id` diferente do usuário for verdadeiro no mesmo comando;
- `VOTO.opcao` é sempre menor que a quantidade de `VOTACAO_OPCAO` da votação;
- `VOTO.usuario_id` precisa estar em `REUNIAO_PARTICIPANTE`, com `REUNIAO_PRESENCA.situacao` igual a `AGUARDANDO` ou `CONFIRMADA` e sem `saida`, no instante do voto;
- os textos de `VOTACAO_OPCAO` não se repetem na mesma votação, ignorando maiúsculas e minúsculas;
- `VOTACAO.aberta_por` é membro `RESPONSAVEL` da comissão da reunião no instante da abertura;
- `VOTACAO.aberta_em` fica entre `REUNIAO.inicio` e `REUNIAO.fim`;
- `VOTACAO.encerrada_em` só existe com `situacao = ENCERRADA` e nunca passa de `REUNIAO.fim`;
- nenhum dado de votação é alterado em uma reunião `CANCELADA`.

## Rastreabilidade

RF-VOT-01 a RF-VOT-04; detalhado em [Dicionário relacional](4%20-%20dicionario-dados-relacional-votacao.md).
