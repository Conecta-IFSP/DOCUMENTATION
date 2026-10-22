# Requisitos funcionais — comissões e complementos

## Contrato comum

URL-base: variável `EXPO_PUBLIC_API_URL` no app; desenvolvimento usual `http://localhost:3000`. Não há prefixo `/api` no servidor. Os caminhos abaixo são relativos a essa base, não apontam para uma API pública externa.

Todos os endpoints usam `JwtAuthGuard.canActivate(context)` e `UsuarioComumGuard.canActivate(context)`. O primeiro extrai `Authorization: Bearer <JWT>`, verifica assinatura/expiração via JwtService.verifyAsync, consulta usuarios por `_id=sub, ativo=true` e atualiza tipo/e-mail a partir do banco. O segundo impede ADMIN_SISTEMA nestes módulos. O identificador do solicitante vem de `req.usuario.sub`, nunca de um campo de autor enviado no JSON. ADMIN da organização e RESPONSAVEL da comissão são papéis diferentes.

`ValidationPipe({whitelist:true,transform:true})` transforma DTOs e remove campos não declarados; não rejeita todos os campos extras. Respostas normais do Nest: GET/PATCH/DELETE 200, POST 201. Erros: 400 dados/regra inválida; 401 token ausente/inválido/expirado ou conta inativa; 403 falta de autorização; 404 recurso inexistente; 409 conflito. Corpo de erro contém `message` (string ou array), `statusCode` e usualmente `error`. Falha de rede é tratada pelo helper api do app; nenhum sucesso é exibido antes da resposta.

Objeto de comissão retornado: `_id,nome,descricao?,organizacao_id,membros,ativo,criado_em,atualizado_em`. Organização populada: `_id,nome,status`; cada membro: `usuario_id` populado com `_id,nome,email,ativo`, `papel,adicionado_em`. Não inclui hash de senha. IDs são serializados como strings; datas como ISO-8601. Referências apagadas externamente podem resultar em null. O módulo não implementa exclusão física.

Objeto de reunião: campos do dicionário de Reuniao, com `comissao_id` populado (`_id,nome,organizacao_id`), `participantes.usuario_id` populado (`_id,nome,email`), `sou_dono` calculado pela comparação com criada_por e `ata` opcional. Campos ata.autor_id/aprovadores/aprovacoes permanecem IDs.

Formulário adotado: os nove campos do exemplo informado pela aluna. O ator é explicitado em Origem. Não foi disponibilizado formulário oficial adicional pelo professor.

## RF-COM-01 — Consultar comissões

**Descrição:** Permitir consultar comissões.

**Entradas:** Filtro opcional organizacao_id (ObjectId); texto e situação Todas/Ativas/Inativas na tela.

**Origem:** Integrante aprovado da organização ou seu administrador.

**Saída:** Resultado descrito na pós-condição ou erro do contrato comum.

**Destino:** Tela de comissões/equipe para COM; tela Ata e lista de presença para ATA/PRE.

**Ação:** `GET /comissoes?organizacao_id=<id>` → `listar(usuarioId, organizacaoId?)`. ComissoesService.listar consulta organizacoes para identificar vínculos. Administrador vê todas da organização; outros veem apenas documentos de comissoes com seu ID em membros.usuario_id. consultar executa find(filtro), populate organizacao_id (nome,status) e membros.usuario_id (nome,email,ativo), sort criado_em decrescente. A tela filtra nome da comissão ou organização sem diferenciar maiúsculas e mostra oito resultados por página. A paginação é local: a API não recebe page/limit.

**Pré-condição:** Usuário ativo, organização APROVADA e vínculo APROVADO.

**Pós-condição:** Lista autorizada exibida, inclusive inativas conforme filtro.

**Efeitos colaterais:** Nenhuma gravação. Sem resultados: lista vazia. Filtro de organização sem permissão: 403.

## RF-COM-02 — Consultar detalhes da comissão

**Descrição:** Permitir consultar detalhes da comissão.

**Entradas:** id da comissão.

**Origem:** Integrante da comissão ou administrador da organização.

**Saída:** Resultado descrito na pós-condição ou erro do contrato comum.

**Destino:** Tela de comissões/equipe para COM; tela Ata e lista de presença para ATA/PRE.

