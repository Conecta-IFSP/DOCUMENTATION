# Testes — Informações do sistema

> Padrão da disciplina · Módulo SIS · Revisado em 26/11/2026

## Resumo

Registra o que foi executado, quando, em qual ambiente e qual foi o resultado. A parte automatizada cobre os guards que protegem as rotas, testados nas rotas administrativas de usuários, e o reset: a validação da palavra de confirmação e as coleções apagadas pelo serviço. As telas foram verificadas pelos roteiros manuais da equipe. Nada aqui está marcado como aprovado sem ter sido executado; o que falta executar está marcado como não executado ou a executar.

## Ambiente

| Item                | Valor                                                                                                                 |
| ------------------- | --------------------------------------------------------------------------------------------------------------------- |
| Data da execução    | Roteiros manuais CT-SIS-01 a CT-SIS-05: relatados pela equipe e registrados em 23/10/2026, sem data própria. Testes automatizados: 23/10/2026 e 26/11/2026. CT-SIS-12: 26/11/2026. |
| Node.js / npm       | 24.19.0 / 11.17.0                                                                                                     |
| Backend             | branch `homolog`: `6f2ffc4` em 23/10/2026; `99b6b4f` com as alterações locais em 26/11/2026; `caca2d2` com as alterações locais do reset, também em 26/11/2026 |
| Aplicativo          | branch `homolog`: `792fab4` em 23/10/2026; `a53acce` com as alterações locais em 26/11/2026; `6ff2d50` com as alterações locais do reset, também em 26/11/2026 |
| Ambiente manual     | "Ambiente integrado" (aplicativo, API e MongoDB), conforme o relato da equipe. O registro não informa o aparelho.      |
| iOS                 | Não executado: a equipe não dispõe de computador Apple nem de aparelho iOS                                            |
| API do CT-SIS-12    | API local, ligada a um banco MongoDB separado, chamada com `curl`                                                     |

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
| 26/11/2026 | BACKEND  | `npx vitest run`, depois das alterações do reset                | 17 arquivos e 234 testes aprovados, nenhuma falha.                                          |
| 26/11/2026 | BACKEND  | `npx vitest run src/sistema/sistema.spec.ts`                    | 1 arquivo e 7 testes aprovados (CT-SIS-10 e CT-SIS-11).                                     |
| 26/11/2026 | BACKEND  | `npm run build` e `npm run lint`, depois das alterações do reset | Sem erro.                                                                                  |
| 26/11/2026 | FRONTEND | `npx tsc --noEmit`, `npm run lint` e `node --test tests/*.test.cjs`, depois das alterações do reset | Sem erro; 5 arquivos e 47 testes aprovados. Nenhum deles cobre a tela "Sistema". |

Os scripts `BACKEND/test/*-mongo.mjs` não cobrem este módulo.

## Matriz de testes automatizados

| ID        | Requisito                         | Cenário e entrada                                                                                                                                                        | Resultado esperado                                                                                                                | Teste                                                                                                   |
| --------- | --------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------- |
| CT-SIS-06 | RF-SIS-01 a RF-SIS-04, RNF-SIS-01 | Com `JwtAuthGuard` e `AdminSistemaGuard`, o mesmo par usado no `SistemaController`: chamar uma rota administrativa com o token de uma conta `ADMIN_SISTEMA`; rebaixar a conta para `USUARIO` no banco e repetir com o mesmo token. | 200 na primeira chamada; depois, 403 em todas as rotas administrativas, sem chamar o serviço. O tipo vale pelo banco, não pelo token. | `BACKEND/src/usuarios/usuarios-acesso.spec.ts`, "revoga todas as operações administrativas sem trocar o token" |
| CT-SIS-07 | RF-SIS-04, RNF-SIS-01             | Com token válido, desativar a conta ou apagá-la do banco e repetir as chamadas.                                                                                          | 401 em todas as rotas, sem chamar o serviço. É o que acontece com as contas apagadas pelo reset.                                  | `BACKEND/src/usuarios/usuarios-acesso.spec.ts`, "bloqueia conta desativada com token válido" e "bloqueia conta excluída com token válido" |
| CT-SIS-10 | RF-SIS-04, RNF-SIS-03             | Validar o `ResetSistemaDto` com `confirmacao` ausente, `""`, `"resetar"`, `"RESET"` e `" RESETAR"`, e depois com `"RESETAR"`. | Os cinco primeiros geram um erro cada, com a mensagem "Digite RESETAR para confirmar o reset"; `"RESETAR"` não gera erro. | `BACKEND/src/sistema/sistema.spec.ts`, "recusa a confirmação %s" (5 casos) e "aceita RESETAR" |
| CT-SIS-11 | RF-SIS-04                         | Chamar `SistemaService.reset` com os quatro models e a conexão simulados.                                                        | Devolve `status` "sistema_resetado"; chama `deleteMany` no model de usuários com `{ tipo: { $ne: "ADMIN_SISTEMA" } }`; pela conexão, usa exatamente as coleções `anexos.chunks`, `anexos.files`, `contadores` e `notificacoes`, e chama `deleteMany({})` em `notificacoes`. | `BACKEND/src/sistema/sistema.spec.ts`, "apaga também as notificações e mantém os administradores do sistema" |

