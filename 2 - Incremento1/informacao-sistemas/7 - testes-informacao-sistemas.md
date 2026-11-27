# Testes — Informações do sistema

> Padrão da disciplina · Módulo SIS · Revisado em 26/11/2026

## Resumo

Registra o que foi executado, quando, em qual ambiente e qual foi o resultado. O serviço deste módulo não tem teste automatizado próprio: a parte automatizada cobre os guards que protegem as rotas, testados nas rotas administrativas de usuários. As telas foram verificadas pelos roteiros manuais da equipe. Nada aqui está marcado como aprovado sem ter sido executado; o que falta executar está marcado como não executado.

## Ambiente

| Item                | Valor                                                                                                                 |
| ------------------- | --------------------------------------------------------------------------------------------------------------------- |
| Data da execução    | Roteiros manuais CT-SIS-01 a CT-SIS-05: relatados pela equipe e registrados em 23/10/2026, sem data própria. Testes automatizados: 23/10/2026 e 26/11/2026. |
| Node.js / npm       | 24.19.0 / 11.17.0                                                                                                     |
| Backend             | branch `homolog`: `6f2ffc4` em 23/10/2026; `99b6b4f` com as alterações locais em 26/11/2026                           |
| Aplicativo          | branch `homolog`: `792fab4` em 23/10/2026; `a53acce` com as alterações locais em 26/11/2026                           |
| Ambiente manual     | "Ambiente integrado" (aplicativo, API e MongoDB), conforme o relato da equipe. O registro não informa o aparelho.      |
| iOS                 | Não executado: a equipe não dispõe de computador Apple nem de aparelho iOS                                            |

## Evidência automatizada

| Data       | Projeto  | Comando                                                         | Resultado                                                                                   |
| ---------- | -------- | --------------------------------------------------------------- | ------------------------------------------------------------------------------------------- |
| 23/10/2026 | BACKEND  | `npm test` (commit `6f2ffc4`)                                   | 13 arquivos e 182 testes aprovados. Suíte global; não há spec exclusivo de `sistema`.        |
| 23/10/2026 | BACKEND  | `npm run build` (commit `6f2ffc4`)                              | Aprovado (TypeScript e NestJS).                                                             |
| 26/11/2026 | BACKEND  | `npx vitest run`                                                | 16 arquivos e 227 testes aprovados, nenhuma falha. Nenhum spec exclusivo de `sistema`.      |
| 26/11/2026 | BACKEND  | `npx vitest run src/usuarios/usuarios-acesso.spec.ts`           | 1 arquivo e 5 testes aprovados (inclui CT-SIS-06 e CT-SIS-07).                              |
| 26/11/2026 | BACKEND  | `npm run build` e `npm run lint`                                | Sem erro.                                                                                   |
| 26/11/2026 | FRONTEND | `npx tsc --noEmit` e `npm run lint`                             | Sem erro.                                                                                   |
| 26/11/2026 | FRONTEND | `node --test tests/*.test.cjs`                                  | 5 arquivos e 47 testes aprovados. Nenhum deles cobre a tela "Sistema".                     |

Os scripts `BACKEND/test/*-mongo.mjs` não cobrem este módulo.

## Matriz de testes automatizados

| ID        | Requisito                         | Cenário e entrada                                                                                                                                                        | Resultado esperado                                                                                                                | Teste                                                                                                   |
| --------- | --------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------- |
| CT-SIS-06 | RF-SIS-01 a RF-SIS-04, RNF-SIS-01 | Com `JwtAuthGuard` e `AdminSistemaGuard`, o mesmo par usado no `SistemaController`: chamar uma rota administrativa com o token de uma conta `ADMIN_SISTEMA`; rebaixar a conta para `USUARIO` no banco e repetir com o mesmo token. | 200 na primeira chamada; depois, 403 em todas as rotas administrativas, sem chamar o serviço. O tipo vale pelo banco, não pelo token. | `BACKEND/src/usuarios/usuarios-acesso.spec.ts`, "revoga todas as operações administrativas sem trocar o token" |
| CT-SIS-07 | RF-SIS-04, RNF-SIS-01             | Com token válido, desativar a conta ou apagá-la do banco e repetir as chamadas.                                                                                          | 401 em todas as rotas, sem chamar o serviço. É o que acontece com as contas apagadas pelo reset.                                  | `BACKEND/src/usuarios/usuarios-acesso.spec.ts`, "bloqueia conta desativada com token válido" e "bloqueia conta excluída com token válido" |

