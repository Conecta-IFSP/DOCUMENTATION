# Diagrama de classes — Relatório individual

> Padrão da disciplina · Módulo REL · Revisado em 27/11/2026

## Resumo

O diagrama mostra as classes que participam do relatório individual, agrupadas em três pacotes, um por camada do sistema: **Aplicativo** (tela, módulos e tipos do aplicativo), **API** (guards, controller, service e objeto de entrada) e **Dados** (os models do Mongoose, marcados `<<Model Mongoose>>`, e os documentos do MongoDB lidos pelo relatório). As bibliotecas chamadas diretamente nos [Diagramas de sequência](6%20-%20diagramas-sequencia-relatorio-individual.md) ficam fora dos pacotes, marcadas `<<biblioteca>>` (Mongoose, Expo) ou `<<API do navegador>>`, só com os membros usados. Toda linha de vida e todo método das sequências aparecem aqui, e cada classe está descrita no [Dicionário de classes](3%20-%20dicionario-dados-classes-relatorio-individual.md). Cada atributo traz o seu tipo, e cada método traz o tipo dos parâmetros e do retorno. Toda classe e todo método do diagrama existem no código ou nas bibliotecas que ele chama; os módulos sem classe, como `api` e `relatorioIndividual`, aparecem com o estereótipo `<<module>>`, e os tipos TypeScript com `<<type>>`.

## Diagrama

