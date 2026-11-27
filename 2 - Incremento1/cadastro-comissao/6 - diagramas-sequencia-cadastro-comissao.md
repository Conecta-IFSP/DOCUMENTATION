# Diagramas de sequência — Cadastro de comissão

> Padrão da disciplina · Módulo COM · Revisado em 26/11/2026

## Resumo

Há um diagrama para cada requisito de [Requisitos funcionais](2%20-%20requisitos-funcionais-cadastro-comissao.md), com o mesmo número. As linhas de vida são classes e módulos que existem no código: as telas do aplicativo, o componente `SeletorPessoaComissao`, os módulos `validacao` e `api`, o `ComissoesController`, o `ComissoesService` e os models do Mongoose. Toda mensagem é a chamada de um método existente, com seus parâmetros, e as mensagens são numeradas na ordem em que acontecem. Regras que não são métodos, como a montagem de um filtro, aparecem em notas.

Em todos os diagramas, a chamada do módulo `api` ao controller é uma requisição HTTP que passa antes por `ApiKeyGuard`, `JwtAuthGuard` e `UsuarioComumGuard`; a rota está indicada na nota. As funções `autorizarLeitura`, `autorizarOrganizacao` e `autorizarComissao` conferem a conta do usuário (`UsuarioModel.findById`) e a organização (`OrganizacaoModel.findById`); essas buscas aparecem por extenso nos diagramas SD-COM-01 a SD-COM-04 e, nos seguintes, ficam dentro da chamada de `autorizarComissao`.

## SD-COM-01 — Listar e filtrar comissões

```mermaid
sequenceDiagram
    autonumber
    actor U as Membro da organização
    participant T as TelaOrganizacao
    participant A as api
    participant C as ComissoesController
    participant S as ComissoesService
    participant UM as UsuarioModel
    participant OM as OrganizacaoModel
    participant M as ComissaoModel
    U->>T: carregar()
    T->>A: api("/comissoes?organizacao_id=id")
    Note over A,C: GET /comissoes?organizacao_id=:id
    A->>C: listar(req, organizacaoId)
    C->>S: listar(usuarioId, organizacaoId)
    S->>S: autorizarLeitura(organizacaoId, usuarioId)
    S->>S: validarId(organizacaoId)
    S->>S: validarUsuario(usuarioId)
    S->>UM: findById(usuarioId)
    UM-->>S: usuario
    S->>OM: findById(organizacaoId)
    OM-->>S: organizacao
    Note over S: Exige organizacao.status APROVADA e um item de membros<br/>com o usuário e status APROVADO, senão 403
    S->>S: administra(organizacao, usuarioId)
    alt administrador da organização
        Note over S: filtro = organizacao_id igual ao da organização
    else demais membros
        Note over S: filtro = organizacao_id igual ao da organização<br/>e membros.usuario_id igual ao usuário
    end
    S->>S: consultar(filtro)
    S->>M: find(filtro)
    S->>M: populate("organizacao_id", "nome status")
    S->>M: populate("membros.usuario_id", "nome email ativo")
    S->>M: sort({ criado_em: -1 })
    M-->>S: comissoes
    S-->>C: comissoes
    C-->>A: 200 comissoes
    A-->>T: Resultado com as comissoes
    T->>T: definirDados(novos)
    U->>T: definirBuscaComissao(texto)
    U->>T: definirFiltro(item)
    Note over T: Mantém as comissões com ativo conforme o filtro e com o nome<br/>em minúsculas contendo o texto. Ordena ativas antes das inativas<br/>e, em cada grupo, por nome crescente (pt-BR)
```

## SD-COM-02 — Consultar comissão

