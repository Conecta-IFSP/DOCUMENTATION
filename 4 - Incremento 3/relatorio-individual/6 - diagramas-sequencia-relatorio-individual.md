# Diagramas de sequência — Relatório individual

> Padrão da disciplina · Módulo REL · Revisado em 27/11/2026

## Resumo

Há um diagrama para cada requisito de [Requisitos funcionais](2%20-%20requisitos-funcionais-relatorio-individual.md), com o mesmo número. As linhas de vida são classes e módulos que existem no código e no [diagrama de classes](3%20-%20diagrama-classes-relatorio-individual.md): a `TelaRelatorioIndividual`, os módulos `relatorioIndividual`, `api` e `exportarRelatorioIndividual`, o `RelatoriosController`, o `RelatoriosService`, os models do Mongoose `UsuarioModel`, `ReuniaoModel`, `OrganizacaoModel` e `ComissaoModel`, guardados nos atributos `usuarios`, `reunioes`, `organizacoes` e `comissoes` do serviço, a `Query` do Mongoose, as bibliotecas `Print`, `Sharing`, `File` e `jsPDF`, os módulos `baixarPdfRelatorio` e `pdfRelatorioIndividual`, que geram e baixam o PDF na web desde o FRONTEND `299afc2`, e as classes do navegador `Document`, `URL` e `HTMLAnchorElement`. Cada mensagem vai para a linha de vida dona do método: `find` e `findById` vão para o model, e `select`, `populate`, `sort`, `lean` e `exec`, para a `Query` que eles devolvem; a linha de vida `Query` representa a consulta montada em cada leitura. O nome da instância aparece entre parênteses quando ajuda a identificá-la, como em `Document (document)` e `HTMLAnchorElement (link)`. Toda mensagem é a chamada de um método existente, com seus parâmetros, e as mensagens são numeradas na ordem em que acontecem. Cada retorno é o nome do valor devolvido, e `void` indica que o método não devolve valor. Regras que não são métodos, como os filtros da consulta, a conversão das datas e o código HTTP da resposta, aparecem em notas.

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
    participant MU as UsuarioModel
    participant MR as ReuniaoModel
    participant Q as Query
    U->>T: carregar()
    Note over U,T: Chamado pelo useFocusEffect ao abrir a aba,<br/>ao puxar a lista ou pelo botão "Atualizar relatório"
    T->>F: consultaRelatorio(periodo)
    F-->>T: consulta
    Note over T,F: Em "Todo período", consulta é o texto vazio
    Note over T,A: caminho = "/relatorios/individual" seguido da consulta
    T->>A: api(caminho)
    Note over A,C: GET /relatorios/individual<br/>ApiKeyGuard, JwtAuthGuard e UsuarioComumGuard executam canActivate(context)
    alt chave, token ou conta recusados pelos guards
        Note over A,C: 401 da chave ou do token, 403 para ADMIN_SISTEMA
        opt resposta 401 com token salvo
            A->>A: removerToken()
        end
        A-->>T: resultado
        Note over T,A: resultado.ok é false, e resultado.erro traz a mensagem
        T->>T: definirErro(resultado.erro)
    else acesso permitido
        A->>C: individual(req, filtros)
        C->>S: individual(req.usuario.sub, filtros)
        S->>MU: findById(usuarioId)
        MU-->>S: query
        S->>Q: select("nome email ativo")
        S->>Q: lean()
        S->>Q: exec()
        Q-->>S: usuario
        Note over S: Conta inexistente ou inativa gera UnauthorizedException (401).<br/>Os vínculos são lidos como em SD-REL-03
        Note over S,MR: filtro = situacao AGENDADA, inicio com $lte de geradaEm e participantes com $elemMatch<br/>do usuário com presenca.situacao CONFIRMADA e presenca.entrada com $lte de geradaEm
        S->>MR: find(filtro)
        MR-->>S: query
        S->>Q: select("titulo local inicio fim dia_inteiro comissao_id participantes")
        Note over S,Q: opcoes = path comissao_id com select nome e organizacao_id<br/>e populate de organizacao_id com select nome
        S->>Q: populate(opcoes)
        S->>Q: sort({ inicio: -1, _id: -1 })
        S->>Q: lean()
        S->>Q: exec()
        Q-->>S: registros
        Note over S: Para cada reunião, usa a presença do usuário se for CONFIRMADA.<br/>minutos = parte inteira de (saida - entrada) em minutos quando a saída<br/>existe, não é anterior à entrada e não é futura, senão null.<br/>O resumo soma os minutos e conta os registros sem duração
        S-->>C: relatorio
        C-->>A: relatorio
        Note over A,C: Resposta 200 com o relatório em JSON
        A-->>T: resultado
        Note over T,A: resultado.ok é true, e resultado.dados traz o relatório
        Note over T: A resposta é descartada se pedido.current mudou depois do pedido
        T->>T: definirRelatorio(resultado.dados)
        T->>F: horasRelatorio(relatorio.resumo.minutos_registrados)
        F-->>T: horas
        Note over T,F: horas no formato "1h 15min"
        T->>F: dataRelatorio(item.inicio)
        F-->>T: dataHora
        Note over T,F: dataHora em pt-BR, no fuso do aparelho
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
    participant MR as ReuniaoModel
    participant Q as Query
    U->>T: definirPeriodo("mes")
    Note over T: carregar depende de periodo, então o useFocusEffect a executa de novo
    T->>T: carregar()
    T->>F: consultaRelatorio(periodo)
    Note over F: agora tem o valor padrão new Date()<br/>de = dia 1 do mês atual às 00:00 do aparelho<br/>ate = dia seguinte ao de hoje às 00:00 do aparelho<br/>os dois convertidos para UTC por toISOString
    F-->>T: consulta
    Note over T,F: consulta traz de com o início do período e ate com o fim, em UTC
    Note over T,A: caminho = "/relatorios/individual" seguido da consulta
    T->>A: api(caminho)
    Note over A,C: GET /relatorios/individual com de e ate na consulta<br/>ValidationPipe aplica RelatorioIndividualDto com IsISO8601 estrito
    alt de ou ate fora do formato ISO 8601 (só em chamada direta à API)
        Note over A,C: 400 "Informe uma data inicial válida" ou "Informe uma data final válida"
        A-->>T: resultado
        T->>T: definirErro(resultado.erro)
    else formato válido
        A->>C: individual(req, filtros)
        C->>S: individual(req.usuario.sub, filtros)
        Note over S: de = new Date(filtros.de) e ate = new Date(filtros.ate), quando informados
        alt de maior ou igual a ate
            S-->>C: BadRequestException
            C-->>A: BadRequestException
            Note over A,C: Resposta 400 com a mensagem "O fim do período deve ser posterior ao início"
            A-->>T: resultado
            T->>T: definirErro(resultado.erro)
        else período válido
            Note over S: Usuário e vínculos lidos como em SD-REL-01 e SD-REL-03.<br/>Os vínculos não usam de nem ate
            Note over S,MR: filtro = situacao AGENDADA, inicio com $lte de geradaEm, $gte de de e $lt de ate<br/>e participantes com $elemMatch do usuário com presenca.situacao CONFIRMADA
            S->>MR: find(filtro)
            MR-->>S: query
            Note over S,Q: select, populate e lean como em SD-REL-01
            S->>Q: sort({ inicio: -1, _id: -1 })
            S->>Q: exec()
            Q-->>S: registros
            S-->>C: relatorio
            Note over S,C: relatorio.periodo.de e relatorio.periodo.ate são os limites recebidos
            C-->>A: relatorio
            A-->>T: resultado
            Note over T: Se o usuário trocou de opção antes, esta resposta é descartada
            T->>T: definirRelatorio(resultado.dados)
            T->>F: periodoRelatorio(relatorio)
            F-->>T: textoPeriodo
            Note over T,F: textoPeriodo no formato "01/11/2026 a 26/11/2026"
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
    participant MU as UsuarioModel
    participant MO as OrganizacaoModel
    participant MC as ComissaoModel
    participant Q as Query
    U->>T: carregar()
    Note over T,A: caminho = "/relatorios/individual" seguido da consulta
    T->>A: api(caminho)
    Note over A,C: GET /relatorios/individual<br/>depois de ApiKeyGuard, JwtAuthGuard e UsuarioComumGuard
    A->>C: individual(req, filtros)
    C->>S: individual(req.usuario.sub, filtros)
    S->>MU: findById(usuarioId)
    MU-->>S: query
    Note over S,Q: select e lean como em SD-REL-01
    S->>Q: exec()
    Q-->>S: usuario
    Note over S,MO: filtro = status APROVADA e membros com $elemMatch<br/>de usuario_id em referenciasUsuario e status APROVADO
    S->>MO: find(filtro)
    MO-->>S: query
    S->>Q: select("nome membros")
    S->>Q: sort({ nome: 1, _id: 1 })
    S->>Q: lean()
    S->>Q: exec()
    Q-->>S: organizacoes
    Note over S,MC: filtro = ativo true, organizacao_id com $in de referenciasOrganizacoes<br/>e membros.usuario_id com $in de referenciasUsuario
    S->>MC: find(filtro)
    MC-->>S: query
    S->>Q: select("nome organizacao_id membros")
    S->>Q: sort({ nome: 1, _id: 1 })
    S->>Q: lean()
    S->>Q: exec()
    Q-->>S: comissoes
    Note over S: papel da organização = papel do item APROVADO do usuário em membros.<br/>papel da comissão = papel do item do usuário em membros.<br/>organizacao da comissão = nome lido na lista de organizações
    Note over S: As reuniões são lidas em seguida, como em SD-REL-01
    S-->>C: relatorio
    Note over S,C: relatorio.vinculos traz as organizações e as comissões
    C-->>A: relatorio
    A-->>T: resultado
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
    participant B as baixarPdfRelatorio
    participant G as pdfRelatorioIndividual
    participant JP as jsPDF
    participant UR as URL
    participant D as Document (document)
    participant L as HTMLAnchorElement (link)
    participant P as Print
    participant H as Sharing
    U->>T: exportar("pdf")
    Note over T: Não faz nada se travaExportacao já estiver ligada
    T->>T: definirExportando("pdf")
    T->>E: exportarRelatorioIndividual(relatorio, "pdf")
    alt Web
        E->>E: logoDoRelatorio()
        Note over E: Na web, a logo é lida por fetch e FileReader e vira data:image/png;base64
        E->>B: baixarPdfRelatorio(relatorio, logo)
        B->>B: fonteBase64(modulo)
        Note over B: Busca as fontes Poppins Regular e SemiBold por fetch, com limite de 15 s cada,<br/>e as converte em base64. Falha: "Não foi possível carregar a fonte do relatório."
        B->>G: gerarPdfRelatorioIndividual(relatorio, recursos)
        Note over B,G: recursos = logo, fonteRegular e fonteNegrito
        G->>JP: addFont(arquivo, "Poppins", estilo)
        G->>JP: text(texto, x, y)
        Note over G,JP: Monta o A4 com a logo, a identificação, os vínculos, o resumo e um bloco por reunião;<br/>abre página nova quando o bloco não cabe e numera as páginas
        G-->>B: pdf
        B->>JP: output("blob")
        JP-->>B: blob
        B->>UR: createObjectURL(blob)
        UR-->>B: url
        B->>D: createElement("a")
        D-->>B: link
        Note over B,L: link.href = url e link.download = "relatorio-individual.pdf".<br/>O link é posto em document.body antes do clique
        B->>L: click()
        Note over L: O navegador baixa relatorio-individual.pdf, sem janela nem diálogo de impressão
        B->>L: remove()
        Note over B,UR: A liberação é agendada por setTimeout para 10 segundos depois
        B->>UR: revokeObjectURL(url)
        B-->>E: void
    else Expo Go
        E->>E: logoDoRelatorio()
        E->>F: htmlRelatorioIndividual(relatorio, logo)
        F-->>E: html
        E->>P: printAsync({ html, width: 595, height: 842 })
        Note over P: Abre a tela de impressão do sistema
        P-->>E: void
    else Aplicativo instalado
        E->>E: logoDoRelatorio()
        E->>F: htmlRelatorioIndividual(relatorio, logo)
        F-->>E: html
        E->>H: isAvailableAsync()
        H-->>E: true
        E->>P: printToFileAsync({ html, width: 595, height: 842 })
        P-->>E: uri
        Note over E,P: uri é o campo do FilePrintResult com o endereço do PDF gerado no cache
        Note over E,H: opcoes = mimeType "application/pdf", UTI "com.adobe.pdf"<br/>e dialogTitle "Exportar relatório individual"
        E->>H: shareAsync(uri, opcoes)
        Note over H: Abre o menu de compartilhamento com o arquivo gerado, sem mover nem renomear
        H-->>E: void
    end
    alt exportação concluída
        E-->>T: void
    else falha da logo, da fonte, da impressão ou do compartilhamento
        E-->>T: erro
        Note over T,E: erro é o Error lançado, com a mensagem em erro.message
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
    participant UR as URL
    participant D as Document (document)
    participant L as HTMLAnchorElement (link)
    participant FS as File
    participant H as Sharing
    U->>T: exportar("csv")
    Note over T: Não faz nada se travaExportacao já estiver ligada
    T->>T: definirExportando("csv")
    T->>E: exportarRelatorioIndividual(relatorio, "csv")
    alt Web
        E->>F: csvRelatorioIndividual(relatorio)
        F->>F: celulaCsv(valor)
        F-->>E: csv
        Note over E,F: csv é o texto com BOM e 19 colunas
        Note over E,UR: blob = new Blob com o csv e o tipo text/csv, charset utf-8
        E->>UR: createObjectURL(blob)
        UR-->>E: url
        E->>D: createElement("a")
        D-->>E: link
        Note over E,L: link.href = url e link.download = "relatorio-individual.csv".<br/>O link é posto em document.body antes do clique
        E->>L: click()
        Note over L: O navegador baixa relatorio-individual.csv
        E->>L: remove()
        Note over E,UR: A liberação é agendada por setTimeout para 1 segundo depois
        E->>UR: revokeObjectURL(url)
    else Aplicativo instalado ou Expo Go
        E->>H: isAvailableAsync()
        H-->>E: true
        Note over E,FS: nome = "relatorio-individual-" seguido de Date.now() e ".csv"
        E->>FS: new File(Paths.cache, nome)
        FS-->>E: arquivo
        E->>FS: create()
        E->>F: csvRelatorioIndividual(relatorio)
        F->>F: celulaCsv(valor)
        F-->>E: csv
        E->>FS: write(csv)
        FS-->>E: void
        Note over E,H: opcoes = mimeType "text/csv", UTI "public.comma-separated-values-text"<br/>e dialogTitle "Exportar relatório individual"
        E->>H: shareAsync(arquivo.uri, opcoes)
        Note over H: Abre o menu de compartilhamento do aparelho
        H-->>E: void
    end
    alt exportação concluída
        E-->>T: void
    else compartilhamento indisponível, falha na gravação ou no compartilhamento
        E-->>T: erro
        Note over T,E: erro é o Error lançado, com a mensagem em erro.message
        T->>T: definirErroExportacao(erro.message)
    end
    T->>T: definirExportando(null)
```

## Rastreabilidade

SD-REL-01 a SD-REL-05 correspondem, na mesma ordem, a UC-REL-01 a UC-REL-05 e a RF-REL-01 a RF-REL-05.
