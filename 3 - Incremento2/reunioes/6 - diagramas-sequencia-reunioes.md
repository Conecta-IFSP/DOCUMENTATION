# Diagramas de Sequência - Reuniões

Os participantes de cada diagrama são classes do sistema, e as mensagens são as chamadas de métodos entre elas. Os objetos `reuniaoModel`, `comissaoModel`, `organizacaoModel` e `usuarioModel` são as instâncias de `Model` usadas pelo `ReunioesService` para acessar as coleções `reunioes`, `comissoes`, `organizacoes` e `usuarios`. As notas descrevem o processamento feito dentro de um método.

---

## RF-01 - Agendar reunião

```mermaid
sequenceDiagram
    autonumber
    actor U as Responsável
    participant T as TelaNovaReuniao
    participant A as api
    participant K as ApiKeyGuard
    participant J as JwtAuthGuard
    participant G as UsuarioComumGuard
    participant P as ValidationPipe
    participant C as ReunioesController
    participant S as ReunioesService
    participant UM as usuarioModel
    participant CM as comissaoModel
    participant OM as organizacaoModel
    participant R as reuniaoModel

    U->>T: preenche os campos e seleciona "Agendar reunião"
    T->>T: salvar()
    Note over T: valida título, horários e local e monta inicio e fim em ISO 8601
    alt campo inválido
        T-->>U: exibe o erro abaixo do campo
    else campos válidos
        T->>A: api('/reunioes', 'POST', {comissao_id, titulo, local, inicio, fim, pauta, convidados_ids})
        A->>A: obterToken()
        A->>K: fetch(URL_API + '/reunioes', {method: 'POST', headers, body})
        K->>K: chavesIguais(recebida, esperada)
        alt chave ausente ou diferente
            K-->>A: 401 Unauthorized
        else chave válida
            K->>J: canActivate(context)
            J->>J: extrairToken(requisicao)
            J->>J: jwtService.verifyAsync(token)
            J->>UM: findOne({_id: sub, ativo: true}).exec()
            UM-->>J: usuário
            alt token inválido ou conta inativa
                J-->>A: 401 Unauthorized
            else token válido
                J->>G: canActivate(context)
                G->>P: transform(corpo, metadata)
                P->>C: criar(dto, req)
                C->>S: criar(dto, req.usuario.sub)
                S->>S: autorizarResponsavel(dto.comissao_id, usuarioId)
                S->>UM: findById(usuarioId).exec()
                S->>CM: findById(comissaoId).exec()
                CM-->>S: comissão
                S->>OM: findById(comissao.organizacao_id).exec()
                OM-->>S: organização
                alt não é responsável, comissão inativa ou vínculo não aprovado
                    S-->>A: 403 Forbidden ou 400 Bad Request
                else autorizado
                    S->>S: lerData(dto.inicio, mensagem)
                    S->>S: lerData(dto.fim, mensagem)
                    S->>S: validarHorario(inicio, fim)
                    Note over S: verifica que cada convidado está em comissao.membros e monta participantes
                    S->>R: create({comissao_id, titulo, local, inicio, fim, pauta, criada_por, situacao: AGENDADA, participantes})
                    R-->>S: reunião
                    S->>S: popular(reuniao)
                    S->>S: apresentar(reuniao, usuarioId)
                    S-->>C: reunião
                    C-->>A: 201 Created, reunião
                    A-->>T: {ok: true, dados}
                    T-->>U: abre a tela da reunião
                end
            end
        end
    end
```

---

## RF-02 - Consultar agenda de reuniões

