# Diagramas de sequência — Votação na reunião

> Padrão da disciplina · Módulo VOT · Revisado em 26/11/2026

## Resumo

Há um diagrama para cada requisito de [Requisitos funcionais](2%20-%20requisitos-funcionais-votacao.md), com o mesmo número. As linhas de vida são classes e módulos que existem no código: a tela `TelaReuniao`, o componente `VotacoesReuniao`, o módulo `api`, o `ReunioesController`, o `ReunioesService` e os models do Mongoose. Toda mensagem é a chamada de um método existente, com seus parâmetros, e as mensagens são numeradas na ordem em que acontecem. Os retornos voltam pela cadeia, na ordem inversa. Regras que não são métodos, como comparações de horário, e o conteúdo dos filtros e das alterações enviados ao banco aparecem em notas; nas mensagens, eles são os parâmetros `filtro`, `alteracao` e `opcoes`.

Em todos os diagramas, a chamada do módulo `api` ao controller é uma requisição HTTP que passa antes por `ApiKeyGuard`, `JwtAuthGuard` e `UsuarioComumGuard`; a rota está indicada na nota. O método `carregar(id, usuarioId)` aparece só com a leitura da reunião; antes dela, ele chama `validarId(id)` e `validarUsuario(usuarioId)`, que lê o usuário com `UsuarioModel.findById`.

## SD-VOT-01 — Abrir votação

```mermaid
sequenceDiagram
    autonumber
    actor U as Presidente
    participant V as VotacoesReuniao
    participant T as TelaReuniao
    participant A as api
    participant C as ReunioesController
    participant S as ReunioesService
    participant CM as ComissaoModel
    participant M as ReuniaoModel
    U->>V: abrirFormulario()
    Note over U,V: Folha "Nova votação" com Sim, Não e Abstenção já preenchidas
    U->>V: enviar()
    V->>V: validar()
    alt problema no formulário
        V->>V: definirErroFormulario(problema)
    else formulário válido
        V->>T: executar("abrir-votacao", acao, "Votação aberta.")
        T->>A: api("/reunioes/:id/votacoes", "POST", { titulo, descricao, opcoes })
        Note over A,C: POST /reunioes/:id/votacoes
        A->>C: abrirVotacao(id, dto, req)
        C->>S: abrirVotacao(id, dto, usuarioId)
        S->>S: carregar(id, usuarioId)
        S->>M: findById(id)
        M-->>S: reuniao
        S->>S: exigirPresidente(reuniao, usuarioId)
        S->>CM: findById(comissao_id)
        CM-->>S: comissao
        Note over S: Exige membro RESPONSAVEL com o usuario_id do usuário e comissão ativa
        S->>S: exigirAgendada(reuniao)
        Note over S: Exige agora maior ou igual a inicio e menor que fim<br/>e opções diferentes, comparadas em minúsculas
        Note over S,M: filtro: _id, situacao AGENDADA e fim maior que agora.<br/>alteracao: $push em votacoes da nova votação ABERTA, com novo _id e votos vazio
        S->>M: findOneAndUpdate(filtro, alteracao, opcoes)
        M-->>S: atualizada
        S->>S: responder(atualizada, usuarioId, conflito)
        alt reunião encontrada pelo filtro
            S-->>C: reuniao
            C-->>A: 201 reuniao
            A-->>T: resultado
            T->>T: definirReuniao(resultado.dados)
            T-->>V: true
            V->>V: definirFormulario(false)
        else reunião cancelada ou terminada nesse intervalo
            S-->>C: ConflictException
            C-->>A: 409 "A reunião mudou. Atualize a tela."
            A-->>T: resultado
            T->>T: definirErro(resultado.erro)
            T-->>V: false
        end
    end
```

## SD-VOT-02 — Votar

```mermaid
sequenceDiagram
    autonumber
    actor U as Participante presente
    participant V as VotacoesReuniao
    participant T as TelaReuniao
    participant A as api
    participant C as ReunioesController
    participant S as ReunioesService
    participant M as ReuniaoModel
    U->>V: pedirVoto(votacao, indice)
    V->>V: definirConfirmacao(pendente)
    Note over U,V: Confirmação "Votar em ...? O voto não pode ser alterado depois."
    U->>V: confirmacao.acao()
    V->>T: executar("votar-:votacaoId", acao, "Voto registrado.")
    T->>A: api("/reunioes/:id/votacoes/:votacaoId/votos", "POST", { opcao })
    Note over A,C: POST /reunioes/:id/votacoes/:votacaoId/votos
    A->>C: votar(id, votacaoId, dto, req)
    C->>S: votar(id, votacaoId, opcao, usuarioId)
    S->>S: carregar(id, usuarioId)
    S->>M: findById(id)
    M-->>S: reuniao
    S->>S: exigirAgendada(reuniao)
    S->>S: buscarVotacao(reuniao, votacaoId)
    Note over S: Exige votação ABERTA e fim maior que agora, opcao menor que a quantidade de opções,<br/>presença não RECUSADA e sem saída, e usuário ausente de votos
    Note over S,M: filtro: _id, AGENDADA, fim maior que agora, votação com o mesmo _id ainda ABERTA<br/>e sem voto do usuário, e participante do usuário com presença AGUARDANDO ou CONFIRMADA<br/>e sem saída. alteracao: $push do voto na votação. opcoes: arrayFilters pelo _id da votação
    S->>M: findOneAndUpdate(filtro, alteracao, opcoes)
    M-->>S: atualizada
    S->>S: responder(atualizada, usuarioId, conflito)
    alt condições ainda valem no banco
        S->>S: apresentarVotacao(votacao, fim, usuarioId)
        S-->>C: reuniao
        C-->>A: 201 reuniao
        A-->>T: resultado
        T->>T: definirReuniao(resultado.dados)
        T->>T: definirSucesso("Voto registrado.")
    else votação encerrada, presença alterada ou voto repetido no intervalo
        S-->>C: ConflictException
        C-->>A: 409 "A votação mudou. Atualize a tela."
        A-->>T: resultado
        T->>T: definirErro(resultado.erro)
    end
    T-->>V: concluído
    V->>V: definirConfirmacao(null)
```