**Ação:** `GET /comissoes/:id` → `buscarPorId(id, usuarioId)`. buscarPorId lê comissoes por _id, autorizarLeitura verifica organizacoes.status e membros.status; verifica administra ou naEquipe. popular resolve organização e integrantes com as projeções de COM-01. A tela divide equipe e reuniões em abas; equipe possui busca e páginas de oito.

**Pré-condição:** Mesmas condições de COM-01; não administrador deve estar em membros da comissão.

**Pós-condição:** Nome, descrição, situação, organização e equipe disponíveis.

**Efeitos colaterais:** Sem gravação. ID inexistente: 404; acesso a comissão alheia: 403.

## RF-COM-03 — Cadastrar comissão com responsável

**Descrição:** Permitir cadastrar comissão com responsável.

**Entradas:** nome string 2–100 após trim; descricao opcional até 1000 após trim; organizacao_id e responsavel_id ObjectId.

**Origem:** Administrador da organização.

**Saída:** Resultado descrito na pós-condição ou erro do contrato comum.

**Destino:** Tela de comissões/equipe para COM; tela Ata e lista de presença para ATA/PRE.

**Ação:** `POST /comissoes` → `criar(dto, usuarioId)`. CreateComissaoDto valida corpo. autorizarOrganizacao confere administrador. exigirElegivel consulta membros aprovados e usuarios ativos. create grava comissoes: nome, descricao, organizacao_id, ativo=true e membros contendo somente {usuario_id:responsavel_id,papel:RESPONSAVEL,adicionado_em:agora}. Timestamps são criados pelo Mongoose.

**Pré-condição:** Organização APROVADA; solicitante ativo com vínculo ADMIN/APROVADO; responsável ativo e APROVADO na mesma organização.

**Pós-condição:** Documento criado com exatamente um responsável; retorna objeto populado e abre seus detalhes.

**Efeitos colaterais:** Novo documento. Campo ausente/inválido ou responsável inelegível: 400; sem autorização: 403. Não há unicidade de nome.

## RF-COM-04 — Editar dados da comissão

**Descrição:** Permitir editar dados da comissão.

**Entradas:** nome opcional 2–100; descricao opcional até 1000.

**Origem:** Administrador da organização.

**Saída:** Resultado descrito na pós-condição ou erro do contrato comum.

**Destino:** Tela de comissões/equipe para COM; tela Ata e lista de presença para ATA/PRE.

**Ação:** `PATCH /comissoes/:id` → `atualizar(id, dto, usuarioId)`. UpdateComissaoDto valida os campos fornecidos. autorizarComissao lê comissão e autoriza organização. findByIdAndUpdate(_id,{$set:dados},{returnDocument:"after",runValidators:true}) altera apenas campos permitidos. organizacao_id e membros não fazem parte desse DTO.

**Pré-condição:** Administrador ADMIN/APROVADO de organização APROVADA; comissão existente.

**Pós-condição:** Dados atualizados retornados e aplicados à tela sem exigir sair e entrar.

**Efeitos colaterais:** Grava campos e atualizado_em. Nome repetido permitido; edição de dados é permitida em comissão inativa. Atualizações simultâneas de nome usam a última escrita, sem controle de versão.

## RF-COM-05 — Desativar comissão

**Descrição:** Permitir desativar comissão.

**Entradas:** id e confirmação explícita no app.

**Origem:** Administrador da organização.

**Saída:** Resultado descrito na pós-condição ou erro do contrato comum.

**Destino:** Tela de comissões/equipe para COM; tela Ata e lista de presença para ATA/PRE.

**Ação:** `DELETE /comissoes/:id` → `remover(id, usuarioId)`. remover chama atualizar(id,{ativo:false},usuarioId). A operação usa $set em comissoes.ativo, preservando equipe e histórico. Retorna {mensagem,comissao}.

**Pré-condição:** Administrador autorizado; comissão existente.

**Pós-condição:** Comissão permanece cadastrada, marcada inativa.

**Efeitos colaterais:** Não apaga documentos nem cancela automaticamente reuniões já marcadas. Nova inclusão/remoção de equipe e novo agendamento ficam bloqueados pelas respectivas regras. Cancelar a confirmação não envia a requisição.

## RF-COM-06 — Reativar comissão

**Descrição:** Permitir reativar comissão.

**Entradas:** ativo=true booleano.

**Origem:** Administrador da organização.

