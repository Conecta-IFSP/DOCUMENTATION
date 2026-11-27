# Diagrama de classes — Informações do sistema

> Padrão da disciplina · Módulo SIS · Revisado em 26/11/2026

## Resumo

O diagrama mostra as classes que participam das informações do sistema, agrupadas em três pacotes, um por camada: **Aplicativo** (telas, componentes de confirmação e de campo de texto e os módulos `api` e `autenticacaoLocal`), **API** (guards, controller, serviço e o objeto de entrada do reset) e **Dados** (documentos guardados no MongoDB que o serviço conta, cria ou apaga). Cada atributo traz o seu tipo, e cada método traz o tipo dos parâmetros e do retorno. Toda classe e todo método do diagrama existem no código. O único objeto de entrada é o `ResetSistemaDto`, corpo do reset; as outras rotas não recebem corpo. O módulo não cria coleção própria.

## Diagrama

```mermaid
classDiagram
    direction LR

    namespace Aplicativo {
        class TelaInicio {
            -administradorDoSistema: boolean
            +carregar(continuar: function) Promise~void~
        }
        class TelaSistema {
            -status: Status
            -relatorios: Relatorios
            -carregando: boolean
            -aviso: Aviso
            -acaoEmAndamento: string
            -confirmacaoReset: boolean
            -textoReset: string
            -confirmacaoTeste: boolean
            +carregar() Promise~void~
            +popularDadosDeTeste() Promise~void~
            +abrirReset() void
            +fecharReset() void
            +resetarSistema() Promise~void~
            -formatarTempoAtivo(segundos: number) string
        }
        class Confirmacao {
            <<component>>
            +titulo: string
            +mensagem: string
            +carregando: boolean
            +rotuloConfirmar: string
            +perigo: boolean
            +confirmar() void
            +cancelar() void
        }
        class FolhaInferior {
            <<component>>
            +visivel: boolean
            +titulo: string
            +acompanharTeclado: boolean
            +aoFechar() void
        }
        class CampoTexto {
            <<component>>
            +rotulo: string
            +value: string
            +onChangeText(texto: string) void
        }
        class api {
            <<module>>
            +api(caminho: string, metodo: string, corpo: unknown) Promise~Resultado~
            +obterToken() Promise~string~
            +removerToken() Promise~void~
        }
        class autenticacaoLocal {
            <<module>>
            +dispositivoProtegido() Promise~boolean~
            +autenticarComDispositivo() Promise~boolean~
        }
    }

    namespace API {
        class ApiKeyGuard {
            -configService: ConfigService
            +canActivate(context: ExecutionContext) boolean
            -isPublicBrowserRedirect(request: Request) boolean
            -chavesIguais(recebida: string, esperada: string) boolean
        }
        class JwtAuthGuard {
            -jwtService: JwtService
            -usuarios: Model~Usuario~
            +canActivate(context: ExecutionContext) Promise~boolean~
            -extrairToken(requisicao: Request) string
        }
        class AdminSistemaGuard {
            +canActivate(context: ExecutionContext) boolean
        }
        class SistemaController {
            -sistemaService: SistemaService
            +status() Promise~object~
            +relatorios() Promise~object~
            +reset(_confirmacao: ResetSistemaDto) Promise~object~
            +popularDadosDeTeste() Promise~object~
        }
        class ResetSistemaDto {
            +confirmacao: string
        }
        class SistemaService {
            -usuarios: Model~Usuario~
            -organizacoes: Model~Organizacao~
            -comissoes: Model~Comissao~
            -reunioes: Model~Reuniao~
            -conexao: Connection
            +status() Promise~object~
            +relatorios() Promise~object~
            +reset() Promise~object~
            +popularDadosDeTeste() Promise~object~
            -obterConfiguracaoUsuariosTeste() object
        }
    }

    namespace Dados {
        class Usuario {
            +_id: ObjectId
            +nome: string
            +email: string
            +senha_hash: string
            +tipo: TipoUsuario
            +ativo: boolean
            +email_verificado: boolean
            +senha_alterada_em: Date
        }
        class Organizacao {
            +_id: ObjectId
            +nome: string
            +cidade: string
            +descricao: string
            +status: StatusOrganizacao
            +criada_por: ObjectId
            +codigo_acesso: string
            +membros: MembroOrganizacao[]
        }
        class MembroOrganizacao {
            +usuario_id: ObjectId
            +papel: PapelOrganizacao
            +status: StatusMembroOrganizacao
            +solicitado_em: Date
            +aprovado_em: Date
        }
        class Comissao {
            +_id: ObjectId
            +nome: string
            +descricao: string
            +organizacao_id: ObjectId
            +membros: MembroComissao[]
            +ativo: boolean
        }
        class MembroComissao {
            +usuario_id: ObjectId
            +papel: PapelComissao
            +adicionado_em: Date
        }
        class Reuniao {
            +_id: ObjectId
            +comissao_id: ObjectId
        }
        class Contador {
            +_id: string
            +valor: number
        }
        class Notificacao {
            +_id: ObjectId
            +usuario_id: ObjectId
            +chave: string
        }
        class TipoUsuario {
            <<enumeration>>
            USUARIO
            ADMIN_SISTEMA
        }
        class StatusOrganizacao {
            <<enumeration>>
            PENDENTE
            APROVADA
            REVOGADA
        }
    }

    TelaInicio ..> TelaSistema : abre
    TelaSistema ..> Confirmacao : usa
    TelaSistema ..> FolhaInferior : usa
    TelaSistema ..> CampoTexto : usa
    TelaSistema ..> autenticacaoLocal : usa
    TelaSistema ..> api : usa
    api ..> SistemaController : requisição HTTP
    SistemaController ..> JwtAuthGuard : UseGuards
    SistemaController ..> AdminSistemaGuard : UseGuards
    SistemaController ..> ResetSistemaDto : recebe
    JwtAuthGuard ..> Usuario : confere a conta

    SistemaController --> "1" SistemaService : sistemaService
    SistemaService ..> Usuario : conta, cria e apaga
    SistemaService ..> Organizacao : conta, cria e apaga
    SistemaService ..> Comissao : conta, cria e apaga
    SistemaService ..> Reuniao : apaga
    SistemaService ..> Contador : apaga pela conexão
    SistemaService ..> Notificacao : apaga pela conexão

    Organizacao "1" *-- "0..*" MembroOrganizacao : membros
    Comissao "1" *-- "1..*" MembroComissao : membros
    Organizacao "0..*" --> "1" Usuario : criada_por
    MembroOrganizacao "0..*" --> "1" Usuario : usuario_id
    MembroComissao "0..*" --> "1" Usuario : usuario_id
    Comissao "0..*" --> "1" Organizacao : organizacao_id
    Reuniao "0..*" --> "1" Comissao : comissao_id
    Notificacao "0..*" --> "1" Usuario : usuario_id
    Usuario ..> TipoUsuario : tipo
    Organizacao ..> StatusOrganizacao : status
```

