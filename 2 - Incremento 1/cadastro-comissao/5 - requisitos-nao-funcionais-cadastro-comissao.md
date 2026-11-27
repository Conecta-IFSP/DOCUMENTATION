# Requisitos não funcionais — Cadastro de comissão

> Padrão da disciplina · Módulo COM · Revisado em 27/11/2026

## Resumo

Os requisitos não funcionais aparecem em dois níveis, como os funcionais. Os **de usuário** dizem, em uma frase, a qualidade esperada. Os **de sistema** completam cada um com a medida que o aprova, a condição em que vale, como foi verificado e onde está a evidência, de modo que qualquer pessoa consiga conferir. Onde ainda não houve medição, isso está escrito.

A classificação segue a árvore de Sommerville usada na disciplina: requisitos de **produto** (facilidade de uso, eficiência, confiabilidade, portabilidade), **organizacionais** (entrega, implementação, padrões) e **externos** (interoperabilidade, éticos, legais).

## Requisitos não funcionais de usuário

**RNF-COM-01 — Acesso só por conta autenticada.** As comissões só podem ser vistas e alteradas pelo aplicativo oficial, por uma pessoa que entrou na própria conta, e nunca pela conta do administrador do sistema.

**RNF-COM-02 — Autorização para gerenciar.** Só o administrador da organização pode cadastrar, alterar, desativar e reativar comissões e mudar a equipe; os demais membros só veem as comissões de que participam.

**RNF-COM-03 — Integridade da equipe gravada.** A equipe de uma comissão nunca pode ficar gravada no banco fora da regra do responsável, mesmo que uma rota deixe passar um pedido errado.

**RNF-COM-04 — Proteção contra gravação simultânea.** Dois administradores mudando a mesma equipe ao mesmo tempo não podem apagar a mudança um do outro.

**RNF-COM-05 — Confirmação de ações destrutivas.** Desativar, reativar, trocar o responsável e remover um integrante devem pedir confirmação antes de acontecer.

**RNF-COM-06 — Tempo de resposta da lista.** A lista de comissões de uma organização com 100 comissões deve chegar ao aplicativo em até 2 segundos.

**RNF-COM-07 — Uso em tela de celular.** A tela da comissão deve caber numa tela de celular sem obrigar a pessoa a rolar por todos os dados de uma vez.

**RNF-COM-08 — Portabilidade.** O cadastro de comissões deve funcionar em celulares Android a partir da versão 7.0 e em iOS.

**RNF-COM-09 — Testabilidade.** Os testes automatizados do módulo devem passar antes de cada entrega.

## Requisitos não funcionais de sistema

### RNF-COM-01 — Acesso só por conta autenticada

| Campo             | Conteúdo                                                                                                                                                                                                 |
| ----------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Classificação** | Externo › Legal › Segurança                                                                                                                                                                              |
| **Critério**      | Toda rota `/comissoes` exige a chave `X-API-Key`, um token JWT válido de conta ativa e uma conta que não seja `ADMIN_SISTEMA`.                                                                           |
| **Medida**        | Sem chave: 401 "Chave de acesso da API inválida ou ausente". Sem token: 401 "Token de acesso não informado". Conta `ADMIN_SISTEMA`: 403 "O administrador do sistema não pode realizar esta operação", e o serviço não é chamado. |
| **Condição**      | Qualquer uma das 12 rotas de `ComissoesController`.                                                                                                                                                      |
| **Verificação**   | Teste automatizado CT-COM-28 (token de usuário comum aceito; administrador do sistema recusado em `GET` e `POST /comissoes`). A recusa sem `X-API-Key` vem do guard global, sem teste próprio neste módulo. |
| **Evidência**     | `@UseGuards(JwtAuthGuard, UsuarioComumGuard)` em `ComissoesController`; `ApiKeyGuard` registrado como `APP_GUARD` em `BACKEND/src/app.module.ts`; `BACKEND/src/comissoes/comissoes.controller.spec.ts`. |
| **Situação**      | Atendido em 26/11/2026 para o JWT e o tipo de conta; a chave `X-API-Key` foi conferida por inspeção. A chave fica no aplicativo (`EXPO_PUBLIC_API_ACCESS_TOKEN`) e pode ser extraída dele; quem protege os dados é o JWT. |

