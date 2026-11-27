# Diagrama de classes — Configurações de usuário

> Padrão da disciplina · Módulo CFG · Revisado em 27/11/2026

## Resumo

O diagrama mostra as classes que participam das configurações da conta, agrupadas em três pacotes, um por camada do sistema: **Aplicativo** (telas, contextos e módulos do aplicativo), **API** (controllers, services, guard e objetos de entrada) e **Dados** (o model do Mongoose `UsuarioModel`, marcado `<<Model Mongoose>>`, o documento guardado no MongoDB e seus enums). As bibliotecas chamadas diretamente nos [Diagramas de sequência](6%20-%20diagramas-sequencia-configuracoes-usuario.md) (`Router`, `SecureStore`, `bcrypt`, `crypto` e `Transporter`) ficam fora dos pacotes, marcadas `<<biblioteca>>`, só com os métodos chamados. Toda linha de vida e todo método das sequências aparecem aqui. Cada atributo traz o seu tipo, e cada método traz o tipo dos parâmetros e do retorno. Toda classe e todo método do diagrama existem no código. Nas telas e nos contextos, os atributos são os estados guardados pelo componente (`useState`, `useRef` ou valor calculado) e os métodos são as funções declaradas dentro do componente ou no mesmo arquivo; os métodos `definir*` são as funções de atualização do `useState`, e `ouvinte` é a função que o `SessaoContext` registra com `observarSessaoInvalida`.

## Diagrama

O diagrama está dividido em 13 partes, para que cada uma caiba na largura da página sem ser reduzida. Cada classe aparece completa, com atributos e métodos, em uma única parte. Quando uma ligação chega a uma classe detalhada em outra parte, essa classe é repetida só com o nome, e o texto abaixo da figura diz em que parte ela está completa. As partes 1 a 4 são do pacote **Aplicativo**, as partes 5 a 10 do pacote **API** e a parte 11 do pacote **Dados**; as bibliotecas, fora dos pacotes, estão nas partes 12 e 13, e o `Transporter`, na parte 6. A parte 4 não tem ligações próprias: as classes dela aparecem uma abaixo da outra, e as ligações estão nas partes indicadas abaixo da figura.

### Parte 1 — Tela "Perfil"

```mermaid
classDiagram
    direction TB

    namespace Aplicativo {
        class TelaPerfil {
            -perfil: Perfil
            -preferenciaSelecionada: PreferenciaTema
            -editandoNome: boolean
            -escolhendoTema: boolean
            -confirmandoSaida: boolean
            -salvandoTema: boolean
            -aviso: Aviso
            -avisoNome: Aviso
            +carregarPerfil(ativa: function) Promise~void~
            +abrirEdicaoNome() void
            +salvarNome() Promise~void~
            +selecionarTema(novaPreferencia: PreferenciaTema) Promise~void~
            +abrirProtegida(destino: Href) Promise~void~
            +sair() Promise~void~
            +definirPerfil(valor: Perfil) void
            +definirAviso(valor: Aviso) void
            +definirAvisoNome(valor: Aviso) void
            +definirEditandoNome(valor: boolean) void
            +definirEscolhendoTema(valor: boolean) void
            +definirConfirmandoSaida(valor: boolean) void
            +definirPreferenciaSelecionada(valor: PreferenciaTema) void
            +definirSalvandoTema(valor: boolean) void
        }
    }
    class api
    class autenticacaoLocal
    class TemaContext
    class SessaoContext
    class validacao
    class ToastContext

    TelaPerfil ..> api : usa
    TelaPerfil ..> autenticacaoLocal : usa
    TelaPerfil ..> TemaContext : usa
    TelaPerfil ..> SessaoContext : usa
    TelaPerfil ..> validacao : usa
    TelaPerfil ..> ToastContext : mostrarToast
```

Figura 2 — Diagrama de classes das configurações de usuário, parte 1: tela "Perfil". Fonte: equipe Conecta+.