Figura 2 — Diagrama de classes das informações do sistema, por camada. Fonte: equipe Conecta+.

## Como ler as ligações

| Ligação                      | Significado                                                             | Exemplo no diagrama                                                                                                      |
| ---------------------------- | ----------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------ |
| Losango cheio (composição)   | A parte só existe dentro do todo e é apagada junto com ele.             | `Organizacao *-- MembroOrganizacao`: os membros ficam dentro do documento da organização e somem quando o reset a apaga.   |
| Seta cheia (associação)      | Uma classe guarda uma referência a outra, que existe por conta própria. | `Comissao --> Organizacao`: `organizacao_id` guarda o identificador de um documento de `organizacoes`.                    |
| Seta tracejada (dependência) | Uma classe usa a outra em uma chamada, sem guardá-la como atributo.     | `SistemaService ..> Reuniao`: o serviço só chama `deleteMany` no model das reuniões.                                     |

Os números nas pontas são as multiplicidades. Por exemplo, uma `Comissao` tem pelo menos um membro (`1..*`), porque o schema exige exatamente um `RESPONSAVEL`; uma `Organizacao` pode ter nenhum ou muitos membros (`0..*`).

`SistemaService` não usa um model para `Contador`, para `Notificacao` nem para os arquivos do GridFS: apaga as coleções `contadores`, `notificacoes`, `anexos.files` e `anexos.chunks` pela conexão do Mongoose, chamando `conexao.db.collection(nome).deleteMany({})`. Por isso as ligações com `Contador` e `Notificacao` são de dependência, e os arquivos do GridFS, que não têm classe no código, aparecem apenas no [diagrama relacional](4%20-%20diagrama-relacional-informacao-sistemas.md).

Os métodos do controller e do serviço devolvem objetos montados na hora, sem classe própria no código; por isso o retorno aparece como `Promise<object>`. Os campos de cada objeto estão no [dicionário de classes](3%20-%20dicionario-dados-classes-informacao-sistemas.md). `Status`, `Relatorios` e `Aviso`, na `TelaSistema`, são tipos TypeScript do aplicativo, também descritos lá. O `ResetSistemaDto` é validado pelo `ValidationPipe` antes do controller; o controller recebe o objeto no parâmetro `_confirmacao`, mas não o repassa ao serviço.

Os guards não chamam uns aos outros: o NestJS executa o método `canActivate` de cada um, na ordem `ApiKeyGuard`, `JwtAuthGuard`, `AdminSistemaGuard`, antes de chamar o controller. `JwtAuthGuard` e `AdminSistemaGuard` estão declarados no decorador `@UseGuards` do `SistemaController`, por isso aparecem como dependências dele. `ApiKeyGuard` vale para toda a API, registrado como `APP_GUARD` em `AppModule`, e por isso não tem ligação com uma classe deste módulo.

## Rotas atendidas pelo controller

As rotas HTTP não são métodos; por isso ficam nesta tabela, e não no diagrama.

| Método do controller  | Rota HTTP                     |
| --------------------- | ----------------------------- |
| `status`              | `GET /sistema/status`         |
| `relatorios`          | `GET /sistema/relatorios`     |
| `popularDadosDeTeste` | `POST /sistema/popular-teste` |
| `reset`               | `POST /sistema/reset`         |

## Regras representadas

- só a conta com `tipo` igual a `ADMIN_SISTEMA` passa pelo `AdminSistemaGuard`;
- `SistemaService` lê e grava diretamente nas coleções de outros módulos; ele não tem schema próprio;
- o reset só chega ao serviço quando o corpo traz `confirmacao` igual a "RESETAR";
- o reset apaga `Organizacao`, `Comissao`, `Reuniao` e `Notificacao` por inteiro, com tudo o que está dentro delas, e de `Usuario` só as contas cujo `tipo` não é `ADMIN_SISTEMA`;
- o aplicativo nunca calcula os totais: ele mostra o que a API devolve.

## Rastreabilidade

RF-SIS-01 a RF-SIS-04; complementado por [Dicionário de classes](3%20-%20dicionario-dados-classes-informacao-sistemas.md) e [Diagramas de sequência](6%20-%20diagramas-sequencia-informacao-sistemas.md).
