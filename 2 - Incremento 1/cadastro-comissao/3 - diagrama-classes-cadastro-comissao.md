# Diagrama de classes — Cadastro de comissão

> Padrão da disciplina · Módulo COM · Revisado em 27/11/2026

## Resumo

O diagrama mostra as classes que participam do cadastro de comissões, agrupadas em três pacotes, um por camada do sistema: **Aplicativo** (telas, componente e módulos do aplicativo), **API** (controller, service e objetos de entrada) e **Dados** (os models do Mongoose `ComissaoModel`, `OrganizacaoModel` e `UsuarioModel`, marcados `<<Model Mongoose>>`, e os documentos guardados no MongoDB). O `Router` do `expo-router`, chamado diretamente pela tela de cadastro, fica no pacote **Bibliotecas**, marcado `<<biblioteca>>`. Para caber legível na largura da página, o diagrama está dividido em sete partes. Toda linha de vida e todo método dos [Diagramas de sequência](6%20-%20diagramas-sequencia-cadastro-comissao.md) aparecem aqui, numa das partes. Cada atributo traz o seu tipo, e cada método traz o tipo dos parâmetros e do retorno. Toda classe e todo método do diagrama existem no código. Nas telas, os métodos `definir*` são as funções de atualização do `useState`, e `reset`, `setValue` e `onChange` são os do formulário do `react-hook-form` que as sequências chamam.

## Diagrama

Cada classe tem os atributos e os métodos numa única parte, indicada na tabela; nas outras partes em que aparece, por ser ponta de uma ligação, ela vem só com o nome, sem atributos nem métodos, e a linha abaixo da figura diz em que parte está detalhada.

| Parte | Figura | Classes detalhadas |
| ----- | ------ | ------------------ |
| 1 | Figura 2 | `TelaNovaComissao`, `Router` |
| 2 | Figura 3 | `TelaComissao`, `SeletorPessoaComissao`, `validacao` |
| 3 | Figura 4 | `TelaOrganizacao`, `api`, `ComissoesController` |
| 4 | Figura 5 | `CreateComissaoDto`, `UpdateComissaoDto`, `AddMembroComissaoDto`, `DefinirResponsavelDto`, `UpdateMembroComissaoDto` |
| 5 | Figura 6 | `ComissoesService` |
| 6 | Figura 7 | `ComissaoModel`, `Comissao`, `MembroComissao`, `PapelComissao`, `Usuario` |
| 7 | Figura 8 | `OrganizacaoModel`, `UsuarioModel`, `Organizacao`, `MembroOrganizacao` |

### Parte 1 — Tela de nova comissão

```mermaid
classDiagram
    direction TB

    namespace Aplicativo {
        class TelaNovaComissao {
            -organizacoes: Organizacao[]
            -candidatos: object
            -seletor: boolean
            -erro: string
            +salvar() Promise~void~
            +setValue(campo: string, valor: string) void
            +onChange(valor: string) void
        }
    }
    namespace Bibliotecas {
        class Router {
            <<biblioteca>>
            +replace(rota: Href) void
        }
    }
    class api
    class validacao
    class SeletorPessoaComissao

    TelaNovaComissao ..> api : usa
    TelaNovaComissao ..> validacao : usa
    TelaNovaComissao ..> SeletorPessoaComissao : mostra
    TelaNovaComissao ..> Router : replace
```

Figura 2 — Diagrama de classes do cadastro de comissão, parte 1: tela "Nova comissão" e `Router`. Fonte: equipe Conecta+.

Só com o nome: `validacao` e `SeletorPessoaComissao` (parte 2) e `api` (parte 3).

### Parte 2 — Tela da comissão

