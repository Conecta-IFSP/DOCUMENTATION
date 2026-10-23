# Diagrama de Classes - Reuniões

O diagrama está dividido em três partes: as classes de dados armazenadas no banco, as classes do servidor que atendem às requisições e as classes do aplicativo usadas pelas telas de reunião.

---

## 1. Classes de Dados

```mermaid
classDiagram
    direction LR

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

    class Comissao {
        +ObjectId _id
        +String nome
        +ObjectId organizacao_id
        +MembroComissao[] membros
        +Boolean ativo
    }

    class MembroComissao {
        +ObjectId usuario_id
        +PapelComissao papel
    }

    class Organizacao {
        +ObjectId _id
        +String nome
        +StatusOrganizacao status
        +MembroOrganizacao[] membros
    }

    class MembroOrganizacao {
        +ObjectId usuario_id
        +PapelOrganizacao papel
        +StatusMembroOrganizacao status
    }

    class Usuario {
        +ObjectId _id
        +String nome
        +String email
        +TipoUsuario tipo
        +Boolean ativo
    }

    class SituacaoReuniao {
        <<enumeration>>
        AGENDADA
        CANCELADA
    }

    class RespostaConvite {
        <<enumeration>>
        PENDENTE
        ACEITO
        RECUSADO
    }

    class SituacaoPresenca {
        <<enumeration>>
        AGUARDANDO
        CONFIRMADA
        RECUSADA
    }

    class OrigemPresenca {
        <<enumeration>>
        PROPRIO
        RESPONSAVEL
    }

    Reuniao "1" *-- "1..*" Participante : participantes
    Participante "1" *-- "0..1" Presenca : presenca
    Comissao "1" o-- "0..*" Reuniao : comissao_id
    Comissao "1" *-- "1..*" MembroComissao : membros
    Organizacao "1" o-- "0..*" Comissao : organizacao_id
    Organizacao "1" *-- "1..*" MembroOrganizacao : membros
    Usuario "1" -- "0..*" Participante : usuario_id
    Usuario "1" -- "0..*" Reuniao : criada_por
    Usuario "1" -- "0..*" MembroComissao : usuario_id
    Usuario "1" -- "0..*" MembroOrganizacao : usuario_id

    Reuniao ..> SituacaoReuniao
    Participante ..> RespostaConvite
    Presenca ..> SituacaoPresenca
    Presenca ..> OrigemPresenca
```

`Participante` e `Presenca` só existem dentro de uma `Reuniao`. `Comissao`, `MembroComissao`, `Organizacao`, `MembroOrganizacao` e `Usuario` pertencem ao Incremento 1 e aparecem apenas com os atributos usados pelas reuniões.

---

## 2. Classes do Servidor