```mermaid
sequenceDiagram
    autonumber
    actor U as Membro da organização
    participant T as TelaComissao
    participant A as api
    participant C as ComissoesController
    participant S as ComissoesService
    participant UM as UsuarioModel
    participant OM as OrganizacaoModel
    participant M as ComissaoModel
    U->>T: buscar()
    Note over T,A: As duas primeiras consultas saem juntas (Promise.all)
    T->>A: api("/comissoes/:id")
    Note over A,C: GET /comissoes/:id
    A->>C: buscarPorId(id, req)
    C->>S: buscarPorId(id, usuarioId)
    S->>S: validarId(id)
    S->>M: findById(id)
    M-->>S: comissao
    S->>S: autorizarLeitura(organizacaoId, usuarioId)
    S->>UM: findById(usuarioId)
    UM-->>S: usuario
    S->>OM: findById(organizacaoId)
    OM-->>S: organizacao
    S->>S: administra(organizacao, usuarioId)
    Note over S: Recusa quem não administra a organização e não está<br/>em comissao.membros ("Você não faz parte desta comissão", 403)
    S->>S: popular(comissao)
    Note over S,M: caminhos = organizacao_id com nome e status<br/>e membros.usuario_id com nome, email e ativo
    S->>M: populate(caminhos)
    M-->>S: comissao preenchida
    S-->>C: comissao
    C-->>A: 200 comissao
    A-->>T: Resultado com a comissao
    T->>A: api("/comissoes/organizacoes")
    Note over A,C: GET /comissoes/organizacoes
    A->>C: listarOrganizacoes(req)
    C->>S: listarOrganizacoes(usuarioId)
    S->>S: validarUsuario(usuarioId)
    S->>UM: findById(usuarioId)
    UM-->>S: usuario
    Note over S,OM: filtro = status APROVADA e membros com $elemMatch<br/>do usuário com papel ADMIN e status APROVADO
    S->>OM: find(filtro)
    OM-->>S: organizacoes com nome, por nome crescente
    S-->>C: organizacoes
    C-->>A: 200 organizacoes
    A-->>T: Resultado com as organizacoes
    opt organização da comissão está entre as administradas
        T->>A: api("/comissoes/organizacoes/:organizacaoId/membros")
        Note over A,C: GET /comissoes/organizacoes/:id/membros
        A->>C: listarMembrosDisponiveis(id, req)
        C->>S: listarMembrosDisponiveis(organizacaoId, usuarioId)
        S->>S: autorizarOrganizacao(organizacaoId, usuarioId)
        S->>UM: findById(usuarioId)
        UM-->>S: usuario
        S->>OM: findById(organizacaoId)
        OM-->>S: organizacao
        Note over S,UM: ids = usuario_id dos membros da organização com status APROVADO
        S->>UM: find({ _id: { $in: ids }, ativo: true })
        UM-->>S: pessoas com nome e email, por nome crescente
        S-->>C: pessoas
        C-->>A: 200 pessoas
        A-->>T: Resultado com as pessoas
    end
    T->>T: aplicar(carga, true)
    U->>T: definirAba("equipe")
    Note over T: Equipe com o responsável primeiro e os demais por nome.<br/>A aba Reuniões usa GET /reunioes?comissao_id=:id (SD-REU-02)
```

## SD-COM-03 — Cadastrar comissão

```mermaid
sequenceDiagram
    autonumber
    actor U as Administrador
    participant T as TelaNovaComissao
    participant V as validacao
    participant A as api
    participant C as ComissoesController
    participant S as ComissoesService
    participant UM as UsuarioModel
    participant OM as OrganizacaoModel
    participant M as ComissaoModel
    Note over U,T: Ao abrir, a tela chama api("/comissoes/organizacoes")<br/>e, para a organização marcada, api("/comissoes/organizacoes/:id/membros"), como no SD-COM-02
    U->>T: setValue("organizacao_id", organizacaoId)
    U->>T: onChange(responsavelId)
    U->>T: salvar()
    T->>V: validarNomeComissao(nome)
    V-->>T: null
    T->>V: validarDescricao(descricao)
    V-->>T: null
    T->>A: api("/comissoes", "POST", corpo)
    Note over A,C: POST /comissoes<br/>O ValidationPipe valida CreateComissaoDto
    A->>C: criar(dto, req)
    C->>S: criar(dto, usuarioId)
    S->>S: autorizarOrganizacao(dto.organizacao_id, usuarioId)
    S->>UM: findById(usuarioId)
    UM-->>S: usuario
    S->>OM: findById(organizacaoId)
    OM-->>S: organizacao
    S->>S: administra(organizacao, usuarioId)
    S->>S: exigirElegivel(organizacaoId, dto.responsavel_id, usuarioId)
    S->>S: listarMembrosDisponiveis(organizacaoId, usuarioId)
    Note over S: listarMembrosDisponiveis repete autorizarOrganizacao
    Note over S,UM: ids = usuario_id dos membros da organização com status APROVADO
    S->>UM: find({ _id: { $in: ids }, ativo: true })
    UM-->>S: pessoas
    Note over S: Recusa responsavel_id fora da lista (400)
    Note over S,M: dados = nome, descricao e organizacao_id do dto, ativo true<br/>e membros com o responsavel_id do dto, papel RESPONSAVEL e adicionado_em agora
    S->>M: create(dados)
    Note over M: O validador de membros exige exatamente um RESPONSAVEL<br/>e nenhum usuario_id repetido
    M-->>S: comissao
    S->>S: popular(comissao)
    S-->>C: comissao
    C-->>A: 201 comissao
    A-->>T: Resultado com a comissao
    Note over T: destino = pathname /comissoes/[id] e params com o id da comissão e criada igual a 1
    T->>T: router.replace(destino)
```