```mermaid
classDiagram
    direction TB

    namespace Aplicativo {
        class TelaComissao {
            -comissao: Comissao
            -gerencia: boolean
            -pessoas: PessoaComissao[]
            -aba: string
            -folha: string
            -escolhido: Integrante
            -confirmacao: AcaoPendente
            -reunioes: Reuniao[]
            -busca: string
            -erro: string
            -sucesso: string
            +buscarReunioes() Promise~object~
            +buscar() Promise~Carga~
            +aplicar(carga: Carga, preencherFormulario: boolean) void
            +atualizar() Promise~void~
            +executar(acao: Function, mensagem: string) Promise~boolean~
            +salvarDados() Promise~void~
            +desativar() Promise~Resultado~
            +trocarResponsavel(alvo: PessoaComissao) void
            +definirAba(valor: string) void
            +definirBusca(valor: string) void
            +definirComissao(valor: Comissao) void
            +definirConfirmacao(valor: AcaoPendente) void
            +definirErro(valor: string) void
            +definirEscolhido(valor: Integrante) void
            +definirFolha(valor: string) void
            +definirSucesso(valor: string) void
            +reset(valores: object) void
        }
        class SeletorPessoaComissao {
            <<component>>
            +visivel: boolean
            +titulo: string
            +pessoas: PessoaComissao[]
            +selecionados: string[]
            +aoAlternar(id: string) void
            +aoFechar() void
            -fechar() void
        }
        class validacao {
            <<module>>
            +validarNomeComissao(nome: string) string
            +validarDescricao(descricao: string) string
        }
    }
    class api

    TelaComissao ..> api : usa
    TelaComissao ..> validacao : usa
    TelaComissao ..> SeletorPessoaComissao : mostra
```

Figura 3 — Diagrama de classes do cadastro de comissão, parte 2: tela da comissão, componente `SeletorPessoaComissao` e módulo `validacao`. Fonte: equipe Conecta+.

Só com o nome: `api` (parte 3).

### Parte 3 — Tela da organização, módulo api e controller

```mermaid
classDiagram
    direction TB

    namespace Aplicativo {
        class TelaOrganizacao {
            -dados: Dados
            -filtro: Filtro
            -buscaComissao: string
            +carregar() Promise~Dados~
            +atualizar() Promise~void~
            +definirDados(valor: Dados) void
            +definirBuscaComissao(valor: string) void
            +definirFiltro(valor: Filtro) void
        }
        class api {
            <<module>>
            +api(caminho: string, metodo: string, corpo: unknown) Promise~Resultado~
        }
    }
    namespace API {
        class ComissoesController {
            -comissoesService: ComissoesService
            +listar(req: RequisicaoAutenticada, organizacaoId: string) Promise~Comissao[]~
            +buscarPorId(id: string, req: RequisicaoAutenticada) Promise~Comissao~
            +listarOrganizacoes(req: RequisicaoAutenticada) Promise~Organizacao[]~
            +listarMembrosDisponiveis(id: string, req: RequisicaoAutenticada) Promise~Usuario[]~
            +listarMinhas(organizacaoId: string, req: RequisicaoAutenticada) Promise~object~
            +criar(dto: CreateComissaoDto, req: RequisicaoAutenticada) Promise~Comissao~
            +atualizar(id: string, dto: UpdateComissaoDto, req: RequisicaoAutenticada) Promise~Comissao~
            +remover(id: string, req: RequisicaoAutenticada) Promise~object~
            +adicionarMembro(id: string, dto: AddMembroComissaoDto, req: RequisicaoAutenticada) Promise~Comissao~
            +atualizarMembro(id: string, usuarioId: string, dto: UpdateMembroComissaoDto, req: RequisicaoAutenticada) Promise~Comissao~
            +definirResponsavel(id: string, dto: DefinirResponsavelDto, req: RequisicaoAutenticada) Promise~Comissao~
            +removerMembro(id: string, usuarioId: string, req: RequisicaoAutenticada) Promise~Comissao~
        }
        class ComissoesService
    }

    TelaOrganizacao ..> api : usa
    api ..> ComissoesController : requisição HTTP
    ComissoesController "1" --> "1" ComissoesService : comissoesService
```

Figura 4 — Diagrama de classes do cadastro de comissão, parte 3: tela da organização, módulo `api` e `ComissoesController`. Fonte: equipe Conecta+.

Só com o nome: `ComissoesService` (parte 5).

### Parte 4 — Objetos de entrada