```mermaid
classDiagram
    direction LR

    namespace Aplicativo {
        class TelaRelatorioIndividual {
            -periodo: PeriodoRelatorio
            -relatorio: RelatorioIndividual | null
            -carregando: boolean
            -erro: string
            -erroExportacao: string
            -exportando: pdf | csv | null
            -pedido: RefObject~number~
            -travaExportacao: RefObject~boolean~
            +carregar() Promise~void~
            +exportar(formato: pdf | csv) Promise~void~
            -definirPeriodo(valor: PeriodoRelatorio) void
            -definirCarregando(valor: boolean) void
            -definirRelatorio(valor: RelatorioIndividual | null) void
            -definirErro(valor: string) void
            -definirErroExportacao(valor: string) void
            -definirExportando(valor: pdf | csv | null) void
        }
        class api {
            <<module>>
            +api(caminho: string, metodo: string, corpo: unknown) Promise~Resultado~
            +removerToken() Promise~void~
        }
        class relatorioIndividual {
            <<module>>
            +consultaRelatorio(periodo: PeriodoRelatorio, agora: Date) string
            +horasRelatorio(minutos: number) string
            +dataRelatorio(data: string | null) string
            +periodoRelatorio(relatorio: RelatorioIndividual) string
            +htmlRelatorioIndividual(r: RelatorioIndividual, logo: string) string
            +celulaCsv(valor: string | number | null) string
            +csvRelatorioIndividual(r: RelatorioIndividual) string
            -escapar(texto: string) string
        }
        class exportarRelatorioIndividual {
            <<module>>
            +exportarRelatorioIndividual(relatorio: RelatorioIndividual, formato: pdf | csv) Promise~void~
            -logoDoRelatorio() Promise~string~
        }
        class baixarPdfRelatorio {
            <<module>>
            +baixarPdfRelatorio(relatorio: RelatorioIndividual, logo: string) Promise~void~
            -fonteBase64(modulo: number) Promise~string~
        }
        class pdfRelatorioIndividual {
            <<module>>
            +gerarPdfRelatorioIndividual(r: RelatorioIndividual, recursos: RecursosPdfRelatorio) jsPDF
        }
        class pdf {
            <<module>>
            +pdfPelaImpressao() boolean
        }
        class RelatorioIndividual {
            <<type>>
            +usuario.nome: string
            +usuario.email: string
            +vinculos?.organizacoes[].id: string
            +vinculos?.organizacoes[].nome: string
            +vinculos?.organizacoes[].papel: ADMIN | MEMBRO
            +vinculos?.comissoes[].id: string
            +vinculos?.comissoes[].nome: string
            +vinculos?.comissoes[].organizacao_id: string
            +vinculos?.comissoes[].organizacao: string
            +vinculos?.comissoes[].papel: RESPONSAVEL | MEMBRO
            +gerada_em: string
            +periodo.de: string | null
            +periodo.ate: string | null
            +resumo.reunioes: number
            +resumo.minutos_registrados: number
            +resumo.registros_sem_duracao: number
            +reunioes: ParticipacaoIndividual[]
        }
        class ParticipacaoIndividual {
            <<type>>
            +reuniao_id: string
            +titulo: string
            +comissao: string
            +organizacao: string
            +local: string | null
            +inicio: string
            +fim: string
            +dia_inteiro: boolean
            +entrada: string
            +saida: string | null
            +saida_automatica?: boolean
            +minutos: number | null
        }
        class PeriodoRelatorio {
            <<type>>
            todos
            mes
            ano
        }
    }

    namespace API {
        class ApiKeyGuard {
            -configService: ConfigService
            +canActivate(context: ExecutionContext) boolean
        }
        class JwtAuthGuard {
            -jwtService: JwtService
            -usuarios: Model~UsuarioDocument~
            +canActivate(context: ExecutionContext) Promise~boolean~
        }
        class UsuarioComumGuard {
            +canActivate(context: ExecutionContext) boolean
        }
        class RelatoriosController {
            -relatorios: RelatoriosService
            +individual(req: RequisicaoAutenticada, filtros: RelatorioIndividualDto) Promise~RelatorioIndividual~
        }
        class RelatoriosService {
            -reunioes: Model~ReuniaoDocument~
            -usuarios: Model~UsuarioDocument~
            -organizacoes: Model~OrganizacaoDocument~
            -comissoes: Model~ComissaoDocument~
            +individual(usuarioId: string, filtros: RelatorioIndividualDto) Promise~RelatorioIndividual~
        }
        class RelatorioIndividualDto {
            +de?: string
            +ate?: string
        }
    }

    namespace Dados {
        class UsuarioModel {
            <<Model Mongoose>>
            +findById(id: string) Query~UsuarioDocument~
            +findOne(filtro: FilterQuery) Query~UsuarioDocument~
        }
        class ReuniaoModel {
            <<Model Mongoose>>
            +find(filtro: FilterQuery) Query~ReuniaoDocument[]~
        }
        class OrganizacaoModel {
            <<Model Mongoose>>
            +find(filtro: FilterQuery) Query~OrganizacaoDocument[]~
        }
        class ComissaoModel {
            <<Model Mongoose>>
            +find(filtro: FilterQuery) Query~ComissaoDocument[]~
        }
        class Usuario {
            +_id: ObjectId
            +nome: string
            +email: string
            +tipo: TipoUsuario
            +ativo: boolean
            +senha_alterada_em?: Date
        }
        class Reuniao {
            +_id: ObjectId
            +comissao_id: ObjectId
            +titulo: string
            +local: string
            +inicio: Date
            +fim: Date
            +dia_inteiro: boolean
            +situacao: SituacaoReuniao
            +participantes: Participante[]
        }
        class Participante {
            +usuario_id: ObjectId
            +convite: RespostaConvite
            +presenca?: Presenca
        }
        class Presenca {
            +entrada: Date
            +saida?: Date
            +saida_automatica?: boolean
            +situacao: SituacaoPresenca
            +marcada_por: OrigemPresenca
        }
        class Organizacao {
            +_id: ObjectId
            +nome: string
            +status: StatusOrganizacao
            +membros: MembroOrganizacao[]
        }
        class MembroOrganizacao {
            +usuario_id: ObjectId
            +papel: PapelOrganizacao
            +status: StatusMembroOrganizacao
        }
        class Comissao {
            +_id: ObjectId
            +nome: string
            +organizacao_id: ObjectId
            +ativo: boolean
            +membros: MembroComissao[]
        }
        class MembroComissao {
            +usuario_id: ObjectId
            +papel: PapelComissao
        }
        class SituacaoPresenca {
            <<enumeration>>
            AGUARDANDO
            CONFIRMADA
            RECUSADA
        }
        class StatusMembroOrganizacao {
            <<enumeration>>
            PENDENTE
            APROVADO
            REJEITADO
            CONVIDADO
        }
    }

    class Query {
        <<biblioteca>>
        +select(campos: string) Query
        +populate(opcoes: PopulateOptions) Query
        +sort(ordem: object) Query
        +lean() Query
        +exec() Promise~unknown~
    }
    class Print {
        <<biblioteca>>
        +printAsync(opcoes: PrintOptions) Promise~void~
        +printToFileAsync(opcoes: FilePrintOptions) Promise~FilePrintResult~
    }
    class Sharing {
        <<biblioteca>>
        +isAvailableAsync() Promise~boolean~
        +shareAsync(url: string, opcoes: SharingOptions) Promise~void~
    }
    class File {
        <<biblioteca>>
        +uri: string
        +File(...uris: string[])
        +create() void
        +write(conteudo: string) Promise~void~
        +base64() Promise~string~
    }
    class jsPDF {
        <<biblioteca>>
        +addFont(arquivo: string, familia: string, estilo: string) string
        +text(texto: string, x: number, y: number) jsPDF
        +output(tipo: blob) Blob
    }
    class Document {
        <<API do navegador>>
        +body: HTMLElement
        +createElement(etiqueta: string) HTMLElement
    }
    class URL {
        <<API do navegador>>
        +createObjectURL(objeto: Blob) string$
        +revokeObjectURL(url: string) void$
    }
    class HTMLAnchorElement {
        <<API do navegador>>
        +href: string
        +download: string
        +click() void
        +remove() void
    }

    TelaRelatorioIndividual ..> api : consulta
    TelaRelatorioIndividual ..> relatorioIndividual : formata
    TelaRelatorioIndividual ..> exportarRelatorioIndividual : exporta
    TelaRelatorioIndividual ..> pdf : usa
    TelaRelatorioIndividual ..> RelatorioIndividual : mostra
    exportarRelatorioIndividual ..> relatorioIndividual : monta HTML e CSV
    exportarRelatorioIndividual ..> Print : usa
    exportarRelatorioIndividual ..> Sharing : usa
    exportarRelatorioIndividual ..> File : usa
    exportarRelatorioIndividual ..> baixarPdfRelatorio : PDF na web
    exportarRelatorioIndividual ..> Document : cria o link do CSV
    exportarRelatorioIndividual ..> URL : endereço do CSV na web
    exportarRelatorioIndividual ..> HTMLAnchorElement : baixa o CSV
    baixarPdfRelatorio ..> pdfRelatorioIndividual : monta o PDF
    baixarPdfRelatorio ..> jsPDF : gera o Blob
    baixarPdfRelatorio ..> Document : cria o link do PDF
    baixarPdfRelatorio ..> URL : endereço do PDF
    baixarPdfRelatorio ..> HTMLAnchorElement : baixa o PDF
    pdfRelatorioIndividual ..> jsPDF : usa
    pdfRelatorioIndividual ..> relatorioIndividual : formata datas, horas e período
    RelatorioIndividual "1" *-- "0..*" ParticipacaoIndividual : reunioes
    api ..> RelatoriosController : requisição HTTP

    RelatoriosController ..> ApiKeyGuard : protegido por
    RelatoriosController ..> JwtAuthGuard : protegido por
    RelatoriosController ..> UsuarioComumGuard : protegido por
    RelatoriosController --> "1" RelatoriosService : relatorios
    RelatoriosController ..> RelatorioIndividualDto : recebe
    RelatoriosService "1" --> "1" UsuarioModel : usuarios
    RelatoriosService "1" --> "1" ReuniaoModel : reunioes
    RelatoriosService "1" --> "1" OrganizacaoModel : organizacoes
    RelatoriosService "1" --> "1" ComissaoModel : comissoes
    JwtAuthGuard "1" --> "1" UsuarioModel : usuarios
    UsuarioModel ..> Usuario : documentos do schema
    ReuniaoModel ..> Reuniao : documentos do schema
    OrganizacaoModel ..> Organizacao : documentos do schema
    ComissaoModel ..> Comissao : documentos do schema
    UsuarioModel ..> Query : devolve
    ReuniaoModel ..> Query : devolve
    OrganizacaoModel ..> Query : devolve
    ComissaoModel ..> Query : devolve

    Reuniao "1" *-- "0..*" Participante : participantes
    Participante "1" *-- "0..1" Presenca : presenca
    Organizacao "1" *-- "0..*" MembroOrganizacao : membros
    Comissao "1" *-- "1..*" MembroComissao : membros
    Participante "0..*" --> "1" Usuario : usuario_id
    MembroOrganizacao "0..*" --> "1" Usuario : usuario_id
    MembroComissao "0..*" --> "1" Usuario : usuario_id
    Reuniao "0..*" --> "1" Comissao : comissao_id
    Comissao "0..*" --> "1" Organizacao : organizacao_id
    Presenca ..> SituacaoPresenca : situacao
    MembroOrganizacao ..> StatusMembroOrganizacao : status
```

