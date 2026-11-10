# Diagramas de sequência — Reuniões

> Padrão da disciplina · Módulo REU · Revisado em 10/11/2026

## Resumo

Há um diagrama para cada requisito funcional, com o mesmo número. As linhas de vida são classes e módulos que existem no código: as telas do aplicativo, o módulo `api`, o `ReunioesController`, o `ReunioesService` e os models do Mongoose. Toda mensagem é a chamada de um método existente, com seus parâmetros, e as mensagens são numeradas na ordem em que acontecem. Regras que não são métodos, como comparações de horário, aparecem em notas.

Em todos os diagramas, a chamada do módulo `api` ao controller é uma requisição HTTP que passa antes por `ApiKeyGuard`, `JwtAuthGuard` e `UsuarioComumGuard`; a rota está indicada na nota.

## SD-REU-01 — Agendar reunião

```mermaid
sequenceDiagram
    autonumber
    actor U as Presidente
    participant F as FormularioReuniao
    participant N as TelaNovaReuniao
    participant A as api
    participant C as ReunioesController
    participant S as ReunioesService
    participant CM as ComissaoModel
    participant M as ReuniaoModel
    U->>F: salvar()
    F->>F: calcularHorario(dados)
    F->>N: aoSalvar(envio)
    N->>A: api("/reunioes", "POST", dados)
    Note over A,C: POST /reunioes
    A->>C: criar(dto, req)
    C->>S: criar(dto, usuarioId)
    S->>S: autorizarResponsavel(comissaoId, usuarioId)
    S->>CM: findById(comissaoId)
    CM-->>S: comissao
    S->>S: lerData(dto.inicio, mensagem)
    S->>S: lerData(dto.fim, mensagem)
    S->>S: validarHorario(inicio, fim)
    Note over S: Confere se cada convidado está em comissao.membros
    S->>M: create(reuniao)
    M-->>S: reuniao
    S->>S: popular(reuniao)
    S->>S: apresentar(reuniao, usuarioId)
    S-->>C: reuniao
    C-->>A: 201 reuniao
    A-->>N: Resultado com a reuniao
    N->>N: router.replace("/reunioes/[id]", id)
    N-->>F: null
```

## SD-REU-02 — Consultar agenda

```mermaid
sequenceDiagram
    autonumber
    actor U as Participante
    participant T as TelaAgenda
    participant A as api
    participant C as ReunioesController
    participant S as ReunioesService
    participant M as ReuniaoModel
    U->>T: carregar()
    T->>A: api("/reunioes")
    Note over A,C: GET /reunioes?de=&ate=&comissao_id=
    A->>C: listar(req, de, ate, comissaoId)
    C->>S: listar(usuarioId, filtros)
    S->>S: validarUsuario(usuarioId)
    Note over S: Consulta: participantes.usuario_id igual ao usuário,<br/>fim maior ou igual a "de", inicio menor ou igual a "ate",<br/>comissao_id igual ao informado
    S->>M: find(consulta)
    S->>M: populate("comissao_id", "nome organizacao_id")
    S->>M: populate("participantes.usuario_id", "nome email")
    S->>M: sort(inicio crescente)
    M-->>S: reunioes
    S->>S: apresentar(reuniao, usuarioId)
    S-->>C: reunioes
    C-->>A: 200 reunioes
    A-->>T: Resultado com as reunioes
    T->>T: definirReunioes(lista.dados)
```

## SD-REU-03 — Visualizar reunião

```mermaid
sequenceDiagram
    autonumber
    actor U as Participante
    participant T as TelaReuniao
    participant A as api
    participant C as ReunioesController
    participant S as ReunioesService
    participant M as ReuniaoModel
    U->>T: buscar()
    T->>A: api("/reunioes/:id")
    Note over A,C: GET /reunioes/:id
    A->>C: buscarPorId(id, req)
    C->>S: buscarPorId(id, usuarioId)
    S->>S: carregar(id, usuarioId)
    S->>M: findById(id)
    M-->>S: reuniao
    Note over S: Sem o usuário em participantes, responde 403
    S->>S: popular(reuniao)
    S->>S: apresentar(reuniao, usuarioId)
    S-->>C: reuniao
    C-->>A: 200 reuniao
    A-->>T: Resultado com a reuniao
    T->>T: definirReuniao(resultado.dados)
    Note over T: buscar() é chamada de novo a cada 30 segundos
```

## SD-REU-04 — Cancelar reunião

