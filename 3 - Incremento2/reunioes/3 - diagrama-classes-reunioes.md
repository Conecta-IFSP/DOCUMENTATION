# Diagrama de classes — Reuniões

> Padrão do Incremento 2 · Classes confirmadas no código auditado

## Resumo

`ReunioesController` expõe as rotas, `ReunioesService` concentra autorização e regras, e `Reuniao` persiste participantes, presenças e ata como subdocumentos no MongoDB. As telas Expo consomem a API por meio do módulo `api`.

## Diagrama

```mermaid
classDiagram
    class ReunioesController {
        +listar(req, de, ate, comissaoId)
        +criar(dto, req)
        +buscarPorId(id, req)
        +atualizar(id, dto, req)
        +responderConvite(id, dto, req)
        +marcarEntrada(id, req)
        +marcarSaida(id, req)
        +atualizarPresenca(id, usuarioId, dto, req)
        +gerarListaPresenca(id, req)
        +salvarAta(id, dto, req)
        +publicarAta(id, dto, req)
        +aprovarAta(id, dto, req)
    }
    class ReunioesService {
        +listar(usuarioId, filtros)
        +criar(dto, usuarioId)
        +buscarPorId(id, usuarioId)
        +atualizar(id, dto, usuarioId)
        +responderConvite(id, aceito, usuarioId)
        +marcarEntrada(id, usuarioId)
        +marcarSaida(id, usuarioId)
        +atualizarPresenca(id, alvoId, situacao, usuarioId)
        +gerarListaPresenca(id, usuarioId)
        +salvarAta(id, dto, usuarioId)
        +publicarAta(id, versao, usuarioId)
        +aprovarAta(id, versao, usuarioId)
    }
    class Reuniao {
        +ObjectId _id
        +ObjectId comissao_id
        +String titulo
        +String local
        +Date inicio
        +Date fim
        +String pauta
        +ObjectId criada_por
        +SituacaoReuniao situacao
        +Participante[] participantes
        +Ata ata
        +Date criado_em
        +Date atualizado_em
    }
    class Participante {
        +ObjectId usuario_id
        +RespostaConvite convite
        +Presenca presenca
    }
    class Presenca {
        +Date entrada
        +Date saida
        +SituacaoPresenca situacao
        +OrigemPresenca marcada_por
    }
    class Ata {
        +String conteudo
        +Number versao
        +String situacao
        +ObjectId autor_id
        +Date atualizada_em
        +ObjectId[] aprovadores
        +ObjectId[] aprovacoes
    }
    class Comissao {
        +ObjectId _id
        +ObjectId organizacao_id
        +Boolean ativo
        +MembroComissao[] membros
    }
    class Usuario {
        +ObjectId _id
        +String nome
        +Boolean ativo
    }
    class TelaAgenda {
        +carregar()
        +responderConvite()
    }
    class TelaNovaReuniao {
        +salvar()
    }
    class TelaReuniao {
        +buscar()
        +executar()
        +marcarPresenca()
    }
    class TelaRegistrosReuniao {
        +carregar()
        +carregarLista()
        +gravar()
        +compartilhar()
    }
    class api {
        +api(caminho, metodo, corpo)
    }

    ReunioesController --> ReunioesService : delega
    ReunioesService --> Reuniao : consulta e atualiza
    ReunioesService --> Comissao : valida papel
    ReunioesService --> Usuario : valida e popula
    Reuniao "1" *-- "1..*" Participante : participantes
    Participante "1" *-- "0..1" Presenca : presenca
    Reuniao "1" *-- "0..1" Ata : ata
    Usuario "1" --> "0..*" Participante : participa
    Usuario "1" --> "0..*" Ata : autor/aprovador
    Comissao "1" --> "0..*" Reuniao : realiza
    TelaAgenda --> api
    TelaNovaReuniao --> api
    TelaReuniao --> api
    TelaRegistrosReuniao --> api
    api --> ReunioesController : HTTP
```

## Regras representadas

- `Participante`, `Presenca` e `Ata` são subdocumentos de `Reuniao`;
- o organizador é identificado por `criada_por` e deve ser o responsável da comissão no agendamento;
- a presença do participante comum começa em `AGUARDANDO`; a do organizador, em `CONFIRMADA`;
- a ata usa versão para impedir sobrescrita concorrente;
- após a publicação, o texto não muda e somente os IDs congelados em `aprovadores` podem aprovar;
- todas as aprovações são necessárias para a situação `APROVADA`.

## Rastreabilidade

RF-REU-01 a RF-REU-12; dicionário de classes e diagramas de sequência do módulo.