Os dois testes verificam os guards nas rotas `/usuarios`; nenhum teste automatizado chama as rotas `/sistema` nem o `SistemaService`.

## Matriz de testes manuais

| ID        | Requisito             | Aparelho                    | Cenário                                          | Resultado esperado                                       | Resultado obtido                       |
| --------- | --------------------- | --------------------------- | ------------------------------------------------ | -------------------------------------------------------- | -------------------------------------- |
| CT-SIS-01 | RF-SIS-01, RNF-SIS-01 | Ambiente integrado          | Consultar o status sem chave, como usuário comum e como administrador do sistema. | 401, 403 e 200, respectivamente.          | Aprovado pela equipe.                  |
| CT-SIS-02 | RF-SIS-02             | Ambiente integrado          | Consultar os relatórios como administrador do sistema. | Totais coerentes com o banco.                        | Aprovado pela equipe.                  |
| CT-SIS-03 | RF-SIS-03, RNF-SIS-02 | Ambiente integrado          | Cancelar e depois confirmar "Popular dados de teste". | Cancelamento não chama a API; confirmação cria o cenário. | Aprovado pela equipe.              |
| CT-SIS-04 | RF-SIS-04, RNF-SIS-02 | Ambiente integrado          | Cancelar o reset.                                | Nenhum dado removido.                                    | Aprovado pela equipe.                  |
| CT-SIS-05 | RF-SIS-04             | Ambiente integrado          | Confirmar o reset.                               | Dados de uso removidos; administrador do sistema preservado. | Aprovado pela equipe.              |
| CT-SIS-08 | RNF-SIS-07            | iOS                         | Repetir CT-SIS-01 a CT-SIS-05 em aparelho iOS.   | Mesmos resultados do Android.                            | Não executado: sem equipamento Apple disponível. |
| CT-SIS-09 | RNF-SIS-05            | API e MongoDB no mesmo computador | Medir 10 vezes o tempo de `GET /sistema/status` e de `GET /sistema/relatorios`, com até 1.000 documentos por coleção. | Maior tempo de cada rota até 1.000 ms. | Não executado. |

A equipe informou a execução dos roteiros administrativos CT-SIS-01 a CT-SIS-05 em ambiente integrado, com todos os cenários aprovados. O registro não guarda data, aparelho nem quais coleções foram conferidas depois do reset.

## Comportamentos identificados por inspeção, sem roteiro executado

Estes pontos foram conferidos lendo o código em 26/11/2026 e estão descritos nos requisitos; nenhum deles tem execução registrada:

- a faixa de sucesso dos dados de teste termina em "Senha padrão: undefined", porque a API não devolve `senha_padrao` (RF-SIS-03);
- as faixas de sucesso de popular e de resetar somem quando a recarga de status e relatórios termina bem (RF-SIS-03 e RF-SIS-04);
- com o MongoDB fora do ar, a tela mostra "Internal server error" na faixa do status, e não "Indisponível" no cartão (RF-SIS-01);
- o reset não apaga `notificacoes` e não usa transação (RF-SIS-04, RNF-SIS-04);
- o reset usa um diálogo simples, sem confirmação reforçada (RNF-SIS-03);
- o reinício de subsistemas citado na tela inicial não existe.

## Rastreabilidade

CT-SIS-01 e CT-SIS-02 cobrem as consultas (RF-SIS-01 e RF-SIS-02); CT-SIS-03 cobre os dados de teste (RF-SIS-03); CT-SIS-04 e CT-SIS-05 cobrem o reset (RF-SIS-04); CT-SIS-06 e CT-SIS-07 cobrem os guards (RNF-SIS-01); CT-SIS-08 e CT-SIS-09 estão pendentes.