```mermaid
classDiagram
    direction TB

    namespace API {
        class ComissoesController
        class CreateComissaoDto {
            +nome: string
            +descricao: string
            +organizacao_id: string
            +responsavel_id: string
        }
        class UpdateComissaoDto {
            +nome: string
            +descricao: string
            +ativo: boolean
        }
        class AddMembroComissaoDto {
            +usuario_id: string
            +papel: PapelComissao
        }
        class DefinirResponsavelDto {
            +usuario_id: string
        }
        class UpdateMembroComissaoDto {
            +papel: PapelComissao
        }
    }

    ComissoesController ..> CreateComissaoDto : recebe
    ComissoesController ..> UpdateComissaoDto : recebe
    ComissoesController ..> AddMembroComissaoDto : recebe
    ComissoesController ..> DefinirResponsavelDto : recebe
    ComissoesController ..> UpdateMembroComissaoDto : recebe
```

Figura 5 — Diagrama de classes do cadastro de comissão, parte 4: objetos de entrada do controller. Fonte: equipe Conecta+.

Só com o nome: `ComissoesController` (parte 3).

### Parte 5 — Serviço

```mermaid
classDiagram
    direction TB

    namespace API {
        class ComissoesService {
            -comissaoModel: Model~Comissao~
            -organizacaoModel: Model~Organizacao~
            -usuarioModel: Model~Usuario~
            +listar(usuarioId: string, organizacaoId: string) Promise~Comissao[]~
            +buscarPorId(id: string, usuarioId: string) Promise~Comissao~
            +listarOrganizacoes(usuarioId: string) Promise~Organizacao[]~
            +listarMembrosDisponiveis(organizacaoId: string, usuarioId: string) Promise~Usuario[]~
            +listarMinhas(organizacaoId: string, usuarioId: string) Promise~object~
            +criar(dto: CreateComissaoDto, usuarioId: string) Promise~Comissao~
            +atualizar(id: string, dto: UpdateComissaoDto, usuarioId: string) Promise~Comissao~
            +remover(id: string, usuarioId: string) Promise~object~
            +adicionarMembro(id: string, dto: AddMembroComissaoDto, usuarioId: string) Promise~Comissao~
            +atualizarMembro(id: string, membroId: string, papel: PapelComissao, usuarioId: string) Promise~Comissao~
            +definirResponsavel(id: string, membroId: string, usuarioId: string) Promise~Comissao~
            +removerMembro(id: string, membroId: string, usuarioId: string) Promise~Comissao~
            -validarId(id: string) void
            -validarUsuario(usuarioId: string) Promise~void~
            -administra(organizacao: Organizacao, usuarioId: string) boolean
            -autorizarOrganizacao(organizacaoId: string, usuarioId: string) Promise~Organizacao~
            -autorizarLeitura(organizacaoId: string, usuarioId: string) Promise~Organizacao~
            -autorizarComissao(id: string, usuarioId: string) Promise~Comissao~
            -popular(comissao: Comissao) Promise~Comissao~
            -consultar(filtro: object) Promise~Comissao[]~
            -exigirResponsavelUnico(comissao: Comissao) void
            -exigirElegivel(organizacaoId: string, membroId: string, usuarioId: string) Promise~void~
            -gravarEquipe(comissao: Comissao, membros: MembroComissao[]) Promise~Comissao~
            -validarEquipe(id: string, membroId: string, usuarioId: string) Promise~Comissao~
        }
    }
    namespace Dados {
        class ComissaoModel
        class OrganizacaoModel
        class UsuarioModel
    }

    ComissoesService "1" --> "1" ComissaoModel : comissaoModel
    ComissoesService "1" --> "1" OrganizacaoModel : organizacaoModel
    ComissoesService "1" --> "1" UsuarioModel : usuarioModel
```

Figura 6 — Diagrama de classes do cadastro de comissão, parte 5: `ComissoesService`. Fonte: equipe Conecta+.

Só com o nome: `ComissaoModel` (parte 6) e `OrganizacaoModel` e `UsuarioModel` (parte 7).

### Parte 6 — Comissão e equipe

