# Requisitos não funcionais — Cadastro de organização

> Padrão da disciplina · Módulo ORG · Revisado em 27/11/2026

## Resumo

Os requisitos não funcionais aparecem em dois níveis, como os funcionais. Os **de usuário** dizem, em uma frase, a qualidade esperada. Os **de sistema** completam cada um com a medida que o aprova, a condição em que vale, como foi verificado e onde está a evidência, de modo que qualquer pessoa consiga conferir. Onde ainda não houve medição, isso está escrito.

A classificação segue a árvore de Sommerville usada na disciplina: requisitos de **produto** (facilidade de uso, eficiência, confiabilidade, portabilidade), **organizacionais** (entrega, implementação, padrões) e **externos** (interoperabilidade, éticos, legais).

## Requisitos não funcionais de usuário

**RNF-ORG-01 — Acesso identificado.** Só quem entrou no aplicativo com uma conta ativa pode consultar ou alterar organizações.

**RNF-ORG-02 — Autorização por papel.** Cada ação sobre organizações só pode ser feita pelo tipo de conta e pelo papel a que ela pertence.

**RNF-ORG-03 — Sigilo do código de acesso.** O código de acesso só deve ser visto por administradores da organização e deve ser difícil de adivinhar.

**RNF-ORG-04 — Proteção contra gravação simultânea.** Duas pessoas agindo ao mesmo tempo sobre os mesmos vínculos não podem desfazer a decisão uma da outra nem deixar a organização sem administrador.

**RNF-ORG-05 — Confirmação de ações destrutivas.** Ações que apagam dados, retiram acesso, mudam papéis ou invalidam o código devem pedir confirmação antes de serem executadas.

**RNF-ORG-06 — Eficiência da busca.** A busca de organizações não deve fazer uma requisição a cada tecla nem mostrar o resultado de uma busca antiga.

**RNF-ORG-07 — Tamanho dos textos de exemplo.** Os textos de exemplo dos campos devem caber na largura do campo, também em tela pequena.

**RNF-ORG-08 — Portabilidade.** As telas de organização devem funcionar em celulares Android a partir da versão 7.0 e em iOS.

**RNF-ORG-09 — Testabilidade.** Os testes automatizados do servidor devem passar antes de cada entrega.

## Requisitos não funcionais de sistema

### RNF-ORG-01 — Acesso identificado

| Campo             | Conteúdo                                                                                                                                                                                               |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Classificação** | Externo › Legal › Segurança                                                                                                                                                                            |
| **Critério**      | Toda rota de `/organizacoes` exige a chave do aplicativo (`X-API-Key`), um token JWT válido e uma conta ativa, relida no banco a cada requisição.                                                       |
| **Medida**        | Nas 12 rotas do módulo, requisição sem token, com token inválido ou expirado, com token emitido antes da última troca de senha, ou de conta desativada ou excluída recebe HTTP 401, e o serviço não é chamado. |
| **Condição**      | Todas as rotas de `OrganizacoesController`.                                                                                                                                                            |
| **Verificação**   | Teste automatizado CT-ORG-27 e, para a rota de remoção de membro, CT-ORG-24. O token anterior à troca de senha não é testado nas rotas deste módulo: a regra é de `JwtAuthGuard` e está coberta pelo CT-USR-15, nos testes do [Cadastro de usuário](../cadastro-usuario/7%20-%20testes-cadastro-usuario.md). |
| **Evidência**     | `@UseGuards(JwtAuthGuard, UsuarioAtivoGuard)` em `BACKEND/src/organizacoes/organizacoes.controller.ts`, `senha_alterada_em` conferido em `BACKEND/src/auth/jwt-auth.guard.ts`, `ApiKeyGuard` global em `BACKEND/src/app.module.ts` e `BACKEND/src/organizacoes/organizacoes-acesso.spec.ts`. A chave `X-API-Key` vai dentro do aplicativo e não é um segredo forte; quem protege os dados de cada pessoa é o token. |
| **Situação**      | Atendido em 26/11/2026.                                                                                                                                                                                |

### RNF-ORG-02 — Autorização por papel

