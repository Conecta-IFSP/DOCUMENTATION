# Requisitos não funcionais — Informações do sistema

> Padrão da disciplina · Módulo SIS · Revisado em 27/11/2026

## Resumo

Os requisitos não funcionais aparecem em dois níveis, como os funcionais. Os **de usuário** dizem, em uma frase, a qualidade esperada. Os **de sistema** completam cada um com a medida que o aprova, a condição em que vale, como foi verificado e onde está a evidência, de modo que qualquer pessoa consiga conferir. Onde ainda não houve medição, ou onde o requisito não está implementado, isso está escrito.

A classificação segue a árvore de Sommerville usada na disciplina: requisitos de **produto** (facilidade de uso, eficiência, confiabilidade, portabilidade), **organizacionais** (entrega, implementação, padrões) e **externos** (interoperabilidade, éticos, legais).

## Requisitos não funcionais de usuário

**RNF-SIS-01 — Acesso só do administrador do sistema.** Só o administrador do sistema pode ver as informações do sistema, criar dados de teste e resetar o sistema.

**RNF-SIS-02 — Confirmação antes de alterar dados.** Criar dados de teste e resetar o sistema devem pedir confirmação antes de mudar qualquer dado.

**RNF-SIS-03 — Confirmação reforçada do reset.** O reset, que apaga quase todos os dados, deve exigir uma confirmação mais forte que um simples toque em "Confirmar".

**RNF-SIS-04 — Reset sem resultado pela metade.** Se o reset falhar, nenhum dado deve ficar apagado só em parte.

**RNF-SIS-05 — Tempo de resposta das consultas.** As informações, o status e os relatórios devem chegar ao administrador em até 1 segundo.

**RNF-SIS-06 — Métricas sem dependência externa.** As métricas do servidor devem ser obtidas sem instalar bibliotecas ou serviços de monitoramento.

**RNF-SIS-07 — Portabilidade.** A tela "Sistema" deve funcionar em celulares Android a partir da versão 7.0, em iOS e no navegador.

**RNF-SIS-08 — Legibilidade nos dois temas.** A tela "Sistema" deve ser legível no tema claro e no tema escuro.

**RNF-SIS-09 — Testabilidade.** Os testes automatizados devem passar antes de cada entrega.

## Requisitos não funcionais de sistema

### RNF-SIS-01 — Acesso só do administrador do sistema

| Campo             | Conteúdo                                                                                                                                                                                                 |
| ----------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Classificação** | Externo › Legal › Segurança                                                                                                                                                                              |
| **Critério**      | As cinco rotas `/sistema` só executam para a conta ativa com `tipo` igual a `ADMIN_SISTEMA` no banco, com a chave do aplicativo correta. O tipo é lido do banco a cada requisição, não do token.          |
| **Medida**        | Sem `X-API-Key` válida: 401 "Chave de acesso da API inválida ou ausente". Sem token, com token inválido ou de conta inativa ou apagada: 401. Conta com `tipo` `USUARIO`: 403 "Apenas administradores do sistema podem realizar esta operação". Nos três casos, o `SistemaService` não é chamado. No aplicativo, a rota `sistema` não existe para a sessão de usuário comum. |
| **Condição**      | Rotas `GET /sistema/sobre`, `GET /sistema/status`, `GET /sistema/relatorios`, `POST /sistema/popular-teste` e `POST /sistema/reset`.                                                                       |
| **Verificação**   | Roteiro manual CT-SIS-01 (401, 403 e 200 na rota de status). Teste automatizado CT-SIS-06, que verifica o mesmo par `JwtAuthGuard` e `AdminSistemaGuard` nas rotas administrativas de usuários: com a conta rebaixada para `USUARIO`, o mesmo token recebe 403 e o serviço não é chamado. |
| **Evidência**     | `BACKEND/src/auth/api-key.guard.ts`, `BACKEND/src/auth/jwt-auth.guard.ts`, `BACKEND/src/auth/admin-sistema.guard.ts`, `@UseGuards` na classe `SistemaController` em `BACKEND/src/sistema/sistema.controller.ts`, que vale também para a rota `sobre`, `Stack.Protected` em `FRONTEND/src/app/_layout.tsx` e `BACKEND/src/usuarios/usuarios-acesso.spec.ts`. |
| **Situação**      | Atendido. CT-SIS-01 aprovado pela equipe, na versão substituída; CT-SIS-06 aprovado em 27/11/2026. Os guards não mudaram na versão atual. Não há teste automatizado que chame as rotas `/sistema` diretamente. |