```mermaid
classDiagram
    direction TB

    namespace Dados {
        class ComissaoModel {
            <<Model Mongoose>>
            +find(filtro: object) Query~ComissaoDocument[]~
            +findById(id: string) Query~ComissaoDocument~
            +create(dados: object) Promise~ComissaoDocument~
            +findByIdAndUpdate(id: string, alteracao: object, opcoes: object) Query~ComissaoDocument~
            +findOneAndUpdate(filtro: object, alteracao: object, opcoes: object) Query~ComissaoDocument~
            +populate(caminhos: object[]) Promise~ComissaoDocument~
        }
        class Comissao {
            +_id: ObjectId
            +nome: string
            +descricao: string
            +organizacao_id: ObjectId
            +ativo: boolean
            +membros: MembroComissao[]
            +criado_em: Date
            +atualizado_em: Date
        }
        class MembroComissao {
            +usuario_id: ObjectId
            +papel: PapelComissao
            +adicionado_em: Date
        }
        class PapelComissao {
            <<enumeration>>
            RESPONSAVEL
            MEMBRO
        }
        class Usuario {
            +_id: ObjectId
            +nome: string
            +email: string
            +tipo: TipoUsuario
            +ativo: boolean
        }
    }
    class Organizacao

    ComissaoModel ..> Comissao : documentos do schema
    Comissao "1" *-- "1..*" MembroComissao : membros
    Comissao "0..*" --> "1" Organizacao : organizacao_id
    MembroComissao "0..*" --> "1" Usuario : usuario_id
    MembroComissao ..> PapelComissao : papel
```

Figura 7 — Diagrama de classes do cadastro de comissão, parte 6: model `ComissaoModel`, documentos `Comissao`, `MembroComissao` e `Usuario` e enumeração `PapelComissao`. Fonte: equipe Conecta+.

Só com o nome: `Organizacao` (parte 7).

### Parte 7 — Organização e usuário

```mermaid
classDiagram
    direction TB

    namespace Dados {
        class OrganizacaoModel {
            <<Model Mongoose>>
            +findById(id: string) Query~OrganizacaoDocument~
            +find(filtro: object) Query~OrganizacaoDocument[]~
        }
        class UsuarioModel {
            <<Model Mongoose>>
            +findById(id: string) Query~UsuarioDocument~
            +find(filtro: object) Query~UsuarioDocument[]~
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
    }
    class Usuario

    OrganizacaoModel ..> Organizacao : documentos do schema
    UsuarioModel ..> Usuario : documentos do schema
    Organizacao "1" *-- "0..*" MembroOrganizacao : membros
    MembroOrganizacao "0..*" --> "1" Usuario : usuario_id
```

Figura 8 — Diagrama de classes do cadastro de comissão, parte 7: models `OrganizacaoModel` e `UsuarioModel` e documentos `Organizacao` e `MembroOrganizacao`. Fonte: equipe Conecta+.

Só com o nome: `Usuario` (parte 6).

## Como ler as ligações

| Ligação                      | Significado                                                             | Exemplo no diagrama                                                                                          |
| ---------------------------- | ----------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------ |
| Losango cheio (composição)   | A parte só existe dentro do todo e é apagada junto com ele.             | `Comissao *-- MembroComissao` (parte 6): cada integrante é gravado dentro do documento da comissão, sem identificador próprio. |
| Seta cheia (associação)      | Uma classe guarda uma referência a outra, que existe por conta própria. | `MembroComissao --> Usuario` (parte 6): `usuario_id` guarda o identificador de um documento de `usuarios`.             |
| Seta tracejada (dependência) | Uma classe usa a outra em uma chamada, sem guardá-la como atributo.     | `TelaNovaComissao ..> Router` (parte 1): a tela abre a comissão criada com `replace`.                                  |

Os números nas pontas são as multiplicidades. Uma `Comissao` tem ao menos um `MembroComissao` (`1..*`), porque o responsável é obrigatório, e pertence a exatamente uma `Organizacao` (`1`); uma organização pode ter qualquer quantidade de comissões (`0..*`). Cada `MembroComissao` aponta para exatamente um `Usuario`, e um usuário pode estar em várias comissões. O `ComissoesService` guarda uma instância de cada model (`1`), injetada pelo NestJS, e cada model lê e grava os documentos do seu schema; `populate` é chamado no documento da comissão já lido.

