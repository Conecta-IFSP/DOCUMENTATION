# Testes — Cadastro de organização

> Padrão do Incremento 1 · Módulo ORG

## Resumo

Registra evidência executada e cenários manuais no formato comum do Incremento 1.

## Evidência automatizada

| Data       | Projeto/branch              | Comando         | Resultado                          | Arquivos relacionados                                                                            |
| ---------- | --------------------------- | --------------- | ---------------------------------- | ------------------------------------------------------------------------------------------------ |
| 23/10/2026 | BACKEND `homolog` `6f2ffc4` | `npm test`      | 13 arquivos e 182 testes aprovados | `organizacoes.controller.spec.ts`, `organizacoes.service.spec.ts`, `organizacoes-acesso.spec.ts` |
| 23/10/2026 | BACKEND `homolog`           | `npm run build` | Aprovado                           | TypeScript/NestJS                                                                                |

## Matriz de testes manuais

| ID        | Requisito       | Cenário/entrada                             | Resultado esperado                                                 | Situação                         |
| --------- | --------------- | ------------------------------------------- | ------------------------------------------------------------------ | -------------------------------- |
| CT-ORG-01 | RF-ORG-01       | Cadastrar nome, cidade e descrição          | Organização pendente; criador admin aprovado.                      | Coberto no backend; UI pendente. |
| CT-ORG-02 | RF-ORG-02       | Buscar por nome/cidade                      | Resultado filtrado dentro do escopo.                               | Pendente em aparelho.            |
| CT-ORG-03 | RF-ORG-03/04/05 | Aprovar, revogar e editar como admin global | Mudanças permitidas; usuário comum recebe 403.                     | Coberto no backend; UI pendente. |
| CT-ORG-04 | RF-ORG-06       | Excluir aprovada e depois revogada          | Primeira tentativa recusada; segunda remove organização/comissões. | Pendente integrado.              |
| CT-ORG-05 | RF-ORG-07       | Entrar e renovar código                     | Vínculo pendente; código antigo deixa de funcionar.                | Coberto no backend; UI pendente. |
| CT-ORG-06 | RF-ORG-08/09    | Convidar, responder, promover e rebaixar    | Estados válidos; último admin preservado.                          | Coberto no backend; UI pendente. |

## Pendências

Executar o roteiro no Expo Go/Android e registrar data, aparelho e resultado obtido.