Só com o nome nesta parte: `SessaoContext` (parte 3); `api`, `autenticacaoLocal`, `TemaContext`, `validacao` e `ToastContext` (parte 4). Ligações em outras partes: `TelaPerfil` (parte 12).

### Parte 2 — Telas "Trocar e-mail" e "Alterar senha"

```mermaid
classDiagram
    direction TB

    namespace Aplicativo {
        class TelaTrocarEmail {
            -etapa: Etapa
            -atual: string
            -novo: string
            -codigo: string
            -reenviado: boolean
            +enviarCodigo() Promise~void~
            +solicitar(email: string) Promise~boolean~
            +reenviar() Promise~void~
            +confirmar() Promise~void~
            +voltar() void
            +definirEtapa(valor: Etapa) void
        }
        class TelaAlterarSenha {
            -erro: string
            +salvar() Promise~void~
            +voltar() void
        }
    }
    class api
    class SessaoContext
    class validacao
    class CampoTexto

    TelaTrocarEmail ..> api : usa
    TelaTrocarEmail ..> SessaoContext : usa
    TelaTrocarEmail ..> validacao : usa
    TelaAlterarSenha ..> api : usa
    TelaAlterarSenha ..> SessaoContext : usa
    TelaAlterarSenha ..> validacao : usa
    TelaAlterarSenha ..> CampoTexto : usa
```

Figura 2 — Diagrama de classes das configurações de usuário, parte 2: telas "Trocar e-mail" e "Alterar senha". Fonte: equipe Conecta+.

Só com o nome nesta parte: `SessaoContext` (parte 3); `api`, `validacao` e `CampoTexto` (parte 4).

### Parte 3 — Tela "Recuperar conta" e contexto da sessão

```mermaid
classDiagram
    direction TB

    namespace Aplicativo {
        class TelaRecuperarConta {
            -etapa: Etapa
            -email: string
            -codigo: string
            -token: string
            -reenviado: boolean
            +solicitar() Promise~void~
            +reenviar() Promise~void~
            +verificarCodigo() Promise~void~
            +redefinir() Promise~void~
            +voltar() void
            +definirEtapa(valor: Etapa) void
            +definirToken(valor: string) void
        }
        class SessaoContext {
            -usuario: Usuario
            -carregando: boolean
            -erro: string
            -aviso: string
            -geracao: number
            +atualizar() Promise~void~
            +entrar(token: string) Promise~boolean~
            +sair(aviso: string) Promise~void~
            +limparAviso() void
            +definirUsuario(valor: Usuario) void
            +definirAviso(valor: string) void
            +ouvinte() void
        }
    }
    class api
    class autenticacaoLocal
    class validacao
    class CampoTexto

    TelaRecuperarConta ..> api : usa
    TelaRecuperarConta ..> autenticacaoLocal : usa
    TelaRecuperarConta ..> validacao : usa
    TelaRecuperarConta ..> CampoTexto : usa
    TelaRecuperarConta ..> SessaoContext : limparAviso
    SessaoContext ..> api : usa
    api ..> SessaoContext : chama o ouvinte
```

Figura 2 — Diagrama de classes das configurações de usuário, parte 3: tela "Recuperar conta" e contexto da sessão. Fonte: equipe Conecta+.

Só com o nome nesta parte: `api`, `autenticacaoLocal`, `validacao` e `CampoTexto` (parte 4). Ligações em outras partes: `SessaoContext` (partes 1 e 2).

### Parte 4 — Contextos, componente e módulos do aplicativo