**Saída:** Resultado descrito na pós-condição ou erro do contrato comum.

**Destino:** Tela de comissões/equipe para COM; tela Ata e lista de presença para ATA/PRE.

**Ação:** `PATCH /comissoes/:id` → `atualizar(id, dto, usuarioId)`. atualizar chama exigirResponsavelUnico antes de gravar comissoes.ativo=true. Se dados antigos tiverem zero ou vários responsáveis, exige regularização por COM-08.

**Pré-condição:** Administrador autorizado; exatamente um membro com papel RESPONSAVEL.

**Pós-condição:** Comissão ativa retornada à tela.

**Efeitos colaterais:** Não modifica reuniões existentes. A reativação verifica quantidade de responsáveis; não revalida automaticamente a elegibilidade de todos os integrantes antigos.

## RF-COM-07 — Adicionar integrante

**Descrição:** Permitir adicionar integrante.

**Entradas:** usuario_id ObjectId; papel omitido ou MEMBRO.

**Origem:** Administrador da organização.

**Saída:** Resultado descrito na pós-condição ou erro do contrato comum.

**Destino:** Tela de comissões/equipe para COM; tela Ata e lista de presença para ATA/PRE.

**Ação:** `POST /comissoes/:id/membros` → `adicionarMembro(id, dto, usuarioId)`. autorizarComissao, exigirResponsavelUnico e exigirElegivel são executados. Duplica ID: 409. Papel RESPONSAVEL: 400, orientando transferência. gravarEquipe substitui o array anterior por cópia acrescida de MEMBRO em uma escrita condicionada ao array e ativo lidos.

**Pré-condição:** Comissão ativa, exatamente um responsável, solicitante administrador; candidato ativo/APROVADO na organização e ainda fora da comissão.

**Pós-condição:** Novo integrante na equipe sem alterar responsável.

**Efeitos colaterais:** Conflito concorrente: 409, atualizar tela e repetir. A busca de candidatos é por nome/e-mail e não envia convite por e-mail.

## RF-COM-08 — Definir ou substituir responsável

**Descrição:** Permitir definir ou substituir responsável.

**Entradas:** usuario_id ObjectId do novo responsável; confirmação no app.

**Origem:** Administrador da organização.

**Saída:** Resultado descrito na pós-condição ou erro do contrato comum.

**Destino:** Tela de comissões/equipe para COM; tela Ata e lista de presença para ATA/PRE.

**Ação:** `PATCH /comissoes/:id/responsavel` → `definirResponsavel(id, membroId, usuarioId)`. DefinirResponsavelDto valida. definirResponsavel lê a equipe e valida o alvo. Mapeia todos os outros para MEMBRO e o alvo para RESPONSAVEL; se ausente, inclui-o. gravarEquipe usa findOneAndUpdate({_id,ativo:valorLido,membros:arrayLido},{$set:{membros:novoArray}},{returnDocument:"after",runValidators:true}). Zero documentos alterados retorna 409.

**Pré-condição:** Administrador autorizado; alvo ativo/APROVADO na organização. Comissão pode estar inativa ou possuir dados antigos inconsistentes.

**Pós-condição:** Exatamente um responsável; antigos responsáveis permanecem integrantes; situação ativo não é alterada.

**Efeitos colaterais:** Não escolhe uma pessoa automaticamente. PATCH /:id/membros/:usuarioId com papel RESPONSAVEL é compatibilidade para integrante já cadastrado em comissão ativa e delega a definirResponsavel. Não transfere autoria criada_por das reuniões antigas.

## RF-COM-09 — Remover integrante

**Descrição:** Permitir remover integrante.

**Entradas:** id da comissão, usuarioId do integrante e confirmação.

**Origem:** Administrador da organização.

**Saída:** Resultado descrito na pós-condição ou erro do contrato comum.

**Destino:** Tela de comissões/equipe para COM; tela Ata e lista de presença para ATA/PRE.

**Ação:** `DELETE /comissoes/:id/membros/:usuarioId` → `removerMembro(id, membroId, usuarioId)`. validarEquipe verifica existência e atividade. removerMembro recusa alvo RESPONSAVEL (400); remove somente alvo do novo array e chama gravarEquipe com comparação do array anterior.

**Pré-condição:** Comissão ativa, responsável único, solicitante administrador e alvo MEMBRO existente.

