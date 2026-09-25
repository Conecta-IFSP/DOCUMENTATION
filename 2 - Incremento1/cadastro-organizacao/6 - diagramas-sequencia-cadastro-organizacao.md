# Diagramas de Sequência - Cadastro de Organização

## 1. Cadastrar organização (RF-01)

```mermaid
sequenceDiagram
    autonumber
    actor Usuario as Usuário Comum
    participant Front as Frontend
    participant API as Backend
    participant DB as MongoDB

    Usuario->>Front: Informa nome e descrição e confirma
    Front->>API: POST /organizacoes {nome, descricao}
    API->>DB: Consulta organização com o mesmo nome
    DB-->>API: Nome disponível
    API->>DB: Cria organização pendente com o criador como administrador
    DB-->>API: Organização criada
    API-->>Front: 201 - Cadastro realizado
    Front->>Usuario: Informa que aguarda autorização
```

## 2. Consultar organizações cadastradas (RF-02)

```mermaid
sequenceDiagram
    autonumber
    actor AdminSis as Administrador do Sistema
    participant Front as Frontend
    participant API as Backend
    participant DB as MongoDB

    AdminSis->>Front: Abre "Gerenciar organizações"
    Front->>API: GET /organizacoes
    API->>DB: Busca todas as organizações
    DB-->>API: Organizações com situação e responsável
    API-->>Front: 200 - Lista completa
    Front->>AdminSis: Exibe a lista com as pendentes no topo
```

## 3. Autorizar cadastro de organização (RF-03)

```mermaid
sequenceDiagram
    autonumber
    actor AdminSis as Administrador do Sistema
    participant Front as Frontend
    participant API as Backend
    participant DB as MongoDB

    AdminSis->>Front: Confirma a autorização da organização
    Front->>API: PATCH /organizacoes/{id} {status: "APROVADA"}
    API->>DB: Grava a situação aprovada
    DB-->>API: Organização atualizada
    API-->>Front: 200 - Organização aprovada
    Front->>AdminSis: Atualiza a lista com a nova situação
```

## 4. Revogar organização (RF-04)

```mermaid
sequenceDiagram
    autonumber
    actor AdminSis as Administrador do Sistema
    participant Front as Frontend
    participant API as Backend
    participant DB as MongoDB

    AdminSis->>Front: Confirma a revogação da organização
    Front->>API: PATCH /organizacoes/{id} {status: "REVOGADA"}
    API->>DB: Grava a situação revogada
    DB-->>API: Organização atualizada
    API-->>Front: 200 - Organização revogada
    Front->>AdminSis: Exibe a organização como revogada
```

## 5. Editar dados da organização (RF-05)

```mermaid
sequenceDiagram
    autonumber
    actor AdminSis as Administrador do Sistema
    participant Front as Frontend
    participant API as Backend
    participant DB as MongoDB

    AdminSis->>Front: Altera nome e descrição e confirma
    Front->>API: PATCH /organizacoes/{id} {nome, descricao}
    API->>DB: Consulta organização com o mesmo nome
    DB-->>API: Nome disponível
    API->>DB: Grava os novos dados
    DB-->>API: Organização atualizada
    API-->>Front: 200 - Dados atualizados
    Front->>AdminSis: Exibe a organização atualizada
```

## 6. Excluir organização revogada (RF-06)

```mermaid
sequenceDiagram
    autonumber
    actor AdminSis as Administrador do Sistema
    participant Front as Frontend
    participant API as Backend
    participant DB as MongoDB

    AdminSis->>Front: Confirma a exclusão da organização revogada
    Front->>API: DELETE /organizacoes/{id}
    API->>DB: Exclui a organização, somente se estiver revogada
    DB-->>API: Organização excluída
    API->>DB: Exclui as comissões da organização
    DB-->>API: Comissões excluídas
    API-->>Front: 200 - Exclusão concluída
    Front->>AdminSis: Remove a organização da lista
```
