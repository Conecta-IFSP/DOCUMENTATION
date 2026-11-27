# Requisitos não funcionais — Informações do sistema

> Padrão da disciplina · Módulo SIS · Revisado em 26/11/2026

## Resumo

Os requisitos não funcionais aparecem em dois níveis, como os funcionais. Os **de usuário** dizem, em uma frase, a qualidade esperada. Os **de sistema** completam cada um com a medida que o aprova, a condição em que vale, como foi verificado e onde está a evidência, de modo que qualquer pessoa consiga conferir. Onde ainda não houve medição, ou onde o requisito não está implementado, isso está escrito.

A classificação segue a árvore de Sommerville usada na disciplina: requisitos de **produto** (facilidade de uso, eficiência, confiabilidade, portabilidade), **organizacionais** (entrega, implementação, padrões) e **externos** (interoperabilidade, éticos, legais).

## Requisitos não funcionais de usuário

**RNF-SIS-01 — Acesso só do administrador do sistema.** Só o administrador do sistema pode ver as informações do sistema, criar dados de teste e resetar o sistema.

**RNF-SIS-02 — Confirmação antes de alterar dados.** Criar dados de teste e resetar o sistema devem pedir confirmação antes de mudar qualquer dado.

**RNF-SIS-03 — Confirmação reforçada do reset.** O reset, que apaga quase todos os dados, deve exigir uma confirmação mais forte que um simples toque em "Confirmar".

**RNF-SIS-04 — Reset sem resultado pela metade.** Se o reset falhar, nenhum dado deve ficar apagado só em parte.

**RNF-SIS-05 — Tempo de resposta das consultas.** O status e os relatórios devem chegar ao administrador em até 1 segundo.

**RNF-SIS-06 — Métricas sem dependência externa.** As métricas do servidor devem ser obtidas sem instalar bibliotecas ou serviços de monitoramento.

**RNF-SIS-07 — Portabilidade.** A tela "Sistema" deve funcionar em celulares Android a partir da versão 7.0, em iOS e no navegador.

**RNF-SIS-08 — Legibilidade nos dois temas.** A tela "Sistema" deve ser legível no tema claro e no tema escuro.

**RNF-SIS-09 — Testabilidade.** Os testes automatizados devem passar antes de cada entrega.

## Requisitos não funcionais de sistema

### RNF-SIS-01 — Acesso só do administrador do sistema

| Campo             | Conteúdo                                                                                                                                                                                                 |
| ----------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Classificação** | Externo › Legal › Segurança                                                                                                                                                                              |
| **Critério**      | As quatro rotas `/sistema` só executam para a conta ativa com `tipo` igual a `ADMIN_SISTEMA` no banco, com a chave do aplicativo correta. O tipo é lido do banco a cada requisição, não do token.         |
| **Medida**        | Sem `X-API-Key` válida: 401 "Chave de acesso da API inválida ou ausente". Sem token, com token inválido ou de conta inativa ou apagada: 401. Conta com `tipo` `USUARIO`: 403 "Apenas administradores do sistema podem realizar esta operação". Nos três casos, o `SistemaService` não é chamado. No aplicativo, a rota `sistema` não existe para a sessão de usuário comum. |
| **Condição**      | Rotas `GET /sistema/status`, `GET /sistema/relatorios`, `POST /sistema/popular-teste` e `POST /sistema/reset`.                                                                                            |
| **Verificação**   | Roteiro manual CT-SIS-01 (401, 403 e 200 na rota de status). Teste automatizado CT-SIS-06, que verifica o mesmo par `JwtAuthGuard` e `AdminSistemaGuard` nas rotas administrativas de usuários: com a conta rebaixada para `USUARIO`, o mesmo token recebe 403 e o serviço não é chamado. |
| **Evidência**     | `BACKEND/src/auth/api-key.guard.ts`, `BACKEND/src/auth/jwt-auth.guard.ts`, `BACKEND/src/auth/admin-sistema.guard.ts`, `@UseGuards` em `BACKEND/src/sistema/sistema.controller.ts`, `Stack.Protected` em `FRONTEND/src/app/_layout.tsx` e `BACKEND/src/usuarios/usuarios-acesso.spec.ts`. |
| **Situação**      | Atendido. CT-SIS-01 aprovado pela equipe; CT-SIS-06 aprovado em 26/11/2026. Não há teste automatizado que chame as rotas `/sistema` diretamente.                                                          |