**Pós-condição:** Integrante removido; responsável preservado.

**Efeitos colaterais:** Reuniões antigas conservam seu snapshot de participantes. Alvo inexistente: 404. Conflito de equipe: 409. Cancelar confirmação não altera nada.

## RF-ATA-01 — Salvar rascunho de ata

**Descrição:** Permitir salvar rascunho de ata.

**Entradas:** conteudo string 20–20000 após trim; versao inteiro >=0 (0 para primeira gravação).

**Origem:** Responsável atual da comissão que participa da reunião.

**Saída:** Resultado descrito na pós-condição ou erro do contrato comum.

**Destino:** Tela de comissões/equipe para COM; tela Ata e lista de presença para ATA/PRE.

**Ação:** `PATCH /reunioes/:id/ata` → `salvarAta(id, dto, usuarioId)`. carregar busca reunioes e participante. autorizarResponsavel consulta comissoes e organizacoes. exigirEncerrada verifica término. Compara versao enviada com ata.versao (ou 0). gravarAta grava subdocumento ata com versao+1, conteudo, autor_id, atualizada_em, situacao=RASCUNHO e arrays vazios de aprovadores/aprovacoes. A escrita exige ata igual ao snapshot lido ou ausente.

**Pré-condição:** Usuário ativo e participante; comissão ativa e organização aprovada com vínculo aprovado; reunião não cancelada, fim <= horário do servidor; ata ausente ou RASCUNHO.

**Pós-condição:** Rascunho salvo, versão incrementada, retornado no objeto da reunião.

**Efeitos colaterais:** Versão antiga ou disputa: 409. Ata já enviada: 400. Não é mantido histórico de textos anteriores: versao é controle de concorrência, não arquivo de versões.

## RF-ATA-02 — Enviar ata para aprovação

**Descrição:** Permitir enviar ata para aprovação.

**Entradas:** versao inteiro >=0; confirmação da publicação.

**Origem:** Responsável atual da comissão que participa da reunião.

**Saída:** Resultado descrito na pós-condição ou erro do contrato comum.

**Destino:** Tela de comissões/equipe para COM; tela Ata e lista de presença para ATA/PRE.

**Ação:** `POST /reunioes/:id/ata/publicar` → `publicarAta(id, versao, usuarioId)`. publicarAta fixa ata.aprovadores com os IDs dos participantes cuja presenca.situacao é CONFIRMADA naquele instante. Mantém conteúdo/versão/autor; define EM_APROVACAO, atualizada_em e aprovacoes=[]. gravarAta condiciona escrita ao snapshot anterior.

**Pré-condição:** Mesma autorização de ATA-01; rascunho existente na versão enviada; pelo menos uma presença CONFIRMADA.

**Pós-condição:** Texto bloqueado e conjunto de aprovadores fixado.

**Efeitos colaterais:** Não aprova automaticamente pelo autor, não dispara e-mail/push. Alterações posteriores de presença não mudam o conjunto congelado. Não há correção, rejeição ou reabertura após publicação nesta versão.

## RF-ATA-03 — Aprovar ata

**Descrição:** Permitir aprovar ata.

**Entradas:** versao inteiro >=0 e confirmação de leitura/aprovação.

**Origem:** Participante incluído em ata.aprovadores.

**Saída:** Resultado descrito na pós-condição ou erro do contrato comum.

**Destino:** Tela de comissões/equipe para COM; tela Ata e lista de presença para ATA/PRE.

**Ação:** `POST /reunioes/:id/ata/aprovar` → `aprovarAta(id, versao, usuarioId)`. aprovarAta valida versão e autorização. Se ID já consta em aprovacoes, retorna reunião sem novo voto. Caso contrário, acrescenta-o, mantém EM_APROVACAO enquanto faltarem IDs e marca APROVADA quando as quantidades coincidem. gravarAta compara snapshot anterior para evitar perda de voto concorrente.

**Pré-condição:** Usuário ativo, participante da reunião encerrada/não cancelada, incluído no conjunto de aprovadores; ata EM_APROVACAO na mesma versão.

**Pós-condição:** Uma aprovação por usuário; aprovação final exige todos os aprovadores fixados.

**Efeitos colaterais:** Conflito 409 exige atualizar e repetir. Não é assinatura digital/certificado. Não há data individual de voto, apenas atualizada_em da ata. A participação histórica é suficiente para leitura/voto: a política herdada não reconsulta vínculo organizacional nessas duas operações.