```mermaid
classDiagram
    direction TB

    namespace Aplicativo {
        class TemaContext {
            -preferencia: PreferenciaTema
            -tema: Tema
            +definirPreferencia(preferencia: PreferenciaTema) void
            +ehPreferenciaTema(valor: unknown) boolean
            -lerPreferencia() Promise~string~
            -gravarPreferencia(preferencia: PreferenciaTema) Promise~void~
        }
        class ToastContext {
            -toast: ToastAtual
            +mostrarToast(mensagem: string, tipo: TipoToast, duracao: number) void
        }
        class CampoTexto {
            +rotulo: string
            +erro: string
            +ajuda: string
            +dica: string
        }
        class validacao {
            <<module>>
            +validarNome(nome: string) string
            +validarEmail(email: string) string
            +validarSenha(senha: string) string
            +validarNovaSenha(senha: string) string
            +validarConfirmacaoSenha(senha: string, confirmacao: string) string
            +normalizarEmail(email: string) string
        }
        class autenticacaoLocal {
            <<module>>
            +dispositivoProtegido() Promise~boolean~
            +autenticarComDispositivo() Promise~boolean~
        }
        class api {
            <<module>>
            +api(caminho: string, metodo: string, corpo: unknown) Promise~Resultado~
            +salvarToken(token: string) Promise~void~
            +obterToken() Promise~string~
            +removerToken() Promise~void~
            +observarSessaoInvalida(ouvinte: function) function
        }
    }
```

Figura 2 — Diagrama de classes das configurações de usuário, parte 4: contextos de tema e de avisos, campo de texto e módulos `validacao`, `autenticacaoLocal` e `api`. Fonte: equipe Conecta+.

Ligações em outras partes: `TemaContext` (partes 1 e 12); `ToastContext` (parte 1); `CampoTexto` (partes 2 e 3); `validacao` (partes 1, 2 e 3); `autenticacaoLocal` (partes 1 e 3); `api` (partes 1, 2, 3, 5, 6 e 12).

### Parte 5 — Controller e serviço de usuários

```mermaid
classDiagram
    direction TB

    namespace API {
        class UsuariosController {
            -usuariosService: UsuariosService
            -authService: AuthService
            +buscarMeuPerfil(requisicao: RequisicaoAutenticada) Promise~Usuario~
            +atualizarMeuPerfil(requisicao: RequisicaoAutenticada, atualizarPerfilDto: AtualizarPerfilDto) Promise~Usuario~
            +solicitarAlteracaoEmail(requisicao: RequisicaoAutenticada, dto: SolicitarAlteracaoEmailDto) Promise~object~
            +confirmarAlteracaoEmail(requisicao: RequisicaoAutenticada, dto: ConfirmarAlteracaoEmailDto) Promise~object~
            +alterarMinhaSenha(requisicao: RequisicaoAutenticada, alterarSenhaDto: AlterarSenhaDto) Promise~object~
        }
        class UsuariosService {
            -usuarioModel: Model~UsuarioDocument~
            +buscarPorId(id: string) Promise~Usuario~
            +atualizarPerfil(id: string, atualizarPerfilDto: AtualizarPerfilDto) Promise~Usuario~
            +alterarSenha(id: string, alterarSenhaDto: AlterarSenhaDto) Promise~object~
            -validarId(id: string) void
        }
    }
    class api

    api ..> UsuariosController : requisição HTTP
    UsuariosController "1" --> "1" UsuariosService : usuariosService
```

Figura 2 — Diagrama de classes das configurações de usuário, parte 5: controller e serviço de usuários. Fonte: equipe Conecta+.

Só com o nome nesta parte: `api` (parte 4). Ligações em outras partes: `UsuariosController` (partes 6, 7, 8 e 10); `UsuariosService` (partes 11 e 13).

### Parte 6 — Controller e serviço de autenticação

```mermaid
classDiagram
    direction TB

    namespace API {
        class AuthController {
            -authService: AuthService
            +recuperarConta(recuperarContaDto: RecuperarContaDto) Promise~object~
            +verificarRecuperacao(dto: VerificarRecuperacaoDto) Promise~object~
            +redefinirSenha(redefinirSenhaDto: RedefinirSenhaDto) Promise~object~
        }
        class AuthService {
            -usuarioModel: Model~UsuarioDocument~
            -transportador: Transporter
            +solicitarAlteracaoEmail(id: string, novoEmail: string) Promise~object~
            +confirmarAlteracaoEmail(id: string, emailInformado: string, codigoInformado: string) Promise~object~
            +solicitarRecuperacao(recuperarContaDto: RecuperarContaDto) Promise~object~
            +verificarRecuperacao(dto: VerificarRecuperacaoDto) Promise~object~
            +redefinirSenha(redefinirSenhaDto: RedefinirSenhaDto) Promise~object~
            -gerarCodigoVerificacao() string
            -hashToken(token: string) string
            -enviarEmail(mensagem: SendMailOptions, aoFalhar: function) void
            -smtpConfigurado() boolean
            -criarTransportador() Transporter
        }
    }
    class Transporter {
        <<biblioteca>>
        +sendMail(mensagem: SendMailOptions) Promise~SentMessageInfo~
    }
    class api
    class UsuariosController

    api ..> AuthController : requisição HTTP
    AuthController "1" --> "1" AuthService : authService
    UsuariosController "1" --> "1" AuthService : authService
    AuthService "1" --> "0..1" Transporter : transportador
```

