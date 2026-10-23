# Testes — Informações do sistema

> Padrão do Incremento 1 · Módulo SIS

## Resumo

Registra evidência executada e cenários manuais no formato comum do Incremento 1.

## Evidência automatizada

| Data       | Projeto/commit    | Comando         | Resultado                          | Arquivos relacionados                            |
| ---------- | ----------------- | --------------- | ---------------------------------- | ------------------------------------------------ |
| 23/10/2026 | BACKEND `6f2ffc4` | `npm test`      | 13 arquivos e 182 testes aprovados | Suíte global; não há spec exclusivo de `sistema` |
| 23/10/2026 | BACKEND `6f2ffc4` | `npm run build` | Aprovado                           | TypeScript/NestJS                                |

## Matriz de testes manuais

| ID        | Requisito | Cenário/entrada                             | Resultado esperado                                     | Situação              |
| --------- | --------- | ------------------------------------------- | ------------------------------------------------------ | --------------------- |
| CT-SIS-01 | RF-SIS-01 | Status sem chave, como usuário e como admin | 401, 403 e 200, respectivamente.                       | Aprovado pela equipe. |
| CT-SIS-02 | RF-SIS-02 | Consultar relatórios como admin             | Totais coerentes com o banco.                          | Aprovado pela equipe. |
| CT-SIS-03 | RF-SIS-03 | Cancelar e confirmar população              | Cancelamento não chama API; confirmação cria cenário.  | Aprovado pela equipe. |
| CT-SIS-04 | RF-SIS-04 | Cancelar reset                              | Nenhum dado removido.                                  | Aprovado pela equipe. |
| CT-SIS-05 | RF-SIS-04 | Confirmar reset                             | Dados operacionais removidos; admin global preservado. | Aprovado pela equipe. |

## Conclusão da validação manual

A equipe informou a execução dos roteiros administrativos em ambiente integrado, com todos os cenários aprovados.
