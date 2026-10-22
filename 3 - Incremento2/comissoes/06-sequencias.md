# Diagramas de sequência

Os lifelines C e S são instâncias das classes reais Controller e Service. M é uma instância de Model do Mongoose, não uma classe chamada Backend. Ator/Tela representa interação do usuário; os componentes React são funções. HTTP aparece em nota, não como nome de método. Cada diagrama corresponde a um RF e herda seu contrato de pré/pós-condições e erros.

## Autenticação comum a todas as rotas

```mermaid
sequenceDiagram
actor A as Usuário
participant G as JwtAuthGuard
participant J as JwtService
participant U as Model de Usuario
participant T as UsuarioComumGuard
A->>G: canActivate(context)
G->>G: extrairToken(requisicao)
G->>J: verifyAsync(token)
J-->>G: payload ou erro
G->>U: findOne({_id:sub,ativo:true}).exec()
U-->>G: usuario
G-->>A: autorizado ou 401
A->>T: canActivate(context)
T-->>A: permitido se tipo comum, senão 403
Note over G,T: Nest executa guards antes do handler e valida o DTO no pipeline
```

## RF-COM-01 — Consultar comissões

```mermaid
sequenceDiagram
actor A as Integrante aprovado da organização ou seu administrador
participant C as ComissoesController
participant S as ComissoesService
participant M as Model de Comissao
participant U as Model de Usuario
Note over A,C: GET /comissoes?organizacao_id=<id>
A->>C: listar(req, organizacaoId)
C->>S: listar(usuarioId, organizacaoId?)
S->>S: autorizarLeitura(organizacaoId, usuarioId)
S->>M: find(filtro).populate(...).sort({criado_em:-1}).exec()
S-->>C: resultado autorizado
C-->>A: dados ou confirmação
```

Sem organizacao_id, listar usa validarUsuario e organizacaoModel.find com vínculo APROVADO, separa administradas/participantes e consulta o $or descrito no RF. Não chama autorizarLeitura nessa alternativa.

## RF-COM-02 — Consultar detalhes da comissão

```mermaid
sequenceDiagram
actor A as Integrante da comissão ou administrador da organização
participant C as ComissoesController
participant S as ComissoesService
participant M as Model de Comissao
participant U as Model de Usuario
Note over A,C: GET /comissoes/:id
A->>C: buscarPorId(id, req)
C->>S: buscarPorId(id, usuarioId)
S->>M: findById(id).exec()
S->>S: autorizarLeitura(organizacao_id, usuarioId)
S->>S: administra(organizacao, usuarioId)
S->>S: popular(comissao)
S-->>C: resultado autorizado
C-->>A: dados ou confirmação
```

## RF-COM-03 — Cadastrar comissão com responsável

```mermaid
sequenceDiagram
actor A as Administrador da organização
participant C as ComissoesController
participant S as ComissoesService
participant M as Model de Comissao
participant U as Model de Usuario
Note over A,C: POST /comissoes
A->>C: criar(dto, req)
C->>S: criar(dto, usuarioId)
S->>S: autorizarOrganizacao(dto.organizacao_id, usuarioId)
S->>S: exigirElegivel(organizacao_id, responsavel_id, usuarioId)
S->>M: create(dadosComUmResponsavel)
S->>S: popular(comissao)
S-->>C: resultado autorizado
C-->>A: dados ou confirmação
```

## RF-COM-04 — Editar dados da comissão

```mermaid
sequenceDiagram
actor A as Administrador da organização
participant C as ComissoesController
participant S as ComissoesService
participant M as Model de Comissao
participant U as Model de Usuario
Note over A,C: PATCH /comissoes/:id
A->>C: atualizar(id, dto, req)
C->>S: atualizar(id, dto, usuarioId)
S->>S: autorizarComissao(id, usuarioId)
S->>M: findByIdAndUpdate(id, {$set:dados}, opcoes).exec()
S->>S: popular(comissao)
S-->>C: resultado autorizado
C-->>A: dados ou confirmação
```

## RF-COM-05 — Desativar comissão

```mermaid
sequenceDiagram
actor A as Administrador da organização
participant C as ComissoesController
participant S as ComissoesService
participant M as Model de Comissao
participant U as Model de Usuario
Note over A,C: DELETE /comissoes/:id
A->>C: remover(id, req)
C->>S: remover(id, usuarioId)
S->>S: atualizar(id, {ativo:false}, usuarioId)
S->>S: autorizarComissao(id, usuarioId)
S->>M: findByIdAndUpdate(id, {$set:dados}, opcoes).exec()
S-->>C: resultado autorizado
C-->>A: dados ou confirmação
```

