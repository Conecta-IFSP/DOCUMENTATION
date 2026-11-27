# Testes — Cadastro de organização

> Padrão do Incremento 1 · Módulo ORG

## Resumo

Registra evidência executada e cenários manuais no formato comum do Incremento 1.

## Evidência automatizada

| Data       | Projeto/commit    | Comando         | Resultado                          | Arquivos relacionados                                                                            |
| ---------- | ----------------- | --------------- | ---------------------------------- | ------------------------------------------------------------------------------------------------ |
| 23/10/2026 | BACKEND `6f2ffc4` | `npm test`      | 13 arquivos e 182 testes aprovados | `organizacoes.controller.spec.ts`, `organizacoes.service.spec.ts`, `organizacoes-acesso.spec.ts` |
| 23/10/2026 | BACKEND `6f2ffc4` | `npm run build` | Aprovado                           | TypeScript/NestJS                                                                                |

## Matriz de testes manuais

| ID        | Requisito       | Cenário/entrada                             | Resultado esperado                                                   | Situação              |
| --------- | --------------- | ------------------------------------------- | -------------------------------------------------------------------- | --------------------- |
| CT-ORG-01 | RF-ORG-01       | Cadastrar nome, cidade e descrição          | Organização criada aguardando aprovação; criador admin aprovado.     | Aprovado pela equipe. |
| CT-ORG-02 | RF-ORG-02       | Buscar por nome/cidade                      | Resultado filtrado dentro do escopo.                                 | Aprovado pela equipe. |
| CT-ORG-03 | RF-ORG-03/04/05 | Aprovar, revogar e editar como admin global | Mudanças permitidas; usuário comum recebe 403.                       | Aprovado pela equipe. |
| CT-ORG-04 | RF-ORG-06       | Excluir aprovada e depois revogada          | Primeira tentativa recusada; segunda remove organização/comissões.   | Aprovado pela equipe. |
| CT-ORG-05 | RF-ORG-07       | Entrar e renovar código                     | Vínculo criado aguardando análise; código antigo deixa de funcionar. | Aprovado pela equipe. |
| CT-ORG-06 | RF-ORG-08/09    | Convidar, responder, promover e rebaixar    | Estados válidos; último admin preservado.                            | Aprovado pela equipe. |

## Conclusão da validação manual

A equipe informou a execução dos roteiros de aceitação em ambiente móvel, com todos os cenários aprovados.