| Campo             | Conteúdo                                                                                                                                                                                                                                       |
| ----------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Classificação** | Externo › Legal › Segurança                                                                                                                                                                                                                    |
| **Critério**      | Só a conta `ADMIN_SISTEMA` autoriza, revoga, edita e exclui organizações; só a conta `USUARIO` cadastra, pede entrada, convida, responde convites e gerencia vínculos; só o vínculo `ADMIN` e `APROVADO` gerencia membros e código. O tipo da conta é o gravado no banco, não o que veio no token. |
| **Medida**        | Cada tentativa fora dessas regras recebe HTTP 403 e nada é gravado, inclusive quando o token foi emitido com um tipo que a conta já não tem.                                                                                                     |
| **Condição**      | Rotas `PATCH` e `DELETE /organizacoes/:id` (administrador do sistema) e demais rotas de alteração (usuário comum e administrador da organização).                                                                                               |
| **Verificação**   | Testes automatizados CT-ORG-12, CT-ORG-17, CT-ORG-26 e CT-ORG-27.                                                                                                                                                                              |
| **Evidência**     | `AdminSistemaGuard` e `UsuarioComumGuard` em `BACKEND/src/auth/admin-sistema.guard.ts`, `UsuarioAtivoGuard` e `OrganizacoesService.exigirAdministrador`.                                                                                       |
| **Situação**      | Atendido em 26/11/2026.                                                                                                                                                                                                                        |

### RNF-ORG-03 — Sigilo do código de acesso

| Campo             | Conteúdo                                                                                                                                                                                                                                                                       |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Classificação** | Externo › Legal › Segurança                                                                                                                                                                                                                                                    |
| **Critério**      | O código só é devolvido a administradores da organização, não se repete entre organizações e não pode ser deduzido.                                                                                                                                                           |
| **Medida**        | 10 caracteres sorteados com `randomInt` (gerador criptográfico) num alfabeto de 31 símbolos, o que dá 31¹⁰, cerca de 8,2 × 10¹⁴ combinações; índice único no banco; o campo não sai na criação nem nas listas (`select: false`); código curto, inexistente ou de organização não autorizada recebem a mesma resposta, "Código inválido". |
| **Condição**      | Cadastro (RF-ORG-01), consulta (RF-ORG-02), entrada por código (RF-ORG-07), troca do código (RF-ORG-10) e consulta de membros (RF-ORG-14).                                                                                                                                      |
| **Verificação**   | Testes automatizados CT-ORG-08, CT-ORG-10, CT-ORG-14 e CT-ORG-20; inspeção de `gerarCodigoAcesso`.                                                                                                                                                                            |
| **Evidência**     | `gerarCodigoAcesso`, `ALFABETO_CODIGO` e o campo `codigo_acesso` em `BACKEND/src/organizacoes/schemas/organizacao.schema.ts`; `OrganizacoesService.criar`, `entrarComCodigo` e `gerarNovoCodigo`.                                                                              |
| **Situação**      | Atendido em 26/11/2026. Não há limite de tentativas de código por conta nem por tempo: um usuário pode tentar códigos sem bloqueio (sem medida definida).                                                                                                                       |

### RNF-ORG-04 — Proteção contra gravação simultânea

| Campo             | Conteúdo                                                                                                                                                                                                                                       |
| ----------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Classificação** | Produto › Confiabilidade                                                                                                                                                                                                                       |
| **Critério**      | Duas ações ao mesmo tempo sobre o mesmo vínculo nunca sobrescrevem uma à outra, e a organização nunca fica sem administrador.                                                                                                                 |
| **Medida**        | Toda gravação de vínculo usa `updateOne` com a situação lida (e o solicitante ainda administrador) no filtro; se outra gravação mudou o vínculo antes, nada é alterado e a resposta é 409. No rebaixamento, o filtro exige outro administrador aprovado no instante da gravação. Na remoção, as comissões só são alteradas depois que o vínculo foi retirado. |
| **Condição**      | Entrar por código, convidar, analisar pedido, responder convite, alterar papel, remover vínculo e gerar código (RF-ORG-07 a RF-ORG-13).                                                                                                         |
| **Verificação**   | Testes automatizados CT-ORG-15, CT-ORG-19, CT-ORG-21, CT-ORG-23 e CT-ORG-25, que simulam `updateOne` sem alteração.                                                                                                                             |
| **Evidência**     | `OrganizacoesService.administradorAprovado` e os filtros de `adicionarMembro`, `atualizarStatusMembro`, `alterarPapelMembro`, `removerMembro`, `convidarMembro`, `responderConvite` e `gerarNovoCodigo`.                                       |
| **Situação**      | Atendido em 26/11/2026 para os vínculos. Não atendido para o nome: a unicidade do nome é conferida por consulta antes da gravação, sem índice, e dois cadastros com o mesmo nome no mesmo instante podem passar (não testado).                   |

