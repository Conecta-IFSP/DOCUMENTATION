# Diagramas de sequência — Reuniões

> Padrão do Incremento 2 · Módulo REU

## Resumo

Cada diagrama usa participantes e métodos confirmados no código: telas Expo, `ReunioesController`, `ReunioesService` e models Mongoose. `ApiKeyGuard`, `JwtAuthGuard` e `UsuarioComumGuard` protegem todas as rotas do módulo.

## SD-REU-01 — Agendar reunião

```mermaid
sequenceDiagram
    actor U as Responsável
    participant T as TelaNovaReuniao
    participant C as ReunioesController
    participant S as ReunioesService
    participant M as ReuniaoModel
    U->>T: informa reunião e convidados
    T->>C: POST /reunioes
    C->>S: criar(dto, usuarioId)
    S->>S: autorizarResponsavel() e validarHorario()
    S->>M: create(reuniao)
    M-->>T: reunião AGENDADA
```

## SD-REU-02 — Consultar agenda

```mermaid
sequenceDiagram
    actor U as Participante
    participant T as TelaAgenda
    participant C as ReunioesController
    participant S as ReunioesService
    participant M as ReuniaoModel
    U->>T: abre agenda/filtros
    T->>C: GET /reunioes?de&ate&comissao_id
    C->>S: listar(usuarioId, filtros)
    S->>M: find(participante, período, comissão)
    M-->>S: reuniões ordenadas
    S-->>T: reuniões populadas
```

## SD-REU-03 — Visualizar reunião

```mermaid
sequenceDiagram
    actor U as Participante
    participant T as TelaReuniao
    participant C as ReunioesController
    participant S as ReunioesService
    participant M as ReuniaoModel
    U->>T: seleciona reunião
    T->>C: GET /reunioes/:id
    C->>S: buscarPorId(id, usuarioId)
    S->>M: findById(id)
    S->>S: validar participação e popular()
    S-->>T: reunião + sou_dono
```

## SD-REU-04 — Cancelar reunião

```mermaid
sequenceDiagram
    actor U as Organizador
    participant T as TelaReuniao
    participant C as ReunioesController
    participant S as ReunioesService
    participant M as ReuniaoModel
    U->>T: confirma cancelamento
    T->>C: PATCH /reunioes/:id {CANCELADA}
    C->>S: atualizar(id, dto, usuarioId)
    S->>M: findOneAndUpdate(organizador, AGENDADA, fim futuro)
    alt estado atual
        M-->>T: reunião CANCELADA
    else estado mudou
        S-->>T: 409 conflito
    end
```

## SD-REU-05 — Responder convite

```mermaid
sequenceDiagram
    actor U as Participante
    participant T as TelaAgenda/TelaReuniao
    participant C as ReunioesController
    participant S as ReunioesService
    participant M as ReuniaoModel
    U->>T: aceita ou recusa
    T->>C: PATCH /reunioes/:id/convite
    C->>S: responderConvite(id, aceito, usuarioId)
    S->>M: findOneAndUpdate(participante e estado válidos)
    M-->>T: convite atualizado
```

## SD-REU-06 — Registrar entrada

```mermaid
sequenceDiagram
    actor U as Participante
    participant T as TelaReuniao
    participant C as ReunioesController
    participant S as ReunioesService
    participant M as ReuniaoModel
    U->>T: marca entrada
    T->>C: POST /reunioes/:id/entrada
    C->>S: marcarEntrada(id, usuarioId)
    S->>S: validar janela, convite e ausência
    S->>M: findOneAndUpdate(presenca inexistente)
    M-->>T: presença criada
```

## SD-REU-07 — Registrar saída

```mermaid
sequenceDiagram
    actor U as Participante
    participant T as TelaReuniao
    participant C as ReunioesController
    participant S as ReunioesService
    participant M as ReuniaoModel
    U->>T: marca saída
    T->>C: POST /reunioes/:id/saida
    C->>S: marcarSaida(id, usuarioId)
    S->>S: calcular min(agora, fim)
    S->>M: findOneAndUpdate(entrada existente, saída ausente)
    M-->>T: saída registrada
```

## SD-REU-08 — Confirmar ou recusar presença

```mermaid
sequenceDiagram
    actor U as Organizador
    participant T as TelaReuniao
    participant C as ReunioesController
    participant S as ReunioesService
    participant M as ReuniaoModel
    U->>T: escolhe participante e situação
    T->>C: PATCH /reunioes/:id/presencas/:usuarioId
    C->>S: atualizarPresenca(id, alvoId, situacao, usuarioId)
    S->>S: validar organizador e início
    S->>M: findOneAndUpdate(presenca)
    M-->>T: presença atualizada
```

## SD-REU-09 — Gerar lista de presença

```mermaid
sequenceDiagram
    actor U as Participante
    participant T as TelaRegistrosReuniao
    participant C as ReunioesController
    participant S as ReunioesService
    participant M as ReuniaoModel
    U->>T: abre aba Presença
    T->>C: GET /reunioes/:id/lista-presenca
    C->>S: gerarListaPresenca(id, usuarioId)
    S->>M: findById(id)
    S->>S: calcular situação e minutos
    S-->>T: lista sem alterar dados
```

## SD-REU-10 — Salvar rascunho da ata

```mermaid
sequenceDiagram
    actor U as Responsável
    participant T as TelaRegistrosReuniao
    participant C as ReunioesController
    participant S as ReunioesService
    participant M as ReuniaoModel
    U->>T: salva conteúdo
    T->>C: PATCH /reunioes/:id/ata {conteudo, versao}
    C->>S: salvarAta(id, dto, usuarioId)
    S->>S: validar responsável, encerramento e versão
    S->>M: findOneAndUpdate(versao atual)
    alt versão atual
        M-->>T: RASCUNHO com versão + 1
    else versão obsoleta
        S-->>T: 409 conflito
    end
```

## SD-REU-11 — Enviar ata para aprovação

```mermaid
sequenceDiagram
    actor U as Responsável
    participant T as TelaRegistrosReuniao
    participant C as ReunioesController
    participant S as ReunioesService
    participant M as ReuniaoModel
    U->>T: confirma publicação
    T->>C: POST /reunioes/:id/ata/publicar {versao}
    C->>S: publicarAta(id, versao, usuarioId)
    S->>S: selecionar presenças CONFIRMADA
    S->>M: findOneAndUpdate(RASCUNHO e versão atual)
    M-->>T: ata EM_APROVACAO e aprovadores
```

## SD-REU-12 — Aprovar ata

```mermaid
sequenceDiagram
    actor U as Aprovador
    participant T as TelaRegistrosReuniao
    participant C as ReunioesController
    participant S as ReunioesService
    participant M as ReuniaoModel
    U->>T: confirma aprovação
    T->>C: POST /reunioes/:id/ata/aprovar {versao}
    C->>S: aprovarAta(id, versao, usuarioId)
    S->>S: validar aprovador e evitar duplicidade
    S->>M: findOneAndUpdate(versão atual)
    alt todos aprovaram
        M-->>T: ata APROVADA
    else ainda faltam aprovações
        M-->>T: ata EM_APROVACAO
    end
```

## Rastreabilidade

SD-REU-01 a SD-REU-12 correspondem, na mesma ordem, a UC-REU-01 a UC-REU-12 e RF-REU-01 a RF-REU-12.