### RNF-COM-02 — Autorização para gerenciar

| Campo             | Conteúdo                                                                                                                                                                                                                          |
| ----------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Classificação** | Externo › Legal › Segurança                                                                                                                                                                                                       |
| **Critério**      | Só quem tem vínculo `ADMIN` `APROVADO` numa organização `APROVADA` grava comissões dessa organização; quem não é administrador só lê as comissões de cuja equipe faz parte.                                                       |
| **Medida**        | Gravação por qualquer outra conta: 403 "Apenas administradores da organização podem gerenciar comissões", sem nenhuma gravação. Leitura de comissão de que a pessoa não participa: 403 "Você não faz parte desta comissão". Vínculo pendente ou rejeitado e organização revogada ou pendente: 403 também na leitura. Conta inativa: 401 "Conta inexistente ou desativada" já no `JwtAuthGuard`; o 403 "Usuário sem acesso ativo" do serviço só aparece se a conta for desativada durante a requisição. |
| **Condição**      | Rotas de cadastro, edição, desativação, reativação, equipe e responsável; rotas de lista e consulta.                                                                                                                              |
| **Verificação**   | Testes automatizados CT-COM-10, CT-COM-11, CT-COM-12, CT-COM-13 e CT-COM-15.                                                                                                                                                      |
| **Evidência**     | `ComissoesService.autorizarOrganizacao`, `autorizarLeitura` e `buscarPorId`; `BACKEND/src/comissoes/comissoes.regras.spec.ts`.                                                                                                    |
| **Situação**      | Atendido em 26/11/2026.                                                                                                                                                                                                           |

### RNF-COM-03 — Integridade da equipe gravada

| Campo             | Conteúdo                                                                                                                                                                                                                   |
| ----------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Classificação** | Produto › Confiabilidade                                                                                                                                                                                                   |
| **Critério**      | Nenhum documento de comissão gravado por este módulo fica com 0 ou 2 ou mais responsáveis, nem com a mesma pessoa duas vezes na equipe. A regra de negócio do responsável está no RF-COM-03, no RF-COM-06 e nos RF-COM-07 a RF-COM-09; aqui o que se exige é que ela seja garantida em duas camadas, no serviço e no próprio schema do banco.                                                                                          |
| **Medida**        | Cadastrar cria a equipe com 1 responsável; trocar o responsável resulta em 1 responsável e mantém o anterior como `MEMBRO`; adicionar como `RESPONSAVEL`, remover o responsável ou rebaixá-lo recebem 400 sem gravação; reativar ou mudar a equipe de uma comissão com 0 ou 2 responsáveis recebe 400 até a troca regularizar. Além disso, o validador do schema recusa gravar qualquer lista fora da regra, mesmo que o pedido passe pelo serviço. Meta: 0 documentos gravados fora da regra. |
| **Condição**      | Rotas `POST /comissoes`, `PATCH /comissoes/:id` (reativação), `POST /comissoes/:id/membros`, `PATCH /comissoes/:id/responsavel`, `PATCH` e `DELETE /comissoes/:id/membros/:usuarioId`.                                    |
| **Verificação**   | Testes automatizados CT-COM-14, CT-COM-19, CT-COM-23 e CT-COM-26 (camada do serviço); inspeção do validador de `membros` no schema (camada do banco), que não tem teste automatizado próprio.                                                                                                                                                         |
| **Evidência**     | Validador de `membros` em `BACKEND/src/comissoes/schemas/comissao.schema.ts`; `ComissoesService.exigirResponsavelUnico` e `definirResponsavel`.                                                                           |
| **Situação**      | Atendido em 26/11/2026 dentro do módulo. Limitação: a remoção de uma pessoa da organização, feita pelo módulo de organização, retira a pessoa das comissões sem passar por esta regra e pode deixar uma comissão sem responsável (ver o [dicionário relacional](4%20-%20dicionario-dados-relacional-cadastro-comissao.md)). |