```mermaid
sequenceDiagram
    autonumber
    actor U as Participante
    participant T as TelaAgenda
    participant TC as TelaComissao
    participant A as api
    participant K as ApiKeyGuard
    participant J as JwtAuthGuard
    participant G as UsuarioComumGuard
    participant C as ReunioesController
    participant S as ReunioesService
    participant UM as usuarioModel
    participant R as reuniaoModel

    alt aba Agenda
        U->>T: abre a aba Agenda
        T->>T: carregar()
        T->>A: api('/reunioes')
    else aba Reuniões da comissão
        U->>TC: abre a aba Reuniões
        TC->>TC: buscarReunioes()
        TC->>A: api('/reunioes?comissao_id=' + id)
    end
    A->>A: obterToken()
    A->>K: fetch(URL_API + '/reunioes', {method: 'GET', headers})
    K->>K: chavesIguais(recebida, esperada)
    K->>J: canActivate(context)
    J->>J: jwtService.verifyAsync(token)
    J->>UM: findOne({_id: sub, ativo: true}).exec()
    J->>G: canActivate(context)
    G->>C: listar(req, de, ate, comissaoId)
    C->>S: listar(req.usuario.sub, {de, ate, comissao_id})
    S->>S: validarUsuario(usuarioId)
    S->>UM: findById(usuarioId).exec()
    alt filtro de data ou comissão inválido
        S-->>A: 400 Bad Request
    else filtros válidos
        S->>R: find(consulta).populate('comissao_id').populate('participantes.usuario_id').sort({inicio: 1}).exec()
        Note over R: consulta com participantes.usuario_id e, se informados, fim >= de, inicio <= ate e comissao_id
        R-->>S: reuniões
        S->>S: apresentar(reuniao, usuarioId)
        S-->>C: reuniões
        C-->>A: 200 OK, reuniões
        alt aba Agenda
            A-->>T: {ok: true, dados}
            T-->>U: marca os dias com reunião e exibe as do dia selecionado
            opt seleciona outro dia
                U->>T: escolherDia(dia)
                T-->>U: exibe as reuniões do dia
            end
        else aba Reuniões da comissão
            A-->>TC: {ok: true, dados}
            TC-->>U: exibe Próximas e Anteriores
        end
    end
```

---

## RF-03 - Visualizar reunião

```mermaid
sequenceDiagram
    autonumber
    actor U as Participante
    participant T as TelaReuniao
    participant A as api
    participant K as ApiKeyGuard
    participant J as JwtAuthGuard
    participant G as UsuarioComumGuard
    participant C as ReunioesController
    participant S as ReunioesService
    participant UM as usuarioModel
    participant R as reuniaoModel

    U->>T: seleciona uma reunião
    loop ao abrir e a cada 30 segundos com a tela em foco
        T->>T: buscar()
        T->>A: api('/reunioes/' + id)
        A->>A: obterToken()
        A->>K: fetch(URL_API + '/reunioes/' + id, {method: 'GET', headers})
        K->>K: chavesIguais(recebida, esperada)
        K->>J: canActivate(context)
        J->>J: jwtService.verifyAsync(token)
        J->>UM: findOne({_id: sub, ativo: true}).exec()
        J->>G: canActivate(context)
        G->>C: buscarPorId(id, req)
        C->>S: buscarPorId(id, req.usuario.sub)
        S->>S: carregar(id, usuarioId)
        S->>UM: findById(usuarioId).exec()
        S->>R: findById(id).exec()
        R-->>S: reunião
        alt reunião inexistente
            S-->>A: 404 Not Found
        else usuário não está em participantes
            S-->>A: 403 Forbidden
            A-->>T: {ok: false, erro}
            T-->>U: exibe "Você não participa desta reunião"
        else participante
            S->>S: popular(reuniao)
            S->>S: apresentar(reuniao, usuarioId)
            S-->>C: reunião
            C-->>A: 200 OK, reunião
            A-->>T: {ok: true, dados}
            Note over T: calcula a situação exibida a partir de situacao, inicio, fim e do convite
            T-->>U: exibe os dados, os cartões de convite e presença e as abas Presença e Pauta
        end
    end
```

---

## RF-04 - Cancelar reunião

