# Diagramas de sequência — Atas de reunião

> Padrão da disciplina · Módulo ATA · Revisado em 10/11/2026

## Resumo

Há um diagrama para cada requisito de [Requisitos funcionais](2%20-%20requisitos-funcionais-atas.md), com o mesmo número. As linhas de vida são classes e módulos que existem no código: as telas do aplicativo, o módulo `api`, o `ReunioesController`, o `ReunioesService` e os models do Mongoose. Toda mensagem é a chamada de um método existente, com seus parâmetros, e as mensagens são numeradas na ordem em que acontecem. Regras que não são métodos, como comparações de versão, aparecem em notas.

Em todos os diagramas, a chamada do módulo `api` ao controller é uma requisição HTTP que passa antes por `ApiKeyGuard`, `JwtAuthGuard` e `UsuarioComumGuard`; a rota está indicada na nota.

## SD-ATA-01 — Salvar rascunho da ata

```mermaid
sequenceDiagram
    autonumber
    actor U as Presidente
    participant T as TelaRegistrosReuniao
    participant A as api
    participant C as ReunioesController
    participant S as ReunioesService
    participant M as ReuniaoModel
    U->>T: handleSubmit(dados)
    T->>T: gravar("/ata", "PATCH", corpo, "Rascunho salvo.")
    T->>A: api("/reunioes/:id/ata", "PATCH", corpo)
    Note over A,C: PATCH /reunioes/:id/ata
    A->>C: salvarAta(id, dto, req)
    C->>S: salvarAta(id, dto, usuarioId)
    S->>S: carregar(id, usuarioId)
    S->>M: findById(id)
    M-->>S: reuniao
    S->>S: autorizarResponsavel(comissaoId, usuarioId, "escrever a ata")
    S->>S: exigirEncerrada(reuniao)
    Note over S: Recusa ata REPROVADA ou já enviada,<br/>e dto.versao diferente de ata.versao
    S->>S: gravarAta(reuniao, ata, usuarioId)
    S->>M: findOneAndUpdate(filtro com _id, AGENDADA e a ata lida, $set ata)
    alt reunião encontrada pelo filtro
        M-->>S: reuniao atualizada
        S->>S: responder(atualizada, usuarioId, conflito)
        S-->>C: reuniao
        C-->>A: 200 reuniao
        A-->>T: Resultado com a reuniao
        T->>T: definirSucesso("Rascunho salvo.")
    else outra gravação chegou antes
        M-->>S: null
        S-->>C: ConflictException
        C-->>A: 409 A ata mudou
        A-->>T: Resultado com o erro
        T->>T: definirErro(resposta.erro)
    end
```

## SD-ATA-02 — Enviar ata para aprovação

```mermaid
sequenceDiagram
    autonumber
    actor U as Presidente
    participant T as TelaRegistrosReuniao
    participant L as autenticacaoLocal
    participant A as api
    participant C as ReunioesController
    participant S as ReunioesService
    participant M as ReuniaoModel
    U->>T: abrirEnvio()
    Note over U,T: A folha mostra os presentes e os prazos de 24 horas, 3 dias e 7 dias
    U->>T: definirPrazo(horas)
    U->>T: enviar()
    opt presidente esteve presente
        T->>L: autenticarComDispositivo()
        L-->>T: true
    end
    T->>T: gravar("/ata/publicar", "POST", corpo, mensagem)
    T->>A: api("/reunioes/:id/ata/publicar", "POST", corpo)
    Note over A,C: POST /reunioes/:id/ata/publicar
    A->>C: publicarAta(id, dto, req)
    C->>S: publicarAta(id, dto, usuarioId)
    S->>S: carregar(id, usuarioId)
    S->>M: findById(id)
    M-->>S: reuniao
    S->>S: autorizarResponsavel(comissaoId, usuarioId, "escrever a ata")
    S->>S: exigirEncerrada(reuniao)
    Note over S: aprovadores: participantes com presenca.situacao CONFIRMADA.<br/>prazo: horário do envio mais dto.prazo_horas
    S->>S: concluirSeAprovada(reuniao, enviada)
    S->>S: gravarAta(reuniao, ata, usuarioId)
    S->>M: findOneAndUpdate(filtro com _id, AGENDADA e a ata lida, $set ata)
    M-->>S: reuniao atualizada
    S->>S: responder(atualizada, usuarioId, conflito)
    S-->>C: reuniao
    C-->>A: 201 reuniao
    A-->>T: Resultado com a reuniao
    T->>T: definirSucesso("Ata enviada para aprovação.")
```

