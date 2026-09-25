# Requisitos Funcionais - Informações do Sistema

| Código | Caso de Uso | Ator |
|--------|-------------|------|
| RF-01 | Consultar status do sistema | Admin do Sistema |
| RF-02 | Consultar relatórios | Admin do Sistema |
| RF-03 | Popular dados de teste | Admin do Sistema |
| RF-04 | Resetar sistema | Admin do Sistema |

---

### RF-01: Consultar status do sistema

| Campo | Descrição |
|-------|-----------|
| **Descrição** | Mostra memória, CPU e tempo ativo do servidor, e se o banco está respondendo. |
| **Pré-condição** | Usuário autenticado como `ADMIN_SISTEMA` |
| **Fluxo Principal** | 1. Admin abre a tela Sistema → 2. Frontend chama `GET /sistema/status` → 3. Backend lê memória/CPU via módulo `os` do Node e conta documentos das 3 coleções → 4. Retorna os números |
| **Regra de Negócio** | O "sistema" monitorado é o servidor backend, não o celular do usuário (React Native não expõe RAM/CPU do aparelho sem módulo nativo) |

---

### RF-02: Consultar relatórios

| Campo | Descrição |
|-------|-----------|
| **Descrição** | Mostra contagens de usuários, organizações e comissões. |
| **Pré-condição** | Usuário autenticado como `ADMIN_SISTEMA` |
| **Fluxo Principal** | 1. Admin abre a tela Sistema → 2. Frontend chama `GET /sistema/relatorios` → 3. Backend conta documentos com `countDocuments()` (total/ativos/pendentes/aprovados) → 4. Retorna os números |

---

### RF-03: Popular dados de teste

| Campo | Descrição |
|-------|-----------|
| **Descrição** | Cria 3 usuários de teste cobrindo os cenários: sem vínculo, admin de organização, e membro de organização + comissão. |
| **Pré-condição** | Usuário autenticado como `ADMIN_SISTEMA` |
| **Fluxo Principal** | 1. Admin clica no botão e confirma → 2. Backend cria 3 usuários (senha `Teste@123`) + 1 organização aprovada + 1 comissão → 3. Retorna a lista criada |
| **Regra de Negócio** | Não mexe no administrador do sistema já existente |

---

### RF-04: Resetar sistema

| Campo | Descrição |
|-------|-----------|
| **Descrição** | Apaga usuários, organizações e comissões, voltando o sistema ao estado inicial. |
| **Pré-condição** | Usuário autenticado como `ADMIN_SISTEMA` |
| **Fluxo Principal** | 1. Admin clica no botão e confirma → 2. Backend apaga tudo de `organizacoes` e `comissoes`, e apaga `usuarios` exceto quem é `ADMIN_SISTEMA` → 3. Retorna sucesso |
| **Regra de Negócio** | Nunca apaga o administrador do sistema (bug corrigido durante o desenvolvimento: a primeira versão apagava todos os usuários, travando o app) |

---

## Sobre "reinicialização de subsistemas"

Esse item do PDF não foi implementado: no NestJS rodando como processo único não existem subsistemas isolados para reiniciar de verdade. Simular isso seria só cosmético. Ponto a validar com o professor.