```mermaid
sequenceDiagram
    autonumber
    actor U as Organizador
    participant T as TelaReuniao
    participant A as api
    participant K as ApiKeyGuard
    participant J as JwtAuthGuard
    participant G as UsuarioComumGuard
    participant P as ValidationPipe
    participant C as ReunioesController
    participant S as ReunioesService
    participant UM as usuarioModel
    participant R as reuniaoModel

    U->>T: seleciona "Mais opções" e "Cancelar reunião"
    T-->>U: pede a confirmação do cancelamento com o título da reunião
    U->>T: confirma
    T->>T: executar('cancelar', acao, 'Reunião cancelada.')
    T->>A: api('/reunioes/' + id, 'PATCH', {situacao: 'CANCELADA'})
    A->>A: obterToken()
    A->>K: fetch(URL_API + '/reunioes/' + id, {method: 'PATCH', headers, body})
    K->>K: chavesIguais(recebida, esperada)
    K->>J: canActivate(context)
    J->>J: jwtService.verifyAsync(token)
    J->>UM: findOne({_id: sub, ativo: true}).exec()
    J->>G: canActivate(context)
    G->>P: transform(corpo, metadata)
    P->>C: atualizar(id, dto, req)
    C->>S: atualizar(id, dto, req.usuario.sub)
    S->>S: carregarComoDono(id, usuarioId)
    S->>UM: findById(usuarioId).exec()
    S->>R: findById(id).exec()
    R-->>S: reunião
    alt usuário não é o organizador
        S-->>A: 403 Forbidden
    else organizador
        S->>S: exigirAgendada(reuniao)
        alt reunião já terminou
            S-->>A: 400 Bad Request
        else reunião em aberto
            S->>R: findOneAndUpdate({_id, criada_por, situacao: AGENDADA, fim: {$gt: agora}}, {$set: {situacao: CANCELADA}}).exec()
            R-->>S: reunião ou null
            S->>S: responder(atualizada, usuarioId, conflito)
            alt reunião alterada antes da gravação
                S-->>A: 409 Conflict
            else gravada
                S-->>C: reunião
                C-->>A: 200 OK, reunião
                A-->>T: {ok: true, dados}
                T-->>U: exibe "Reunião cancelada."
            end
        end
    end
```

---

## RF-05 - Responder convite

```mermaid
sequenceDiagram
    autonumber
    actor U as Participante
    participant T as TelaReuniao
    participant A as api
    participant K as ApiKeyGuard
    participant J as JwtAuthGuard
    participant G as UsuarioComumGuard
    participant P as ValidationPipe
    participant C as ReunioesController
    participant S as ReunioesService
    participant UM as usuarioModel
    participant R as reuniaoModel

    U->>T: seleciona "Aceitar" ou "Recusar" no cartão Convite
    T->>T: executar('aceitar' ou 'recusar', acao)
    T->>A: api('/reunioes/' + id + '/convite', 'PATCH', {aceito})
    A->>A: obterToken()
    A->>K: fetch(URL_API + '/reunioes/' + id + '/convite', {method: 'PATCH', headers, body})
    K->>K: chavesIguais(recebida, esperada)
    K->>J: canActivate(context)
    J->>J: jwtService.verifyAsync(token)
    J->>UM: findOne({_id: sub, ativo: true}).exec()
    J->>G: canActivate(context)
    G->>P: transform(corpo, metadata)
    P->>C: responderConvite(id, dto, req)
    C->>S: responderConvite(id, dto.aceito, req.usuario.sub)
    S->>S: carregar(id, usuarioId)
    S->>UM: findById(usuarioId).exec()
    S->>R: findById(id).exec()
    R-->>S: reunião
    alt usuário é o organizador
        S-->>A: 403 Forbidden
    else participante convidado
        S->>S: exigirAgendada(reuniao)
        alt reunião cancelada ou encerrada, ou recusa depois da entrada
            S-->>A: 400 Bad Request
        else pode responder
            Note over S,R: na recusa, o filtro exige também presenca inexistente
            S->>R: findOneAndUpdate({_id, situacao: AGENDADA, participantes: {$elemMatch: {usuario_id}}}, {$set: {'participantes.$.convite': ACEITO ou RECUSADO}}).exec()
            R-->>S: reunião
            S->>S: responder(atualizada, usuarioId, conflito)
            S-->>C: reunião
            C-->>A: 200 OK, reunião
            A-->>T: {ok: true, dados}
            T-->>U: atualiza o cartão do convite
        end
    end
```

Na agenda, o método `TelaAgenda.responder(reuniao, aceito)` envia a mesma requisição a partir do item da lista.

---

## RF-06 - Registrar entrada

