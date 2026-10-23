# Catálogo da API e regras implementadas

Fonte: `BACKEND`, commit `6f2ffc4`. Todas as rotas, salvo as exceções indicadas, exigem `X-API-Key`. `ApiKeyGuard` compara a chave com `timingSafeEqual`.

## Autenticação e cadastro

| Método e rota                   |              JWT | Entrada                                            | Controller → service                                                                         | Persistência/regra                                                                                      |
| ------------------------------- | ---------------: | -------------------------------------------------- | -------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------- |
| `POST /usuarios`                |              Não | `nome`, `email`, `senha`, `tema?`                  | `UsuariosController.criar` → `UsuariosService.criar` + `AuthService.enviarCodigoVerificacao` | cria `usuarios` não verificado, bcrypt 10 e código de 7 caracteres/30 min; remove conta se envio falhar |
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
| `POST /usuarios/me/email/solicitar` | Sim | valida e-mail diferente/único e envia código ao novo endereço |
| `POST /usuarios/me/email/confirmar` | Sim | valida código e efetiva o novo e-mail                         |
| `PATCH /usuarios/me/senha`          | Sim | compara senha atual, impede repetição e grava bcrypt 10       |
| `POST /auth/recuperar-conta`        | Não | resposta genérica; envia código de 7 caracteres válido 30 min |
| `POST /auth/verificar-recuperacao`  | Não | consome código e retorna token aleatório de 32 bytes          |
| `POST /auth/redefinir-senha`        | Não | consome token, grava senha e limpa recuperação                |

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
| `PATCH /organizacoes/:id/membros/:usuarioId`       | administrador aprovado                | `status=APROVADO                                                                                | REJEITADO`                                                                             | analisa pendente; permite reverter `REJEITADO→APROVADO` |
| `PATCH /organizacoes/:id/membros/:usuarioId/papel` | administrador aprovado                | `papel=ADMIN                                                                                    | MEMBRO`                                                                                | promove/rebaixa; impede organização sem administrador   |
| `DELETE /organizacoes/:id/membros/:usuarioId`      | administrador aprovado                | membro não administrador                                                                        | remove vínculo e o usuário das comissões da organização                                |

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
| `PATCH /comissoes/:id/responsavel`         | `usuario_id`                                             | substitui atomicamente o responsável e rebaixa o anterior                                   |
| `PATCH /comissoes/:id/membros/:usuarioId`  | `papel`                                                  | `RESPONSAVEL` delega à substituição; não permite retirar o único responsável sem substituto |
| `DELETE /comissoes/:id/membros/:usuarioId` | ID                                                       | remove membro comum; responsável precisa ser substituído antes                              |

O schema valida integrantes únicos e exatamente um `RESPONSAVEL`. Alterações de equipe usam compare-and-swap para não sobrescrever atualização concorrente.

## Informações do sistema

Todas exigem JWT e `ADMIN_SISTEMA`.

| Método e rota                 | Origem dos dados             | Resposta/efeito                                                          |
| ----------------------------- | ---------------------------- | ------------------------------------------------------------------------ |
| `GET /sistema/status`         | `node:os` + `countDocuments` | memória, CPU, uptime e contagens de usuários/organizações/comissões      |
| `GET /sistema/relatorios`     | `countDocuments`             | totais/ativos, organizações pendentes/aprovadas e comissões ativas       |
| `POST /sistema/popular-teste` | variáveis `TEST_USER_*`      | cria 3 usuários, 1 organização e 1 comissão; falha se variáveis ausentes |
| `POST /sistema/reset`         | quatro models                | apaga usuários não administradores, organizações, comissões e reuniões   |

## Reuniões, presenças e atas

Todas exigem JWT e `USUARIO`. O service valida participação e papel de responsável.

| Método e rota                              | Regra principal                                                              |
| ------------------------------------------ | ---------------------------------------------------------------------------- |
| `GET /reunioes?de=&ate=&comissao_id=`      | lista reuniões do participante, com filtros opcionais                        |
| `POST /reunioes`                           | somente responsável da comissão ativa; cria participantes e convite          |
| `GET /reunioes/:id`                        | somente participante                                                         |
| `PATCH /reunioes/:id`                      | organizador; implementação aceita apenas cancelamento (`situacao=CANCELADA`) |
| `PATCH /reunioes/:id/convite`              | participante não organizador responde antes do término                       |
| `POST /reunioes/:id/entrada`               | participante não recusado, durante o horário, uma vez                        |
| `POST /reunioes/:id/saida`                 | participante com entrada; limita saída ao término                            |
| `PATCH /reunioes/:id/presencas/:usuarioId` | organizador confirma/recusa presença                                         |
| `GET /reunioes/:id/lista-presenca`         | participante; calcula duração e `SEM_REGISTRO`                               |
| `PATCH /reunioes/:id/ata`                  | responsável salva rascunho versionado                                        |
| `POST /reunioes/:id/ata/publicar`          | responsável publica versão para aprovação                                    |
| `POST /reunioes/:id/ata/aprovar`           | participante aprova a versão atual; service controla quorum/estado           |

## Códigos HTTP usuais

| Código | Significado no projeto                                                   |
| -----: | ------------------------------------------------------------------------ |
|    400 | DTO inválido ou regra de negócio violada                                 |
|    401 | chave da API/JWT ausente ou inválido, conta inativa                      |
|    403 | autenticado, mas tipo/papel/vínculo não autoriza a operação              |
|    404 | recurso ou vínculo não encontrado                                        |
|    409 | duplicidade, estado já alterado ou conflito concorrente                  |
|    503 | configuração obrigatória ausente ou envio inicial de e-mail indisponível |
