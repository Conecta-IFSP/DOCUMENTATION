# Diagramas de sequência — Informações do sistema

> Padrão da disciplina · Módulo SIS · Revisado em 26/11/2026

## Resumo

Há um diagrama para cada requisito de [Requisitos funcionais](2%20-%20requisitos-funcionais-informacao-sistemas.md), com o mesmo número. As linhas de vida são classes e módulos que existem no código: as telas do aplicativo, os módulos `api` e `autenticacaoLocal`, os guards, o `SistemaController`, o `SistemaService`, o módulo `os` do Node.js e os models do Mongoose. Toda mensagem é a chamada de um método existente, com seus parâmetros, e as mensagens são numeradas na ordem em que acontecem. Regras que não são métodos, como a comparação do tipo da conta, e as rotas HTTP aparecem em notas.

O SD-SIS-01 mostra as três verificações que o NestJS executa antes do controller (`ApiKeyGuard`, `JwtAuthGuard` e `AdminSistemaGuard`), cada uma pelo seu método `canActivate`. Nos outros diagramas, essas verificações acontecem da mesma forma e estão resumidas numa nota.

## SD-SIS-01 — Consultar status do servidor

```mermaid
sequenceDiagram
    autonumber
    actor U as Administrador do sistema
    participant I as TelaInicio
    participant T as TelaSistema
    participant A as api
    participant K as ApiKeyGuard
    participant J as JwtAuthGuard
    participant G as AdminSistemaGuard
    participant C as SistemaController
    participant S as SistemaService
    participant O as os
    participant MU as UsuarioModel
    participant MO as OrganizacaoModel
    participant MC as ComissaoModel
    U->>I: aoTocar()
    I->>T: router.push("/sistema")
    T->>T: carregar()
    Note over T: useFocusEffect chama carregar ao abrir a tela.<br/>Ao mesmo tempo, carregar pede os relatórios (SD-SIS-02)
    T->>A: api("/sistema/status")
    A->>A: obterToken()
    Note over A,K: GET /sistema/status com os cabeçalhos X-API-Key e Authorization Bearer
    A->>K: canActivate(context)
    K->>K: chavesIguais(recebida, esperada)
    K-->>A: true
    A->>J: canActivate(context)
    J->>J: extrairToken(requisicao)
    Note over J: jwtService.verifyAsync confere assinatura e validade.<br/>sub precisa ser um ObjectId válido
    J->>MU: findOne({ _id: requisicao.usuario.sub, ativo: true })
    MU-->>J: usuario
    Note over J: Recusa token emitido antes de senha_alterada_em.<br/>Copia tipo e email do banco para a requisição
    J-->>A: true
    A->>G: canActivate(context)
    Note over G: tipo igual a ADMIN_SISTEMA, senão 403
    G-->>A: true
    A->>C: status()
    C->>S: status()
    S->>O: totalmem()
    O-->>S: bytes totais
    S->>O: freemem()
    O-->>S: bytes livres
    S->>O: loadavg()
    O-->>S: cargas de 1, 5 e 15 minutos
    S->>MU: countDocuments()
    S->>MO: countDocuments()
    S->>MC: countDocuments()
    Note over S,MC: As três contagens rodam em paralelo (Promise.all)
    alt contagens concluídas
        MU-->>S: total de usuarios
        MO-->>S: total de organizacoes
        MC-->>S: total de comissoes
        S->>O: cpus()
        O-->>S: lista de processadores
        S->>O: uptime()
        O-->>S: segundos desde que o computador ligou
        Note over S: Converte bytes em MB, calcula uso_percentual,<br/>arredonda a carga para 2 casas e define conectado true
        S-->>C: objeto de status
        C-->>A: 200 objeto de status
        A-->>T: Resultado com os dados
        Note over T: Só grava se os relatórios também deram certo
        T->>T: definirAviso(null)
        T->>T: definirStatus(resultadoStatus.dados)
    else falha do MongoDB
        MU-->>S: erro
        S-->>C: erro não tratado
        C-->>A: 500 Internal server error
        A-->>T: Resultado com o erro
        Note over T: aviso = tom "erro" e mensagem resultadoStatus.erro
        T->>T: definirAviso(aviso)
        Note over T: Faixa "Não foi possível carregar o status"
    end
```

Se uma das verificações recusa a requisição, o controller não é chamado e o guard devolve 401, 403 ou 503; a tela mostra a mensagem na mesma faixa vermelha (ver RF-SIS-01).

## SD-SIS-02 — Consultar relatórios