```mermaid
sequenceDiagram
    autonumber
    actor U as Participante
    participant T as TelaReuniao
    participant A as api
    participant K as ApiKeyGuard
    participant J as JwtAuthGuard
    participant G as UsuarioComumGuard
    participant C as ReunioesController
    participant S as ReunioesService
    participant UM as usuarioModel
    participant R as reuniaoModel

    U->>T: seleciona "Marcar entrada"
    T->>T: executar('entrada', acao, 'Entrada marcada.')
    T->>A: api('/reunioes/' + id + '/entrada', 'POST')
    A->>A: obterToken()
    A->>K: fetch(URL_API + '/reunioes/' + id + '/entrada', {method: 'POST', headers})
    K->>K: chavesIguais(recebida, esperada)
    K->>J: canActivate(context)
    J->>J: jwtService.verifyAsync(token)
    J->>UM: findOne({_id: sub, ativo: true}).exec()
    J->>G: canActivate(context)
    G->>C: marcarEntrada(id, req)
    C->>S: marcarEntrada(id, req.usuario.sub)
    S->>S: carregar(id, usuarioId)
    S->>UM: findById(usuarioId).exec()
    S->>R: findById(id).exec()
    R-->>S: reunião
    S->>S: exigirAgendada(reuniao)
    alt antes do início, depois do término ou convite recusado
        S-->>A: 400 Bad Request
    else entrada já registrada
        S-->>A: 409 Conflict
    else pode registrar
        S->>R: findOneAndUpdate({_id, situacao: AGENDADA, participantes: {$elemMatch: {usuario_id, convite: {$ne: RECUSADO}, presenca: {$exists: false}}}}, {$set: {'participantes.$.presenca': {entrada, situacao, marcada_por: PROPRIO}}}).exec()
        Note over S,R: situacao é CONFIRMADA para o organizador e AGUARDANDO para os demais
        R-->>S: reunião
        S->>S: responder(atualizada, usuarioId, conflito)
        S-->>C: reunião
        C-->>A: 200 OK, reunião
        A-->>T: {ok: true, dados}
        T-->>U: exibe "Entrada marcada." e a situação da presença
    end
```

---

## RF-07 - Registrar saída

```mermaid
sequenceDiagram
    autonumber
    actor U as Participante
    participant T as TelaReuniao
    participant A as api
    participant K as ApiKeyGuard
    participant J as JwtAuthGuard
    participant G as UsuarioComumGuard
    participant C as ReunioesController
    participant S as ReunioesService
    participant UM as usuarioModel
    participant R as reuniaoModel

    U->>T: seleciona "Marcar saída"
    T->>T: executar('saida', acao, 'Saída marcada.')
    T->>A: api('/reunioes/' + id + '/saida', 'POST')
    A->>A: obterToken()
    A->>K: fetch(URL_API + '/reunioes/' + id + '/saida', {method: 'POST', headers})
    K->>K: chavesIguais(recebida, esperada)
    K->>J: canActivate(context)
    J->>J: jwtService.verifyAsync(token)
    J->>UM: findOne({_id: sub, ativo: true}).exec()
    J->>G: canActivate(context)
    G->>C: marcarSaida(id, req)
    C->>S: marcarSaida(id, req.usuario.sub)
    S->>S: carregar(id, usuarioId)
    S->>UM: findById(usuarioId).exec()
    S->>R: findById(id).exec()
    R-->>S: reunião
    S->>S: exigirAgendada(reuniao)
    alt sem presença ou presença recusada
        S-->>A: 400 Bad Request
    else saída já registrada
        S-->>A: 409 Conflict
    else pode registrar
        Note over S: saida recebe a data e hora atuais ou reuniao.fim, o que for menor
        S->>R: findOneAndUpdate({_id, participantes: {$elemMatch: {usuario_id, 'presenca.saida': {$exists: false}, 'presenca.situacao': {$ne: RECUSADA}}}}, {$set: {'participantes.$.presenca.saida': saida}}).exec()
        R-->>S: reunião
        S->>S: responder(atualizada, usuarioId, conflito)
        S-->>C: reunião
        C-->>A: 200 OK, reunião
        A-->>T: {ok: true, dados}
        T-->>U: exibe "Saída marcada." e o período
    end
```

---

## RF-08 - Confirmar ou recusar presença

