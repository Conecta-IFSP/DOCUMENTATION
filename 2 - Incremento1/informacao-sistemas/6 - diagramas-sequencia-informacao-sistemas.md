# Diagramas de sequência — Informações do sistema

> Padrão do Incremento 1 · Módulo SIS

## Resumo

Mostra a ordem entre tela, guards, `SistemaController`, `SistemaService` e MongoDB.

Todos os fluxos passam por `ApiKeyGuard`, `JwtAuthGuard` e `AdminSistemaGuard` antes de chegar ao controller.

## SD-SIS-01 — Status

```mermaid
sequenceDiagram
    actor A as Administrador do sistema
    participant T as TelaSistema
    participant C as SistemaController
    participant S as SistemaService
    participant DB as MongoDB
    A->>T: abre a tela
    T->>C: GET /sistema/status
    C->>S: status()
    S->>S: lê CPU, memória e uptime pelo módulo os
    S->>DB: conta documentos para validar disponibilidade
    S-->>T: métricas do backend e estado do banco
```

## SD-SIS-02 — Relatórios

```mermaid
sequenceDiagram
    actor A as Administrador do sistema
    participant T as TelaSistema
    participant C as SistemaController
    participant S as SistemaService
    participant DB as MongoDB
    A->>T: consulta relatórios
    T->>C: GET /sistema/relatorios
    C->>S: relatorios()
    S->>DB: countDocuments com filtros de situação
    S-->>T: totais de usuários, organizações e comissões
```

## SD-SIS-03 — Popular dados de teste

```mermaid
sequenceDiagram
    actor A as Administrador do sistema
    participant T as TelaSistema
    participant C as SistemaController
    participant S as SistemaService
    participant DB as MongoDB
    A->>T: confirma criação dos dados
    T->>C: POST /sistema/popular-teste
    C->>S: popularDadosDeTeste()
    S->>DB: cria três usuários, organização e comissão
    S-->>T: dados criados
```

## SD-SIS-04 — Reset

```mermaid
sequenceDiagram
    actor A as Administrador do sistema
    participant T as TelaSistema
    participant C as SistemaController
    participant S as SistemaService
    participant DB as MongoDB
    A->>T: confirma a ação destrutiva
    T->>C: POST /sistema/reset
    C->>S: reset()
    S->>DB: remove reuniões, comissões e organizações
    S->>DB: remove usuários exceto ADMIN_SISTEMA
    S-->>T: status=sistema_resetado e data
```