## RF-COM-06 — Reativar comissão

```mermaid
sequenceDiagram
actor A as Administrador da organização
participant C as ComissoesController
participant S as ComissoesService
participant M as Model de Comissao
participant U as Model de Usuario
Note over A,C: PATCH /comissoes/:id
A->>C: atualizar(id, dto, req)
C->>S: atualizar(id, dto, usuarioId)
S->>S: autorizarComissao(id, usuarioId)
S->>S: exigirResponsavelUnico(atual)
S->>M: findByIdAndUpdate(id, {$set:dados}, opcoes).exec()
S-->>C: resultado autorizado
C-->>A: dados ou confirmação
```

## RF-COM-07 — Adicionar integrante

```mermaid
sequenceDiagram
actor A as Administrador da organização
participant C as ComissoesController
participant S as ComissoesService
participant M as Model de Comissao
participant U as Model de Usuario
Note over A,C: POST /comissoes/:id/membros
A->>C: adicionarMembro(id, dto, req)
C->>S: adicionarMembro(id, dto, usuarioId)
S->>S: autorizarComissao(id, usuarioId)
S->>S: exigirResponsavelUnico(comissao)
S->>S: exigirElegivel(organizacao_id, dto.usuario_id, usuarioId)
S->>S: gravarEquipe(comissao, membrosComNovoIntegrante)
S->>M: findOneAndUpdate(filtroSnapshot, {$set:{membros}}, opcoes).exec()
alt snapshot alterado por outra requisição
M-->>S: null
S-->>C: ConflictException
C-->>A: 409, atualizar e tentar novamente
else escrita aceita
M-->>S: documento atualizado
S-->>C: documento populado
C-->>A: resultado de sucesso
end
```

## RF-COM-08 — Definir ou substituir responsável

```mermaid
sequenceDiagram
actor A as Administrador da organização
participant C as ComissoesController
participant S as ComissoesService
participant M as Model de Comissao
participant U as Model de Usuario
Note over A,C: PATCH /comissoes/:id/responsavel
A->>C: definirResponsavel(id, dto, req)
C->>S: definirResponsavel(id, membroId, usuarioId)
S->>S: autorizarComissao(id, usuarioId)
S->>S: exigirElegivel(organizacao_id, membroId, usuarioId)
S->>S: gravarEquipe(comissao, membrosComResponsavelUnico)
S->>M: findOneAndUpdate(filtroSnapshot, {$set:{membros}}, opcoes).exec()
alt snapshot alterado por outra requisição
M-->>S: null
S-->>C: ConflictException
C-->>A: 409, atualizar e tentar novamente
else escrita aceita
M-->>S: documento atualizado
S-->>C: documento populado
C-->>A: resultado de sucesso
end
```

## RF-COM-09 — Remover integrante

```mermaid
sequenceDiagram
actor A as Administrador da organização
participant C as ComissoesController
participant S as ComissoesService
participant M as Model de Comissao
participant U as Model de Usuario
Note over A,C: DELETE /comissoes/:id/membros/:usuarioId
A->>C: removerMembro(id, usuarioId, req)
C->>S: removerMembro(id, membroId, usuarioId)
S->>S: validarEquipe(id, membroId, usuarioId)
S->>S: exigirResponsavelUnico(comissao)
S->>S: gravarEquipe(comissao, membrosSemAlvo)
S->>M: findOneAndUpdate(filtroSnapshot, {$set:{membros}}, opcoes).exec()
alt snapshot alterado por outra requisição
M-->>S: null
S-->>C: ConflictException
C-->>A: 409, atualizar e tentar novamente
else escrita aceita
M-->>S: documento atualizado
S-->>C: documento populado
C-->>A: resultado de sucesso
end
```

## RF-ATA-01 — Salvar rascunho de ata

```mermaid
sequenceDiagram
actor A as Responsável atual da comissão que participa da reunião
participant C as ReunioesController
participant S as ReunioesService
participant M as Model de Reuniao
participant U as Model de Usuario
Note over A,C: PATCH /reunioes/:id/ata
A->>C: salvarAta(id, dto, req)
C->>S: salvarAta(id, dto, usuarioId)
S->>S: carregar(id, usuarioId)
S->>S: autorizarResponsavel(comissao_id, usuarioId)
S->>S: exigirEncerrada(reuniao)
S->>S: gravarAta(reuniao, novoRascunho, usuarioId)
S->>M: findOneAndUpdate(filtroAtaAnterior, {$set:{ata}}, opcoes).exec()
alt snapshot alterado por outra requisição
M-->>S: null
S-->>C: ConflictException
C-->>A: 409, atualizar e tentar novamente
else escrita aceita
M-->>S: documento atualizado
S-->>C: documento populado
C-->>A: resultado de sucesso
end
```