```mermaid
sequenceDiagram
    autonumber
    actor U as Presidente
    participant T as TelaReuniao
    participant A as api
    participant C as ReunioesController
    participant S as ReunioesService
    participant M as ReuniaoModel
    Note over U,T: Mais opções, Cancelar reunião e confirmação
    U->>T: executar("cancelar", acao, "Reunião cancelada.")
    T->>A: api("/reunioes/:id", "PATCH", situacao CANCELADA)
    Note over A,C: PATCH /reunioes/:id
    A->>C: atualizar(id, dto, req)
    C->>S: atualizar(id, dto, usuarioId)
    S->>S: cancelar(id, usuarioId)
    S->>S: carregarComoDono(id, usuarioId)
    S->>M: findById(id)
    M-->>S: reuniao
    S->>S: exigirAgendada(reuniao)
    Note over S: Recusa se reuniao.fim já passou
    S->>M: findOneAndUpdate(filtro com _id, criada_por, AGENDADA e fim futuro, $set situacao)
    alt reunião encontrada pelo filtro
        M-->>S: reuniao cancelada
        S->>S: responder(atualizada, usuarioId, conflito)
        S-->>C: reuniao
        C-->>A: 200 reuniao
        A-->>T: Resultado com a reuniao
    else reunião mudou no intervalo
        M-->>S: null
        S-->>C: ConflictException
        C-->>A: 409 A reunião mudou
        A-->>T: Resultado com o erro
    end
```

## SD-REU-05 — Responder convite

```mermaid
sequenceDiagram
    autonumber
    actor U as Participante
    participant T as TelaAgenda
    participant A as api
    participant C as ReunioesController
    participant S as ReunioesService
    participant M as ReuniaoModel
    U->>T: responder(reuniao, aceito)
    T->>A: api("/reunioes/:id/convite", "PATCH", aceito)
    Note over A,C: PATCH /reunioes/:id/convite
    A->>C: responderConvite(id, dto, req)
    C->>S: responderConvite(id, aceito, usuarioId)
    S->>S: carregar(id, usuarioId)
    S->>M: findById(id)
    M-->>S: reuniao
    Note over S: Recusa quem agendou a reunião
    S->>S: exigirAgendada(reuniao)
    Note over S: Recusa se a reunião terminou ou,<br/>na recusa do convite, se já há entrada marcada
    S->>M: findOneAndUpdate(filtro com _id, AGENDADA e participantes.usuario_id, $set participantes.$.convite)
    M-->>S: reuniao atualizada
    S->>S: responder(atualizada, usuarioId, conflito)
    S-->>C: reuniao
    C-->>A: 200 reuniao
    A-->>T: Resultado com a reuniao
    T->>T: definirReunioes(lista atualizada)
```

## SD-REU-06 — Registrar entrada

```mermaid
sequenceDiagram
    autonumber
    actor U as Participante
    participant T as TelaReuniao
    participant A as api
    participant C as ReunioesController
    participant S as ReunioesService
    participant M as ReuniaoModel
    U->>T: executar("entrada", acao, "Entrada marcada.")
    T->>A: api("/reunioes/:id/entrada", "POST")
    Note over A,C: POST /reunioes/:id/entrada
    A->>C: marcarEntrada(id, req)
    C->>S: marcarEntrada(id, usuarioId)
    S->>S: carregar(id, usuarioId)
    S->>M: findById(id)
    M-->>S: reuniao
    S->>S: exigirAgendada(reuniao)
    Note over S: Recusa antes de inicio, depois de fim,<br/>com convite RECUSADO ou com presenca já criada
    S->>M: findOneAndUpdate(filtro com participante sem presenca e convite não recusado, $set participantes.$.presenca)
    M-->>S: reuniao atualizada
    S->>S: responder(atualizada, usuarioId, conflito)
    S-->>C: reuniao
    C-->>A: 201 reuniao
    A-->>T: Resultado com a reuniao
    T->>T: definirReuniao(resultado.dados)
```

## SD-REU-07 — Registrar saída

```mermaid
sequenceDiagram
    autonumber
    actor U as Participante
    participant T as TelaReuniao
    participant A as api
    participant C as ReunioesController
    participant S as ReunioesService
    participant M as ReuniaoModel
    U->>T: executar("saida", acao, "Saída marcada.")
    T->>A: api("/reunioes/:id/saida", "POST")
    Note over A,C: POST /reunioes/:id/saida
    A->>C: marcarSaida(id, req)
    C->>S: marcarSaida(id, usuarioId)
    S->>S: carregar(id, usuarioId)
    S->>M: findById(id)
    M-->>S: reuniao
    S->>S: exigirAgendada(reuniao)
    Note over S: Recusa sem entrada, com presença recusada ou com saída já marcada.<br/>O horário gravado é o menor entre agora e reuniao.fim
    S->>M: findOneAndUpdate(filtro com participante sem presenca.saida, $set participantes.$.presenca.saida)
    M-->>S: reuniao atualizada
    S->>S: responder(atualizada, usuarioId, conflito)
    S-->>C: reuniao
    C-->>A: 201 reuniao
    A-->>T: Resultado com a reuniao
    T->>T: definirReuniao(resultado.dados)
```