## SD-ATA-03 — Aprovar ata

```mermaid
sequenceDiagram
    autonumber
    actor U as Participante presente
    participant T as TelaRegistrosReuniao
    participant L as autenticacaoLocal
    participant A as api
    participant C as ReunioesController
    participant S as ReunioesService
    participant M as ReuniaoModel
    participant N as ContadorModel
    U->>T: aprovar()
    T->>L: dispositivoProtegido()
    alt aparelho com biometria ou PIN
        L-->>T: true
        T->>L: autenticarComDispositivo()
        L-->>T: true
    else aparelho sem bloqueio de tela
        L-->>T: false
        T->>T: definirConfirmacao(pendente)
        Note over U,T: O participante confirma no diálogo "Aprovar esta ata?"
    end
    T->>T: gravar("/ata/aprovar", "POST", corpo, "Ata aprovada.")
    T->>A: api("/reunioes/:id/ata/aprovar", "POST", corpo)
    Note over A,C: POST /reunioes/:id/ata/aprovar
    A->>C: aprovarAta(id, dto, req)
    C->>S: aprovarAta(id, versao, usuarioId)
    S->>S: avaliarAta(id, versao, usuarioId, avaliacao)
    S->>S: carregar(id, usuarioId)
    S->>M: findById(id)
    M-->>S: reuniao
    S->>S: exigirEncerrada(reuniao)
    Note over S: Recusa versão diferente, usuário fora de ata.aprovadores,<br/>decisão já registrada ou ata fora de EM_APROVACAO
    S->>S: concluirSeAprovada(reuniao, avaliada)
    opt todos os aprovadores aprovaram
        S->>S: proximoNumeroAta(comissaoId, ano)
        S->>N: findOneAndUpdate(filtro com _id da sequência, $inc valor, upsert)
        N-->>S: contador
    end
    S->>S: gravarAta(reuniao, ata, usuarioId)
    S->>M: findOneAndUpdate(filtro com _id, AGENDADA e a ata lida, $set ata)
    M-->>S: reuniao atualizada
    S->>S: responder(atualizada, usuarioId, conflito)
    S-->>C: reuniao
    C-->>A: 201 reuniao
    A-->>T: Resultado com a reuniao
    T->>T: definirSucesso("Ata aprovada.")
```

## SD-ATA-04 — Reprovar ata

```mermaid
sequenceDiagram
    autonumber
    actor U as Participante presente
    participant T as TelaRegistrosReuniao
    participant A as api
    participant C as ReunioesController
    participant S as ReunioesService
    participant M as ReuniaoModel
    U->>T: definirTrecho(paragrafo)
    U->>T: definirReprovacaoAberta(true)
    U->>T: definirJustificativa(texto)
    U->>T: reprovar()
    T->>T: gravar("/ata/reprovar", "POST", corpo, mensagem)
    T->>A: api("/reunioes/:id/ata/reprovar", "POST", corpo)
    Note over A,C: POST /reunioes/:id/ata/reprovar
    A->>C: reprovarAta(id, dto, req)
    C->>S: reprovarAta(id, dto, usuarioId)
    S->>S: avaliarAta(id, versao, usuarioId, avaliacao)
    S->>S: carregar(id, usuarioId)
    S->>M: findById(id)
    M-->>S: reuniao
    S->>S: exigirEncerrada(reuniao)
    Note over S: Além das regras da aprovação, recusa trecho<br/>que não aparece em ata.conteudo
    S->>S: concluirSeAprovada(reuniao, avaliada)
    S->>S: gravarAta(reuniao, ata, usuarioId)
    S->>M: findOneAndUpdate(filtro com _id, AGENDADA e a ata lida, $set ata)
    M-->>S: reuniao atualizada
    S->>S: responder(atualizada, usuarioId, conflito)
    S-->>C: reuniao
    C-->>A: 201 reuniao
    A-->>T: Resultado com a reuniao
    T->>T: definirSucesso("Ata reprovada. Ela voltou para correção.")
```

## SD-ATA-05 — Criar nova versão da ata

