# Dicionário de Dados - Visão Relacional

Sem tabelas novas. Este incremento só opera sobre as tabelas já existentes:

| Tabela | Operação usada aqui |
|--------|----------------------|
| `usuario` | `COUNT`, `INSERT` (teste), `DELETE` (reset, exceto ADMIN_SISTEMA) |
| `organizacao` | `COUNT`, `INSERT` (teste), `DELETE` (reset) |
| `comissao` | `COUNT`, `INSERT` (teste), `DELETE` (reset) |

**Regra de integridade:** o reset nunca remove usuários com `tipo = ADMIN_SISTEMA`, senão o sistema fica sem administrador.