## SD-VOT-03 — Encerrar votação

```mermaid
sequenceDiagram
    autonumber
    actor U as Presidente
    participant V as VotacoesReuniao
    participant T as TelaReuniao
    participant A as api
    participant C as ReunioesController
    participant S as ReunioesService
    participant CM as ComissaoModel
    participant M as ReuniaoModel
    U->>V: pedirEncerramento(votacao)
    V->>V: definirConfirmacao(pendente)
    Note over U,V: Confirmação "Encerrar a votação ...? Ninguém mais poderá votar."
    U->>V: confirmacao.acao()
    V->>T: executar("encerrar-:votacaoId", acao, "Votação encerrada.")
    T->>A: api("/reunioes/:id/votacoes/:votacaoId/encerrar", "POST")
    Note over A,C: POST /reunioes/:id/votacoes/:votacaoId/encerrar
    A->>C: encerrarVotacao(id, votacaoId, req)
    C->>S: encerrarVotacao(id, votacaoId, usuarioId)
    S->>S: carregar(id, usuarioId)
    S->>M: findById(id)
    M-->>S: reuniao
    S->>S: exigirPresidente(reuniao, usuarioId)
    S->>CM: findById(comissao_id)
    CM-->>S: comissao
    S->>S: exigirAgendada(reuniao)
    S->>S: buscarVotacao(reuniao, votacaoId)
    Note over S: Recusa votação gravada como ENCERRADA.<br/>encerrada_em: o menor entre agora e reuniao.fim
    Note over S,M: filtro: _id, AGENDADA e votação com o mesmo _id ainda ABERTA.<br/>alteracao: $set da situacao ENCERRADA e de encerrada_em nessa votação
    S->>M: findOneAndUpdate(filtro, alteracao, opcoes)
    M-->>S: atualizada
    S->>S: responder(atualizada, usuarioId, conflito)
    alt votação ainda aberta
        S-->>C: reuniao
        C-->>A: 201 reuniao
        A-->>T: resultado
        T->>T: definirReuniao(resultado.dados)
        T->>T: definirSucesso("Votação encerrada.")
    else encerrada por outra chamada no intervalo
        S-->>C: ConflictException
        C-->>A: 409 "A votação mudou. Atualize a tela."
        A-->>T: resultado
        T->>T: definirErro(resultado.erro)
    end
    T-->>V: concluído
    V->>V: definirConfirmacao(null)
```

## SD-VOT-04 — Consultar votações e resultado

```mermaid
sequenceDiagram
    autonumber
    actor U as Participante da reunião
    participant T as TelaReuniao
    participant V as VotacoesReuniao
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
    S->>S: popular(reuniao)
    S->>S: apresentar(reuniao, usuarioId)
    loop cada votação
        S->>S: apresentarVotacao(votacao, fim, usuarioId)
        Note over S: Retira votos, calcula total_votos e meu_voto.<br/>ABERTA com fim já passado é apresentada como ENCERRADA.<br/>resultado só quando ENCERRADA
    end
    S-->>C: reuniao
    C-->>A: 200 reuniao
    A-->>T: resultado
    T->>T: definirReuniao(resultado.dados)
    T->>A: api("/comissoes/:comissaoId")
    A-->>T: comissao
    T->>T: definirPresidente(presidente)
    Note over T: buscar() se repete a cada 30 segundos e ao puxar a tela
    U->>T: definirAba("votacoes")
    T->>V: VotacoesReuniao(reuniao, presidente, emAndamento, motivoSemVoto, salvando, executar)
    loop cada votação, abertas primeiro e mais recentes antes
        alt votação aberta
            V->>V: renderizarAberta(votacao)
        else votação encerrada
            V->>V: renderizarResultado(votacao)
        end
    end
```

## Rastreabilidade

SD-VOT-01 a SD-VOT-04 correspondem a RF-VOT-01 a RF-VOT-04 e aos casos de uso UC-VOT-01 a UC-VOT-04.