```mermaid
classDiagram
    direction LR

    class ReunioesController {
        -ReunioesService reunioesService
        +listar(req, de, ate, comissaoId) Reuniao[]
        +criar(dto: CreateReuniaoDto, req) Reuniao
        +buscarPorId(id, req) Reuniao
        +atualizar(id, dto: UpdateReuniaoDto, req) Reuniao
        +responderConvite(id, dto: ResponderConviteReuniaoDto, req) Reuniao
        +marcarEntrada(id, req) Reuniao
        +marcarSaida(id, req) Reuniao
        +atualizarPresenca(id, usuarioId, dto: AtualizarPresencaDto, req) Reuniao
        +gerarListaPresenca(id, req) ListaPresenca
    }

    class ReunioesService {
        -Model~Reuniao~ reuniaoModel
        -Model~Comissao~ comissaoModel
        -Model~Organizacao~ organizacaoModel
        -Model~Usuario~ usuarioModel
        +criar(dto, usuarioId) Reuniao
        +listar(usuarioId, filtros) Reuniao[]
        +buscarPorId(id, usuarioId) Reuniao
        +atualizar(id, dto, usuarioId) Reuniao
        +responderConvite(id, aceito, usuarioId) Reuniao
        +marcarEntrada(id, usuarioId) Reuniao
        +marcarSaida(id, usuarioId) Reuniao
        +atualizarPresenca(id, participanteId, situacao, usuarioId) Reuniao
        +gerarListaPresenca(id, usuarioId) ListaPresenca
        -validarId(id) void
        -validarUsuario(usuarioId) void
        -lerData(valor, mensagem) Date
        -validarHorario(inicio, fim) void
        -autorizarResponsavel(comissaoId, usuarioId) Comissao
        -carregar(id, usuarioId) CargaReuniao
        -carregarComoDono(id, usuarioId) CargaReuniao
        -exigirAgendada(reuniao) void
        -popular(reuniao) Reuniao
        -apresentar(reuniao, usuarioId) Reuniao
        -responder(reuniao, usuarioId, conflito) Reuniao
    }

    class Model~T~ {
        <<Mongoose>>
        +find(filtro) T[]
        +findById(id) T
        +findOne(filtro) T
        +create(dados) T
        +findOneAndUpdate(filtro, atualizacao, opcoes) T
    }

    class CreateReuniaoDto {
        +String comissao_id
        +String titulo
        +String local
        +String inicio
        +String fim
        +String pauta
        +String[] convidados_ids
    }

    class UpdateReuniaoDto {
        +SituacaoReuniao situacao
    }

    class ResponderConviteReuniaoDto {
        +Boolean aceito
    }

    class AtualizarPresencaDto {
        +SituacaoPresenca situacao
    }

    class ApiKeyGuard {
        -ConfigService configService
        +canActivate(context) Boolean
        -chavesIguais(recebida, esperada) Boolean
    }

    class JwtAuthGuard {
        -JwtService jwtService
        -Model~Usuario~ usuarios
        +canActivate(context) Boolean
        -extrairToken(requisicao) String
    }

    class UsuarioComumGuard {
        +canActivate(context) Boolean
    }

    class ValidationPipe {
        <<NestJS>>
        +transform(valor, metadata) Object
    }

    class JwtService {
        <<NestJS>>
        +verifyAsync(token) UsuarioAutenticado
    }

    ReunioesController --> ReunioesService
    ReunioesService --> Model
    JwtAuthGuard --> JwtService
    JwtAuthGuard --> Model
    ReunioesController ..> CreateReuniaoDto
    ReunioesController ..> UpdateReuniaoDto
    ReunioesController ..> ResponderConviteReuniaoDto
    ReunioesController ..> AtualizarPresencaDto
    ValidationPipe ..> CreateReuniaoDto : valida
    ApiKeyGuard ..> ReunioesController : protege
    JwtAuthGuard ..> ReunioesController : protege
    UsuarioComumGuard ..> ReunioesController : protege
```

---

## 3. Classes do Aplicativo

```mermaid
classDiagram
    direction LR

    class TelaNovaReuniao {
        <<tela>>
        +salvar() void
    }

    class TelaAgenda {
        <<tela>>
        +carregar() void
        +escolherDia(dia) void
        +responder(reuniao, aceito) void
    }

    class TelaComissao {
        <<tela>>
        +buscarReunioes() Reuniao[]
    }

    class TelaReuniao {
        <<tela>>
        +buscar() void
        +executar(chave, acao, mensagem) void
        +marcarPresenca(pessoa, situacao) void
    }

    class TelaRegistrosReuniao {
        <<tela>>
        +carregar() void
        +carregarLista() void
        +compartilhar() void
    }

    class reunioes {
        <<módulo>>
        +comissoesQueComando(comissoes, usuarioId) Comissao[]
    }

    class api {
        <<módulo>>
        +api(caminho, metodo, corpo) Resultado
        +obterToken() String
    }

    TelaNovaReuniao ..> api
    TelaAgenda ..> api
    TelaAgenda ..> reunioes
    TelaNovaReuniao ..> reunioes
    TelaComissao ..> api
    TelaReuniao ..> api
    TelaRegistrosReuniao ..> api
```