No código, são opcionais: `descricao`, em `Comissao` e em `CreateComissaoDto`; `nome`, `descricao` e `ativo`, em `UpdateComissaoDto`; e `papel`, em `AddMembroComissaoDto`. `validarNomeComissao` e `validarDescricao` devolvem o texto do erro ou `null` quando o valor é válido. `Organizacao`, `MembroOrganizacao` e `Usuario` aparecem só com os campos que este módulo usa; a descrição completa está nos módulos de [organização](../cadastro-organizacao/README.md) e de [usuário](../cadastro-usuario/README.md).

## Rotas atendidas pelo controller

As rotas HTTP não são métodos; por isso ficam nesta tabela, e não no diagrama.

| Método do controller       | Rota HTTP                                  | Uso                                                                                                   |
| -------------------------- | ------------------------------------------ | ----------------------------------------------------------------------------------------------------- |
| `listar`                   | `GET /comissoes`                           | RF-COM-01, com ou sem `?organizacao_id=`.                                                             |
| `buscarPorId`              | `GET /comissoes/:id`                       | RF-COM-02.                                                                                            |
| `listarOrganizacoes`       | `GET /comissoes/organizacoes`              | RF-COM-02 e RF-COM-03.                                                                                |
| `listarMembrosDisponiveis` | `GET /comissoes/organizacoes/:id/membros`  | RF-COM-02 e RF-COM-03.                                                                                |
| `criar`                    | `POST /comissoes`                          | RF-COM-03.                                                                                            |
| `atualizar`                | `PATCH /comissoes/:id`                     | RF-COM-04 e RF-COM-06.                                                                                |
| `remover`                  | `DELETE /comissoes/:id`                    | RF-COM-05. Apesar do verbo, desativa; não apaga o documento.                                          |
| `adicionarMembro`          | `POST /comissoes/:id/membros`              | RF-COM-07.                                                                                            |
| `definirResponsavel`       | `PATCH /comissoes/:id/responsavel`         | RF-COM-08.                                                                                            |
| `removerMembro`            | `DELETE /comissoes/:id/membros/:usuarioId` | RF-COM-09.                                                                                            |
| `listarMinhas`             | `GET /comissoes/minhas/:organizacaoId`     | Não usada pelo aplicativo. Devolve as comissões de cuja equipe o usuário faz parte, com `meu_papel`.  |
| `atualizarMembro`          | `PATCH /comissoes/:id/membros/:usuarioId`  | Não usada pelo aplicativo. Exige a comissão ativa e a pessoa na equipe (`validarEquipe`). Com `papel = RESPONSAVEL`, chama `definirResponsavel`; com `MEMBRO`, chama `exigirResponsavelUnico`, recusa rebaixar o responsável ("Substitua o responsável antes de alterar seu papel") e, para um membro, devolve a comissão sem gravar nada. |

## Regras representadas

- `MembroComissao` não existe fora de `Comissao`;
- em `membros` há exatamente um item com `papel = RESPONSAVEL` e nenhum `usuario_id` repetido;
- quem entra em `membros` precisa estar ativo e com vínculo `APROVADO` na organização da comissão;
- a troca de responsável regrava a lista inteira numa só operação: o escolhido vira `RESPONSAVEL` e todos os outros viram `MEMBRO`;
- toda mudança de equipe só é gravada se a equipe e a situação não mudaram desde a leitura;
- o aplicativo nunca decide quem pode gerenciar: ele mostra os botões conforme a resposta de `GET /comissoes/organizacoes`, e a API confere de novo em cada operação.

## Rastreabilidade

RF-COM-01 a RF-COM-09; complementado por [Dicionário de classes](3%20-%20dicionario-dados-classes-cadastro-comissao.md) e [Diagramas de sequência](6%20-%20diagramas-sequencia-cadastro-comissao.md).
