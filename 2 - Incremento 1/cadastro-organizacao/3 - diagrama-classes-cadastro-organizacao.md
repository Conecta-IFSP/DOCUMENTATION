# Diagrama de classes — Cadastro de organização

> Padrão da disciplina · Módulo ORG · Revisado em 27/11/2026

## Resumo

O diagrama mostra as classes que participam do cadastro de organização, agrupadas em três pacotes, um por camada do sistema: **Aplicativo** (telas, componente de confirmação e módulos do aplicativo), **API** (controller, serviço, guardas de acesso e objetos de entrada) e **Dados** (os models do Mongoose `OrganizacaoModel`, `UsuarioModel` e `ComissaoModel`, marcados `<<Model Mongoose>>`, o módulo `organizacaoSchema`, os documentos guardados no MongoDB e as enumerações). Para caber legível na largura da página, o diagrama está dividido em oito partes. Toda linha de vida e todo método dos [Diagramas de sequência](6%20-%20diagramas-sequencia-cadastro-organizacao.md) aparecem aqui, numa das partes. Cada atributo traz o seu tipo, e cada método traz o tipo dos parâmetros e do retorno. Toda classe e todo método do diagrama existem no código. As telas são componentes de função do React: os atributos são os estados mais relevantes e os métodos são as funções internas que o usuário dispara, as funções de atualização do `useState` (`definir*`) chamadas nas sequências e, em `TelaGerenciarOrganizacoes`, o `reset` do formulário de edição.

## Diagrama

Cada classe tem os atributos e os métodos numa única parte, indicada na tabela; nas outras partes em que aparece, por ser ponta de uma ligação, ela vem só com o nome, sem atributos nem métodos, e a linha abaixo da figura diz em que parte está detalhada.

