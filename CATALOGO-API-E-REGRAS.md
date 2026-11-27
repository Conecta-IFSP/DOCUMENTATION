# Catálogo da API e regras implementadas

Fonte: `BACKEND`, branch `main`, conferido em 27/11/2026, na versão com o voto em branco e nulo. Todas as rotas, salvo as exceções indicadas, exigem `X-API-Key`. `ApiKeyGuard` é global (`APP_GUARD`), compara a chave com `timingSafeEqual` e só dispensa a chave nas requisições `OPTIONS` e nos dois callbacks do Google; sem `API_ACCESS_TOKEN` no servidor, responde 503. O `ValidationPipe` global descarta campos que não estão no DTO (`whitelist`). Não há limite de requisições por tempo.

## Autenticação e cadastro

| Método e rota                   |              JWT | Entrada                                            | Controller → service                                                                         | Persistência/regra                                                                                      |
| ------------------------------- | ---------------: | -------------------------------------------------- | -------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------- |
| `POST /usuarios`                |              Não | `nome`, `email`, `senha`, `tema?`                  | `UsuariosController.criar` → `UsuariosService.criar` + `AuthService.enviarCodigoVerificacao` | `nome` ≥2; senha ≥8 com maiúscula, minúscula, número e símbolo; e-mail já cadastrado: 409; cria `usuarios` não verificado, bcrypt 10 e código de 7 caracteres/30 min; responde sem esperar o e-mail e remove a conta se o envio falhar |
| `POST /auth/verificar-cadastro` |              Não | `email`, `codigo` no formato `XXX-XXXX`            | `verificarCadastro`                                                                          | valida SHA-256/expiração; define `email_verificado=true`; código errado ou vencido: 400                 |
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
| `PATCH /usuarios/:id`  | altera `nome`, `email` (único), `senha` (mesma regra do cadastro), `tipo`, `tema` e `ativo`; a senha trocada por aqui não grava `senha_alterada_em`, então os tokens já emitidos continuam válidos |
| `DELETE /usuarios/:id` | desativação lógica (`ativo=false`)              |

## Organizações e vínculos

Todas exigem JWT, conta ativa e, no nível global, API key. Análise de solicitação, troca de papel, remoção de membro e convite exigem a organização `APROVADA`.

| Método e rota                                      | Autorização de domínio                | Entrada/consulta                                                                                | Efeito                                                                                 |
| -------------------------------------------------- | ------------------------------------- | ----------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------- |
| `POST /organizacoes`                               | `USUARIO`                             | `nome` 2–100, `cidade` 2–60, `descricao?` ≤200                                                  | cria `PENDENTE`; nome repetido, sem diferenciar maiúsculas: 409; criador é `ADMIN/APROVADO`; código único de 10 caracteres fica oculto, com até 5 tentativas em colisão |
| `GET /organizacoes?busca=`                         | autenticado                           | admin do sistema vê todas; usuário vê as organizações em que tem vínculo, em qualquer situação, com `meu_vinculo`; busca no nome sem diferenciar maiúsculas, com os 100 primeiros caracteres do termo | lista ordenada por `criado_em` desc.                                                   |
| `GET /organizacoes/:id`                            | administrador aprovado da organização | ID                                                                                              | retorna detalhes, membros populados e `codigo_acesso`                                  |
| `PATCH /organizacoes/:id`                          | `ADMIN_SISTEMA`                       | nome/cidade/descrição/status                                                                    | atualiza, aceita transições entre `PENDENTE`, `APROVADA`, `REVOGADA`; nome repetido: 409 |
| `DELETE /organizacoes/:id`                         | `ADMIN_SISTEMA`                       | organização `REVOGADA`                                                                          | exclusão física e exclusão das comissões; reuniões não são removidas explicitamente    |
| `POST /organizacoes/entrar`                        | `USUARIO`                             | `codigo` até 20 caracteres; hífens e espaços são ignorados                                      | cria/reabre solicitação `PENDENTE` somente em organização aprovada; código inexistente: 404; já membro, pendente ou convidado: 409 |
| `POST /organizacoes/:id/codigo`                    | administrador aprovado                | —                                                                                               | troca código de acesso; até 5 tentativas em colisão                                    |
| `POST /organizacoes/:id/convites`                  | administrador aprovado                | `email`                                                                                         | cria vínculo `CONVIDADO` para usuário ativo que não é administrador do sistema, ou convida de novo quem foi rejeitado; membro, pendente ou já convidado: 409 |
| `PATCH /organizacoes/:id/convite`                  | convidado                             | `aceitar:boolean`                                                                               | aceita como `APROVADO` ou remove convite recusado                                      |
| `PATCH /organizacoes/:id/membros/:usuarioId`       | administrador aprovado                | `status` = `APROVADO` ou `REJEITADO`                                                            | analisa pendente; permite reverter `REJEITADO→APROVADO` |
| `PATCH /organizacoes/:id/membros/:usuarioId/papel` | administrador aprovado                | `papel` = `ADMIN` ou `MEMBRO`                                                                   | promove/rebaixa; impede organização sem administrador   |
| `DELETE /organizacoes/:id/membros/:usuarioId`      | administrador aprovado                | membro não administrador; solicitação pendente precisa ser analisada antes (409)                | remove vínculo e o usuário das comissões da organização; se ele era o responsável de uma comissão, ela fica sem responsável |