```mermaid
sequenceDiagram
    autonumber
    actor U as Organizador
    participant T as TelaReuniao
    participant A as api
    participant K as ApiKeyGuard
    participant J as JwtAuthGuard
    participant G as UsuarioComumGuard
    participant P as ValidationPipe
    participant C as ReunioesController
    participant S as ReunioesService
    participant UM as usuarioModel
    participant R as reuniaoModel

    U->>T: seleciona confirmar, recusar ou "Marcar presente" ao lado de um participante
    T->>T: marcarPresenca(pessoa, situacao)
    T->>A: api('/reunioes/' + id + '/presencas/' + usuarioId, 'PATCH', {situacao})
    A->>A: obterToken()
    A->>K: fetch(URL_API + '/reunioes/' + id + '/presencas/' + usuarioId, {method: 'PATCH', headers, body})
    K->>K: chavesIguais(recebida, esperada)
    K->>J: canActivate(context)
    J->>J: jwtService.verifyAsync(token)
    J->>UM: findOne({_id: sub, ativo: true}).exec()
    J->>G: canActivate(context)
    G->>P: transform(corpo, metadata)
    P->>C: atualizarPresenca(id, usuarioId, dto, req)
    C->>S: atualizarPresenca(id, usuarioId, dto.situacao, req.usuario.sub)
    S->>S: validarId(participanteId)
    S->>S: carregarComoDono(id, organizadorId)
    S->>UM: findById(organizadorId).exec()
    S->>R: findById(id).exec()
    R-->>S: reunião
    S->>S: exigirAgendada(reuniao)
    alt antes do início, pessoa de fora ou recusa sem entrada
        S-->>A: 400 Bad Request ou 404 Not Found
    else participante com entrada registrada
        S->>R: findOneAndUpdate({_id, situacao: AGENDADA, participantes: {$elemMatch: {usuario_id, presenca: {$exists: true}}}}, {$set: {'participantes.$.presenca.situacao': situacao}}).exec()
        R-->>S: reunião
    else participante sem entrada marcado como presente
        S->>R: findOneAndUpdate({_id, situacao: AGENDADA, participantes: {$elemMatch: {usuario_id, presenca: {$exists: false}}}}, {$set: {'participantes.$.presenca': {entrada: inicio, situacao: CONFIRMADA, marcada_por: RESPONSAVEL}}}).exec()
        R-->>S: reunião
    end
    S->>S: responder(atualizada, organizadorId, conflito)
    S-->>C: reunião
    C-->>A: 200 OK, reunião
    A-->>T: {ok: true, dados}
    T-->>U: move o participante para Presentes ou Ausentes
```

---

## RF-09 - Gerar lista de presença

```mermaid
sequenceDiagram
    autonumber
    actor U as Participante
    participant T as TelaRegistrosReuniao
    participant A as api
    participant K as ApiKeyGuard
    participant J as JwtAuthGuard
    participant G as UsuarioComumGuard
    participant C as ReunioesController
    participant S as ReunioesService
    participant UM as usuarioModel
    participant R as reuniaoModel

    U->>T: seleciona a aba Presença
    T->>T: carregarLista()
    T->>A: api('/reunioes/' + id + '/lista-presenca')
    A->>A: obterToken()
    A->>K: fetch(URL_API + '/reunioes/' + id + '/lista-presenca', {method: 'GET', headers})
    K->>K: chavesIguais(recebida, esperada)
    K->>J: canActivate(context)
    J->>J: jwtService.verifyAsync(token)
    J->>UM: findOne({_id: sub, ativo: true}).exec()
    J->>G: canActivate(context)
    G->>C: gerarListaPresenca(id, req)
    C->>S: gerarListaPresenca(id, req.usuario.sub)
    S->>S: carregar(id, usuarioId)
    S->>UM: findById(usuarioId).exec()
    S->>R: findById(id).exec()
    R-->>S: reunião
    alt usuário não está em participantes
        S-->>A: 403 Forbidden
    else participante
        S->>UM: find({_id: {$in: participantes.usuario_id}}).select('nome').exec()
        UM-->>S: nomes
        Note over S: monta nome, convite, situacao ou SEM_REGISTRO, entrada, saida e minutos de cada participante
        S-->>C: lista com gerada_em
        C-->>A: 200 OK, lista
        A-->>T: {ok: true, dados}
        T-->>U: exibe os presentes e o período de cada participante
        opt seleciona "Compartilhar"
            U->>T: seleciona "Compartilhar"
            T->>T: compartilhar()
            T-->>U: abre o menu de compartilhamento do aparelho com a lista em texto
        end
    end
```