## RF-PRE-01 — Gerar e compartilhar lista de presença

**Descrição:** Permitir gerar e compartilhar lista de presença.

**Entradas:** id da reunião; compartilhar é ação local opcional.

**Origem:** Participante da reunião.

**Saída:** Resultado descrito na pós-condição ou erro do contrato comum.

**Destino:** Tela de comissões/equipe para COM; tela Ata e lista de presença para ATA/PRE.

**Ação:** `GET /reunioes/:id/lista-presenca` → `gerarListaPresenca(id, usuarioId)`. carregar autoriza. usuarioModel.find({_id:{$in:idsParticipantes}}).select(nome) resolve nomes. Retorna reuniao_id,titulo,inicio,fim,gerada_em e participantes: usuario_id,nome,convite,situacao,entrada,saida,minutos. Ausência de presença vira SEM_REGISTRO e datas/minutos null; minutos=floor((saida-entrada)/60000), mínimo 0, somente se há saída. Tela mostra oito por página e Share.share gera texto por ação do usuário.

**Pré-condição:** Usuário ativo e participante; não exige reunião encerrada.

**Pós-condição:** Relatório consultado/compartilhável, indicando que apenas CONFIRMADA é presença validada.

**Efeitos colaterais:** Não grava presença nem cria arquivo no servidor. Pode incluir reunião cancelada ou em curso como fotografia dos registros. Nome indisponível recebe rótulo Usuário indisponível. Compartilhamento depende do sistema operacional.

## Consultas auxiliares (passos dos casos de uso, não objetivos independentes)

- `GET /comissoes/organizacoes` → listarOrganizacoes(usuarioId): busca organizacoes com status APROVADA e `$elemMatch` do mesmo integrante ADMIN/APROVADO; retorna `_id,nome`, ordem nome crescente. Apoia COM-01/03.
- `GET /comissoes/organizacoes/:id/membros` → listarMembrosDisponiveis(id,usuarioId): autoriza ADMIN, obtém vínculos APROVADO, consulta usuarios `_id in ids, ativo:true`, seleciona `_id,nome,email`, ordem nome crescente. Apoia COM-03/07/08.
- `GET /comissoes/minhas/:organizacaoId` → listarMinhas: exige vínculo APROVADO em organização APROVADA, busca comissoes por organizacao_id e membros.usuario_id. Retorna organizacao:{_id,nome} e comissoes:[{_id,nome,descricao,ativo,meu_papel}], ordem nome crescente. É outra apresentação de COM-01.
- `GET /reunioes/:id` → buscarPorId: carrega a reunião e ata para participante ativo. Apoia ATA-01/02/03 e PRE-01.

## Integrações existentes preservadas

O agendamento existente (`POST /reunioes`, criar) aceita comissao_id, titulo 3–120, local 2–120, inicio/fim ISO-8601, pauta opcional de até 30 strings de até 200 caracteres. Exige responsável e fim maior que início e que agora. Aceita convidados_ids opcional: array de ObjectIds únicos de integrantes da comissão. Quando informado, grava somente os selecionados e inclui sempre o organizador; array vazio significa somente o organizador. Por compatibilidade, clientes antigos que omitem o campo ainda convidam toda a comissão. Autor inicia ACEITO, demais PENDENTE. No app atualizado o campo é sempre enviado, inicialmente somente com o organizador. Convites são registros internos, não integração com Google/Teams ou envio automático de e-mail.

`PATCH /reunioes/:id` (atualizar) permite edição/cancelamento ao criador. `PATCH /:id/convite` (responderConvite) recebe aceito booleano. `POST /:id/entrada` (marcarEntrada) registra entrada durante a reunião: criador CONFIRMADA, demais AGUARDANDO. `POST /:id/saida` (marcarSaida) registra saída limitada ao fim. `PATCH /:id/presencas/:usuarioId` (atualizarPresenca) permite ao criador CONFIRMADA/RECUSADA; sem entrada, confirmar cria entrada igual ao início e origem RESPONSAVEL. Essa hora é um lançamento administrativo, não medição real de chegada. O criador e o responsável atual podem divergir após COM-08. Esses requisitos pertencem ao módulo de reuniões já existente; não são atribuídos como implementação nova de Rafaela.