## Comissões

Todas exigem JWT e `USUARIO`. Leitura exige vínculo aprovado; escrita exige administrador aprovado da organização. Nos dois casos a organização precisa estar `APROVADA`. Listas por `criado_em` decrescente, exceto as de organizações, membros e `minhas`, por nome.

| Método e rota                              | Consulta/entrada                                         | Efeito/regra                                                                                |
| ------------------------------------------ | -------------------------------------------------------- | ------------------------------------------------------------------------------------------- |
| `GET /comissoes`                           | `organizacao_id?`                                        | admin vê todas das organizações administradas; membro vê somente as próprias                |
| `GET /comissoes/minhas/:organizacaoId`     | organização                                              | devolve comissões em que o usuário está na equipe e `meu_papel`                             |
| `GET /comissoes/organizacoes`              | —                                                        | organizações aprovadas que o usuário administra                                             |
| `GET /comissoes/organizacoes/:id/membros`  | organização                                              | usuários ativos e aprovados elegíveis                                                       |
| `POST /comissoes`                          | `nome` 2–100, `descricao?` ≤1000, `organizacao_id`, `responsavel_id` | cria ativa com **exatamente um** responsável elegível                                       |
| `GET /comissoes/:id`                       | ID                                                       | admin da organização ou integrante da comissão                                              |
| `PATCH /comissoes/:id`                     | `nome` 2–100, `descricao` ≤1000, `ativo`                 | edição e ativação; ativação exige exatamente um responsável                                 |
| `DELETE /comissoes/:id`                    | ID                                                       | desativação lógica (`ativo=false`)                                                          |
| `POST /comissoes/:id/membros`              | `usuario_id`, papel opcional                             | adiciona somente `MEMBRO` em comissão ativa; responsável usa rota específica                |
| `PATCH /comissoes/:id/responsavel`         | `usuario_id`                                             | substitui atomicamente o responsável e rebaixa o anterior; aceita também comissão inativa |
| `PATCH /comissoes/:id/membros/:usuarioId`  | `papel`                                                  | `RESPONSAVEL` delega à substituição; não permite retirar o único responsável sem substituto |
| `DELETE /comissoes/:id/membros/:usuarioId` | ID                                                       | remove membro comum; responsável precisa ser substituído antes                              |

O schema valida integrantes únicos e exatamente um `RESPONSAVEL`. Alterações de equipe usam compare-and-swap para não sobrescrever atualização concorrente.

