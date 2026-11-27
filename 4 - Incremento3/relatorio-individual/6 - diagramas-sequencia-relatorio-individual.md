# Diagramas de sequência — Relatório individual

> Padrão da disciplina · Módulo REL · Revisado em 26/11/2026

## Resumo

Há um diagrama para cada requisito de [Requisitos funcionais](2%20-%20requisitos-funcionais-relatorio-individual.md), com o mesmo número. As linhas de vida são classes e módulos que existem no código e no [diagrama de classes](3%20-%20diagrama-classes-relatorio-individual.md): a `TelaRelatorioIndividual`, os módulos `relatorioIndividual`, `api` e `exportarRelatorioIndividual`, o `RelatoriosController`, o `RelatoriosService`, os models do Mongoose guardados nos atributos `usuarios`, `reunioes`, `organizacoes` e `comissoes` do serviço, e as bibliotecas `Print`, `Sharing`, `File` e `window`. As chamadas `select`, `populate` e `sort` são feitas na `Query` devolvida por `find` ou `findById` e aparecem na mesma linha de vida do model. Toda mensagem é a chamada de um método existente, com seus parâmetros, e as mensagens são numeradas na ordem em que acontecem. Regras que não são métodos, como os filtros da consulta, aparecem em notas.

Nos diagramas que chamam a API, a requisição passa antes pelos guards `ApiKeyGuard`, `JwtAuthGuard` e `UsuarioComumGuard`, que o NestJS executa pelo método `canActivate(context)`; a rota e os guards estão indicados na nota.

## SD-REL-01 — Consultar relatório individual

```mermaid
sequenceDiagram
    autonumber
    actor U as Usuário comum
    participant T as TelaRelatorioIndividual
    participant F as relatorioIndividual
    participant A as api
    participant C as RelatoriosController
    participant S as RelatoriosService
    participant MU as usuarios (Model Usuario)
    participant MR as reunioes (Model Reuniao)
    U->>T: carregar()
    Note over U,T: Chamado pelo useFocusEffect ao abrir a aba,<br/>ao puxar a lista ou pelo botão "Atualizar relatório"
    T->>F: consultaRelatorio(periodo)
    F-->>T: texto vazio em "Todo período"
    T->>A: api("/relatorios/individual")
    Note over A,C: GET /relatorios/individual<br/>ApiKeyGuard, JwtAuthGuard e UsuarioComumGuard executam canActivate(context)
    alt chave, token ou conta recusados pelos guards
        Note over A,C: 401 da chave ou do token, 403 para ADMIN_SISTEMA
        opt resposta 401 com token salvo
            A->>A: removerToken()
        end
        A-->>T: Resultado com o erro
        T->>T: definirErro(resultado.erro)
    else acesso permitido
        A->>C: individual(req, filtros)
        C->>S: individual(req.usuario.sub, filtros)
        S->>MU: findById(usuarioId)
        S->>MU: select("nome email ativo")
        MU-->>S: usuario
        Note over S: Conta inexistente ou inativa gera UnauthorizedException (401).<br/>Os vínculos são lidos como em SD-REL-03
        Note over S,MR: filtro = situacao AGENDADA, inicio com $lte de geradaEm e participantes com $elemMatch<br/>do usuário com presenca.situacao CONFIRMADA e presenca.entrada com $lte de geradaEm
        S->>MR: find(filtro)
        S->>MR: select("titulo local inicio fim dia_inteiro comissao_id participantes")
        Note over S,MR: opcoes = path comissao_id com select nome e organizacao_id<br/>e populate de organizacao_id com select nome
        S->>MR: populate(opcoes)
        S->>MR: sort({ inicio: -1, _id: -1 })
        MR-->>S: registros
        Note over S: Para cada reunião, usa a presença do usuário se for CONFIRMADA.<br/>minutos = parte inteira de (saida - entrada) em minutos quando a saída<br/>existe, não é anterior à entrada e não é futura, senão null.<br/>O resumo soma os minutos e conta os registros sem duração
        S-->>C: relatório
        C-->>A: 200 relatório
        A-->>T: Resultado com o relatório
        Note over T: A resposta é descartada se pedido.current mudou depois do pedido
        T->>T: definirRelatorio(resultado.dados)
        T->>F: horasRelatorio(relatorio.resumo.minutos_registrados)
        F-->>T: "1h 15min"
        T->>F: dataRelatorio(item.inicio)
        F-->>T: data e hora no fuso do aparelho
    end
```

## SD-REL-02 — Filtrar relatório por período