### RNF-COM-04 — Proteção contra gravação simultânea

| Campo             | Conteúdo                                                                                                                                                                                      |
| ----------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Classificação** | Produto › Confiabilidade                                                                                                                                                                      |
| **Critério**      | Duas mudanças de equipe feitas ao mesmo tempo nunca sobrescrevem uma à outra.                                                                                                                 |
| **Medida**        | Toda mudança de equipe usa `findOneAndUpdate` com o `_id`, o `ativo` e a lista `membros` lidos como filtro. Se o documento já mudou, a resposta é 409 "A comissão mudou. Atualize e tente novamente." e nada é alterado. |
| **Condição**      | Adicionar integrante, trocar o responsável e remover integrante.                                                                                                                              |
| **Verificação**   | Testes automatizados CT-COM-21 e CT-COM-24, que simulam o `findOneAndUpdate` devolvendo `null`. A disputa real entre duas gravações no MongoDB era coberta pelo script CT-COM-30, que foi descontinuado; não há teste com MongoDB real. |
| **Evidência**     | `ComissoesService.gravarEquipe`.                                                                                                                                                              |
| **Situação**      | Atendido nos testes automatizados em 26/11/2026; sem verificação com MongoDB real.                                                                                                       |

### RNF-COM-05 — Confirmação de ações destrutivas

| Campo             | Conteúdo                                                                                                                                                                                   |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Classificação** | Produto › Facilidade de uso                                                                                                                                                                |
| **Critério**      | Desativar, reativar, trocar o responsável e remover integrante abrem um diálogo de confirmação; nada é enviado à API antes de a pessoa confirmar.                                          |
| **Medida**        | As 4 ações mostram o diálogo (`Confirmacao`) com o nome da comissão ou da pessoa e os botões "Cancelar" e a ação ("Desativar", "Reativar", "Confirmar" ou "Remover"); desativar e remover aparecem em vermelho. "Cancelar" fecha o diálogo sem chamada à API. |
| **Condição**      | Tela da comissão, com a comissão ativa (desativar, trocar, remover) ou inativa (reativar).                                                                                                 |
| **Verificação**   | Inspeção de `FRONTEND/src/app/comissoes/[id].tsx` em 26/11/2026; o roteiro CT-COM-06 registrou a remoção após confirmação, e o CT-COM-34, em 27/11/2026, o diálogo da desativação no Galaxy S24. O roteiro CT-COM-31, em 27/11/2026, registrou o cancelamento e a confirmação da troca do responsável. |
| **Evidência**     | `TelaComissao`: estado `confirmacao` e funções `trocarResponsavel` e `executar`.                                                                                                           |
| **Situação**      | Atendido por inspeção. Em aparelho, o CT-COM-31 (27/11/2026, emulador Android 14) confirmou o diálogo da troca do responsável: "Cancelar" não mudou nada e "Confirmar" trocou as etiquetas. O cancelamento dos diálogos de remover integrante não foi feito, e o de desativar e reativar não foi registrado.                                                                                                    |

### RNF-COM-06 — Tempo de resposta da lista

| Campo             | Conteúdo                                                                                                                                         |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Classificação** | Produto › Eficiência › Desempenho                                                                                                                |
| **Critério**      | O tempo de resposta da rota da lista de comissões de uma organização fica abaixo de um limite fixo.                                             |
| **Medida**        | `GET /comissoes?organizacao_id=` responde em até 2 segundos, medidos do envio da requisição ao recebimento da resposta.                          |
| **Condição**      | Organização com 100 comissões de 10 integrantes cada, API e MongoDB no mesmo computador da equipe, sem outras requisições ao mesmo tempo.        |
| **Verificação**   | Medição do tempo da requisição nas ferramentas de rede do navegador, na versão web do aplicativo.                                               |
| **Evidência**     | `ComissoesService.listar` e `consultar`.                                                                                                         |
| **Situação**      | Critério proposto; medição pendente.                                                                                                             |

### RNF-COM-07 — Uso em tela de celular