## Informações do sistema

Todas exigem `X-API-Key`, JWT e `ADMIN_SISTEMA`.

| Método e rota                 | Origem dos dados                                                       | Resposta/efeito                                                          |
| ----------------------------- | ---------------------------------------------------------------------- | ------------------------------------------------------------------------ |
| `GET /sistema/sobre`          | `package.json` da API, `process`, `node:os` e comando `buildInfo`      | nome "Conecta+", versão da API, ambiente (`NODE_ENV` ou `development`), início e fuso horário da API; sete tecnologias (Node.js, TypeScript, NestJS, MongoDB, Mongoose, bcrypt e class-validator), com a versão declarada no `package.json` sem `^`/`~` (Node.js e MongoDB com a versão em execução); sistema operacional, arquitetura, processador, núcleos e memória total do servidor |
| `GET /sistema/status`         | `node:os`, `process.uptime()`, comandos `ping` e `dbStats` + `countDocuments` | memória total, livre e em uso (%), núcleos e CPU em uso em porcentagem (medida em 200 ms), tempo ativo do computador (`os.uptime()`) e da API (`process.uptime()`); banco com `conectado`, latência do `ping`, armazenamento em MB e contagens de usuários/organizações/comissões/reuniões, ou só `conectado: false` se uma dessas consultas falhar |
| `GET /sistema/relatorios`     | `countDocuments`                                                       | usuários total/ativos, organizações total/pendentes/aprovadas, comissões total/ativas, reuniões total/agendadas/canceladas e atas total/em aprovação/aprovadas, pela `ata.situacao` atual de cada reunião |
| `POST /sistema/popular-teste` | variáveis `TEST_USERS_PASSWORD` e `TEST_USER_*`                        | cria 3 usuários, 1 organização `APROVADA` e 1 comissão e devolve nome, e-mail e cenário de cada usuário, sem a senha; variável ausente: 400 com os nomes que faltam |
| `POST /sistema/reset`         | `ResetarSistemaDto` + quatro models + GridFS                           | corpo `{ "senha": "<senha do administrador>", "confirmacao": "RESET" }`; outra `confirmacao`: 400 "Digite RESET para confirmar"; `senha` ausente ou que não é texto: 400 "Informe a senha do administrador"; `confirmarSenhaAdmin` compara a senha com o `senha_hash` de quem pede, por bcrypt: 400 "Senha incorreta" ou, sem senha cadastrada, 400 "Defina uma senha na sua conta para resetar o sistema", sem apagar nada; com a senha certa, apaga usuários não administradores, organizações, comissões, reuniões, os anexos guardados em `anexos.files`/`anexos.chunks`, os contadores de numeração das atas e as notificações, sem transação, e devolve 201 `{ status: "sistema_resetado", resetado_em }` |

## Reuniões, presenças e anexos

Todas exigem JWT e `USUARIO`. O service valida participação e papel de responsável. Organizador é quem agendou a reunião (`criada_por`).

