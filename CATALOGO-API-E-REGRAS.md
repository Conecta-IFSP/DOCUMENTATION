# Catálogo da API e regras implementadas

Fonte: `BACKEND`, branch `homolog` com as alterações do Incremento 3 (sobre o commit `6f2ffc4`), conferido em 09/11/2026. Todas as rotas, salvo as exceções indicadas, exigem `X-API-Key`. `ApiKeyGuard` compara a chave com `timingSafeEqual`.

## Autenticação e cadastro

| Método e rota                   |              JWT | Entrada                                            | Controller → service                                                                         | Persistência/regra                                                                                      |
| ------------------------------- | ---------------: | -------------------------------------------------- | -------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------- |
| `POST /usuarios`                |              Não | `nome`, `email`, `senha`, `tema?`                  | `UsuariosController.criar` → `UsuariosService.criar` + `AuthService.enviarCodigoVerificacao` | senha ≥8 com maiúscula, minúscula, número e símbolo; cria `usuarios` não verificado, bcrypt 10 e código de 7 caracteres/30 min; responde sem esperar o e-mail e remove a conta se o envio falhar |
| `POST /auth/verificar-cadastro` |              Não | `email`, `codigo`                                  | `verificarCadastro`                                                                          | valida SHA-256/expiração; define `email_verificado=true`                                                |
| `POST /auth/login`              |              Não | `email`, `senha`                                   | `login`                                                                                      | exige conta ativa e e-mail confirmado; retorna JWT de 1 h                                               |
| `POST /auth/google`             |              Não | `id_token` ou `code`, `redirect_uri`, `plataforma` | `loginComGoogle`                                                                             | valida Google; cria ou vincula conta por `google_id`/e-mail; retorna JWT                                |
| `GET /auth/google/callback`     | Não; sem API key | `code`, `state?`                                   | callback móvel                                                                               | redireciona para `conectamais://auth/google`                                                            |
| `GET /auth/google/callback-web` | Não; sem API key | `code`, `state?`                                   | callback web                                                                                 | redireciona para `FRONTEND_URL/login/google-callback`                                                   |

## Configurações de usuário

| Método e rota                       | JWT | Regra principal                                               |
| ----------------------------------- | --: | ------------------------------------------------------------- |
| `GET /usuarios/me`                  | Sim | busca `usuarios._id = JWT.sub`                                |
| `PATCH /usuarios/me`                | Sim | altera `nome`/`tema`; e-mail direto é recusado                |
| `POST /usuarios/me/email/solicitar` | Sim | valida e-mail diferente/único e envia código ao novo endereço, sem esperar o envio |
| `POST /usuarios/me/email/confirmar` | Sim | valida código e efetiva o novo e-mail                         |
| `PATCH /usuarios/me/senha`          | Sim | nova senha ≥8 com maiúscula, minúscula, número e símbolo; compara senha atual, impede repetição, grava bcrypt 10 e `senha_alterada_em`; tokens anteriores passam a receber 401 |
| `POST /auth/recuperar-conta`        | Não | resposta genérica; envia código de 7 caracteres válido 30 min, sem esperar o envio |
| `POST /auth/verificar-recuperacao`  | Não | consome código e retorna token aleatório de 32 bytes          |
| `POST /auth/redefinir-senha`        | Não | nova senha ≥8 com maiúscula, minúscula, número e símbolo; consome token, grava senha e `senha_alterada_em` e limpa recuperação; tokens anteriores passam a receber 401 |

## Administração de usuários

Todas exigem `JwtAuthGuard` e `AdminSistemaGuard`.

| Método e rota          | Operação                                        |
| ---------------------- | ----------------------------------------------- |
| `GET /usuarios`        | lista usuários por `criado_em` decrescente      |
| `GET /usuarios/:id`    | consulta usuário                                |
| `PATCH /usuarios/:id`  | altera campos permitidos por `UpdateUsuarioDto` |
| `DELETE /usuarios/:id` | desativação lógica (`ativo=false`)              |

## Organizações e vínculos

Todas exigem JWT, conta ativa e, no nível global, API key.