Figura 2 — Diagrama de classes das configurações de usuário, parte 6: controller e serviço de autenticação, com a biblioteca `Transporter`. Fonte: equipe Conecta+.

Só com o nome nesta parte: `api` (parte 4); `UsuariosController` (parte 5). Ligações em outras partes: `AuthController` (partes 9 e 10); `AuthService` (partes 11 e 13).

### Parte 7 — DTOs do perfil e da troca de senha

```mermaid
classDiagram
    direction TB

    namespace API {
        class AtualizarPerfilDto {
            +nome: string
            +email: string
            +tema: TemaUsuario
        }
        class AlterarSenhaDto {
            +senha_atual: string
            +nova_senha: string
        }
    }
    class UsuariosController

    UsuariosController ..> AtualizarPerfilDto : recebe
    UsuariosController ..> AlterarSenhaDto : recebe
```

Figura 2 — Diagrama de classes das configurações de usuário, parte 7: DTOs do perfil e da troca de senha. Fonte: equipe Conecta+.

Só com o nome nesta parte: `UsuariosController` (parte 5).

### Parte 8 — DTOs da troca de e-mail

```mermaid
classDiagram
    direction TB

    namespace API {
        class SolicitarAlteracaoEmailDto {
            +email: string
        }
        class ConfirmarAlteracaoEmailDto {
            +email: string
            +codigo: string
        }
    }
    class UsuariosController

    UsuariosController ..> SolicitarAlteracaoEmailDto : recebe
    UsuariosController ..> ConfirmarAlteracaoEmailDto : recebe
```

Figura 2 — Diagrama de classes das configurações de usuário, parte 8: DTOs da troca de e-mail. Fonte: equipe Conecta+.

Só com o nome nesta parte: `UsuariosController` (parte 5).

### Parte 9 — DTOs da recuperação de conta

```mermaid
classDiagram
    direction TB

    namespace API {
        class RecuperarContaDto {
            +email: string
        }
        class VerificarRecuperacaoDto {
            +email: string
            +codigo: string
        }
    }
    class AuthController

    AuthController ..> RecuperarContaDto : recebe
    AuthController ..> VerificarRecuperacaoDto : recebe
```

Figura 2 — Diagrama de classes das configurações de usuário, parte 9: DTOs da recuperação de conta. Fonte: equipe Conecta+.

Só com o nome nesta parte: `AuthController` (parte 6).

### Parte 10 — DTO da nova senha e guard de autenticação

```mermaid
classDiagram
    direction TB

    namespace API {
        class RedefinirSenhaDto {
            +token: string
            +nova_senha: string
        }
        class JwtAuthGuard {
            -jwtService: JwtService
            -usuarios: Model~UsuarioDocument~
            +canActivate(context: ExecutionContext) Promise~boolean~
            -extrairToken(requisicao: Request) string
        }
    }
    class AuthController
    class UsuariosController

    AuthController ..> RedefinirSenhaDto : recebe
    UsuariosController ..> JwtAuthGuard : protegido por
```

Figura 2 — Diagrama de classes das configurações de usuário, parte 10: DTO da nova senha e guard de autenticação. Fonte: equipe Conecta+.

Só com o nome nesta parte: `UsuariosController` (parte 5); `AuthController` (parte 6). Ligações em outras partes: `JwtAuthGuard` (parte 11).