### RNF-SIS-02 — Confirmação antes de alterar dados

| Campo             | Conteúdo                                                                                                                                                       |
| ----------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Classificação** | Produto › Facilidade de uso                                                                                                                                    |
| **Critério**      | Os botões "Popular dados de teste" e "Resetar sistema" só abrem um diálogo; a requisição só sai depois do toque em "Confirmar".                                |
| **Medida**        | Tocar em "Cancelar", ou no botão voltar do Android, não envia nenhuma requisição e não muda nenhum dado.                                                       |
| **Condição**      | Tela "Sistema", diálogos "Criar 3 usuários de teste?" e "Apagar todos os usuários, organizações e comissões? O administrador do sistema será mantido.".       |
| **Verificação**   | Roteiros manuais CT-SIS-03 e CT-SIS-04.                                                                                                                         |
| **Evidência**     | `FRONTEND/src/app/sistema.tsx` (estados `confirmacaoTeste` e `confirmacaoReset`) e `FRONTEND/src/components/Confirmacao.tsx`.                                  |
| **Situação**      | Atendido. CT-SIS-03 e CT-SIS-04 aprovados pela equipe.                                                                                                          |

### RNF-SIS-03 — Confirmação reforçada do reset

| Campo             | Conteúdo                                                                                                                                                                       |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Classificação** | Externo › Legal › Segurança                                                                                                                                                    |
| **Critério**      | O reset exige uma segunda prova de intenção além do toque em "Confirmar", por exemplo digitar uma palavra combinada ou confirmar a identidade com biometria ou PIN do aparelho. |
| **Medida**        | Sem essa segunda prova, `POST /sistema/reset` não é enviado.                                                                                                                    |
| **Condição**      | Botão "Resetar sistema" da tela "Sistema".                                                                                                                                      |
| **Verificação**   | Inspeção de `FRONTEND/src/app/sistema.tsx`.                                                                                                                                     |
| **Evidência**     | O diálogo do reset usa o mesmo componente `Confirmacao` dos dados de teste, sem campo de texto, sem `autenticarComDispositivo` e sem a opção `perigo`.                          |
| **Situação**      | **Não implementado.** Pedido na avaliação do Incremento 1 ("Resetar sistema precisa de uma confirmação mais segura").                                                          |

### RNF-SIS-04 — Reset sem resultado pela metade

| Campo             | Conteúdo                                                                                                                                                                                |
| ----------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Classificação** | Produto › Confiabilidade                                                                                                                                                                |
| **Critério**      | O reset apaga todas as coleções previstas ou nenhuma.                                                                                                                                   |
| **Medida**        | Se uma das exclusões falhar, as outras são desfeitas e a resposta é de erro.                                                                                                            |
| **Condição**      | `POST /sistema/reset` com falha do MongoDB durante as exclusões.                                                                                                                        |
| **Verificação**   | Inspeção de `SistemaService.reset`.                                                                                                                                                     |
| **Evidência**     | `BACKEND/src/sistema/sistema.service.ts`: as sete exclusões rodam ao mesmo tempo com `Promise.all`, sem sessão nem transação do MongoDB.                                               |
| **Situação**      | **Não atendido.** Uma falha no meio pode deixar parte das coleções apagada e parte não; a API responde 500 e o administrador precisa conferir o banco.                                  |

### RNF-SIS-05 — Tempo de resposta das consultas

| Campo             | Conteúdo                                                                                                                                                                         |
| ----------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Classificação** | Produto › Eficiência › Desempenho                                                                                                                                                |
| **Critério**      | As rotas de status e de relatórios respondem rápido o suficiente para a tela abrir sem espera perceptível.                                                                        |
| **Medida**        | Cada rota responde em até 1.000 ms, do envio da requisição à chegada da resposta completa, em 10 chamadas seguidas; vale o maior dos 10 tempos.                                   |
| **Condição**      | API e MongoDB no mesmo computador, sem outras requisições ao mesmo tempo, com até 1.000 documentos em cada uma das coleções `usuarios`, `organizacoes` e `comissoes`.            |
| **Verificação**   | CT-SIS-09: medir com uma ferramenta de requisições HTTP (por exemplo, Postman ou `curl` com `-w "%{time_total}"`).                                                               |
| **Evidência**     | `SistemaService.status` (três contagens em paralelo) e `SistemaService.relatorios` (sete contagens em paralelo), em `BACKEND/src/sistema/sistema.service.ts`.                     |
| **Situação**      | Medição pendente: CT-SIS-09 não foi executado.                                                                                                                                   |