## SD-COM-04 — Editar comissão

```mermaid
sequenceDiagram
    autonumber
    actor U as Administrador
    participant T as TelaComissao
    participant V as validacao
    participant A as api
    participant C as ComissoesController
    participant S as ComissoesService
    participant UM as UsuarioModel
    participant OM as OrganizacaoModel
    participant M as ComissaoModel
    U->>T: definirFolha("opcoes")
    U->>T: reset({ nome, descricao })
    U->>T: definirFolha("editar")
    U->>T: salvarDados()
    T->>V: validarNomeComissao(nome)
    V-->>T: null
    T->>V: validarDescricao(descricao)
    V-->>T: null
    T->>T: executar(acao, "Dados da comissão atualizados.")
    T->>A: api("/comissoes/:id", "PATCH", { nome: nome.trim(), descricao: texto.trim() })
    Note over A,C: PATCH /comissoes/:id<br/>O ValidationPipe valida UpdateComissaoDto
    A->>C: atualizar(id, dto, req)
    C->>S: atualizar(id, dto, usuarioId)
    S->>S: autorizarComissao(id, usuarioId)
    S->>S: validarId(id)
    S->>M: findById(id)
    M-->>S: comissao
    S->>S: autorizarOrganizacao(organizacaoId, usuarioId)
    S->>UM: findById(usuarioId)
    UM-->>S: usuario
    S->>OM: findById(organizacaoId)
    OM-->>S: organizacao
    S->>S: administra(organizacao, usuarioId)
    Note over S: dados recebe só os campos enviados (nome e descricao)
    Note over S,M: opcoes = returnDocument after e runValidators true
    S->>M: findByIdAndUpdate(id, { $set: dados }, opcoes)
    M-->>S: comissao atualizada
    S->>S: popular(comissao)
    S-->>C: comissao
    C-->>A: 200 comissao
    A-->>T: Resultado com a comissao
    T->>T: definirComissao(comissao)
    T->>T: definirSucesso("Dados da comissão atualizados.")
    T->>T: definirFolha(null)
```

## SD-COM-05 — Desativar comissão

```mermaid
sequenceDiagram
    autonumber
    actor U as Administrador
    participant T as TelaComissao
    participant A as api
    participant C as ComissoesController
    participant S as ComissoesService
    participant M as ComissaoModel
    U->>T: definirFolha("opcoes")
    U->>T: definirConfirmacao(pendente)
    Note over U,T: "Desativar comissão" abre o diálogo<br/>"Desativar ...? Ela poderá ser reativada depois." com "Cancelar" e "Desativar"
    U->>T: executar(desativar, "Comissão desativada.")
    T->>T: desativar()
    T->>A: api("/comissoes/:id", "DELETE")
    Note over A,C: DELETE /comissoes/:id
    A->>C: remover(id, req)
    C->>S: remover(id, usuarioId)
    S->>S: atualizar(id, { ativo: false }, usuarioId)
    S->>S: autorizarComissao(id, usuarioId)
    S->>M: findById(id)
    M-->>S: comissao
    Note over S,M: dados = ativo false<br/>opcoes = returnDocument after e runValidators true
    S->>M: findByIdAndUpdate(id, { $set: dados }, opcoes)
    M-->>S: comissao atualizada
    S->>S: popular(comissao)
    S-->>C: mensagem e comissao
    C-->>A: 200 mensagem e comissao
    A-->>T: Resultado com mensagem e comissao
    T->>T: definirComissao(comissao)
    T->>T: definirSucesso("Comissão desativada.")
```

## SD-COM-06 — Reativar comissão