| Método e rota                                      | Autorização de domínio                | Entrada/consulta                                                                                | Efeito                                                                                 |
| -------------------------------------------------- | ------------------------------------- | ----------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------- |
| `POST /organizacoes`                               | `USUARIO`                             | `nome` 2–100, `cidade` 2–60, `descricao?` ≤200                                                  | cria `PENDENTE`; criador é `ADMIN/APROVADO`; código único de 10 caracteres fica oculto |
| `GET /organizacoes?busca=`                         | autenticado                           | admin do sistema vê todas; usuário vê somente vínculos; busca nome case-insensitive, termo ≤100 | lista ordenada por `criado_em` desc.                                                   |
| `GET /organizacoes/:id`                            | administrador aprovado da organização | ID                                                                                              | retorna detalhes, membros populados e `codigo_acesso`                                  |
| `PATCH /organizacoes/:id`                          | `ADMIN_SISTEMA`                       | nome/cidade/descrição/status                                                                    | atualiza, aceita transições entre `PENDENTE`, `APROVADA`, `REVOGADA`                   |
| `DELETE /organizacoes/:id`                         | `ADMIN_SISTEMA`                       | organização `REVOGADA`                                                                          | exclusão física e exclusão das comissões; reuniões não são removidas explicitamente    |
| `POST /organizacoes/entrar`                        | `USUARIO`                             | código                                                                                          | cria/reabre solicitação `PENDENTE` somente em organização aprovada                     |
| `POST /organizacoes/:id/codigo`                    | administrador aprovado                | —                                                                                               | troca código de acesso; até 5 tentativas em colisão                                    |
| `POST /organizacoes/:id/convites`                  | administrador aprovado                | `email`                                                                                         | cria vínculo `CONVIDADO` para usuário ativo                                            |
| `PATCH /organizacoes/:id/convite`                  | convidado                             | `aceitar:boolean`                                                                               | aceita como `APROVADO` ou remove convite recusado                                      |
| `PATCH /organizacoes/:id/membros/:usuarioId`       | administrador aprovado                | `status` = `APROVADO` ou `REJEITADO`                                                            | analisa pendente; permite reverter `REJEITADO→APROVADO` |
| `PATCH /organizacoes/:id/membros/:usuarioId/papel` | administrador aprovado                | `papel` = `ADMIN` ou `MEMBRO`                                                                   | promove/rebaixa; impede organização sem administrador   |
| `DELETE /organizacoes/:id/membros/:usuarioId`      | administrador aprovado                | membro não administrador                                                                        | remove vínculo e o usuário das comissões da organização; se ele era o responsável de uma comissão, ela fica sem responsável |

## Comissões

Todas exigem JWT e `USUARIO`. Leitura exige vínculo aprovado; escrita exige administrador aprovado da organização.

| Método e rota                              | Consulta/entrada                                         | Efeito/regra                                                                                |
| ------------------------------------------ | -------------------------------------------------------- | ------------------------------------------------------------------------------------------- |
| `GET /comissoes`                           | `organizacao_id?`                                        | admin vê todas das organizações administradas; membro vê somente as próprias                |
| `GET /comissoes/minhas/:organizacaoId`     | organização                                              | devolve comissões em que o usuário está na equipe e `meu_papel`                             |
| `GET /comissoes/organizacoes`              | —                                                        | organizações aprovadas que o usuário administra                                             |
| `GET /comissoes/organizacoes/:id/membros`  | organização                                              | usuários ativos e aprovados elegíveis                                                       |
| `POST /comissoes`                          | `nome`, `descricao?`, `organizacao_id`, `responsavel_id` | cria ativa com **exatamente um** responsável elegível                                       |
| `GET /comissoes/:id`                       | ID                                                       | admin da organização ou integrante da comissão                                              |
| `PATCH /comissoes/:id`                     | nome/descrição/ativo                                     | edição e ativação; ativação exige exatamente um responsável                                 |
| `DELETE /comissoes/:id`                    | ID                                                       | desativação lógica (`ativo=false`)                                                          |
| `POST /comissoes/:id/membros`              | `usuario_id`, papel opcional                             | adiciona somente `MEMBRO`; responsável usa rota específica                                  |
| `PATCH /comissoes/:id/responsavel`         | `usuario_id`                                             | substitui atomicamente o responsável e rebaixa o anterior; aceita também comissão inativa |
| `PATCH /comissoes/:id/membros/:usuarioId`  | `papel`                                                  | `RESPONSAVEL` delega à substituição; não permite retirar o único responsável sem substituto |
| `DELETE /comissoes/:id/membros/:usuarioId` | ID                                                       | remove membro comum; responsável precisa ser substituído antes                              |

O schema valida integrantes únicos e exatamente um `RESPONSAVEL`. Alterações de equipe usam compare-and-swap para não sobrescrever atualização concorrente.

## Informações do sistema

Todas exigem `X-API-Key`, JWT e `ADMIN_SISTEMA`.