### RNF-SIS-02 — Confirmação antes de alterar dados

| Campo             | Conteúdo                                                                                                                                                       |
| ----------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Classificação** | Produto › Facilidade de uso                                                                                                                                    |
| **Critério**      | Os botões "Popular dados de teste" e "Resetar sistema" só abrem um diálogo ou um cartão de confirmação; a requisição só sai depois do toque em "Confirmar", nos dados de teste, ou em "Apagar tudo", no reset. |
| **Medida**        | Tocar em "Cancelar" ou no botão voltar do Android, no diálogo, ou em "Cancelar", no cartão do reset, não envia nenhuma requisição e não muda nenhum dado.      |
| **Condição**      | Tela "Sistema": diálogo "Criar 3 usuários de teste?" e cartão "Resetar sistema".                                                                               |
| **Verificação**   | Roteiros manuais CT-SIS-03 e CT-SIS-04, executados em versões substituídas; CT-SIS-17 e CT-SIS-18, com a versão atual.                                         |
| **Evidência**     | `FRONTEND/src/app/sistema.tsx` (estados `confirmacaoTeste` e `resetAberto` e função `fecharReset`) e `FRONTEND/src/components/Confirmacao.tsx`.                 |
| **Situação**      | Implementado e conferido por inspeção em 27/11/2026. Na versão atual, CT-SIS-17 e CT-SIS-18 foram executados em parte em 27/11/2026 (o diálogo "Criar 3 usuários de teste?" e o "Apagar tudo" inativo com "RESE" foram vistos); o "Cancelar" dos dois não foi registrado. |

### RNF-SIS-03 — Confirmação reforçada do reset

| Campo             | Conteúdo                                                                                                                                                                       |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Classificação** | Externo › Legal › Segurança                                                                                                                                                    |
| **Critério**      | O reset exige duas provas além do toque no botão: digitar a palavra "RESET" e a senha de quem pede. As duas são conferidas pela API, e a senha é comparada com o hash bcrypt guardado na conta, o que vale também na web e para chamadas feitas fora do aplicativo. |
| **Medida**        | Na tela, "Apagar tudo" fica inativo enquanto a palavra não é exatamente "RESET" ou a senha está vazia. Na API: sem `confirmacao` igual a "RESET", 400 "Digite RESET para confirmar"; sem `senha` em texto, 400 "Informe a senha do administrador"; com a senha errada, 400 "Senha incorreta"; conta sem senha, 400 "Defina uma senha na sua conta para resetar o sistema". Nos quatro casos nada é apagado. |
| **Condição**      | Cartão "Resetar sistema" da tela "Sistema" e rota `POST /sistema/reset`. Não há limite de tentativas: quem tem o token de um administrador do sistema pode tentar várias senhas seguidas. |
| **Verificação**   | Testes automatizados CT-SIS-10 (validação do `ResetarSistemaDto`), CT-SIS-11 (senha certa) e CT-SIS-15 (senha errada, sem apagar nada); roteiros CT-SIS-17 no aparelho e CT-SIS-19 com chamadas diretas à API. |
| **Evidência**     | `BACKEND/src/sistema/dto/resetar-sistema.dto.ts`, `BACKEND/src/sistema/sistema.controller.ts`, `confirmarSenhaAdmin` em `BACKEND/src/sistema/sistema.service.ts`, `BACKEND/src/sistema/sistema.spec.ts` e `FRONTEND/src/app/sistema.tsx` (`resetValido` e `resetarSistema`). |
| **Situação**      | Implementado em `931c2c8` e `795d49b`, atendendo à avaliação do Incremento 1 ("Resetar sistema precisa de uma confirmação mais segura"). CT-SIS-10, CT-SIS-11 e CT-SIS-15 aprovados em 27/11/2026; CT-SIS-17 executado em parte em 27/11/2026, no emulador Android 14 e no Galaxy S24: "Apagar tudo" inativo com "RESE", "Senha incorreta" com a senha errada sem apagar nada e, com a senha certa, só a conta `ADMIN_SISTEMA` no banco; CT-SIS-19 a executar. |