```mermaid
sequenceDiagram
    autonumber
    actor U as Presidente
    participant T as TelaRegistrosReuniao
    participant A as api
    participant C as ReunioesController
    participant S as ReunioesService
    participant M as ReuniaoModel
    U->>T: refazer()
    T->>T: definirConfirmacao(pendente)
    Note over U,T: O presidente confirma no diálogo "A versão N será cancelada"
    T->>T: gravar("/ata/nova", "POST", undefined, mensagem)
    T->>A: api("/reunioes/:id/ata/nova", "POST")
    Note over A,C: POST /reunioes/:id/ata/nova
    A->>C: refazerAta(id, req)
    C->>S: refazerAta(id, usuarioId)
    S->>S: carregar(id, usuarioId)
    S->>M: findById(id)
    M-->>S: reuniao
    S->>S: autorizarResponsavel(comissaoId, usuarioId, "escrever a ata")
    S->>S: exigirEncerrada(reuniao)
    Note over S: Recusa ata fora de REPROVADA
    S->>M: findOneAndUpdate(filtro com _id, AGENDADA e a ata lida, $push atas_canceladas e $set ata)
    M-->>S: reuniao atualizada
    S->>S: responder(atualizada, usuarioId, conflito)
    S-->>C: reuniao
    C-->>A: 201 reuniao
    A-->>T: Resultado com a reuniao
    T->>T: definirSucesso("Nova versão criada.")
```

## SD-ATA-06 — Consultar ata

```mermaid
sequenceDiagram
    autonumber
    actor U as Participante
    participant T as TelaRegistrosReuniao
    participant R as reunioes (utilitário)
    participant D as diferenca
    participant A as api
    participant C as ReunioesController
    participant S as ReunioesService
    participant M as ReuniaoModel
    U->>T: carregar()
    T->>A: api("/reunioes/:id")
    Note over A,C: GET /reunioes/:id
    A->>C: buscarPorId(id, req)
    C->>S: buscarPorId(id, usuarioId)
    S->>S: carregar(id, usuarioId)
    S->>M: findById(id)
    M-->>S: reuniao
    S->>S: popular(reuniao)
    S->>S: apresentar(reuniao, usuarioId)
    S-->>C: reuniao
    C-->>A: 200 reuniao
    A-->>T: Resultado com a reuniao
    T->>T: definirReuniao(resposta.dados)
    T->>R: situacaoDaAta(ata)
    R-->>T: texto e tom da etiqueta
    opt ata em aprovação com versão cancelada anterior
        T->>D: mudancas(textoAnterior, ata.conteudo)
        D-->>T: trechos novos e removidos
    end
```

## SD-ATA-07 — Ver atas pendentes

```mermaid
sequenceDiagram
    autonumber
    actor U as Participante
    participant T as TelaInicio
    participant A as api
    participant C as ReunioesController
    participant S as ReunioesService
    participant M as ReuniaoModel
    U->>T: carregar()
    T->>A: api("/reunioes/atas/pendentes")
    Note over A,C: GET /reunioes/atas/pendentes
    A->>C: atasPendentes(req)
    C->>S: atasPendentes(usuarioId)
    S->>S: validarUsuario(usuarioId)
    Note over S: Consulta: reunião AGENDADA e (ata EM_APROVACAO com o usuário<br/>em aprovadores e sem avaliação dele) ou (ata REPROVADA e autor_id igual ao usuário)
    S->>M: find(consulta)
    S->>M: select("titulo comissao_id ata.situacao ata.prazo")
    S->>M: populate("comissao_id", "nome")
    S->>M: sort(ata.prazo crescente)
    M-->>S: reunioes
    S-->>C: atas pendentes
    C-->>A: 200 lista
    A-->>T: Resultado com a lista
    T->>T: definirAtas(atasPendentes.dados)
    Note over U,T: O usuário toca em uma ata da lista
    T->>T: router.push("/reunioes/registros", id)
```

## SD-ATA-08 — Gerar PDF da ata

```mermaid
sequenceDiagram
    autonumber
    actor U as Participante
    participant T as TelaRegistrosReuniao
    participant P as pdfAta
    participant I as Print
    participant F as File
    participant H as Sharing
    U->>T: baixarPdf()
    T->>T: definirGerandoPdf(true)
    T->>P: compartilharPdfAta(reuniao)
    P->>P: htmlDaAta(reuniao)
    P->>I: printToFileAsync(html, width 595, height 842)
    I-->>P: uri do arquivo temporário
    P->>F: move(destino)
    P->>H: shareAsync(destino.uri, opcoes)
    Note over H: Abre o menu de compartilhamento do aparelho
    H-->>P: concluído
    P-->>T: concluído
    T->>T: definirGerandoPdf(false)
```

## Rastreabilidade

SD-ATA-01 a SD-ATA-08 correspondem, na mesma ordem, a UC-ATA-01 a UC-ATA-08 e a RF-ATA-01 a RF-ATA-08.