### RNF-ORG-05 — Confirmação de ações destrutivas

| Campo             | Conteúdo                                                                                                                                                                                                                                       |
| ----------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Classificação** | Produto › Facilidade de uso                                                                                                                                                                                                                    |
| **Critério**      | Toda ação que apaga dados, retira acesso, muda papel ou invalida o código abre o diálogo `Confirmacao`, e "Cancelar" fecha o diálogo sem chamar a API.                                                                                         |
| **Medida**        | Nove ações com diálogo: revogar e excluir organização; recusar pedido; remover acesso, cancelar convite e excluir pedido recusado; tornar administrador e tornar membro (ou deixar de ser administrador); gerar novo código. Nenhuma requisição é feita antes do toque no botão de confirmação, e os botões do diálogo ficam inativos enquanto a requisição está em andamento. |
| **Condição**      | Telas "Gerenciar organizações" e tela da organização.                                                                                                                                                                                          |
| **Verificação**   | Inspeção do código em 27/11/2026. Roteiro em aparelho CT-ORG-28, executado em 27/11/2026 no emulador Android 14 nas nove ações com diálogo e no "Aprovar acesso" de um pedido recusado. CT-ORG-31, em 27/11/2026, conferiu no Galaxy S24 o diálogo e a confirmação de "Gerar novo código".                                                                |
| **Evidência**     | `FRONTEND/src/components/Confirmacao.tsx`, `TelaGerenciarOrganizacoes.confirmarAcao` (`FRONTEND/src/app/organizacoes/aprovacao.tsx`) e `TelaOrganizacao.pedir` e `confirmar` (`FRONTEND/src/app/(abas)/organizacao.tsx`).                    |
| **Situação**      | Atendido em 27/11/2026 (CT-ORG-28 e CT-ORG-31): em todas as ações com diálogo, "Cancelar" não muda nada e a confirmação executa a ação. Autorizar organização, aprovar pedido, aprovar pedido recusado e responder convite não pedem confirmação, por não retirarem acesso de ninguém.                                  |

### RNF-ORG-06 — Eficiência da busca

| Campo             | Conteúdo                                                                                                                                                                                                         |
| ----------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Classificação** | Produto › Eficiência › Desempenho                                                                                                                                                                                |
| **Critério**      | A busca do administrador do sistema não envia uma requisição por tecla nem mostra o resultado de uma busca mais antiga; o termo não pode virar uma expressão regular escolhida pelo usuário.                      |
| **Medida**        | Uma requisição só depois de 350 ms sem digitação; respostas que chegam fora de ordem são descartadas; o servidor usa no máximo 100 caracteres do termo, com os caracteres especiais escapados. Tempo de resposta: sem medição registrada. |
| **Condição**      | Tela "Gerenciar organizações". A busca da aba "Organizações" e a da aba "Membros" são feitas no próprio aparelho, sem requisição.                                                                                 |
| **Verificação**   | Inspeção de `ATRASO_BUSCA_MS` e `ultimaConsulta` em `FRONTEND/src/app/organizacoes/aprovacao.tsx`; teste automatizado CT-ORG-11.                                                                                 |
| **Evidência**     | `TelaGerenciarOrganizacoes.carregar`, `OrganizacoesService.listar` e `escaparRegex`.                                                                                                                             |
| **Situação**      | Critério atendido no código e no teste em 26/11/2026; medição do tempo de resposta pendente.                                                                                                                     |