### Parte 11 — Model, documento e enums do usuário

```mermaid
classDiagram
    direction TB

    namespace Dados {
        class UsuarioModel {
            <<Model Mongoose>>
            +findById(id: string) Query~UsuarioDocument~
            +findOne(filtro: object) Query~UsuarioDocument~
            +save() Promise~UsuarioDocument~
        }
        class Usuario {
            +_id: ObjectId
            +nome: string
            +email: string
            -senha_hash: string
            +senha_alterada_em: Date
            +tipo: TipoUsuario
            +tema: TemaUsuario
            -reset_senha_codigo_hash: string
            -reset_senha_token_hash: string
            -reset_senha_expira_em: Date
            -email_pendente: string
            -email_pendente_codigo_hash: string
            -email_pendente_expira_em: Date
            +email_verificado: boolean
            +ativo: boolean
            +provedor_autenticacao: ProvedorAutenticacao
            +criado_em: Date
            +atualizado_em: Date
        }
        class TemaUsuario {
            <<enumeration>>
            claro
            escuro
            sistema
        }
        class TipoUsuario {
            <<enumeration>>
            USUARIO
            ADMIN_SISTEMA
        }
        class ProvedorAutenticacao {
            <<enumeration>>
            LOCAL
            GOOGLE
            AMBOS
        }
    }
    class UsuariosService
    class AuthService
    class JwtAuthGuard

    UsuariosService "1" --> "1" UsuarioModel : usuarioModel
    AuthService "1" --> "1" UsuarioModel : usuarioModel
    JwtAuthGuard "1" --> "1" UsuarioModel : usuarios
    UsuarioModel ..> Usuario : documentos do schema
    Usuario ..> TemaUsuario : tema
    Usuario ..> TipoUsuario : tipo
    Usuario ..> ProvedorAutenticacao : provedor_autenticacao
```

Figura 2 — Diagrama de classes das configurações de usuário, parte 11: model, documento e enums do usuário. Fonte: equipe Conecta+.

Só com o nome nesta parte: `UsuariosService` (parte 5); `AuthService` (parte 6); `JwtAuthGuard` (parte 10).

### Parte 12 — Bibliotecas do aplicativo

```mermaid
classDiagram
    direction TB

    class Router {
        <<biblioteca>>
        +push(rota: Href) void
    }
    class SecureStore {
        <<biblioteca>>
        +setItemAsync(chave: string, valor: string) Promise~void~
        +deleteItemAsync(chave: string) Promise~void~
    }
    class TelaPerfil
    class api
    class TemaContext

    TelaPerfil ..> Router : push
    api ..> SecureStore : guarda e apaga o token
    TemaContext ..> SecureStore : grava a preferência
```

Figura 2 — Diagrama de classes das configurações de usuário, parte 12: bibliotecas do aplicativo. Fonte: equipe Conecta+.

Só com o nome nesta parte: `TelaPerfil` (parte 1); `api` e `TemaContext` (parte 4).

### Parte 13 — Bibliotecas da API

```mermaid
classDiagram
    direction TB

    class bcrypt {
        <<biblioteca>>
        +hash(dado: string, custo: number) Promise~string~
        +compare(dado: string, hash: string) Promise~boolean~
    }
    class crypto {
        <<biblioteca>>
        +randomBytes(tamanho: number) Buffer
    }
    class UsuariosService
    class AuthService

    UsuariosService ..> bcrypt : compare e hash
    AuthService ..> bcrypt : hash
    AuthService ..> crypto : randomBytes
```

Figura 2 — Diagrama de classes das configurações de usuário, parte 13: bibliotecas da API. Fonte: equipe Conecta+.

Só com o nome nesta parte: `UsuariosService` (parte 5); `AuthService` (parte 6).

## Como ler as ligações