| Método e rota                 | Origem dos dados             | Resposta/efeito                                                          |
| ----------------------------- | ---------------------------- | ------------------------------------------------------------------------ |
| `GET /sistema/status`         | `node:os` + `countDocuments` | memória, CPU, tempo ligado do computador (`os.uptime()`, não da API) e contagens de usuários/organizações/comissões |
| `GET /sistema/relatorios`     | `countDocuments`             | totais/ativos, organizações pendentes/aprovadas e comissões ativas       |
| `POST /sistema/popular-teste` | variáveis `TEST_USER_*`      | cria 3 usuários, 1 organização e 1 comissão e devolve nome, e-mail e cenário de cada usuário, sem a senha; falha se variáveis ausentes |
| `POST /sistema/reset`         | quatro models + GridFS       | apaga usuários não administradores, organizações, comissões, reuniões, os anexos guardados em `anexos.files`/`anexos.chunks` e os contadores de numeração das atas; não apaga `notificacoes` |

## Reuniões, presenças e anexos

Todas exigem JWT e `USUARIO`. O service valida participação e papel de responsável.

| Método e rota                              | Regra principal                                                                                   |
| ------------------------------------------ | --------------------------------------------------------------------------------------------------- |
| `GET /reunioes?de=&ate=&comissao_id=`      | lista reuniões do participante, com filtros opcionais                                             |
| `POST /reunioes`                           | somente responsável da comissão ativa; cria participantes e convite; `local` é opcional e `dia_inteiro` marca a reunião de dia inteiro |
| `GET /reunioes/:id`                        | somente participante                                                                               |
| `PATCH /reunioes/:id`                      | organizador; cancela (`situacao=CANCELADA`) ou edita título, local, data, horário, pauta e convidados |
| `PATCH /reunioes/:id/convite`              | participante não organizador responde antes do término                                            |
| `POST /reunioes/:id/entrada`               | participante não recusado, durante o horário, uma vez                                             |
| `POST /reunioes/:id/saida`                 | participante com entrada; limita saída ao término                                                 |
| `PATCH /reunioes/:id/presencas/:usuarioId` | organizador confirma/recusa presença                                                              |
| `GET /reunioes/:id/lista-presenca`         | participante; calcula duração e `SEM_REGISTRO`                                                    |
| `POST /reunioes/:id/anexos`                | organizador envia arquivo de até 10 MB para a pauta, no máximo 10 por reunião (GridFS)            |
| `GET /reunioes/:id/anexos/:anexoId`        | participante baixa o anexo                                                                         |
| `DELETE /reunioes/:id/anexos/:anexoId`     | organizador remove o anexo e o arquivo no GridFS                                                  |

## Atas

Documentadas em [Incremento 3 — Atas](4%20-%20Incremento3/atas/README.md). A ata só existe depois do término da reunião.

| Método e rota                     | Regra principal                                                                                       |
| --------------------------------- | ------------------------------------------------------------------------------------------------------- |
| `GET /reunioes/atas/pendentes`    | lista as atas que esperam avaliação do usuário e as atas dele que foram reprovadas                   |
| `PATCH /reunioes/:id/ata`         | responsável salva rascunho versionado                                                                  |
| `POST /reunioes/:id/ata/publicar` | responsável envia a versão para aprovação, com prazo de 24, 72 ou 168 horas; congela os presentes; no fim do prazo, a rotina do servidor (a cada 60 s) registra a aprovação automática de quem não respondeu e conclui a ata |
| `POST /reunioes/:id/ata/aprovar`  | presente aprova a versão atual antes do fim do prazo; com todas as aprovações a ata é numerada por comissão e ano |
| `POST /reunioes/:id/ata/reprovar` | presente reprova a versão atual antes do fim do prazo, com trecho e justificativa opcionais; a ata volta para o responsável |
| `POST /reunioes/:id/ata/nova`     | responsável cancela a ata reprovada e abre a versão seguinte com o mesmo texto                        |

## Presença por link

Documentada em [Incremento 2 — Reuniões](3%20-%20Incremento2/reunioes/README.md), RF-REU-17 a RF-REU-22. Todas exigem `X-API-Key`, JWT e `USUARIO`.