Figura 2 — Diagrama de classes do relatório individual, por camada. Fonte: equipe Conecta+.

## Como ler as ligações

| Ligação                      | Significado                                                             | Exemplo no diagrama                                                                                       |
| ---------------------------- | ----------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------- |
| Losango cheio (composição)   | A parte só existe dentro do todo e é apagada junto com ele.             | `Organizacao *-- MembroOrganizacao`: o vínculo é gravado dentro do documento da organização.              |
| Seta cheia (associação)      | Uma classe guarda uma referência a outra, que existe por conta própria. | `Comissao --> Organizacao`: `organizacao_id` guarda o identificador de um documento de `organizacoes`.    |
| Seta tracejada (dependência) | Uma classe usa a outra em uma chamada, sem guardá-la como atributo.     | `ComissaoModel ..> Comissao`: o model lê os documentos do schema `Comissao`.                               |

Os números nas pontas são as multiplicidades. Por exemplo, uma `Comissao` tem pelo menos um membro (`1..*`), porque precisa ter exatamente um responsável; um `Participante` tem no máximo uma `Presenca` (`0..1`); um relatório tem qualquer quantidade de participações (`0..*`), inclusive nenhuma; e o `RelatoriosService` guarda exatamente um model de cada coleção (`1`), injetado pelo NestJS.