| Campo             | Conteúdo                                                                                                                                                                                           |
| ----------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Classificação** | Produto › Facilidade de uso                                                                                                                                                                        |
| **Critério**      | A tela da comissão separa reuniões e equipe em abas, e as ações de gerência abrem em folhas, sem formulários fixos na tela.                                                                        |
| **Medida**        | A tela da comissão mostra uma aba por vez ("Reuniões" ou "Equipe"); editar dados, adicionar integrante e escolher o responsável abrem em folhas inferiores; nenhum texto fica cortado nem exige rolagem horizontal numa tela de 720 × 1280. |
| **Condição**      | Emulador Android 7.0 (720 × 1280) e Galaxy S24.                                                                                                                                                    |
| **Verificação**   | Inspeção de `FRONTEND/src/app/comissoes/[id].tsx` (componente `Abas` e `FolhaInferior`) em 26/11/2026; roteiro CT-COM-31.                                                                          |
| **Evidência**     | `TelaComissao`, estados `aba` e `folha`.                                                                                                                                                           |
| **Situação**      | Atendido na tela de 720 × 1280 em 27/11/2026 (CT-COM-31, emulador Android 14 com a tela forçada para 720 × 1280 e 320 dpi): as abas "Reuniões" e "Equipe" mostraram uma por vez, e "Mais opções" e as folhas "Editar comissão" e "Adicionar integrante" abriram sem texto cortado (o nome longo do responsável aparece com reticências). O Galaxy S24 não foi usado nesse roteiro.                                                                                                             |

### RNF-COM-08 — Portabilidade

| Campo             | Conteúdo                                                                                                                                      |
| ----------------- | --------------------------------------------------------------------------------------------------------------------------------------------- |
| **Classificação** | Produto › Portabilidade                                                                                                                       |
| **Critério**      | O fluxo completo funciona do Android 7.0 (API 24, o mínimo do aplicativo) ao Android 16, e no iOS.                                            |
| **Medida**        | Listar, consultar, cadastrar, editar, desativar, reativar, adicionar, trocar o responsável e remover, sem erro e sem texto cortado.          |
| **Condição**      | Emulador Android 7.0, Galaxy S24 com Android 16 e aparelho iOS.                                                                               |
| **Verificação**   | Roteiros CT-COM-31, CT-COM-33 e CT-COM-34 (Android) e CT-COM-32 (iOS).                                                                                               |
| **Evidência**     | Os roteiros CT-COM-01 a CT-COM-06 foram aprovados pela equipe em ambiente móvel, sem registro de aparelho nem de versão do sistema.           |
| **Situação**      | Parcial: em 27/11/2026, CT-COM-31 foi executado em parte no emulador Android 14 (consultar a comissão e trocar o responsável), e CT-COM-33 e CT-COM-34 foram aprovados no Galaxy S24 com Android 16 (cadastrar e desativar). O emulador Android 7.0 não foi usado, porque tem PIN de bloqueio; editar, reativar, adicionar e remover não foram concluídos em aparelho com as telas atuais; iOS não executado por falta de equipamento.                                                                |

### RNF-COM-09 — Testabilidade

| Campo             | Conteúdo                                                                                                         |
| ----------------- | ---------------------------------------------------------------------------------------------------------------- |
| **Classificação** | Organizacional › Entrega                                                                                         |
| **Critério**      | A suíte do backend passa inteira antes de cada entrega, incluindo os testes do módulo.                           |
| **Medida**        | `npx vitest run` sem nenhuma falha; os três arquivos de teste de `BACKEND/src/comissoes/` aprovados.             |
| **Condição**      | Código da branch `main`.                                                                                         |
| **Verificação**   | Execução registrada em [Testes](7%20-%20testes-cadastro-comissao.md).                                            |
| **Evidência**     | 17 arquivos e 246 testes aprovados em 27/11/2026 (backend na versão com o voto em branco e nulo), dos quais 3 arquivos e 40 testes do módulo. |
| **Situação**      | Atendido em 27/11/2026. O script com MongoDB real (CT-COM-30) foi descontinuado. |

## Rastreabilidade

RNF-COM-01 a RNF-COM-09 cobrem RF-COM-01 a RF-COM-09 e são verificados pela matriz CT-COM.