| Método e rota | Quem usa | Regra principal |
| --- | --- | --- |
| `GET /presencas/:id/gestao` | responsável da comissão | informa se há link ativo, a revisão atual das presenças e o histórico de correções (RF-REU-22) |
| `POST /presencas/:id/link` | responsável da comissão | gera o link de presença com código durante a reunião (RF-REU-17) |
| `DELETE /presencas/:id/link` | responsável da comissão | invalida o link ativo e expira as notificações de presença da reunião (RF-REU-18) |
| `POST /presencas/:id/enviar` | responsável da comissão | envia o link aos convidados por notificação no aplicativo ou por e-mail; o e-mail exige SMTP configurado (RF-REU-19) |
| `GET /presencas/:id/acesso?codigo=` | participante convidado | consulta a reunião pelo código sem registrar presença (RF-REU-20) |
| `POST /presencas/:id/acesso` | participante convidado | registra a entrada ou a saída do próprio participante pelo código (RF-REU-20) |
| `PATCH /presencas/:id/horarios/:usuarioId` | responsável da comissão | corrige entrada e saída com motivo de 5 a 500 caracteres, guarda os valores anteriores no histórico e exige a revisão atual das presenças (RF-REU-21) |
| `GET /presencas/:id/historico` | responsável da comissão | lista as correções de horário registradas (RF-REU-22) |

## Notificações

Exigem `X-API-Key`, JWT e `USUARIO`.

| Método e rota | Regra principal |
| --- | --- |
| `GET /notificacoes` | gera, sem repetir, as notificações de convite pendente, cancelamento, mudança de horário e ata a avaliar das reuniões do usuário; devolve até 100 notificações não expiradas, da mais recente para a mais antiga, e a quantidade de não lidas |
| `PATCH /notificacoes/:id/lida` | marca como lida uma notificação do próprio usuário; de outro usuário ou inexistente: 404 "Notificação não encontrada" |

## Relatórios

Documentado em [Incremento 3 — Relatório individual](4%20-%20Incremento3/relatorio-individual/README.md). Exige JWT e `USUARIO`. O usuário vem só do token; a rota não aceita o identificador de outra pessoa.

| Método e rota | Entrada | Controller → service | Resposta/regra |
| --- | --- | --- | --- |
| `GET /relatorios/individual?de=&ate=` | `de` e `ate` opcionais, datas ISO 8601; `ate` é exclusivo e precisa ser posterior a `de` | `RelatoriosController.individual` → `RelatoriosService.individual` | `usuario` (nome e e-mail), `gerada_em`, `periodo`, `resumo` (reuniões, minutos registrados e registros sem duração), `vinculos` e `reunioes`. `reunioes` traz as reuniões não canceladas, já iniciadas, em que a presença do usuário é `CONFIRMADA`, da mais recente para a mais antiga. Cada item de `reunioes` traz `saida_automatica`, verdadeiro quando a saída foi registrada no término da reunião. `vinculos.organizacoes` traz `id`, `nome` e `papel` (`ADMIN` ou `MEMBRO`) das organizações `APROVADA` em que o vínculo do usuário é `APROVADO`; `vinculos.comissoes` traz `id`, `nome`, `organizacao_id`, `organizacao` e `papel` (`RESPONSAVEL` ou `MEMBRO`) das comissões ativas dessas organizações em que o usuário está na equipe; as duas listas são ordenadas por nome e, no empate, por `_id`, e não dependem do período. Datas inválidas: 400 "Informe uma data inicial válida" ou "Informe uma data final válida"; `ate` não posterior a `de`: 400 "O fim do período deve ser posterior ao início"; conta inativa: 401 |

## Votações

Documentadas em [Incremento 3 — Votação](4%20-%20Incremento3/votacao/README.md). As votações ficam em `reunioes.votacoes[]` e são devolvidas em toda resposta de reunião, sem a lista de votos.

| Método e rota | Regra principal |
| --- | --- |
| `POST /reunioes/:id/votacoes` | responsável da comissão abre a votação com a reunião em andamento; de 2 a 10 opções sem repetição |
| `POST /reunioes/:id/votacoes/:votacaoId/votos` | participante com entrada não recusada e sem saída vota uma única vez, antes do fim da reunião |
| `POST /reunioes/:id/votacoes/:votacaoId/encerrar` | responsável da comissão encerra a votação aberta; o resultado por opção passa a ser devolvido |

## Códigos HTTP usuais

| Código | Significado no projeto                                                   |
| -----: | ------------------------------------------------------------------------ |
|    400 | DTO inválido ou regra de negócio violada                                 |
|    401 | chave da API/JWT ausente ou inválido, conta inativa ou token emitido antes da última troca de senha |
|    403 | autenticado, mas tipo/papel/vínculo não autoriza a operação              |
|    404 | recurso ou vínculo não encontrado                                        |
|    409 | duplicidade, estado já alterado ou conflito concorrente                  |
|    503 | configuração obrigatória ausente, falha ao gravar o código do cadastro ou envio de e-mail de presença indisponível |