```mermaid
sequenceDiagram
    autonumber
    actor U as Administrador do sistema
    participant T as TelaSistema
    participant A as api
    participant C as SistemaController
    participant S as SistemaService
    participant MU as UsuarioModel
    participant MO as OrganizacaoModel
    participant MC as ComissaoModel
    U->>T: carregar()
    Note over U,T: Ao abrir a tela ou puxar a tela para baixo
    T->>A: api("/sistema/relatorios")
    Note over A,C: GET /sistema/relatorios. Antes do controller,<br/>ApiKeyGuard, JwtAuthGuard e AdminSistemaGuard como em SD-SIS-01
    A->>C: relatorios()
    C->>S: relatorios()
    S->>MU: countDocuments()
    S->>MU: countDocuments({ ativo: true })
    S->>MO: countDocuments()
    S->>MO: countDocuments({ status: StatusOrganizacao.PENDENTE })
    S->>MO: countDocuments({ status: StatusOrganizacao.APROVADA })
    S->>MC: countDocuments()
    S->>MC: countDocuments({ ativo: true })
    Note over S,MC: As sete contagens rodam em paralelo (Promise.all)<br/>e todos os filtros são por igualdade
    MU-->>S: total e ativos
    MO-->>S: total, pendentes e aprovadas
    MC-->>S: total e ativas
    S-->>C: objeto de relatórios
    C-->>A: 200 objeto de relatórios
    A-->>T: Resultado com os dados
    alt status e relatórios deram certo
        T->>T: definirAviso(null)
        T->>T: definirStatus(resultadoStatus.dados)
        T->>T: definirRelatorios(resultadoRelatorios.dados)
    else só os relatórios falharam
        Note over T: aviso = tom "erro" e mensagem resultadoRelatorios.erro
        T->>T: definirAviso(aviso)
        Note over T: Faixa "Não foi possível carregar os relatórios"
    end
```

## SD-SIS-03 — Popular dados de teste

```mermaid
sequenceDiagram
    autonumber
    actor U as Administrador do sistema
    participant T as TelaSistema
    participant A as api
    participant C as SistemaController
    participant S as SistemaService
    participant B as bcrypt
    participant MU as UsuarioModel
    participant MO as OrganizacaoModel
    participant MC as ComissaoModel
    U->>T: definirConfirmacaoTeste(true)
    Note over U,T: Toque em "Popular dados de teste".<br/>O diálogo mostra "Criar 3 usuários de teste?" com Cancelar e Confirmar
    alt Cancelar
        U->>T: definirConfirmacaoTeste(false)
        Note over T: O diálogo fecha e nada é enviado
    else Confirmar
        U->>T: popularDadosDeTeste()
        T->>T: definirAcaoEmAndamento("popular-teste")
        T->>A: api("/sistema/popular-teste", "POST")
        Note over A,C: POST /sistema/popular-teste, sem corpo. Antes do controller,<br/>ApiKeyGuard, JwtAuthGuard e AdminSistemaGuard como em SD-SIS-01
        A->>C: popularDadosDeTeste()
        C->>S: popularDadosDeTeste()
        S->>S: obterConfiguracaoUsuariosTeste()
        Note over S: Lê TEST_USERS_PASSWORD e as seis variáveis TEST_USER_*.<br/>Faltando alguma, 400 "Configure as contas de teste no .env: " e os nomes
        S->>B: hash(senha, 10)
        B-->>S: senha_hash
        loop três contas, em paralelo
            S->>MU: create(nome, email, senha_hash)
            MU-->>S: usuario
        end
        Note over S,MU: E-mail já cadastrado: o índice único recusa e a API responde 500
        Note over S,MO: dados = nome "Organização de Teste", cidade, descricao, criada_por, status APROVADA<br/>e membros com um ADMIN e um MEMBRO, ambos APROVADO
        S->>MO: create(dados)
        MO-->>S: organizacao
        Note over S,MC: dados = nome "Comissão de Teste", descricao, organizacao_id<br/>e membros com um RESPONSAVEL
        S->>MC: create(dados)
        MC-->>S: comissao
        S-->>C: status dados_de_teste_criados e usuarios
        C-->>A: 201 objeto
        A-->>T: Resultado com os dados
        T->>T: definirAcaoEmAndamento(null)
        T->>T: definirConfirmacaoTeste(false)
        T->>T: carregar()
        Note over T: carregar pede status e relatórios (SD-SIS-01 e SD-SIS-02)<br/>e, quando os dois dão certo, chama definirAviso(null)
        Note over T: sucesso = tom "sucesso", titulo "Dados de teste criados"<br/>e mensagem "3 usuários de teste criados."
        T->>T: definirAviso(sucesso)
        Note over T: A faixa é definida depois da recarga e continua na tela
    end
```