```mermaid
sequenceDiagram
    autonumber
    actor U as Usuário comum
    participant T as TelaRelatorioIndividual
    participant F as relatorioIndividual
    participant A as api
    participant C as RelatoriosController
    participant S as RelatoriosService
    participant MR as reunioes (Model Reuniao)
    U->>T: definirPeriodo("mes")
    Note over T: carregar depende de periodo, então o useFocusEffect a executa de novo
    T->>T: carregar()
    T->>F: consultaRelatorio("mes", agora)
    Note over F: de = dia 1 do mês atual às 00:00 do aparelho<br/>ate = dia seguinte ao de hoje às 00:00 do aparelho<br/>os dois convertidos para UTC por toISOString
    F-->>T: consulta com de e ate
    Note over T,A: caminho = "/relatorios/individual" seguido da consulta
    T->>A: api(caminho)
    Note over A,C: GET /relatorios/individual com de e ate na consulta<br/>ValidationPipe aplica RelatorioIndividualDto com IsISO8601 estrito
    alt de ou ate fora do formato ISO 8601 (só em chamada direta à API)
        Note over A,C: 400 "Informe uma data inicial válida" ou "Informe uma data final válida"
        A-->>T: Resultado com o erro
    else formato válido
        A->>C: individual(req, filtros)
        C->>S: individual(req.usuario.sub, filtros)
        S->>S: new Date(filtros.de), new Date(filtros.ate)
        alt de maior ou igual a ate
            S-->>C: BadRequestException
            C-->>A: 400 "O fim do período deve ser posterior ao início"
            A-->>T: Resultado com o erro
            T->>T: definirErro(resultado.erro)
        else período válido
            Note over S: Usuário e vínculos lidos como em SD-REL-01 e SD-REL-03.<br/>Os vínculos não usam de nem ate
            Note over S,MR: filtro = situacao AGENDADA, inicio com $lte de geradaEm, $gte de de e $lt de ate<br/>e participantes com $elemMatch do usuário com presenca.situacao CONFIRMADA
            S->>MR: find(filtro)
            S->>MR: sort({ inicio: -1, _id: -1 })
            MR-->>S: registros do período
            S-->>C: relatório com periodo.de e periodo.ate
            C-->>A: 200 relatório
            A-->>T: Resultado com o relatório
            Note over T: Se o usuário trocou de opção antes, esta resposta é descartada
            T->>T: definirRelatorio(resultado.dados)
            T->>F: periodoRelatorio(relatorio)
            F-->>T: "01/11/2026 a 26/11/2026"
        end
    end
```

## SD-REL-03 — Ver vínculos atuais

```mermaid
sequenceDiagram
    autonumber
    actor U as Usuário comum
    participant T as TelaRelatorioIndividual
    participant A as api
    participant C as RelatoriosController
    participant S as RelatoriosService
    participant MU as usuarios (Model Usuario)
    participant MO as organizacoes (Model Organizacao)
    participant MC as comissoes (Model Comissao)
    U->>T: carregar()
    Note over T,A: caminho = "/relatorios/individual" seguido da consulta
    T->>A: api(caminho)
    Note over A,C: GET /relatorios/individual<br/>depois de ApiKeyGuard, JwtAuthGuard e UsuarioComumGuard
    A->>C: individual(req, filtros)
    C->>S: individual(req.usuario.sub, filtros)
    S->>MU: findById(usuarioId)
    MU-->>S: usuario ativo
    Note over S,MO: filtro = status APROVADA e membros com $elemMatch<br/>de usuario_id em referenciasUsuario e status APROVADO
    S->>MO: find(filtro)
    S->>MO: select("nome membros")
    S->>MO: sort({ nome: 1, _id: 1 })
    MO-->>S: organizacoes
    Note over S,MC: filtro = ativo true, organizacao_id com $in de referenciasOrganizacoes<br/>e membros.usuario_id com $in de referenciasUsuario
    S->>MC: find(filtro)
    S->>MC: select("nome organizacao_id membros")
    S->>MC: sort({ nome: 1, _id: 1 })
    MC-->>S: comissoes
    Note over S: papel da organização = papel do item APROVADO do usuário em membros.<br/>papel da comissão = papel do item do usuário em membros.<br/>organizacao da comissão = nome lido na lista de organizações
    Note over S: As reuniões são lidas em seguida, como em SD-REL-01
    S-->>C: relatório com vinculos
    C-->>A: 200 relatório
    A-->>T: Resultado com o relatório
    T->>T: definirRelatorio(resultado.dados)
    Note over U,T: Cartão "Meus vínculos atuais": "nome · Administrador",<br/>"comissão · organização · Responsável" ou as frases de lista vazia
```