| Ligação                      | Significado                                                                        | Exemplo no diagrama                                                                                          |
| ---------------------------- | ---------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------ |
| Seta cheia (associação)      | Uma classe guarda uma referência a outra, que existe por conta própria.            | `UsuariosController --> "1" UsuariosService` (parte 5): o controller recebe uma única instância do serviço no construtor. |
| Seta tracejada (dependência) | Uma classe usa a outra em uma chamada, sem guardá-la como atributo.                | `TelaAlterarSenha ..> SessaoContext` (parte 2): a tela chama `sair` depois da troca de senha.                 |

O número na ponta da associação é a multiplicidade: cada controller tem exatamente uma instância de cada serviço (`1`), criada pela injeção de dependências do NestJS. Os services e o `JwtAuthGuard` guardam o mesmo `UsuarioModel`, e o `AuthService` guarda no máximo um `Transporter` (`0..1`), criado no primeiro envio. O `UsuarioModel` lê e grava os documentos do schema `Usuario`; `save()` é chamado no documento devolvido por `findById` ou `findOne`, que nos diagramas de sequência aparece como a linha de vida "UsuarioModel (documento usuario)".

Não há composição neste módulo. Os dados temporários da troca de e-mail e da recuperação são campos soltos do próprio documento `Usuario`, e não subdocumentos; o [diagrama relacional](4%20-%20diagrama-relacional-configuracoes-usuario.md) os separa apenas como visão lógica.

Os atributos com `-` em `Usuario` são os campos com `select: false` no schema: o Mongoose só os lê quando o serviço pede, com `select('+campo')`, e eles nunca aparecem nas respostas da API. Os DTOs repetem os nomes dos campos da API em `snake_case`. Os métodos que devolvem `Promise~object~` devolvem um objeto simples, como `{ mensagem }`, `{ mensagem, email }` ou `{ token }`, detalhado no [dicionário de classes](3%20-%20dicionario-dados-classes-configuracoes-usuario.md).

`CampoTexto` é o componente de campo de texto usado nas telas; aparece no diagrama por causa do atributo `ajuda`, que mostra a regra da senha abaixo do campo "Nova senha".

## Rotas atendidas pelos controllers

As rotas HTTP não são métodos; por isso ficam nesta tabela, e não no diagrama.

| Método do controller                         | Rota HTTP                           | Exige JWT |
| -------------------------------------------- | ----------------------------------- | --------- |
| `UsuariosController.buscarMeuPerfil`         | `GET /usuarios/me`                  | Sim       |
| `UsuariosController.atualizarMeuPerfil`      | `PATCH /usuarios/me`                | Sim       |
| `UsuariosController.solicitarAlteracaoEmail` | `POST /usuarios/me/email/solicitar` | Sim       |
| `UsuariosController.confirmarAlteracaoEmail` | `POST /usuarios/me/email/confirmar` | Sim       |
| `UsuariosController.alterarMinhaSenha`       | `PATCH /usuarios/me/senha`          | Sim       |
| `AuthController.recuperarConta`              | `POST /auth/recuperar-conta`        | Não       |
| `AuthController.verificarRecuperacao`        | `POST /auth/verificar-recuperacao`  | Não       |
| `AuthController.redefinirSenha`              | `POST /auth/redefinir-senha`        | Não       |

Todas exigem o cabeçalho `X-API-Key`, conferido pelo `ApiKeyGuard` registrado para a API inteira.

## Regras representadas

- o usuário só lê e altera a própria conta: os controllers usam o `sub` do token, e não um identificador vindo da tela;
- `AuthService` concentra tudo o que envolve código enviado por e-mail: troca de e-mail e recuperação;
- `JwtAuthGuard` consulta `Usuario` em toda requisição autenticada e recusa a conta inativa e o token emitido antes de `senha_alterada_em`;
- o aplicativo nunca guarda a senha: guarda só o token da sessão (`conecta_mais_token`) e a preferência de tema (`conecta_mais_tema`).

## Rastreabilidade

RF-CFG-01 a RF-CFG-06; complementado por [Dicionário de classes](3%20-%20dicionario-dados-classes-configuracoes-usuario.md) e [Diagramas de sequência](6%20-%20diagramas-sequencia-configuracoes-usuario.md).