`UsuarioModel`, `ReuniaoModel`, `OrganizacaoModel` e `ComissaoModel` são os models do Mongoose guardados nos atributos `usuarios`, `reunioes`, `organizacoes` e `comissoes` do serviço (o `JwtAuthGuard` guarda o mesmo `UsuarioModel` em `usuarios`); no código, o tipo de cada atributo é `Model<...Document>`. Cada model mostra só os métodos chamados por este módulo: `findById` e `findOne` em `UsuarioModel` e `find` nos outros três. Esses métodos devolvem uma `Query`, e é na `Query` que o serviço chama `select`, `populate`, `sort`, `lean` e `exec`; por isso, nas sequências, essas chamadas vão para a linha de vida `Query`. Os métodos `definir…` da tela são as funções de atualização de estado criadas por `useState`.

Na web, desde o FRONTEND `299afc2`, o PDF é gerado pelo módulo `pdfRelatorioIndividual` (`src/utils/pdf-relatorio-individual.ts`) com a biblioteca `jsPDF` 4.2.1 e baixado pelo módulo `baixarPdfRelatorio` (`src/services/baixar-pdf-relatorio.web.ts`; no celular, o Metro usa `baixar-pdf-relatorio.ts`, que só lança um erro e não é chamado). A classe `jsPDF` mostra só os métodos que aparecem nas sequências; o módulo também chama `addFileToVFS`, `setProperties`, `addImage`, `splitTextToSize`, `addPage`, `setFont`, `setFontSize`, `setTextColor`, `setDrawColor`, `line`, `getNumberOfPages` e `setPage`. A exportação usa as classes do navegador: `Document` para `document`, no PDF e no CSV; `URL` para os métodos estáticos `createObjectURL` e `revokeObjectURL`, sublinhados no diagrama; e `HTMLAnchorElement` para o link criado por `document.createElement('a')`.

O objeto devolvido pela API não tem classe própria no servidor: o tipo é deduzido pelo TypeScript a partir do que `RelatoriosService.individual` monta. O tipo `RelatorioIndividual` do aplicativo descreve exatamente esse objeto e por isso aparece como retorno do controller e do service. Os campos `usuario`, `vinculos`, `periodo` e `resumo` são objetos escritos dentro do próprio tipo, sem nome; no diagrama, eles aparecem com o caminho do campo, como `usuario.nome` e `vinculos?.organizacoes[].papel`, em que `[]` indica um item da lista. Os valores `pdf | csv` são os dois textos aceitos como formato de exportação. O `?` indica campo opcional: `vinculos` é opcional no aplicativo para que ele funcione com um servidor que ainda não envia vínculos. As enumerações não mostradas (`TipoUsuario`, `SituacaoReuniao`, `RespostaConvite`, `OrigemPresenca`, `StatusOrganizacao`, `PapelOrganizacao` e `PapelComissao`) estão no [dicionário de classes](3%20-%20dicionario-dados-classes-relatorio-individual.md).

## Rotas atendidas pelo controller

As rotas HTTP não são métodos; por isso ficam nesta tabela, e não no diagrama.

| Método do controller | Rota HTTP                              |
| -------------------- | -------------------------------------- |
| `individual`         | `GET /relatorios/individual?de=&ate=`  |

## Regras representadas

- o relatório só lê: nenhuma classe do pacote Dados é gravada por este módulo;
- o usuário do relatório vem sempre do token (`req.usuario.sub`); `RelatorioIndividualDto` não tem campo de usuário;
- `ParticipacaoIndividual` não existe no banco: é calculada a partir de uma `Reuniao` e da `Presenca` do usuário nela;
- os vínculos dependem de `Organizacao.status`, `MembroOrganizacao.status`, `Comissao.ativo` e da presença do usuário em `Comissao.membros`;
- a exportação usa só o objeto já carregado na tela; `Print`, `Sharing` e `File` são usados no celular, e `jsPDF`, `Document`, `URL` e `HTMLAnchorElement`, na web.

## Rastreabilidade

RF-REL-01 a RF-REL-05; complementado por [Dicionário de classes](3%20-%20dicionario-dados-classes-relatorio-individual.md) e [Diagramas de sequência](6%20-%20diagramas-sequencia-relatorio-individual.md).
