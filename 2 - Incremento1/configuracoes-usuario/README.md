# Configurações de usuário

## Objetivo e escopo

O módulo permite consultar e alterar o próprio perfil, confirmar um novo e-mail, trocar ou recuperar a senha, escolher o tema e encerrar a sessão. O usuário nunca informa seu `_id`: nas rotas autenticadas, a conta é obtida do `sub` do JWT.

## Atores

- **Visitante:** solicita e conclui a recuperação da conta.
- **Usuário autenticado:** consulta e altera os próprios dados.
- **Serviço de e-mail:** entrega códigos de 7 caracteres.
- **Sistema operacional:** confirma identidade por biometria, PIN ou senha do aparelho quando existe credencial cadastrada.

## Documentos

1. [Casos de uso](1%20-%20casos-uso-configuracoes-usuario.md)
2. [Requisitos funcionais](2%20-%20requisitos-funcionais-configuracoes-usuario.md)
3. [Diagrama de classes](3%20-%20diagrama-classes-configuracoes-usuario.md)
4. [Dicionário de classes](3%20-%20dicionario-dados-classes-configuracoes-usuario.md)
5. [Diagrama relacional](4%20-%20diagrama-relacional-configuracoes-usuario.md)
6. [Dicionário relacional](4%20-%20dicionario-dados-relacional-configuracoes-usuario.md)
7. [Requisitos não funcionais](5%20-%20requisitos-nao-funcionais-configuracoes-usuario.md)
8. [Diagramas de sequência](6%20-%20diagramas-sequencia-configuracoes-usuario.md)
9. [Testes](7%20-%20testes-configuracoes-usuario.md)

## Rastreabilidade

| Caso de uso | Requisito | Rotas principais                                                           | Testes automatizados relacionados        |
| ----------- | --------- | -------------------------------------------------------------------------- | ---------------------------------------- |
| UC-CFG-01   | RF-CFG-01 | `GET/PATCH /usuarios/me`                                                   | testes de controller/service de usuários |
| UC-CFG-02   | RF-CFG-02 | `POST /usuarios/me/email/solicitar`, `/confirmar`                          | testes de `AuthService`                  |
| UC-CFG-03   | RF-CFG-03 | `PATCH /usuarios/me/senha`                                                 | testes de usuários                       |
| UC-CFG-04   | RF-CFG-04 | `POST /auth/recuperar-conta`, `/verificar-recuperacao`, `/redefinir-senha` | testes de `AuthService`                  |
| UC-CFG-05   | RF-CFG-05 | `PATCH /usuarios/me`                                                       | inspeção e teste manual pendente         |
| UC-CFG-06   | RF-CFG-06 | operação local                                                             | testes de sessão da API                  |

## Pendências conhecidas

- A troca e a redefinição de senha não invalidam JWTs já emitidos.
- A troca de senha autenticada não encerra a sessão atual, apesar da recomendação registrada na avaliação.
- A autenticação local é dispensada na web e em aparelho sem credencial cadastrada; ela é uma proteção de interface, não substitui autenticação do servidor.
- Não há limitação de tentativas por IP/e-mail nos endpoints de recuperação.