| Método e rota                              | Regra principal                                                                                   |
| ------------------------------------------ | --------------------------------------------------------------------------------------------------- |
| `GET /reunioes?de=&ate=&comissao_id=`      | lista as reuniões em que o usuário é participante, por `inicio` crescente; `de` traz as que terminam a partir da data, `ate` as que começam até a data e `comissao_id` filtra a comissão |
| `POST /reunioes`                           | somente responsável da comissão ativa; `titulo` 3–120, `local` opcional 2–120, `pauta` ≤2000, `inicio` e `fim` ISO 8601, com término depois do início e ainda não passado; `dia_inteiro` marca a reunião de dia inteiro; `convidados_ids` só com integrantes da comissão (sem o campo, convida a equipe toda; lista vazia, só o organizador); o organizador fica com convite `ACEITO` e os demais `PENDENTE` |
| `GET /reunioes/:id`                        | somente participante                                                                               |
| `PATCH /reunioes/:id`                      | organizador, antes do término; cancela (`situacao=CANCELADA`) ou edita título, local, data, horário, dia inteiro, pauta e convidados, nunca as duas coisas juntas; data e horário não mudam depois do início, e mudar o horário volta os convites para `PENDENTE` |
| `PATCH /reunioes/:id/convite`              | participante não organizador responde (`aceito`) antes do término; não recusa depois de marcar entrada |
| `POST /reunioes/:id/entrada`               | participante não recusado, durante o horário, uma vez; a entrada do organizador já fica `CONFIRMADA` e a dos demais `AGUARDANDO`; fechada depois do envio da ata |
| `POST /reunioes/:id/saida`                 | participante com entrada; limita saída ao término; fechada depois do envio da ata                 |
| `PATCH /reunioes/:id/presencas/:usuarioId` | organizador, depois do início, confirma ou recusa (`situacao`) a presença; confirmar quem não marcou entrada registra a entrada no início da reunião; também desmarca uma presença confirmada: `RECUSADA` sobre uma presença `CONFIRMADA` troca só a situação e mantém a entrada e a saída; recusar quem não tem presença responde "Esta pessoa não marcou entrada"; exige quem agendou a reunião, reunião não cancelada e alvo que participa dela; fechada depois do envio da ata (aceita sem ata ou com a ata em rascunho) |
| `GET /reunioes/:id/lista-presenca`         | participante; calcula duração e `SEM_REGISTRO`                                                    |
| `POST /reunioes/:id/anexos`                | organizador, antes do término, envia o campo `arquivo` (PDF, JPEG, PNG, texto, CSV ou documento do Office ou do LibreOffice) de até 10 MB para a pauta, no máximo 10 por reunião (GridFS); maior que 10 MB: 413 |
| `GET /reunioes/:id/anexos/:anexoId`        | participante baixa o anexo                                                                         |
| `DELETE /reunioes/:id/anexos/:anexoId`     | organizador, antes do término, remove o anexo e o arquivo no GridFS                               |

## Atas

Documentadas em [Incremento 3 — Atas](4%20-%20Incremento%203/atas/README.md). A ata só existe depois do término da reunião.

| Método e rota                     | Regra principal                                                                                       |
| --------------------------------- | ------------------------------------------------------------------------------------------------------- |
| `GET /reunioes/atas/pendentes`    | lista as atas que esperam avaliação do usuário e as atas dele que foram reprovadas                   |
| `PATCH /reunioes/:id/ata`         | responsável da comissão salva o rascunho (`conteudo` de 20 a 20000 caracteres e `versao` atual); ata enviada ou reprovada não é editada; versão desatualizada: 409 |
| `POST /reunioes/:id/ata/publicar` | responsável envia a versão para aprovação, com `prazo_horas` de 24, 72 ou 168; congela os presentes e exige ao menos uma presença confirmada; quem envia e esteve presente já conta como aprovação; no fim do prazo, a rotina do servidor (a cada 60 s) registra a aprovação automática de quem não respondeu e conclui a ata |
| `POST /reunioes/:id/ata/aprovar`  | presente aprova a versão atual antes do fim do prazo; com todas as aprovações a ata é numerada por comissão e ano |
| `POST /reunioes/:id/ata/reprovar` | presente reprova a versão atual antes do fim do prazo, com `trecho` (precisa estar no texto da ata) e `justificativa` (até 500 caracteres) opcionais; a ata volta para o responsável |
| `POST /reunioes/:id/ata/nova`     | responsável cancela a ata reprovada e abre a versão seguinte com o mesmo texto                        |

## Presença por link

Documentada em [Incremento 2 — Reuniões](3%20-%20Incremento%202/reunioes/README.md), RF-REU-17 a RF-REU-22. Todas exigem `X-API-Key`, JWT e `USUARIO`.

