# Diagrama de Casos de Uso

```mermaid
flowchart LR
    AdminSistema["⚙️ Admin do Sistema"]

    UC1["Consultar status do sistema"]
    UC2["Consultar relatórios"]
    UC3["Popular dados de teste"]
    UC4["Resetar sistema"]

    AdminSistema --> UC1
    AdminSistema --> UC2
    AdminSistema --> UC3
    AdminSistema --> UC4

    classDef actor stroke:#1976d2,stroke-width:2px;
    classDef uc stroke:#7b1fa2,stroke-width:2px;

    class AdminSistema actor;
    class UC1,UC2,UC3,UC4 uc;
```

**Observação:** não há caso de uso de "reinicialização de subsistemas" — a arquitetura (backend único, sem subsistemas isolados) não permite reiniciar um módulo de forma real. Decisão explicada no arquivo de requisitos funcionais.
