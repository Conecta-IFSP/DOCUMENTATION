# Diagrama de classes — Cadastro de comissão

> Padrão do Incremento 1 · Classes efetivamente envolvidas na branch `homolog`.

## Resumo

O controller recebe as requisições, o service aplica autorização e regras de equipe, e o documento `Comissao` persiste membros embutidos. `Organizacao` e `Usuario` são consultados para validar autorização e elegibilidade.

## Diagrama

```mermaid
classDiagram
    class ComissoesController {
        +listar()
        +buscarPorId(id)
        +criar(dto)
        +atualizar(id, dto)
        +adicionarMembro(id, dto)
        +definirResponsavel(id, dto)
        +removerMembro(id, usuarioId)
    }
    class ComissoesService {
        +listar(usuarioId)
        +buscarPorId(id, usuarioId)
        +criar(dto, usuarioId)
        +atualizar(id, dto, usuarioId)
        +adicionarMembro(id, dto, usuarioId)
        +definirResponsavel(id, dto, usuarioId)
        +removerMembro(id, alvoId, usuarioId)
    }
    class Comissao {
        +ObjectId _id
        +String nome
        +String descricao
        +ObjectId organizacao_id
        +Boolean ativo
        +MembroComissao[] membros
        +Date criado_em
        +Date atualizado_em
    }
    class MembroComissao {
        +ObjectId usuario_id
        +PapelComissao papel
    }
    class PapelComissao {
        <<enumeration>>
        RESPONSAVEL
        MEMBRO
    }
    class Organizacao
    class Usuario

    ComissoesController --> ComissoesService : delega
    ComissoesService --> Comissao : consulta e atualiza
    ComissoesService --> Organizacao : autoriza
    ComissoesService --> Usuario : valida elegibilidade
    Comissao "1" *-- "1..*" MembroComissao : contém
    MembroComissao --> PapelComissao : papel
```

## Regras representadas

- cada comissão pertence a uma organização;
- `membros.usuario_id` não se repete;
- existe exatamente um `RESPONSAVEL`;
- a substituição do responsável promove o alvo e rebaixa o anterior na mesma escrita.

## Rastreabilidade

RF-COM-01 a RF-COM-09 e dicionário de classes do módulo.
