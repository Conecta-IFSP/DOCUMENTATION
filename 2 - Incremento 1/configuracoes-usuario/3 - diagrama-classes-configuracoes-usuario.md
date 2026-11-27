# Diagrama de classes — Configurações de usuário

> Padrão do Incremento 1 · Módulo CFG

## Resumo

Representa as telas, contexts, controllers, services e entidade `Usuario` envolvidos nas configurações.

O diagrama representa classes reais do backend. `Usuario` é persistente; DTOs validam entradas; controllers delegam regras aos services.

```mermaid
classDiagram
direction LR
class Usuario {
  +ObjectId _id
  +String nome
  +String email
  -String senha_hash
  +TipoUsuario tipo
  +TemaUsuario tema
  -String reset_senha_codigo_hash
  -String reset_senha_token_hash
  -Date reset_senha_expira_em
  -String email_pendente
  -String email_pendente_codigo_hash
  -Date email_pendente_expira_em
  +Boolean email_verificado
  +Boolean ativo
  +ProvedorAutenticacao provedor_autenticacao
  +Date criado_em
  +Date atualizado_em
}
class TemaUsuario {
  <<enumeration>>
  claro
  escuro
  sistema
}
class UsuariosController {
  +buscarMeuPerfil(req)
  +atualizarMeuPerfil(req,dto)
  +solicitarAlteracaoEmail(req,dto)
  +confirmarAlteracaoEmail(req,dto)
  +alterarMinhaSenha(req,dto)
}
class UsuariosService {
  +buscarPorId(id)
  +atualizarPerfil(id,dto)
  +alterarSenha(id,dto)
}
class AuthController {
  +recuperarConta(dto)
  +verificarRecuperacao(dto)
  +redefinirSenha(dto)
}
class AuthService {
  +solicitarAlteracaoEmail(id,email)
  +confirmarAlteracaoEmail(id,email,codigo)
  +solicitarRecuperacao(dto)
  +verificarRecuperacao(dto)
  +redefinirSenha(dto)
  -gerarCodigoVerificacao()
  -hashToken(token)
}
class AtualizarPerfilDto {
  +String nome?
  +String email?
  +TemaUsuario tema?
}
class AlterarSenhaDto {
  +String senha_atual
  +String nova_senha
}
class RecuperarContaDto { +String email }
class VerificarRecuperacaoDto {
  +String email
  +String codigo
}
class RedefinirSenhaDto {
  +String token
  +String nova_senha
}
class SolicitarAlteracaoEmailDto { +String email }
class ConfirmarAlteracaoEmailDto {
  +String email
  +String codigo
}

Usuario --> TemaUsuario : tema
UsuariosController --> UsuariosService : delega perfil/senha
UsuariosController --> AuthService : delega troca de e-mail
AuthController --> AuthService : delega recuperação
UsuariosService --> Usuario : consulta e atualiza
AuthService --> Usuario : códigos, token, e-mail e senha
UsuariosController ..> AtualizarPerfilDto
UsuariosController ..> AlterarSenhaDto
UsuariosController ..> SolicitarAlteracaoEmailDto
UsuariosController ..> ConfirmarAlteracaoEmailDto
AuthController ..> RecuperarContaDto
AuthController ..> VerificarRecuperacaoDto
AuthController ..> RedefinirSenhaDto
```

## Relações importantes

- `UsuariosController` usa `UsuariosService` para perfil/senha e `AuthService` para troca confirmada de e-mail.
- `AuthController` expõe o fluxo público de recuperação.
- Hashes e campos temporários usam `select: false`; não aparecem em consultas comuns nem nas respostas da API.
- `AtualizarPerfilDto.email` existe para validação defensiva, porém `UsuariosService.atualizarPerfil()` recusa alteração direta.
