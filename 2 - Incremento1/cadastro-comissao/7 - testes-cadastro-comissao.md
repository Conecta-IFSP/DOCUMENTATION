# Testes — Cadastro de comissão

> Padrão do Incremento 1 · Módulo COM

## Resumo

Registra evidência executada e cenários manuais no formato comum do Incremento 1.

## Evidência automatizada

| Data       | Projeto/branch              | Comando         | Resultado                          | Arquivos relacionados                                                                   |
| ---------- | --------------------------- | --------------- | ---------------------------------- | --------------------------------------------------------------------------------------- |
| 23/10/2026 | BACKEND `homolog` `6f2ffc4` | `npm test`      | 13 arquivos e 182 testes aprovados | `comissoes.controller.spec.ts`, `comissoes.service.spec.ts`, `comissoes.regras.spec.ts` |
| 23/10/2026 | BACKEND `homolog`           | `npm run build` | Aprovado                           | TypeScript/NestJS                                                                       |

## Matriz de testes manuais

| ID        | Requisito       | Cenário/entrada                             | Resultado esperado                                      | Situação                         |
| --------- | --------------- | ------------------------------------------- | ------------------------------------------------------- | -------------------------------- |
| CT-COM-01 | RF-COM-01/02    | Listar e abrir comissão permitida           | Lista e detalhes exibidos; acesso alheio recusado.      | Pendente em aparelho.            |
| CT-COM-02 | RF-COM-03       | Criar com responsável aprovado              | Comissão ativa com exatamente um responsável.           | Pendente em aparelho.            |
| CT-COM-03 | RF-COM-04/05/06 | Editar, desativar e reativar                | Dados e situação atualizados; equipe preservada.        | Pendente em aparelho.            |
| CT-COM-04 | RF-COM-07       | Adicionar membro aprovado e repetir         | Primeira inclusão funciona; repetição retorna conflito. | Coberto no backend; UI pendente. |
| CT-COM-05 | RF-COM-08       | Substituir responsável                      | Novo promovido e anterior rebaixado na mesma resposta.  | Coberto no backend; UI pendente. |
| CT-COM-06 | RF-COM-09       | Tentar remover responsável e remover membro | Responsável recusado; membro removido após confirmação. | Coberto no backend; UI pendente. |

## Pendências

Executar o roteiro no Expo Go/Android e registrar data, aparelho e resultado obtido.