CT-SIS-06 e CT-SIS-07 verificam os guards nas rotas `/usuarios`. CT-SIS-10 e CT-SIS-11 testam o DTO e o serviço do reset sem subir o NestJS: nenhum teste automatizado chama as rotas `/sistema` nem passa pelo `ValidationPipe`; essa parte foi conferida à mão no CT-SIS-12.

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
| CT-SIS-12 | RF-SIS-04, RNF-SIS-03 | API do CT-SIS-12            | Com o token de um administrador do sistema, chamar `POST /sistema/reset` sem corpo, com `confirmacao` "resetar" e com "RESETAR"; depois, conferir as contas e as notificações. | 400 "Digite RESETAR para confirmar o reset" nos dois primeiros; 201 no terceiro, ficando só as contas `ADMIN_SISTEMA` e nenhuma notificação. | Aprovado em 26/11/2026. Sem corpo e com "resetar", 400 com a mensagem; com "RESETAR", 201 com `sistema_resetado`; ficou só o usuário `ADMIN_SISTEMA` e 0 notificações. |
| CT-SIS-13 | RF-SIS-04, RNF-SIS-02, RNF-SIS-03 | Celular Android | Tocar em "Resetar sistema"; digitar "RESETA" e depois "RESETAR"; fechar a folha tocando fora dela; abrir de novo, digitar "RESETAR", tocar no botão da folha e cancelar o pedido de identidade; repetir confirmando com a digital ou o PIN. | O botão da folha só fica ativo com "RESETAR"; fechar não envia nada; com o pedido cancelado, a faixa "Não foi possível confirmar sua identidade." e nada apagado; confirmado, a faixa "Sistema resetado" com "Usuários, organizações, comissões, reuniões e notificações apagados.", que continua depois da recarga, e o relatório só com os administradores do sistema. | Parcialmente executado em 26/11/2026, Galaxy S24 (Android 16), APK com a API local num banco de teste separado: o botão da folha só ficou ativo com "RESETAR"; ao tocar, o Android abriu o pedido de biometria (`BiometricPromptRoot`); cancelado, a tela rolou até a faixa "Não foi possível confirmar sua identidade." e nada foi apagado (a notificação de teste continuou no banco). A confirmação com a digital não foi executada; o caminho de sucesso foi conferido por curl no CT-SIS-12. |
| CT-SIS-14 | RF-SIS-03             | Celular Android             | Num banco sem as contas de teste, tocar em "Popular dados de teste" e confirmar. | A faixa "Dados de teste criados" com "3 usuários de teste criados.", sem "Senha padrão", que continua depois da recarga; os cartões com os novos totais. | Aprovado em 26/11/2026, Galaxy S24, APK com a API local num banco de teste separado. Sem as variáveis `TEST_USER_*`, a tela rolou até a faixa de erro com a lista das variáveis. Com as variáveis (valores fictícios), apareceu "Dados de teste criados" com "3 usuários de teste criados.", a faixa continuou depois da recarga e os cartões passaram a 4 de 4 usuários, 1 de 1 organização e 1 de 1 comissão. |

A equipe informou a execução dos roteiros administrativos CT-SIS-01 a CT-SIS-05 em ambiente integrado, com todos os cenários aprovados. O registro não guarda data, aparelho nem quais coleções foram conferidas depois do reset. CT-SIS-04 e CT-SIS-05 foram executados com o diálogo simples que o reset usava antes; o fluxo atual, com a palavra e a confirmação de identidade, é o do CT-SIS-13.

## Comportamentos identificados por inspeção, sem roteiro executado

Estes pontos foram conferidos lendo o código em 26/11/2026 e estão descritos nos requisitos; nenhum deles tem execução registrada:

- as faixas de sucesso de popular e de resetar são definidas depois da recarga de status e relatórios; se a recarga falhar, a faixa de sucesso toma o lugar da faixa de erro e os cartões ficam com os números antigos (RF-SIS-03 e RF-SIS-04);
- com o MongoDB fora do ar, a tela mostra "Internal server error" na faixa do status, e não "Indisponível" no cartão (RF-SIS-01);
- o reset não usa transação (RF-SIS-04, RNF-SIS-04);
- na web e em aparelho sem bloqueio de tela, o reset não pede identidade, e só a palavra "RESETAR" o protege; se a consulta do nível de segurança do aparelho falhar, `autenticarComDispositivo` também segue sem pedir (RNF-SIS-03);
- o reinício de subsistemas citado na tela inicial não existe.

## Rastreabilidade

CT-SIS-01 e CT-SIS-02 cobrem as consultas (RF-SIS-01 e RF-SIS-02); CT-SIS-03 e CT-SIS-14 cobrem os dados de teste (RF-SIS-03); CT-SIS-04, CT-SIS-05 e CT-SIS-10 a CT-SIS-13 cobrem o reset (RF-SIS-04 e RNF-SIS-03); CT-SIS-06 e CT-SIS-07 cobrem os guards (RNF-SIS-01); CT-SIS-08 e CT-SIS-09 estão pendentes, e CT-SIS-13 foi executado em parte (falta confirmar com a digital).