### RNF-SIS-04 — Reset sem resultado pela metade

| Campo             | Conteúdo                                                                                                                                                                                |
| ----------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Classificação** | Produto › Confiabilidade                                                                                                                                                                |
| **Critério**      | O reset apaga todas as coleções previstas ou nenhuma.                                                                                                                                   |
| **Medida**        | Se uma das exclusões falhar, as outras são desfeitas e a resposta é de erro.                                                                                                            |
| **Condição**      | `POST /sistema/reset` com a senha certa e falha do MongoDB durante as exclusões.                                                                                                        |
| **Verificação**   | Inspeção de `SistemaService.reset`.                                                                                                                                                     |
| **Evidência**     | `BACKEND/src/sistema/sistema.service.ts`: depois de `confirmarSenhaAdmin`, as oito exclusões rodam ao mesmo tempo com `Promise.all`, sem sessão nem transação do MongoDB.                |
| **Situação**      | **Não atendido.** Uma falha no meio pode deixar parte das coleções apagada e parte não; a API responde 500 e o administrador precisa conferir o banco. A senha errada, conferida antes, não apaga nada. |

### RNF-SIS-05 — Tempo de resposta das consultas

| Campo             | Conteúdo                                                                                                                                                                         |
| ----------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Classificação** | Produto › Eficiência › Desempenho                                                                                                                                                |
| **Critério**      | As rotas de informações, de status e de relatórios respondem rápido o suficiente para a tela abrir sem espera perceptível.                                                        |
| **Medida**        | Cada rota responde em até 1.000 ms, do envio da requisição à chegada da resposta completa, em 10 chamadas seguidas; vale o maior dos 10 tempos.                                   |
| **Condição**      | API e MongoDB no mesmo computador, sem outras requisições ao mesmo tempo, com até 1.000 documentos em cada uma das coleções `usuarios`, `organizacoes`, `comissoes` e `reunioes`. |
| **Verificação**   | CT-SIS-09: medir com uma ferramenta de requisições HTTP (por exemplo, Postman ou `curl` com `-w "%{time_total}"`).                                                               |
| **Evidência**     | `SistemaService.sobre` (um comando `buildInfo`), `SistemaService.status` (espera fixa de 200 ms para medir a CPU, em paralelo com `ping`, `dbStats` e quatro contagens) e `SistemaService.relatorios` (treze contagens em paralelo), em `BACKEND/src/sistema/sistema.service.ts`. |
| **Situação**      | Medição pendente: CT-SIS-09 não foi executado. A rota de status gasta pelo menos 200 ms só na medição da CPU.                                                                     |

### RNF-SIS-06 — Métricas sem dependência externa

| Campo             | Conteúdo                                                                                                                                                |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Classificação** | Organizacional › Implementação                                                                                                                          |
| **Critério**      | Memória, CPU, processador e tempo ativo vêm dos módulos `node:os` e `process`, que fazem parte do Node.js; a situação do banco vem de comandos do próprio MongoDB (`ping`, `dbStats` e `buildInfo`); o módulo não cria coleção própria. |
| **Medida**        | `sistema.service.ts` importa só `node:module`, `node:os`, `bcrypt`, `@nestjs/*`, `mongoose` e os schemas do projeto; o `sistema.module.ts` registra só schemas de outros módulos. |
| **Condição**      | Código da branch `homolog`.                                                                                                                             |
| **Verificação**   | Inspeção das importações de `BACKEND/src/sistema/sistema.service.ts` e `BACKEND/src/sistema/sistema.module.ts`.                                          |
| **Evidência**     | Os dois arquivos citados.                                                                                                                                |
| **Situação**      | Atendido por inspeção em 27/11/2026.                                                                                                                     |

### RNF-SIS-07 — Portabilidade

