# Diagramas de sequência — Cadastro de organização

Os participantes nomeiam classes reais da branch `homolog`. O aplicativo chama o controller; o controller delega ao service; o service consulta o MongoDB via Mongoose.

## SD-ORG-01 — Cadastrar organização

```mermaid
sequenceDiagram
    actor U as Usuário
    participant T as TelaNovaOrganizacao
    participant C as OrganizacoesController
    participant S as OrganizacoesService
    participant DB as MongoDB
    U->>T: nome, cidade e descrição
    T->>C: POST /organizacoes
    C->>S: criar(dto, usuarioId)
    S->>DB: verifica nome sem diferenciar maiúsculas
    S->>DB: cria PENDENTE, código e criador ADMIN/APROVADO
    S-->>T: organização criada
```

## SD-ORG-02 — Consultar e pesquisar

```mermaid
sequenceDiagram
    actor U as Usuário ou Admin do sistema
    participant T as TelaOrganizacoes
    participant C as OrganizacoesController
    participant S as OrganizacoesService
    participant DB as MongoDB
    U->>T: informa termo opcional
    T->>C: GET /organizacoes?busca=termo
    C->>S: listar(usuario, busca)
    S->>DB: filtra nome/cidade e escopo permitido
    DB-->>T: organizações ordenadas por criação
```

## SD-ORG-03 — Alterar dados ou situação

```mermaid
sequenceDiagram
    actor A as Administrador do sistema
    participant T as TelaGestaoOrganizacoes
    participant C as OrganizacoesController
    participant S as OrganizacoesService
    participant DB as MongoDB
    A->>T: edita dados, aprova ou revoga
    T->>C: PATCH /organizacoes/:id
    C->>S: atualizar(id, dto, usuario)
    S->>DB: valida unicidade e grava campos permitidos
    DB-->>T: organização atualizada
```

## SD-ORG-04 — Excluir revogada

```mermaid
sequenceDiagram
    actor A as Administrador do sistema
    participant T as TelaGestaoOrganizacoes
    participant C as OrganizacoesController
    participant S as OrganizacoesService
    participant DB as MongoDB
    A->>T: confirma exclusão
    T->>C: DELETE /organizacoes/:id
    C->>S: remover(id)
    S->>DB: exclui somente se status=REVOGADA
    S->>DB: exclui comissões vinculadas
    S-->>T: confirmação
```

## SD-ORG-05 — Entrar e renovar código

```mermaid
sequenceDiagram
    actor U as Usuário
    actor A as Administrador da organização
    participant T as Aplicativo
    participant C as OrganizacoesController
    participant S as OrganizacoesService
    participant DB as MongoDB
    alt entrada
        U->>T: código de 10 caracteres
        T->>C: POST /organizacoes/entrar
        C->>S: entrarComCodigo(codigo, usuarioId)
        S->>DB: busca APROVADA e adiciona MEMBRO/PENDENTE
    else renovação
        A->>T: solicita novo código
        T->>C: POST /organizacoes/:id/codigo
        C->>S: renovarCodigo(id, adminId)
        S->>DB: substitui código após autorizar ADMIN/APROVADO
    end
```

## SD-ORG-06 — Convidar e responder

```mermaid
sequenceDiagram
    actor A as Administrador da organização
    actor U as Convidado
    participant T as Aplicativo
    participant C as OrganizacoesController
    participant S as OrganizacoesService
    participant DB as MongoDB
    A->>T: e-mail do usuário
    T->>C: POST /organizacoes/:id/convites
    C->>S: convidar(...)
    S->>DB: cria vínculo MEMBRO/CONVIDADO
    U->>T: aceita ou recusa
    T->>C: PATCH /organizacoes/:id/convite
    C->>S: responderConvite(...)
    S->>DB: aprova ou remove vínculo
```

## SD-ORG-07 — Analisar solicitação

```mermaid
sequenceDiagram
    actor A as Administrador da organização
    participant T as TelaMembros
    participant C as OrganizacoesController
    participant S as OrganizacoesService
    participant DB as MongoDB
    A->>T: aprova ou rejeita
    T->>C: PATCH /organizacoes/:id/membros/:usuarioId
    C->>S: alterarStatusMembro(...)
    S->>DB: atualiza com comparação do estado original
    DB-->>T: vínculo atualizado ou 409 por concorrência
```

## SD-ORG-08 — Alterar papel

```mermaid
sequenceDiagram
    actor A as Administrador da organização
    participant T as TelaMembros
    participant C as OrganizacoesController
    participant S as OrganizacoesService
    participant DB as MongoDB
    A->>T: promove ou rebaixa
    T->>C: PATCH /organizacoes/:id/membros/:usuarioId/papel
    C->>S: alterarPapelMembro(...)
    S->>DB: atualiza membro aprovado
    S->>S: impede rebaixar o último ADMIN
    S-->>T: organização atualizada
```