## SD-SIS-04 — Resetar sistema

```mermaid
sequenceDiagram
    autonumber
    actor U as Administrador do sistema
    participant T as TelaSistema
    participant L as autenticacaoLocal
    participant A as api
    participant C as SistemaController
    participant S as SistemaService
    participant MU as UsuarioModel
    participant MO as OrganizacaoModel
    participant MC as ComissaoModel
    participant MR as ReuniaoModel
    participant X as Connection
    U->>T: abrirReset()
    T->>T: definirTextoReset("")
    T->>T: definirConfirmacaoReset(true)
    Note over U,T: A folha "Resetar sistema" mostra o aviso, o campo "Digite RESETAR para confirmar"<br/>e o botão "Resetar sistema", inativo enquanto o texto não for igual a RESETAR
    Note over U,T: texto = o que o administrador digita no campo
    U->>T: definirTextoReset(texto)
    alt fechar a folha
        U->>T: fecharReset()
        T->>T: definirConfirmacaoReset(false)
        Note over T: Toque fora da folha ou botão voltar do Android. Nada é enviado
    else botão Resetar sistema da folha, com RESETAR digitado
        U->>T: resetarSistema()
        T->>T: definirConfirmacaoReset(false)
        T->>L: autenticarComDispositivo()
        L->>L: dispositivoProtegido()
        Note over U,L: Com biometria ou PIN cadastrado, o aparelho pede a identidade.<br/>Na web e sem bloqueio de tela, devolve true sem pedir
        alt identidade não confirmada
            L-->>T: false
            Note over T: aviso = tom "erro" e mensagem "Não foi possível confirmar sua identidade."
            T->>T: definirAviso(aviso)
            Note over T: Nada é enviado
        else identidade confirmada
            L-->>T: true
            T->>T: definirAcaoEmAndamento("reset")
            Note over T,A: corpo = { confirmacao: textoReset }
            T->>A: api("/sistema/reset", "POST", corpo)
            Note over A,C: POST /sistema/reset. Antes do controller, ApiKeyGuard, JwtAuthGuard e AdminSistemaGuard como em SD-SIS-01.<br/>Depois, o ValidationPipe confere o ResetSistemaDto. Outro valor: 400 "Digite RESETAR para confirmar o reset"
            A->>C: reset(_confirmacao)
            C->>S: reset()
            S->>MU: deleteMany({ tipo: { $ne: TipoUsuario.ADMIN_SISTEMA } })
            S->>MO: deleteMany({})
            S->>MC: deleteMany({})
            S->>MR: deleteMany({})
            S->>X: db.collection("anexos.files").deleteMany({})
            S->>X: db.collection("anexos.chunks").deleteMany({})
            S->>X: db.collection("contadores").deleteMany({})
            S->>X: db.collection("notificacoes").deleteMany({})
            Note over S,X: As oito exclusões rodam em paralelo (Promise.all), sem transação
            MU-->>S: resultado
            MO-->>S: resultado
            MC-->>S: resultado
            MR-->>S: resultado
            X-->>S: resultados
            S-->>C: status sistema_resetado e resetado_em
            C-->>A: 201 objeto
            A-->>T: Resultado com os dados
            T->>T: definirAcaoEmAndamento(null)
            T->>T: carregar()
            Note over T: carregar pede status e relatórios e, quando os dois dão certo, chama definirAviso(null)
            Note over T: sucesso = tom "sucesso", titulo "Sistema resetado" e mensagem<br/>"Usuários, organizações, comissões, reuniões e notificações apagados."
            T->>T: definirAviso(sucesso)
        end
    end
```

Se o `ValidationPipe` recusar o corpo, o controller não é chamado e a tela mostra a faixa "Não foi possível resetar" com "Digite RESETAR para confirmar o reset"; pela tela isso não acontece, porque o botão da folha só fica ativo com a palavra exata. Se uma das exclusões falhar, a API responde 500 "Internal server error", a tela mostra a mesma faixa e as exclusões que já terminaram não são desfeitas (ver RF-SIS-04 e RNF-SIS-04).

## Rastreabilidade

SD-SIS-01 a SD-SIS-04 correspondem, na mesma ordem, a UC-SIS-01 a UC-SIS-04 e a RF-SIS-01 a RF-SIS-04.