## SD-REU-08 — Confirmar ou recusar presença

```mermaid
sequenceDiagram
    autonumber
    actor U as Presidente
    participant T as TelaReuniao
    participant A as api
    participant C as ReunioesController
    participant S as ReunioesService
    participant M as ReuniaoModel
    U->>T: marcarPresenca(pessoa, situacao)
    T->>T: executar(chave, acao)
    T->>A: api("/reunioes/:id/presencas/:usuarioId", "PATCH", situacao)
    Note over A,C: PATCH /reunioes/:id/presencas/:usuarioId
    A->>C: atualizarPresenca(id, usuarioId, dto, req)
    C->>S: atualizarPresenca(id, participanteId, situacao, usuarioId)
    S->>S: validarId(participanteId)
    S->>S: carregarComoDono(id, usuarioId)
    S->>M: findById(id)
    M-->>S: reuniao
    S->>S: exigirAgendada(reuniao)
    Note over S: Recusa antes de reuniao.inicio, se a pessoa não participa<br/>ou ao recusar quem não marcou entrada
    alt participante já tem presenca
        S->>M: findOneAndUpdate(filtro com participante e presenca existente, $set participantes.$.presenca.situacao)
    else participante sem presenca
        S->>M: findOneAndUpdate(filtro com participante sem presenca, $set participantes.$.presenca)
    end
    M-->>S: reuniao atualizada
    S->>S: responder(atualizada, usuarioId, conflito)
    S-->>C: reuniao
    C-->>A: 200 reuniao
    A-->>T: Resultado com a reuniao
```

## SD-REU-09 — Gerar lista de presença

```mermaid
sequenceDiagram
    autonumber
    actor U as Participante
    participant T as TelaRegistrosReuniao
    participant A as api
    participant C as ReunioesController
    participant S as ReunioesService
    participant M as ReuniaoModel
    participant UM as UsuarioModel
    U->>T: carregarLista()
    T->>A: api("/reunioes/:id/lista-presenca")
    Note over A,C: GET /reunioes/:id/lista-presenca
    A->>C: gerarListaPresenca(id, req)
    C->>S: gerarListaPresenca(id, usuarioId)
    S->>S: carregar(id, usuarioId)
    S->>M: findById(id)
    M-->>S: reuniao
    S->>UM: find(_id dentro de participantes.usuario_id)
    S->>UM: select("nome")
    UM-->>S: pessoas
    Note over S: Para cada participante: situacao de presenca ou SEM_REGISTRO,<br/>minutos entre entrada e saida, arredondados para baixo
    S-->>C: lista de presença
    C-->>A: 200 lista
    A-->>T: Resultado com a lista
    T->>T: definirLista(resposta.dados)
    U->>T: compartilhar()
    T->>T: Share.share(message)
```

## SD-REU-13 — Editar reunião

```mermaid
sequenceDiagram
    autonumber
    actor U as Presidente
    participant F as FormularioReuniao
    participant E as TelaEditarReuniao
    participant A as api
    participant C as ReunioesController
    participant S as ReunioesService
    participant M as ReuniaoModel
    Note over F,E: O formulário abre preenchido com a reunião buscada por api("/reunioes/:id")
    U->>F: salvar()
    F->>F: calcularHorario(dados)
    F->>E: aoSalvar(envio)
    E->>A: api("/reunioes/:id", "PATCH", campos)
    Note over A,C: PATCH /reunioes/:id
    A->>C: atualizar(id, dto, req)
    C->>S: atualizar(id, dto, usuarioId)
    S->>S: editar(id, campos, usuarioId)
    S->>S: carregarComoDono(id, usuarioId)
    S->>M: findById(id)
    M-->>S: reuniao
    S->>S: exigirAgendada(reuniao)
    alt data ou horário mudaram
        Note over S: Recusa se reuniao.inicio já passou
        S->>S: validarHorario(inicio, fim)
    end
    opt dto com convidados_ids
        S->>S: autorizarResponsavel(comissaoId, usuarioId)
    end
    S->>M: findOneAndUpdate(filtro com _id, AGENDADA, fim futuro e atualizado_em, $set campos)
    alt reunião encontrada pelo filtro
        M-->>S: reuniao atualizada
        S->>S: responder(atualizada, usuarioId, conflito)
        S-->>C: reuniao
        C-->>A: 200 reuniao
        A-->>E: Resultado com a reuniao
        E-->>F: null
        F->>E: aoConcluir()
    else reunião mudou durante a edição
        M-->>S: null
        S-->>C: ConflictException
        C-->>A: 409 A reunião mudou
        A-->>E: Resultado com o erro
        E-->>F: mensagem de erro
    end
```

## SD-REU-14 — Anexar arquivo à pauta