```mermaid
sequenceDiagram
    autonumber
    actor U as Administrador
    participant T as TelaComissao
    participant A as api
    participant C as ComissoesController
    participant S as ComissoesService
    participant M as ComissaoModel
    U->>T: definirFolha("opcoes")
    U->>T: definirConfirmacao(pendente)
    Note over U,T: "Reativar comissão" abre o diálogo com "Cancelar" e "Reativar"
    U->>T: executar(acao, "Comissão reativada.")
    T->>A: api("/comissoes/:id", "PATCH", { ativo: true })
    Note over A,C: PATCH /comissoes/:id
    A->>C: atualizar(id, dto, req)
    C->>S: atualizar(id, dto, usuarioId)
    S->>S: autorizarComissao(id, usuarioId)
    S->>M: findById(id)
    M-->>S: comissao
    S->>S: exigirResponsavelUnico(comissao)
    alt exatamente um RESPONSAVEL em membros
        Note over S,M: dados = ativo true<br/>opcoes = returnDocument after e runValidators true
        S->>M: findByIdAndUpdate(id, { $set: dados }, opcoes)
        M-->>S: comissao atualizada
        S->>S: popular(comissao)
        S-->>C: comissao
        C-->>A: 200 comissao
        A-->>T: Resultado com a comissao
        T->>T: definirSucesso("Comissão reativada.")
    else nenhum ou mais de um RESPONSAVEL
        S-->>C: BadRequestException
        C-->>A: 400 Defina exatamente um responsável
        A-->>T: Resultado com o erro
        T->>T: definirErro(resultado.erro)
    end
```

## SD-COM-07 — Adicionar integrante

```mermaid
sequenceDiagram
    autonumber
    actor U as Administrador
    participant T as TelaComissao
    participant A as api
    participant C as ComissoesController
    participant S as ComissoesService
    participant UM as UsuarioModel
    participant M as ComissaoModel
    U->>T: definirFolha("adicionar")
    U->>T: definirBusca(texto)
    Note over T: Lista as pessoas elegíveis fora da equipe cujo<br/>"nome e-mail" em minúsculas contém o texto
    U->>T: executar(acao, "... entrou na comissão.")
    T->>A: api("/comissoes/:id/membros", "POST", { usuario_id: pessoa._id, papel: "MEMBRO" })
    Note over A,C: POST /comissoes/:id/membros<br/>O ValidationPipe valida AddMembroComissaoDto
    A->>C: adicionarMembro(id, dto, req)
    C->>S: adicionarMembro(id, dto, usuarioId)
    S->>S: autorizarComissao(id, usuarioId)
    S->>M: findById(id)
    M-->>S: comissao
    Note over S: Recusa comissão inativa (400)
    S->>S: exigirResponsavelUnico(comissao)
    Note over S: Recusa dto.papel RESPONSAVEL (400)
    S->>S: exigirElegivel(organizacaoId, dto.usuario_id, usuarioId)
    S->>S: listarMembrosDisponiveis(organizacaoId, usuarioId)
    Note over S,UM: ids = usuario_id dos membros da organização com status APROVADO
    S->>UM: find({ _id: { $in: ids }, ativo: true })
    UM-->>S: pessoas
    Note over S: Recusa usuario_id já presente em membros (409)
    Note over S: membros = itens atuais mais o usuario_id do dto com papel MEMBRO e adicionado_em agora
    S->>S: gravarEquipe(comissao, membros)
    Note over S,M: filtro = _id, ativo e membros iguais aos da comissão lida<br/>opcoes = returnDocument after e runValidators true
    S->>M: findOneAndUpdate(filtro, { $set: { membros } }, opcoes)
    alt equipe igual à lida
        M-->>S: comissao atualizada
        S->>S: popular(comissao)
        S-->>C: comissao
        C-->>A: 201 comissao
        A-->>T: Resultado com a comissao
        T->>T: definirComissao(comissao)
    else outra gravação chegou antes
        M-->>S: null
        S-->>C: ConflictException
        C-->>A: 409 A comissão mudou
        A-->>T: Resultado com o erro
        T->>T: definirErro(resultado.erro)
    end
```

## SD-COM-08 — Substituir responsável