### RNF-ORG-07 — Tamanho dos textos de exemplo

| Campo             | Conteúdo                                                                                                                                                                                                                    |
| ----------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Classificação** | Produto › Facilidade de uso                                                                                                                                                                                                 |
| **Critério**      | O texto de exemplo (placeholder) de cada campo cabe na largura do campo, sem ultrapassar a borda.                                                                                                                           |
| **Medida**        | Os textos de exemplo das telas do módulo têm no máximo 11 caracteres: "Organização", "Cidade", "Descrição", "Nome", "Código", "E-mail" e "Buscar". Nos campos de uma linha montados com `CampoTexto`, o exemplo é desenhado numa linha só e cortado com reticências se faltar espaço. |
| **Condição**      | Menor tela usada nos testes do projeto: 720 × 1280 com 320 dpi (360 × 640 dp), a do emulador Android 7.0.                                                                                                                                                   |
| **Verificação**   | Inspeção do código em 27/11/2026; CT-ORG-29, em 27/11/2026, num emulador Android 14 com a tela forçada para 720 × 1280 e 320 dpi.                                                                                                                                         |
| **Evidência**     | Atributos `placeholder` em `FRONTEND/src/app/organizacoes/*.tsx`, `FRONTEND/src/app/(abas)/organizacoes.tsx` e `FRONTEND/src/app/(abas)/organizacao.tsx`; `numberOfLines={1}` e `ellipsizeMode="tail"` em `FRONTEND/src/components/CampoTexto.tsx`. |
| **Situação**      | Atendido em 27/11/2026: no CT-ORG-29, os textos de exemplo de "Criar organização", "Entrar com código", da folha "Convidar", da edição em "Gerenciar organizações" e das buscas couberam nos campos. Na mesma tela pequena, a linha com a etiqueta do papel e os números do cartão de "Suas organizações" passava da borda; desde o FRONTEND `4f53a25`, ela quebra quando não cabe (RF-ORG-02).                                                                                                                                                       |

### RNF-ORG-08 — Portabilidade

| Campo             | Conteúdo                                                                                                                                                         |
| ----------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Classificação** | Produto › Portabilidade                                                                                                                                          |
| **Critério**      | O módulo funciona do Android 7.0 (API 24, o mínimo do aplicativo) ao Android 16, e no iOS.                                                                       |
| **Medida**        | Os roteiros CT-ORG-01 a CT-ORG-06 terminam sem erro e sem texto cortado.                                                                                         |
| **Condição**      | Aplicativo em celular.                                                                                                                                           |
| **Verificação**   | Roteiros CT-ORG-01 a CT-ORG-06, aprovados pela equipe em ambiente móvel (registro de 23/10/2026, sem o aparelho e a versão do sistema anotados); CT-ORG-30 no iOS. |
| **Evidência**     | Registro dos roteiros em [Testes](7%20-%20testes-cadastro-organizacao.md).                                                                                       |
| **Situação**      | Android aprovado pela equipe, sem registro do aparelho e da versão. iOS não executado por falta de equipamento; validação pendente.                              |

### RNF-ORG-09 — Testabilidade

| Campo             | Conteúdo                                                                                                  |
| ----------------- | --------------------------------------------------------------------------------------------------------- |
| **Classificação** | Organizacional › Entrega                                                                                  |
| **Critério**      | A suíte do backend passa inteira antes de cada entrega, incluindo os testes deste módulo.                 |
| **Medida**        | `npx vitest run` sem nenhuma falha.                                                                       |
| **Condição**      | Código da branch `homolog`, commit `d1f1aeb`, em 27/11/2026.                                              |
| **Verificação**   | Execução registrada em [Testes](7%20-%20testes-cadastro-organizacao.md).                                  |
| **Evidência**     | 17 arquivos e 246 testes aprovados, dos quais 3 arquivos e 74 testes são deste módulo, em 27/11/2026.     |
| **Situação**      | Atendido em 27/11/2026.                                                                                   |

## Rastreabilidade

RNF-ORG-01 a RNF-ORG-09 cobrem RF-ORG-01 a RF-ORG-14 e são verificados pela matriz CT-ORG.