No `OrganizacoesController` e no `OrganizacoesService`, os parâmetros dos métodos aparecem só com o tipo, na ordem do código, porque as assinaturas com os nomes passam da largura da página; o nome de cada parâmetro está no [dicionário de classes](3%20-%20dicionario-dados-classes-cadastro-organizacao.md#operações-do-organizacoescontroller).

| Parte | Figura | Classes detalhadas |
| ----- | ------ | ------------------ |
| 1 | Figura 2 | `TelaCadastroOrganizacao`, `TelaEntrarComCodigo`, `TelaInicio`, `validacao` |
| 2 | Figura 3 | `TelaOrganizacoes`, `TelaOrganizacao`, `TelaGerenciarOrganizacoes`, `Confirmacao` |
| 3 | Figura 4 | `api`, `OrganizacoesController` |
| 4 | Figura 5 | `JwtAuthGuard`, `UsuarioAtivoGuard`, `UsuarioComumGuard`, `AdminSistemaGuard`, `CreateOrganizacaoDto`, `UpdateOrganizacaoDto`, `EntrarComCodigoDto`, `ConvidarMembroDto`, `ResponderConviteDto`, `UpdateStatusMembroDto`, `UpdatePapelMembroDto` |
| 5 | Figura 6 | `OrganizacoesService` |
| 6 | Figura 7 | `OrganizacaoModel` |
| 7 | Figura 8 | `ComissaoModel`, `Comissao`, `organizacaoSchema`, `Organizacao`, `MembroOrganizacao`, `StatusOrganizacao`, `PapelOrganizacao`, `StatusMembroOrganizacao` |
| 8 | Figura 9 | `UsuarioModel`, `Usuario`, `TipoUsuario` |

### Parte 1 — Telas de cadastro, de entrada e inicial

```mermaid
classDiagram
    direction TB

    namespace Aplicativo {
        class TelaCadastroOrganizacao {
            -erro: string
            -enviada: DadosOrganizacao
            +enviar() Promise~void~
            +definirErro(valor: string) void
            +definirEnviada(valor: DadosOrganizacao) void
        }
        class TelaEntrarComCodigo {
            -erro: string
            -enviado: Pedido
            +enviar() Promise~void~
            +definirEnviado(valor: Pedido) void
        }
        class TelaInicio {
            -organizacoes: Organizacao[]
            -convitesAbertos: boolean
            +carregar(continuar: Function) Promise~void~
            +responder(convite: Convite, aceito: boolean) Promise~void~
        }
        class validacao {
            <<module>>
            +validarNomeOrganizacao(nome: string) string
            +validarCidade(cidade: string) string
            +validarDescricao(descricao: string) string
            +validarCodigo(texto: string) string
            +mascararCodigo(texto: string) string
            +validarEmail(email: string) string
        }
        class api
    }

    TelaCadastroOrganizacao ..> api : usa
    TelaCadastroOrganizacao ..> validacao : usa
    TelaEntrarComCodigo ..> api : usa
    TelaEntrarComCodigo ..> validacao : usa
    TelaInicio ..> api : usa
```

Figura 2 — Diagrama de classes do cadastro de organização, parte 1: telas de cadastro, de entrada com código e inicial e módulo `validacao`. Fonte: equipe Conecta+.

Só com o nome: `api` (parte 3).

### Parte 2 — Telas de organizações e de gerenciamento

```mermaid
classDiagram
    direction TB

    namespace Aplicativo {
        class TelaOrganizacoes {
            -organizacoes: Organizacao[]
            -busca: string
            -respondendo: string
            -aviso: Aviso
            +carregar(continuar: Function) Promise~void~
            +atualizar() Promise~void~
            +responder(organizacao: Organizacao, aceitar: boolean) Promise~void~
            +definirOrganizacoes(valor: Organizacao[]) void
            +definirAviso(valor: Aviso) void
        }
        class TelaOrganizacao {
            -dados: Dados
            -busca: string
            -email: string
            -menu: Membro
            -confirmacao: Pendente
            -aviso: Aviso
            -convidarAberto: boolean
            +carregar() Promise~Dados~
            +atualizar() Promise~void~
            +aprovar(membro: Membro) Promise~void~
            +pedir(pendente: Pendente) void
            +confirmar() Promise~void~
            +convidar() Promise~void~
            +copiarCodigo() Promise~void~
            +pedirNovoCodigo() void
            +opcoesDe(membro: Membro) object[]
            +caminho(usuarioId: string, sufixo: string) string
            +definirDados(valor: Dados) void
            +definirAviso(valor: Aviso) void
            +definirMenu(valor: Membro) void
            +definirConvidarAberto(valor: boolean) void
            +definirEmail(valor: string) void
        }
        class TelaGerenciarOrganizacoes {
            -organizacoes: Organizacao[]
            -busca: string
            -editandoId: string
            -acaoPendente: AcaoPendente
            -erro: string
            +carregar(termoBusca: string) Promise~void~
            +atualizar(id: string, dados: Partial~Organizacao~) Promise~boolean~
            +salvarEdicao() Promise~void~
            +excluir(organizacao: Organizacao) Promise~void~
            +confirmarAcao() Promise~void~
            +definirOrganizacoes(valor: SetStateAction~Organizacao[]~) void
            +definirBusca(valor: string) void
            +definirEditandoId(valor: string) void
            +definirAcaoPendente(valor: AcaoPendente) void
            +definirErro(valor: string) void
            +reset(valores: object) void
        }
        class Confirmacao {
            <<component>>
            +mensagem: string
            +rotuloConfirmar: string
            +perigo: boolean
            +carregando: boolean
            +confirmar() void
            +cancelar() void
        }
        class api
        class validacao
    }

    TelaOrganizacoes ..> api : usa
    TelaOrganizacao ..> api : usa
    TelaOrganizacao ..> validacao : usa
    TelaOrganizacao ..> Confirmacao : usa
    TelaGerenciarOrganizacoes ..> api : usa
    TelaGerenciarOrganizacoes ..> validacao : usa
    TelaGerenciarOrganizacoes ..> Confirmacao : usa
```

Figura 3 — Diagrama de classes do cadastro de organização, parte 2: telas "Organizações", da organização e "Gerenciar organizações" e componente `Confirmacao`. Fonte: equipe Conecta+.

Só com o nome: `validacao` (parte 1) e `api` (parte 3).

### Parte 3 — Módulo api e controller

```mermaid
classDiagram
    direction TB

    namespace Aplicativo {
        class api {
            <<module>>
            +api(caminho: string, metodo: string, corpo: unknown) Promise~Resultado~
        }
    }
    namespace API {
        class OrganizacoesController {
            -organizacoesService: OrganizacoesService
            +criar(CreateOrganizacaoDto, RequisicaoAutenticada) Promise~object~
            +listar(RequisicaoAutenticada, string) Promise~object[]~
            +buscarPorId(string, RequisicaoAutenticada) Promise~Organizacao~
            +atualizar(string, UpdateOrganizacaoDto) Promise~Organizacao~
            +remover(string) Promise~object~
            +entrarComCodigo(EntrarComCodigoDto, RequisicaoAutenticada) Promise~object~
            +gerarNovoCodigo(string, RequisicaoAutenticada) Promise~object~
            +removerMembro(string, string, RequisicaoAutenticada) Promise~object~
            +atualizarStatusMembro(string, string, UpdateStatusMembroDto, RequisicaoAutenticada) Promise~object~
            +alterarPapelMembro(string, string, UpdatePapelMembroDto, RequisicaoAutenticada) Promise~object~
            +convidarMembro(string, ConvidarMembroDto, RequisicaoAutenticada) Promise~object~
            +responderConvite(string, ResponderConviteDto, RequisicaoAutenticada) Promise~object~
        }
        class OrganizacoesService
    }

    api ..> OrganizacoesController : requisição HTTP
    OrganizacoesController "1" --> "1" OrganizacoesService : organizacoesService
```

Figura 4 — Diagrama de classes do cadastro de organização, parte 3: módulo `api` e `OrganizacoesController`. Fonte: equipe Conecta+.

Só com o nome: `OrganizacoesService` (parte 5).

### Parte 4 — Guardas e objetos de entrada

```mermaid
classDiagram
    direction TB

    namespace API {
        class OrganizacoesController
        class JwtAuthGuard {
            +canActivate(context: ExecutionContext) Promise~boolean~
        }
        class UsuarioAtivoGuard {
            -usuarios: Model~Usuario~
            +canActivate(context: ExecutionContext) Promise~boolean~
        }
        class UsuarioComumGuard {
            +canActivate(context: ExecutionContext) boolean
        }
        class AdminSistemaGuard {
            +canActivate(context: ExecutionContext) boolean
        }
        class CreateOrganizacaoDto {
            +nome: string
            +cidade: string
            +descricao: string
        }
        class UpdateOrganizacaoDto {
            +nome: string
            +cidade: string
            +descricao: string
            +status: StatusOrganizacao
        }
        class EntrarComCodigoDto {
            +codigo: string
        }
        class ConvidarMembroDto {
            +email: string
        }
        class ResponderConviteDto {
            +aceitar: boolean
        }
        class UpdateStatusMembroDto {
            +status: StatusMembroOrganizacao
        }
        class UpdatePapelMembroDto {
            +papel: PapelOrganizacao
        }
    }

    OrganizacoesController ..> JwtAuthGuard : protegido por
    OrganizacoesController ..> UsuarioAtivoGuard : protegido por
    OrganizacoesController ..> UsuarioComumGuard : protegido por
    OrganizacoesController ..> AdminSistemaGuard : protegido por
    OrganizacoesController ..> CreateOrganizacaoDto : recebe
    OrganizacoesController ..> UpdateOrganizacaoDto : recebe
    OrganizacoesController ..> EntrarComCodigoDto : recebe
    OrganizacoesController ..> ConvidarMembroDto : recebe
    OrganizacoesController ..> ResponderConviteDto : recebe
    OrganizacoesController ..> UpdateStatusMembroDto : recebe
    OrganizacoesController ..> UpdatePapelMembroDto : recebe
```

Figura 5 — Diagrama de classes do cadastro de organização, parte 4: guardas de acesso e objetos de entrada do controller. Fonte: equipe Conecta+.

Só com o nome: `OrganizacoesController` (parte 3). A ligação de `UsuarioAtivoGuard` com `UsuarioModel` está na parte 8.

### Parte 5 — Serviço

```mermaid
classDiagram
    direction TB

    namespace API {
        class OrganizacoesService {
            -organizacaoModel: Model~Organizacao~
            -usuarioModel: Model~Usuario~
            -comissaoModel: Model~Comissao~
            +criar(CreateOrganizacaoDto, string) Promise~object~
            +listar(TipoUsuario, string, string) Promise~object[]~
            +buscarPorId(string, string) Promise~Organizacao~
            +atualizar(string, UpdateOrganizacaoDto) Promise~Organizacao~
            +remover(string) Promise~object~
            +entrarComCodigo(string, string) Promise~object~
            +adicionarMembro(string, string) Promise~object~
            +gerarNovoCodigo(string, string) Promise~object~
            +atualizarStatusMembro(string, string, UpdateStatusMembroDto, string) Promise~object~
            +removerMembro(string, string, string) Promise~object~
            +alterarPapelMembro(string, string, UpdatePapelMembroDto, string) Promise~object~
            +convidarMembro(string, ConvidarMembroDto, string) Promise~object~
            +responderConvite(string, string, ResponderConviteDto) Promise~object~
            -detalhes(string) Promise~Organizacao~
            -reabrirSolicitacao(string, string) Promise~object~
            -vinculoDe(Organizacao, string) MembroOrganizacao
            -administradorAprovado(string) object
            -codigoRepetido(unknown) boolean
            -escaparRegex(string) string
            -exigirAdministrador(string, string) Promise~Organizacao~
            -exigirOrganizacaoAprovada(Organizacao) void
            -exigirNomeDisponivel(string, string) Promise~void~
            -validarId(string) void
        }
    }
    namespace Dados {
        class OrganizacaoModel
        class UsuarioModel
        class ComissaoModel
        class organizacaoSchema
    }

    OrganizacoesService "1" --> "1" OrganizacaoModel : organizacaoModel
    OrganizacoesService "1" --> "1" UsuarioModel : usuarioModel
    OrganizacoesService "1" --> "1" ComissaoModel : comissaoModel
    OrganizacoesService ..> organizacaoSchema : gerarCodigoAcesso
```

Figura 6 — Diagrama de classes do cadastro de organização, parte 5: `OrganizacoesService`. Fonte: equipe Conecta+.

Só com o nome: `OrganizacaoModel` (parte 6), `ComissaoModel` e `organizacaoSchema` (parte 7) e `UsuarioModel` (parte 8).

### Parte 6 — Model da organização

```mermaid
classDiagram
    direction TB

    namespace Dados {
        class OrganizacaoModel {
            <<Model Mongoose>>
            +create(dados: object) Promise~OrganizacaoDocument~
            +exists(filtro: object) Query~object~
            +find(filtro: object) Query~OrganizacaoDocument[]~
            +findById(id: string) Query~OrganizacaoDocument~
            +findOne(filtro: object) Query~OrganizacaoDocument~
            +findByIdAndUpdate(id: string, dados: object, opcoes: object) Query~OrganizacaoDocument~
            +updateOne(filtro: object, alteracao: object, opcoes: object) Query~UpdateResult~
            +deleteOne(filtro: object) Query~DeleteResult~
        }
    }
    class Organizacao

    OrganizacaoModel ..> Organizacao : documentos do schema
```

Figura 7 — Diagrama de classes do cadastro de organização, parte 6: model `OrganizacaoModel`. Fonte: equipe Conecta+.

Só com o nome: `Organizacao` (parte 7).

### Parte 7 — Organização, vínculos e comissão

```mermaid
classDiagram
    direction TB

    namespace Dados {
        class ComissaoModel {
            <<Model Mongoose>>
            +updateMany(filtro: object, alteracao: object) Query~UpdateResult~
            +deleteMany(filtro: object) Query~DeleteResult~
        }
        class Comissao {
            +_id: ObjectId
            +nome: string
            +organizacao_id: ObjectId
            +membros: MembroComissao[]
            +ativo: boolean
        }
        class organizacaoSchema {
            <<module>>
            +gerarCodigoAcesso() string
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
            +criado_em: Date
            +atualizado_em: Date
        }
        class MembroOrganizacao {
            +usuario_id: ObjectId
            +papel: PapelOrganizacao
            +status: StatusMembroOrganizacao
            +solicitado_em: Date
            +aprovado_em: Date
        }
        class StatusOrganizacao {
            <<enumeration>>
            PENDENTE
            APROVADA
            REVOGADA
        }
        class PapelOrganizacao {
            <<enumeration>>
            ADMIN
            MEMBRO
        }
        class StatusMembroOrganizacao {
            <<enumeration>>
            PENDENTE
            APROVADO
            REJEITADO
            CONVIDADO
        }
    }
    class Usuario

    ComissaoModel ..> Comissao : documentos do schema
    Comissao "0..*" --> "1" Organizacao : organizacao_id
    organizacaoSchema ..> Organizacao : default de codigo_acesso
    Organizacao "1" *-- "1..*" MembroOrganizacao : membros
    Organizacao "0..*" --> "1" Usuario : criada_por
    MembroOrganizacao "0..*" --> "1" Usuario : usuario_id
    Organizacao ..> StatusOrganizacao : status
    MembroOrganizacao ..> PapelOrganizacao : papel
    MembroOrganizacao ..> StatusMembroOrganizacao : status
```

Figura 8 — Diagrama de classes do cadastro de organização, parte 7: documentos `Organizacao`, `MembroOrganizacao` e `Comissao`, model `ComissaoModel`, módulo `organizacaoSchema` e enumerações. Fonte: equipe Conecta+.

Só com o nome: `Usuario` (parte 8).

### Parte 8 — Usuário

```mermaid
classDiagram
    direction TB

    namespace Dados {
        class UsuarioModel {
            <<Model Mongoose>>
            +findById(id: string) Query~UsuarioDocument~
            +findOne(filtro: object) Query~UsuarioDocument~
        }
        class Usuario {
            +_id: ObjectId
            +nome: string
            +email: string
            +tipo: TipoUsuario
            +ativo: boolean
        }
        class TipoUsuario {
            <<enumeration>>
            USUARIO
            ADMIN_SISTEMA
        }
    }
    class UsuarioAtivoGuard

    UsuarioAtivoGuard "1" --> "1" UsuarioModel : usuarios
    UsuarioModel ..> Usuario : documentos do schema
    Usuario ..> TipoUsuario : tipo
```

Figura 9 — Diagrama de classes do cadastro de organização, parte 8: model `UsuarioModel`, documento `Usuario` e enumeração `TipoUsuario`. Fonte: equipe Conecta+.

Só com o nome: `UsuarioAtivoGuard` (parte 4).

## Como ler as ligações

| Ligação                      | Significado                                                             | Exemplo no diagrama                                                                                                        |
| ---------------------------- | ----------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------- |
| Losango cheio (composição)   | A parte só existe dentro do todo e é apagada junto com ele.             | `Organizacao *-- MembroOrganizacao` (parte 7): o vínculo é gravado dentro do documento da organização e não tem identificador próprio. |
| Seta cheia (associação)      | Uma classe guarda uma referência a outra, que existe por conta própria. | `MembroOrganizacao --> Usuario` (parte 7): `usuario_id` guarda o identificador de um documento de `usuarios`.                         |
| Seta tracejada (dependência) | Uma classe usa a outra em uma chamada, sem guardá-la como atributo.     | `OrganizacoesService ..> organizacaoSchema` (parte 5): o serviço sorteia um código com `gerarCodigoAcesso`.                          |

Os números nas pontas são as multiplicidades. Uma `Organizacao` tem ao menos um vínculo (`1..*`), porque nasce com o do criador e nunca fica sem administrador; cada vínculo aponta para exatamente um `Usuario` (`1`), e um usuário pode ter vínculos com várias organizações (`0..*`). Cada `Comissao` pertence a uma única organização. O `OrganizacoesService` guarda uma instância de cada model (`1`), injetada pelo NestJS, e o `UsuarioAtivoGuard` guarda o `UsuarioModel`; cada model lê e grava os documentos do seu schema. O `ComissaoModel` é usado aqui só para apagar as comissões de uma organização excluída e para tirar das comissões quem perde o vínculo.

No código, os atributos opcionais são `descricao`, em `Organizacao`, e `aprovado_em`, em `MembroOrganizacao`. `codigo_acesso` é obrigatório, mas não sai nas consultas comuns (`select: false`). A obrigatoriedade de cada atributo está no [dicionário de classes](3%20-%20dicionario-dados-classes-cadastro-organizacao.md). `MembroComissao` pertence ao módulo de comissões e não é detalhado aqui.

## Rotas atendidas pelo controller

As rotas HTTP não são métodos; por isso ficam nesta tabela, e não no diagrama. Todas passam por `ApiKeyGuard` (global), `JwtAuthGuard` e `UsuarioAtivoGuard`.

| Método do controller    | Rota HTTP                                          | Guarda adicional    | Requisito |
| ----------------------- | -------------------------------------------------- | ------------------- | --------- |
| `criar`                 | `POST /organizacoes`                               | `UsuarioComumGuard` | RF-ORG-01 |
| `listar`                | `GET /organizacoes?busca=`                         | —                   | RF-ORG-02 |
| `buscarPorId`           | `GET /organizacoes/:id`                            | —                   | RF-ORG-14 |
| `atualizar`             | `PATCH /organizacoes/:id`                          | `AdminSistemaGuard` | RF-ORG-03, RF-ORG-04, RF-ORG-05 |
| `remover`               | `DELETE /organizacoes/:id`                         | `AdminSistemaGuard` | RF-ORG-06 |
| `entrarComCodigo`       | `POST /organizacoes/entrar`                        | `UsuarioComumGuard` | RF-ORG-07 |
| `convidarMembro`        | `POST /organizacoes/:id/convites`                  | `UsuarioComumGuard` | RF-ORG-08 |
| `atualizarStatusMembro` | `PATCH /organizacoes/:id/membros/:usuarioId`       | `UsuarioComumGuard` | RF-ORG-09 |
| `gerarNovoCodigo`       | `POST /organizacoes/:id/codigo`                    | `UsuarioComumGuard` | RF-ORG-10 |
| `responderConvite`      | `PATCH /organizacoes/:id/convite`                  | `UsuarioComumGuard` | RF-ORG-11 |
| `alterarPapelMembro`    | `PATCH /organizacoes/:id/membros/:usuarioId/papel` | `UsuarioComumGuard` | RF-ORG-12 |
| `removerMembro`         | `DELETE /organizacoes/:id/membros/:usuarioId`      | `UsuarioComumGuard` | RF-ORG-13 |

## Regras representadas

- `MembroOrganizacao` não existe fora de `Organizacao`;
- cada usuário aparece no máximo uma vez em `membros` da mesma organização;
- toda organização tem ao menos um vínculo com papel `ADMIN` e situação `APROVADO`;
- o papel na organização (`PapelOrganizacao`) é independente do tipo da conta (`TipoUsuario`): as rotas que criam ou mudam vínculos recusam a conta `ADMIN_SISTEMA`, que por isso não cadastra organização, não pede entrada nem recebe convite;
- o aplicativo não decide a situação da organização nem do vínculo: ele mostra o que a API devolve.

## Rastreabilidade

RF-ORG-01 a RF-ORG-14; complementado por [Dicionário de classes](3%20-%20dicionario-dados-classes-cadastro-organizacao.md) e [Diagramas de sequência](6%20-%20diagramas-sequencia-cadastro-organizacao.md).