### RNF-SIS-06 — Métricas sem dependência externa

| Campo             | Conteúdo                                                                                                                                                |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Classificação** | Organizacional › Implementação                                                                                                                          |
| **Critério**      | Memória, CPU e tempo ligado vêm do módulo `node:os`, que faz parte do Node.js; o módulo não cria coleção própria.                                        |
| **Medida**        | `sistema.service.ts` importa só `node:os`, `bcrypt`, `@nestjs/*`, `mongoose` e os schemas do projeto; o `sistema.module.ts` registra só schemas de outros módulos. |
| **Condição**      | Código da branch `homolog`.                                                                                                                             |
| **Verificação**   | Inspeção das importações de `BACKEND/src/sistema/sistema.service.ts` e `BACKEND/src/sistema/sistema.module.ts`.                                          |
| **Evidência**     | Os dois arquivos citados.                                                                                                                                |
| **Situação**      | Atendido por inspeção em 26/11/2026.                                                                                                                     |

### RNF-SIS-07 — Portabilidade

| Campo             | Conteúdo                                                                                                                                       |
| ----------------- | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| **Classificação** | Produto › Portabilidade                                                                                                                        |
| **Critério**      | A tela "Sistema" funciona do Android 7.0 (API 24, o mínimo do aplicativo) ao Android 16, no iOS e no navegador.                                 |
| **Medida**        | Consultar, popular e resetar sem erro e sem texto cortado nos cartões e nos diálogos.                                                          |
| **Condição**      | Emuladores Android 7.0 e 16, Galaxy S24 com Android 16, aparelho iOS e navegador.                                                              |
| **Verificação**   | Roteiros CT-SIS-01 a CT-SIS-05 e CT-SIS-08.                                                                                                    |
| **Evidência**     | O registro de CT-SIS-01 a CT-SIS-05 não informa em qual aparelho foram executados.                                                             |
| **Situação**      | Pendente: não há execução registrada por aparelho; iOS não executado (CT-SIS-08).                                                              |

### RNF-SIS-08 — Legibilidade nos dois temas

| Campo             | Conteúdo                                                                                                                                    |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------- |
| **Classificação** | Produto › Facilidade de uso                                                                                                                 |
| **Critério**      | A tela "Sistema" e seus diálogos usam as cores do tema ativo, sem cor fixa no código.                                                       |
| **Medida**        | Nenhuma cor literal em `sistema.tsx`: todas vêm de `useTema().tema.cores`, assim como em `Confirmacao`, `Botao`, `Cartao` e `FaixaAviso`. |
| **Condição**      | Tema do aplicativo em Claro e em Escuro.                                                                                                    |
| **Verificação**   | Inspeção de `FRONTEND/src/app/sistema.tsx`.                                                                                                 |
| **Evidência**     | `FRONTEND/src/app/sistema.tsx`.                                                                                                             |
| **Situação**      | Atendido por inspeção em 26/11/2026; não há teste registrado em aparelho no tema escuro.                                                    |

### RNF-SIS-09 — Testabilidade

| Campo             | Conteúdo                                                                                                               |
| ----------------- | ---------------------------------------------------------------------------------------------------------------------- |
| **Classificação** | Organizacional › Entrega                                                                                               |
| **Critério**      | A suíte do backend e a checagem de tipos e de estilo do aplicativo passam inteiras antes de cada entrega.              |
| **Medida**        | `npx vitest run` no BACKEND sem falha; `npx tsc --noEmit` e `npm run lint` no FRONTEND sem erro.                        |
| **Condição**      | Código da branch `homolog`.                                                                                            |
| **Verificação**   | Execução registrada em [Testes](7%20-%20testes-informacao-sistemas.md).                                                |
| **Evidência**     | 16 arquivos e 227 testes aprovados em 26/11/2026. Nenhum deles chama `SistemaController` ou `SistemaService` diretamente. |
| **Situação**      | Atendido em 26/11/2026 para a suíte existente; o serviço deste módulo não tem teste automatizado próprio.              |

## Rastreabilidade

RNF-SIS-01 a RNF-SIS-09 cobrem RF-SIS-01 a RF-SIS-04 e são verificados pela matriz CT-SIS.