| Método e rota | Quem usa | Regra principal |
| --- | --- | --- |
| `GET /presencas/:id/gestao` | responsável da comissão | informa se há link ativo, o início e o fim, os participantes e a revisão atual das presenças (RF-REU-22) |
| `POST /presencas/:id/link` | responsável da comissão | gera o link com código de 64 caracteres hexadecimais, válido até o término; pode ser gerado antes do início; um novo link substitui o anterior e expira as notificações de presença; sem `FRONTEND_URL` http(s): 503 (RF-REU-17) |
| `DELETE /presencas/:id/link` | responsável da comissão | invalida o link ativo e expira as notificações de presença da reunião (RF-REU-18) |
| `POST /presencas/:id/enviar` | responsável da comissão | corpo `codigo` e `canal` (`APP` ou `EMAIL`); envia o link aos convidados que não recusaram, por notificação no aplicativo ou por e-mail; o e-mail exige SMTP configurado (503) e devolve quantos foram enviados (RF-REU-19) |
| `GET /presencas/:id/acesso?codigo=` | participante convidado | consulta a reunião pelo código sem registrar presença (RF-REU-20) |
| `POST /presencas/:id/acesso` | participante convidado | corpo `codigo` e `acao` (`ENTRADA` ou `SAIDA`); com convite `ACEITO` e durante a reunião, registra a entrada já `CONFIRMADA` ou a saída do próprio participante (RF-REU-20) |
| `PATCH /presencas/:id/horarios/:usuarioId` | responsável da comissão | corrige entrada e saída com motivo de 5 a 500 caracteres, horários já ocorridos dentro da reunião e saída depois da entrada; a presença fica `CONFIRMADA`, os valores anteriores vão para o histórico e a revisão desatualizada recebe 409 (RF-REU-21) |
| `GET /presencas/:id/historico` | responsável da comissão | lista as correções de horário registradas (RF-REU-22) |

## Notificações

Exigem `X-API-Key`, JWT e `USUARIO`.

| Método e rota | Regra principal |
| --- | --- |
| `GET /notificacoes` | gera, sem repetir, as notificações de convite pendente, cancelamento, mudança de horário e ata a avaliar das reuniões do usuário; devolve até 100 notificações não expiradas, da mais recente para a mais antiga, e a quantidade de não lidas |
| `PATCH /notificacoes/:id/lida` | marca como lida uma notificação do próprio usuário; de outro usuário ou inexistente: 404 "Notificação não encontrada" |

## Relatórios

Documentado em [Incremento 3 — Relatório individual](4%20-%20Incremento%203/relatorio-individual/README.md). Exige JWT e `USUARIO`. O usuário vem só do token; a rota não aceita o identificador de outra pessoa.

| Método e rota | Entrada | Controller → service | Resposta/regra |
| --- | --- | --- | --- |
| `GET /relatorios/individual?de=&ate=` | `de` e `ate` opcionais, datas ISO 8601 comparadas com o `inicio` da reunião; `ate` é exclusivo e precisa ser posterior a `de` | `RelatoriosController.individual` → `RelatoriosService.individual` | `usuario` (nome e e-mail), `gerada_em`, `periodo`, `resumo` (reuniões, minutos registrados e registros sem duração), `vinculos` e `reunioes`. `reunioes` traz as reuniões não canceladas, já iniciadas, em que a presença do usuário é `CONFIRMADA`, da mais recente para a mais antiga. Cada item de `reunioes` traz `saida_automatica`, verdadeiro quando a saída foi registrada no término da reunião. `vinculos.organizacoes` traz `id`, `nome` e `papel` (`ADMIN` ou `MEMBRO`) das organizações `APROVADA` em que o vínculo do usuário é `APROVADO`; `vinculos.comissoes` traz `id`, `nome`, `organizacao_id`, `organizacao` e `papel` (`RESPONSAVEL` ou `MEMBRO`) das comissões ativas dessas organizações em que o usuário está na equipe; as duas listas são ordenadas por nome e, no empate, por `_id`, e não dependem do período. Datas inválidas: 400 "Informe uma data inicial válida" ou "Informe uma data final válida"; `ate` não posterior a `de`: 400 "O fim do período deve ser posterior ao início"; conta inativa: 401 |

