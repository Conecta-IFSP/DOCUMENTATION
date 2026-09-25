# Diagramas de sequência

## SD-COM-01 — Listar e filtrar comissões

```mermaid
sequenceDiagram
    actor A as Administrador
    participant T as Tela
    participant API as API NestJS
    participant S as ComissoesService
    participant DB as MongoDB

    A->>T: Abre a tela Comissões
    T->>API: GET /comissoes
    API->>S: listar(usuarioId)
    S->>DB: Busca organizações administradas
    DB-->>S: Organizações
    S->>DB: Busca comissões
    DB-->>S: Comissões
    S-->>API: Lista permitida
    API-->>T: JSON com as comissões
    T->>T: Aplica busca e filtro
    T-->>A: Exibe a lista
```

## SD-COM-02 — Consultar comissão

```mermaid
sequenceDiagram
    actor A as Administrador
    participant T as Tela
    participant API as API NestJS
    participant S as ComissoesService
    participant DB as MongoDB

    A->>T: Seleciona uma comissão
    T->>API: GET /comissoes/:id
    API->>S: buscarPorId(id, usuarioId)
    S->>DB: Busca comissão
    DB-->>S: Comissão
    S->>S: Verifica permissão
    S-->>API: Dados da comissão
    API-->>T: JSON
    T->>API: GET /comissoes/organizacoes/:id/membros
    API->>S: listarMembrosDisponiveis()
    S->>DB: Busca usuários aprovados e ativos
    DB-->>S: Usuários
    S-->>API: Lista de usuários
    API-->>T: JSON
    T-->>A: Exibe dados e equipe
```

## SD-COM-03 — Cadastrar comissão

```mermaid
sequenceDiagram
    actor A as Administrador
    participant T as Tela Nova comissão
    participant API as API NestJS
    participant S as ComissoesService
    participant DB as MongoDB

    A->>T: Preenche nome, descrição e organização
    A->>T: Toca em Salvar
    T->>API: POST /comissoes + JSON
    API->>API: Valida JWT e DTO
    API->>S: criar(dto, usuarioId)
    S->>S: Verifica permissão na organização
    S->>DB: create(comissão)
    DB-->>S: Comissão criada
    S-->>API: Comissão
    API-->>T: JSON da comissão criada
    T-->>A: Abre a comissão cadastrada
```

## SD-COM-04 — Editar comissão

```mermaid
sequenceDiagram
    actor A as Administrador
    participant T as Tela
    participant API as API NestJS
    participant S as ComissoesService
    participant DB as MongoDB

    A->>T: Altera nome ou descrição
    A->>T: Toca em Salvar
    T->>API: PATCH /comissoes/:id + JSON
    API->>API: Valida JWT e DTO
    API->>S: atualizar(id, dto, usuarioId)
    S->>S: Verifica permissão
    S->>DB: findByIdAndUpdate + $set
    DB-->>S: Comissão atualizada
    S-->>API: Comissão
    API-->>T: JSON atualizado
    T-->>A: Exibe mensagem de sucesso
```

## SD-COM-05 — Desativar comissão

```mermaid
sequenceDiagram
    actor A as Administrador
    participant T as Tela
    participant API as API NestJS
    participant S as ComissoesService
    participant DB as MongoDB

    A->>T: Toca em Desativar
    T-->>A: Pede confirmação
    A->>T: Confirma
    T->>API: DELETE /comissoes/:id
    API->>S: remover(id, usuarioId)
    S->>S: Verifica permissão
    S->>DB: Atualiza ativo = false
    DB-->>S: Comissão inativa
    S-->>API: Resultado
    API-->>T: JSON
    T-->>A: Mostra comissão como inativa
```

## SD-COM-06 — Reativar comissão

```mermaid
sequenceDiagram
    actor A as Administrador
    participant T as Tela
    participant API as API NestJS
    participant S as ComissoesService
    participant DB as MongoDB

    A->>T: Toca em Reativar
    T->>API: PATCH /comissoes/:id
    Note over T,API: JSON: { "ativo": true }
    API->>S: atualizar(id, dto, usuarioId)
    S->>S: Verifica permissão
    S->>DB: Atualiza ativo = true
    DB-->>S: Comissão ativa
    S-->>API: Comissão
    API-->>T: JSON
    T-->>A: Mostra comissão ativa
```

## SD-COM-07 — Adicionar integrante

```mermaid
sequenceDiagram
    actor A as Administrador
    participant T as Tela
    participant API as API NestJS
    participant S as ComissoesService
    participant DB as MongoDB

    A->>T: Escolhe usuário e papel
    A->>T: Toca em Adicionar
    T->>API: POST /comissoes/:id/membros
    Note over T,API: JSON com usuario_id e papel
    API->>S: adicionarMembro()
    S->>S: Verifica comissão ativa e permissão
    S->>DB: Confere usuário ativo e aprovado
    DB-->>S: Usuário elegível
    S->>DB: $push no array membros
    DB-->>S: Comissão atualizada
    S-->>API: Comissão
    API-->>T: JSON
    T-->>A: Exibe novo integrante
```

## SD-COM-08 — Alterar papel do integrante

```mermaid
sequenceDiagram
    actor A as Administrador
    participant T as Tela
    participant API as API NestJS
    participant S as ComissoesService
    participant DB as MongoDB

    A->>T: Toca em Tornar responsável ou Tornar membro
    T->>API: PATCH /comissoes/:id/membros/:usuarioId
    Note over T,API: JSON com novo papel
    API->>S: atualizarMembro()
    S->>S: Verifica comissão, permissão e integrante
    S->>DB: $set em membros.$.papel
    DB-->>S: Comissão atualizada
    S-->>API: Comissão
    API-->>T: JSON
    T-->>A: Exibe o novo papel
```

## SD-COM-09 — Remover integrante

```mermaid
sequenceDiagram
    actor A as Administrador
    participant T as Tela
    participant API as API NestJS
    participant S as ComissoesService
    participant DB as MongoDB

    A->>T: Toca em Remover
    T-->>A: Pede confirmação
    A->>T: Confirma
    T->>API: DELETE /comissoes/:id/membros/:usuarioId
    API->>S: removerMembro()
    S->>S: Verifica comissão, permissão e integrante
    S->>DB: $pull no array membros
    DB-->>S: Comissão atualizada
    S-->>API: Comissão
    API-->>T: JSON
    T-->>A: Atualiza a equipe na tela
```

---

# Resumo das rotas usadas

| Método | Rota | Função |
|---|---|---|
| GET | `/comissoes` | Lista comissões. |
| GET | `/comissoes/:id` | Busca uma comissão. |
| GET | `/comissoes/organizacoes` | Lista organizações que o usuário administra. |
| GET | `/comissoes/organizacoes/:id/membros` | Lista usuários que podem entrar na comissão. |
| POST | `/comissoes` | Cadastra comissão. |
| PATCH | `/comissoes/:id` | Edita ou reativa comissão. |
| DELETE | `/comissoes/:id` | Desativa comissão. |
| POST | `/comissoes/:id/membros` | Adiciona integrante. |
| PATCH | `/comissoes/:id/membros/:usuarioId` | Altera papel. |
| DELETE | `/comissoes/:id/membros/:usuarioId` | Remove integrante. |

---