## RF-ATA-02 — Enviar ata para aprovação

```mermaid
sequenceDiagram
actor A as Responsável atual da comissão que participa da reunião
participant C as ReunioesController
participant S as ReunioesService
participant M as Model de Reuniao
participant U as Model de Usuario
Note over A,C: POST /reunioes/:id/ata/publicar
A->>C: publicarAta(id, dto, req)
C->>S: publicarAta(id, versao, usuarioId)
S->>S: carregar(id, usuarioId)
S->>S: autorizarResponsavel(comissao_id, usuarioId)
S->>S: exigirEncerrada(reuniao)
S->>S: gravarAta(reuniao, ataComAprovadores, usuarioId)
S->>M: findOneAndUpdate(filtroAtaAnterior, {$set:{ata}}, opcoes).exec()
alt snapshot alterado por outra requisição
M-->>S: null
S-->>C: ConflictException
C-->>A: 409, atualizar e tentar novamente
else escrita aceita
M-->>S: documento atualizado
S-->>C: documento populado
C-->>A: resultado de sucesso
end
```

## RF-ATA-03 — Aprovar ata

```mermaid
sequenceDiagram
actor A as Participante incluído em ata.aprovadores
participant C as ReunioesController
participant S as ReunioesService
participant M as Model de Reuniao
participant U as Model de Usuario
Note over A,C: POST /reunioes/:id/ata/aprovar
A->>C: aprovarAta(id, dto, req)
C->>S: aprovarAta(id, versao, usuarioId)
S->>S: carregar(id, usuarioId)
S->>S: exigirEncerrada(reuniao)
S->>S: gravarAta(reuniao, ataComVotoUnico, usuarioId)
S->>M: findOneAndUpdate(filtroAtaAnterior, {$set:{ata}}, opcoes).exec()
alt snapshot alterado por outra requisição
M-->>S: null
S-->>C: ConflictException
C-->>A: 409, atualizar e tentar novamente
else escrita aceita
M-->>S: documento atualizado
S-->>C: documento populado
C-->>A: resultado de sucesso
end
```

Antes de gravarAta, o serviço compara versão, exige presença em aprovadores e verifica voto existente. Voto repetido chama buscarPorId(id,usuarioId), retornando sem escrever. Falta de autorização: 403; estado inválido: 400; versão divergente: 409.

## RF-PRE-01 — Gerar e compartilhar lista de presença

```mermaid
sequenceDiagram
actor A as Participante da reunião
participant C as ReunioesController
participant S as ReunioesService
participant M as Model de Reuniao
participant U as Model de Usuario
Note over A,C: GET /reunioes/:id/lista-presenca
A->>C: gerarListaPresenca(id, req)
C->>S: gerarListaPresenca(id, usuarioId)
S->>S: carregar(id, usuarioId)
S->>U: find({_id:{$in:ids}}).select("nome").exec()
S-->>C: resultado autorizado
C-->>A: dados ou confirmação
```

## Detalhamento das colaborações internas

```mermaid
sequenceDiagram
participant S as ComissoesService
participant U as Model de Usuario
participant O as Model de Organizacao
S->>S: autorizarOrganizacao(organizacaoId, usuarioId)
S->>S: validarId(organizacaoId)
S->>S: validarUsuario(usuarioId)
S->>U: findById(usuarioId).exec()
U-->>S: usuario ativo
S->>O: findById(organizacaoId).exec()
O-->>S: organizacao
S->>S: administra(organizacao, usuarioId)
Note over S,O: Exige APROVADA e mesmo vínculo ADMIN/APROVADO, ou 403
```

```mermaid
sequenceDiagram
participant S as ReunioesService
participant U as Model de Usuario
participant R as Model de Reuniao
S->>S: carregar(id, usuarioId)
S->>S: validarId(id)
S->>S: validarUsuario(usuarioId)
S->>U: findById(usuarioId).exec()
U-->>S: usuario
S->>R: findById(id).exec()
R-->>S: reuniao
Note over S: Exige usuario ativo e ID em participantes e calcula dono por criada_por
```

Os nomes novoRascunho, filtroSnapshot e ataComVotoUnico são argumentos descritivos, não métodos inventados. Definições exatas estão nos RF e em gravarEquipe/gravarAta. Falhas de autenticação, DTO e autorização interrompem o fluxo antes da escrita. Retornos de consultas internas foram abreviados nos diagramas principais para legibilidade. As sequências de reuniões já existentes permanecem sob responsabilidade do respectivo módulo; os treze requisitos próprios desta revisão têm diagrama acima.