## Votações

Documentadas em [Incremento 3 — Votação](4%20-%20Incremento%203/votacao/README.md). As votações ficam em `reunioes.votacoes[]` e são devolvidas em toda resposta de reunião, sem a lista de votos.

| Método e rota | Corpo | Regra principal |
| --- | --- | --- |
| `POST /reunioes/:id/votacoes` | `titulo`, `descricao` (opcional), `opcoes` e `modo` (opcional: `FECHADA`, o padrão, ou `ABERTA`) | responsável da comissão ativa abre a votação com a reunião em andamento; `titulo` de 3 a 200 caracteres; de 2 a 10 opções de até 80 caracteres, sem repetição; outro `modo`: 400 "Escolha se a votação é aberta ou fechada"; o `modo` não muda depois |
| `POST /reunioes/:id/votacoes/:votacaoId/votos` | `tipo` (opcional: `OPCAO`, o padrão, `BRANCO` ou `NULO`) e `opcao` (inteiro a partir de 0, exigido só no tipo `OPCAO`) | participante com entrada não recusada e sem saída vota uma única vez, antes do fim da reunião; com `BRANCO` ou `NULO`, `opcao` é ignorada e não é gravada; outro `tipo`: 400 "Tipo de voto inválido"; `opcao` ausente, não inteira ou negativa no tipo `OPCAO`: 400 "Escolha uma opção de voto"; `opcao` fora da lista: 400 "Opção de voto inválida" |
| `POST /reunioes/:id/votacoes/:votacaoId/encerrar` | nenhum | responsável da comissão encerra a votação aberta; o resultado por opção, os brancos, os nulos e, no `modo` `ABERTA`, a lista nominal passam a ser devolvidos |

Cada item de `votacoes` na resposta traz `_id`, `titulo`, `descricao`, `opcoes`, `modo`, `situacao` (`ABERTA` ou `ENCERRADA`; a votação de reunião terminada é devolvida como `ENCERRADA`), `aberta_em`, `encerrada_em`, `total_votos` (inclui brancos e nulos), `meu_voto` (posição da opção, `BRANCO`, `NULO` ou `null`), `resultado` (votos por opção, sem brancos e nulos), `brancos`, `nulos` e `votos_nominais`. Enquanto a votação não está encerrada, `resultado`, `brancos`, `nulos` e `votos_nominais` são `null`. `votos_nominais` só é preenchido com a votação encerrada e `modo` `ABERTA`: uma lista de `{ usuario_id, nome, voto }` em ordem alfabética de nome, com "Usuário indisponível" quando o votante não está mais entre os participantes; no `modo` `FECHADA`, é sempre `null`. Votações gravadas sem `modo` são devolvidas como `FECHADA`, e votos gravados sem `tipo` contam como voto em opção.

## Códigos HTTP usuais

| Código | Significado no projeto                                                   |
| -----: | ------------------------------------------------------------------------ |
|    400 | DTO inválido ou regra de negócio violada                                 |
|    401 | chave da API/JWT ausente ou inválido, conta inativa ou token emitido antes da última troca de senha |
|    403 | autenticado, mas tipo/papel/vínculo não autoriza a operação              |
|    404 | recurso ou vínculo não encontrado                                        |
|    409 | duplicidade, estado já alterado ou conflito concorrente                  |
|    413 | anexo maior que 10 MB                                                    |
|    503 | configuração obrigatória ausente, falha ao gravar o código do cadastro ou envio de e-mail de presença indisponível |