| Campo             | Conteúdo                                                                                                                                       |
| ----------------- | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| **Classificação** | Produto › Portabilidade                                                                                                                        |
| **Critério**      | A tela "Sistema" funciona do Android 7.0 (API 24, o mínimo do aplicativo) ao Android 16, no iOS e no navegador.                                 |
| **Medida**        | Consultar, popular e resetar sem erro e sem texto cortado nos cartões, no diálogo e no cartão do reset.                                        |
| **Condição**      | Emuladores Android 7.0 e 16, Galaxy S24 com Android 16, aparelho iOS e navegador.                                                              |
| **Verificação**   | Roteiros CT-SIS-08 e CT-SIS-16 a CT-SIS-18.                                                                                                    |
| **Evidência**     | O registro de CT-SIS-01 a CT-SIS-05 não informa em qual aparelho foram executados; CT-SIS-13 e CT-SIS-14 foram executados no Galaxy S24, com a versão substituída; CT-SIS-16 a CT-SIS-18, com a versão atual, no emulador Android 14 e no Galaxy S24 (Android 16), em 27/11/2026. |
| **Situação**      | Parcial: a versão atual funcionou no Android 14 e no Android 16 em 27/11/2026, em parte dos passos de CT-SIS-16 a CT-SIS-18; Android 7.0, navegador e iOS (CT-SIS-08) não executados. |

### RNF-SIS-08 — Legibilidade nos dois temas

| Campo             | Conteúdo                                                                                                                                    |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------- |
| **Classificação** | Produto › Facilidade de uso                                                                                                                 |
| **Critério**      | A tela "Sistema", seu diálogo, seu cartão de reset e os avisos flutuantes usam as cores do tema ativo, sem cor fixa no código.              |
| **Medida**        | Nenhuma cor literal em `sistema.tsx` nem em `Confirmacao`, `CampoTexto`, `Botao`, `Cartao`, `FaixaAviso` e `ToastContext`: as cores vêm de `useTema().tema.cores`. Exceções: o texto do `Botao` primário usa a constante `MARCA.branco`, sobre o degradê do tema, e a sombra do aviso flutuante usa `#000`. A tela "Sistema" não tem botão primário. |
| **Condição**      | Tema do aplicativo em Claro e em Escuro.                                                                                                    |
| **Verificação**   | Inspeção de `FRONTEND/src/app/sistema.tsx` e dos componentes citados.                                                                       |
| **Evidência**     | `FRONTEND/src/app/sistema.tsx` e `FRONTEND/src/contexts/ToastContext.tsx`.                                                                  |
| **Situação**      | Atendido por inspeção em 27/11/2026; não há teste registrado em aparelho no tema escuro.                                                    |

### RNF-SIS-09 — Testabilidade

| Campo             | Conteúdo                                                                                                               |
| ----------------- | ---------------------------------------------------------------------------------------------------------------------- |
| **Classificação** | Organizacional › Entrega                                                                                               |
| **Critério**      | A suíte do backend e a checagem de tipos e de estilo do aplicativo passam inteiras antes de cada entrega.              |
| **Medida**        | `npx vitest run` no BACKEND sem falha; `npx tsc --noEmit` e `npm run lint` no FRONTEND sem erro.                        |
| **Condição**      | Código da branch `homolog`.                                                                                            |
| **Verificação**   | Execução registrada em [Testes](7%20-%20testes-informacao-sistemas.md).                                                |
| **Evidência**     | Na última execução, em 27/11/2026, sobre o backend `d1f1aeb` e o frontend `10abe49`: 17 arquivos e 246 testes aprovados no backend, `npx tsc --noEmit` e `npm run lint` sem erro no FRONTEND. O `sistema.spec.ts` tem 6 testes: valida o `ResetarSistemaDto` e chama `SistemaService.reset` com models simulados. Nenhum teste chama `SistemaController` nem as rotas `/sistema`. |
| **Situação**      | Atendido em 27/11/2026. Só o reset tem teste automatizado próprio; informações, status, relatórios e dados de teste não têm. |

## Rastreabilidade

RNF-SIS-01 a RNF-SIS-09 cobrem RF-SIS-01 a RF-SIS-05 e são verificados pela matriz CT-SIS.