## SD-REL-04 — Exportar relatório em PDF

```mermaid
sequenceDiagram
    autonumber
    actor U as Usuário comum
    participant T as TelaRelatorioIndividual
    participant E as exportarRelatorioIndividual
    participant F as relatorioIndividual
    participant W as window
    participant P as Print
    participant H as Sharing
    U->>T: exportar("pdf")
    Note over T: Não faz nada se travaExportacao já estiver ligada
    T->>T: definirExportando("pdf")
    T->>E: exportarRelatorioIndividual(relatorio, "pdf")
    alt Web
        E->>E: imprimirRelatorioWeb(relatorio)
        E->>W: open("", "_blank")
        W-->>E: janela
        E->>E: logoDoRelatorio()
        E->>F: htmlRelatorioIndividual(relatorio, logo)
        F->>F: escapar(texto)
        F-->>E: html
        E->>W: janela.document.write(html)
        E->>W: janela.print()
        Note over W: Abre o diálogo de impressão do navegador, com "Salvar como PDF"
    else Expo Go
        E->>E: logoDoRelatorio()
        E->>F: htmlRelatorioIndividual(relatorio, logo)
        F-->>E: html
        E->>P: printAsync({ html, width: 595, height: 842 })
        Note over P: Abre a tela de impressão do sistema
        P-->>E: concluído
    else Aplicativo instalado
        E->>E: logoDoRelatorio()
        E->>F: htmlRelatorioIndividual(relatorio, logo)
        F-->>E: html
        E->>H: isAvailableAsync()
        H-->>E: true
        E->>P: printToFileAsync({ html, width: 595, height: 842 })
        P-->>E: uri do PDF no cache
        E->>H: shareAsync(uri, { mimeType: "application/pdf" })
        Note over H: Abre o menu de compartilhamento com o arquivo gerado, sem mover nem renomear
        H-->>E: concluído
    end
    alt exportação concluída
        E-->>T: concluído
    else janela bloqueada, falha da logo, da impressão ou do compartilhamento
        E-->>T: Error com a mensagem
        T->>T: definirErroExportacao(erro.message)
    end
    T->>T: definirExportando(null)
```

## SD-REL-05 — Exportar relatório em CSV

```mermaid
sequenceDiagram
    autonumber
    actor U as Usuário comum
    participant T as TelaRelatorioIndividual
    participant E as exportarRelatorioIndividual
    participant F as relatorioIndividual
    participant W as window
    participant FS as File
    participant H as Sharing
    U->>T: exportar("csv")
    Note over T: Não faz nada se travaExportacao já estiver ligada
    T->>T: definirExportando("csv")
    T->>E: exportarRelatorioIndividual(relatorio, "csv")
    alt Web
        E->>F: csvRelatorioIndividual(relatorio)
        F->>F: celulaCsv(valor)
        F-->>E: csv com BOM, 19 colunas
        E->>W: URL.createObjectURL(blob)
        W-->>E: url
        E->>W: document.createElement("a")
        W-->>E: link
        E->>W: link.click()
        Note over W: O navegador baixa relatorio-individual.csv
        E->>W: URL.revokeObjectURL(url)
        Note over E,W: A liberação é agendada por setTimeout para 1 segundo depois
    else Aplicativo instalado ou Expo Go
        E->>H: isAvailableAsync()
        H-->>E: true
        E->>FS: new File(Paths.cache, "relatorio-individual-" + Date.now() + ".csv")
        E->>FS: create()
        E->>F: csvRelatorioIndividual(relatorio)
        F->>F: celulaCsv(valor)
        F-->>E: csv com BOM, 19 colunas
        E->>FS: write(csv)
        FS-->>E: gravado
        E->>H: shareAsync(arquivo.uri, { mimeType: "text/csv" })
        Note over H: Abre o menu de compartilhamento do aparelho
        H-->>E: concluído
    end
    alt exportação concluída
        E-->>T: concluído
    else compartilhamento indisponível, falha na gravação ou no compartilhamento
        E-->>T: Error com a mensagem
        T->>T: definirErroExportacao(erro.message)
    end
    T->>T: definirExportando(null)
```

## Rastreabilidade

SD-REL-01 a SD-REL-05 correspondem, na mesma ordem, a UC-REL-01 a UC-REL-05 e a RF-REL-01 a RF-REL-05.
