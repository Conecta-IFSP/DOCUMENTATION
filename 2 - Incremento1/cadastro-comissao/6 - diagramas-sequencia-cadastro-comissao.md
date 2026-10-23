# Diagramas de sequência — Cadastro de comissão

> Padrão do Incremento 1 · Módulo COM

## Resumo

Cada diagrama usa participantes reais: tela Expo, `ComissoesController`, `ComissoesService` e models Mongoose.

## SD-COM-01 — Listar e filtrar comissões

```mermaid
sequenceDiagram
    actor U as Integrante
    participant T as TelaComissoes
    participant C as ComissoesController
    participant S as ComissoesService
    participant M as ComissaoModel
    U->>T: abre lista e informa filtros
    T->>C: GET /comissoes
    C->>S: listar(usuarioId)
    S->>M: find(organizacoes permitidas)
    M-->>T: comissões autorizadas
    T->>T: aplica busca e situação
```

## SD-COM-02 — Consultar comissão

```mermaid
sequenceDiagram
    actor U as Integrante
    participant T as TelaGerenciarComissao
    participant C as ComissoesController
    participant S as ComissoesService
    participant M as ComissaoModel
    U->>T: seleciona comissão
    T->>C: GET /comissoes/:id
    C->>S: buscarPorId(id, usuarioId)
    S->>M: findById(id)
    S->>S: valida autorização
    S-->>T: comissão e equipe
```

## SD-COM-03 — Cadastrar comissão

```mermaid
sequenceDiagram
    actor A as Administrador
    participant T as TelaNovaComissao
    participant C as ComissoesController
    participant S as ComissoesService
    participant O as OrganizacaoModel
    participant U as UsuarioModel
    participant M as ComissaoModel
    A->>T: informa dados e responsável
    T->>C: POST /comissoes
    C->>S: criar(dto, usuarioId)
    S->>O: valida organização e administração
    S->>U: valida responsável ativo/aprovado
    S->>M: create(comissão com um RESPONSAVEL)
    M-->>T: comissão criada
```

## SD-COM-04 — Editar comissão

```mermaid
sequenceDiagram
    actor A as Administrador
    participant T as TelaGerenciarComissao
    participant C as ComissoesController
    participant S as ComissoesService
    participant M as ComissaoModel
    A->>T: salva nome/descrição
    T->>C: PATCH /comissoes/:id
    C->>S: atualizar(id, dto, usuarioId)
    S->>S: valida administração
    S->>M: findByIdAndUpdate(campos permitidos)
    M-->>T: comissão atualizada
```

## SD-COM-05 — Desativar comissão

```mermaid
sequenceDiagram
    actor A as Administrador
    participant T as TelaGerenciarComissao
    participant C as ComissoesController
    participant S as ComissoesService
    participant M as ComissaoModel
    A->>T: confirma desativação
    T->>C: PATCH /comissoes/:id {ativo:false}
    C->>S: atualizar(id, dto, usuarioId)
    S->>M: grava ativo=false
    M-->>T: comissão inativa
```

## SD-COM-06 — Reativar comissão

```mermaid
sequenceDiagram
    actor A as Administrador
    participant T as TelaGerenciarComissao
    participant C as ComissoesController
    participant S as ComissoesService
    participant M as ComissaoModel
    A->>T: seleciona reativar
    T->>C: PATCH /comissoes/:id {ativo:true}
    C->>S: atualizar(id, dto, usuarioId)
    S->>M: grava ativo=true
    M-->>T: comissão ativa
```

## SD-COM-07 — Adicionar integrante

```mermaid
sequenceDiagram
    actor A as Administrador
    participant T as TelaGerenciarComissao
    participant C as ComissoesController
    participant S as ComissoesService
    participant U as UsuarioModel
    participant M as ComissaoModel
    A->>T: seleciona integrante
    T->>C: POST /comissoes/:id/membros
    C->>S: adicionarMembro(id, dto, adminId)
    S->>U: valida ativo/aprovado
    S->>M: findOneAndUpdate(equipe original, novo MEMBRO)
    M-->>T: equipe atualizada ou 409
```

## SD-COM-08 — Substituir responsável

```mermaid
sequenceDiagram
    actor A as Administrador
    participant T as TelaGerenciarComissao
    participant C as ComissoesController
    participant S as ComissoesService
    participant U as UsuarioModel
    participant M as ComissaoModel
    A->>T: define novo responsável
    T->>C: PATCH /comissoes/:id/responsavel
    C->>S: definirResponsavel(id, dto, adminId)
    S->>U: valida elegibilidade do alvo
    S->>M: escrita atômica promove alvo e rebaixa anterior
    M-->>T: equipe com exatamente um responsável
```

## SD-COM-09 — Remover integrante

```mermaid
sequenceDiagram
    actor A as Administrador
    participant T as TelaGerenciarComissao
    participant C as ComissoesController
    participant S as ComissoesService
    participant M as ComissaoModel
    A->>T: confirma remoção
    T->>C: DELETE /comissoes/:id/membros/:usuarioId
    C->>S: removerMembro(id, alvoId, adminId)
    S->>S: impede remover RESPONSAVEL
    S->>M: findOneAndUpdate(equipe original, $pull)
    M-->>T: equipe atualizada ou 409
```

## Rastreabilidade

SD-COM-01 a SD-COM-09 correspondem a RF-COM-01 a RF-COM-09.