```mermaid
sequenceDiagram
    autonumber
    actor U as Presidente
    participant T as TelaReuniao
    participant X as anexos
    participant C as ReunioesController
    participant S as ReunioesService
    participant G as GridFSBucket
    participant M as ReuniaoModel
    U->>T: adicionarAnexos()
    T->>X: escolherArquivos()
    X-->>T: arquivos escolhidos
    T->>X: validarArquivos(novos, jaAnexados)
    X-->>T: null
    loop para cada arquivo
        T->>X: enviarAnexo(reuniaoId, arquivo)
        Note over X,C: POST /reunioes/:id/anexos, multipart, limite de 10 MB no FileInterceptor
        X->>C: adicionarAnexo(id, arquivo, req)
        C->>S: adicionarAnexo(id, arquivo, usuarioId)
        S->>S: carregarComoDono(id, usuarioId)
        S->>M: findById(id)
        M-->>S: reuniao
        S->>S: exigirAnexosAbertos(reuniao)
        Note over S: Confere o tipo em TIPOS_ANEXO e o limite MAXIMO_ANEXOS
        S->>S: arquivos()
        S->>G: openUploadStream(nome, opcoes)
        G-->>S: envio com o id do arquivo
        S->>M: findOneAndUpdate(filtro com _id, AGENDADA e menos de 10 anexos, $push anexos)
        alt reunião encontrada pelo filtro
            M-->>S: reuniao atualizada
        else reunião mudou durante o envio
            M-->>S: null
            S->>G: delete(envio.id)
        end
        S->>S: responder(atualizada, usuarioId, conflito)
        S-->>C: reuniao
        C-->>X: 201 reuniao
        X-->>T: Resultado com a reuniao
    end
    T->>T: definirSucesso(mensagem)
```

## SD-REU-15 — Baixar anexo da pauta

```mermaid
sequenceDiagram
    autonumber
    actor U as Participante
    participant T as TelaReuniao
    participant X as anexos
    participant FS as File
    participant C as ReunioesController
    participant S as ReunioesService
    participant M as ReuniaoModel
    participant G as GridFSBucket
    U->>T: abrir(item)
    T->>X: abrirAnexo(reuniaoId, anexo)
    X->>FS: downloadFileAsync(url, destino, opcoes)
    Note over FS,C: GET /reunioes/:id/anexos/:anexoId
    FS->>C: abrirAnexo(id, anexoId, req)
    C->>S: abrirAnexo(id, anexoId, usuarioId)
    S->>S: carregar(id, usuarioId)
    S->>M: findById(id)
    M-->>S: reuniao
    S->>S: buscarAnexo(reuniao, anexoId)
    S->>S: arquivos()
    S->>G: openDownloadStream(anexo._id)
    G-->>S: fluxo de leitura
    S-->>C: anexo e conteudo
    C-->>FS: 200 StreamableFile
    FS-->>X: arquivo no cache
    X->>FS: canPreview(arquivo, opcoes)
    FS-->>X: true
    X->>FS: preview(arquivo, opcoes)
    X-->>T: null
```

## SD-REU-16 — Remover anexo da pauta

```mermaid
sequenceDiagram
    autonumber
    actor U as Presidente
    participant T as TelaReuniao
    participant X as anexos
    participant A as api
    participant C as ReunioesController
    participant S as ReunioesService
    participant M as ReuniaoModel
    participant G as GridFSBucket
    U->>T: confirmarRemocao(item)
    Note over U,T: O sistema pergunta "Remover o anexo?" e o presidente confirma
    T->>T: executar("anexo", acao, "Anexo removido.")
    T->>X: removerAnexo(reuniaoId, anexoId)
    X->>A: api("/reunioes/:id/anexos/:anexoId", "DELETE")
    Note over A,C: DELETE /reunioes/:id/anexos/:anexoId
    A->>C: removerAnexo(id, anexoId, req)
    C->>S: removerAnexo(id, anexoId, usuarioId)
    S->>S: carregarComoDono(id, usuarioId)
    S->>M: findById(id)
    M-->>S: reuniao
    S->>S: exigirAnexosAbertos(reuniao)
    S->>S: buscarAnexo(reuniao, anexoId)
    S->>M: findOneAndUpdate(filtro com _id, AGENDADA e anexos._id, $pull anexos)
    M-->>S: reuniao sem o anexo
    S->>S: arquivos()
    S->>G: delete(anexo._id)
    S->>S: responder(atualizada, usuarioId, conflito)
    S-->>C: reuniao
    C-->>A: 200 reuniao
    A-->>T: Resultado com a reuniao
```

## Rastreabilidade

SD-REU-01 a SD-REU-09 e SD-REU-13 a SD-REU-16 correspondem aos requisitos funcionais de mesmo número. Os diagramas da ata estão em [SD-ATA-01 a SD-ATA-08](../../4%20-%20Incremento3/atas/6%20-%20diagramas-sequencia-atas.md).