```mermaid
sequenceDiagram
    autonumber
    actor U as Administrador
    participant P as SeletorPessoaComissao
    participant T as TelaComissao
    participant A as api
    participant C as ComissoesController
    participant S as ComissoesService
    participant UM as UsuarioModel
    participant M as ComissaoModel
    alt pela folha Responsável
        U->>T: definirFolha("responsavel")
        Note over U,P: O administrador toca em outra pessoa da lista
        P->>T: aoAlternar(pessoaId)
        T->>T: trocarResponsavel(alvo)
        T->>T: definirConfirmacao(pendente)
        P->>P: fechar()
    else pela opção "Tornar responsável" de um membro
        U->>T: definirEscolhido(integrante)
        U->>T: trocarResponsavel(alvo)
        T->>T: definirConfirmacao(pendente)
    end
    Note over U,T: "... será o responsável. ... passa a membro." com "Cancelar" e "Confirmar"
    U->>T: executar(acao, "Responsável atualizado.")
    T->>A: api("/comissoes/:id/responsavel", "PATCH", { usuario_id: alvo._id })
    Note over A,C: PATCH /comissoes/:id/responsavel<br/>O ValidationPipe valida DefinirResponsavelDto
    A->>C: definirResponsavel(id, dto, req)
    C->>S: definirResponsavel(id, dto.usuario_id, usuarioId)
    S->>S: autorizarComissao(id, usuarioId)
    S->>M: findById(id)
    M-->>S: comissao
    S->>S: exigirElegivel(organizacaoId, membroId, usuarioId)
    S->>S: listarMembrosDisponiveis(organizacaoId, usuarioId)
    Note over S,UM: ids = usuario_id dos membros da organização com status APROVADO
    S->>UM: find({ _id: { $in: ids }, ativo: true })
    UM-->>S: pessoas
    Note over S: Nova lista: o escolhido vira RESPONSAVEL e os demais MEMBRO,<br/>com adicionado_em mantido. Fora da equipe, o escolhido entra como RESPONSAVEL
    S->>S: gravarEquipe(comissao, membros)
    Note over S,M: filtro = _id, ativo e membros iguais aos da comissão lida<br/>opcoes = returnDocument after e runValidators true
    S->>M: findOneAndUpdate(filtro, { $set: { membros } }, opcoes)
    M-->>S: comissao atualizada
    S->>S: popular(comissao)
    S-->>C: comissao
    C-->>A: 200 comissao
    A-->>T: Resultado com a comissao
    T->>T: definirComissao(comissao)
    T->>T: definirSucesso("Responsável atualizado.")
```

## SD-COM-09 — Remover integrante

```mermaid
sequenceDiagram
    autonumber
    actor U as Administrador
    participant T as TelaComissao
    participant A as api
    participant C as ComissoesController
    participant S as ComissoesService
    participant M as ComissaoModel
    U->>T: definirEscolhido(integrante)
    U->>T: definirConfirmacao(pendente)
    Note over U,T: "Remover da comissão" abre o diálogo com "Cancelar" e "Remover"
    U->>T: executar(acao, "Integrante removido da comissão.")
    T->>A: api("/comissoes/:id/membros/:usuarioId", "DELETE")
    Note over A,C: DELETE /comissoes/:id/membros/:usuarioId
    A->>C: removerMembro(id, usuarioId, req)
    C->>S: removerMembro(id, membroId, usuarioId)
    S->>S: validarEquipe(id, membroId, usuarioId)
    S->>S: validarId(membroId)
    S->>S: autorizarComissao(id, usuarioId)
    S->>M: findById(id)
    M-->>S: comissao
    Note over S: Recusa comissão inativa (400) e pessoa fora de membros (404)
    S->>S: exigirResponsavelUnico(comissao)
    Note over S: Recusa remover o RESPONSAVEL<br/>("Substitua o responsável antes de removê-lo", 400)
    Note over S: membros = itens atuais sem o de usuario_id igual a membroId
    S->>S: gravarEquipe(comissao, membros)
    Note over S,M: filtro = _id, ativo e membros iguais aos da comissão lida<br/>opcoes = returnDocument after e runValidators true
    S->>M: findOneAndUpdate(filtro, { $set: { membros } }, opcoes)
    M-->>S: comissao atualizada
    S->>S: popular(comissao)
    S-->>C: comissao
    C-->>A: 200 comissao
    A-->>T: Resultado com a comissao
    T->>T: definirComissao(comissao)
    T->>T: definirSucesso("Integrante removido da comissão.")
```

## Rastreabilidade

SD-COM-01 a SD-COM-09 correspondem, na mesma ordem, a UC-COM-01 a UC-COM-09 e a RF-COM-01 a RF-COM-09.
