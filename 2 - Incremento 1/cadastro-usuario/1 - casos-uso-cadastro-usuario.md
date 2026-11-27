# Casos de uso — Cadastro de usuário

> Padrão da disciplina · Módulo USR · Revisado em 26/11/2026

## Resumo

Este documento mostra quem cria a conta e quem entra no aplicativo, e o que cada um pode fazer. Cada caso de uso é descrito em detalhe, como requisito funcional no formulário padrão, em [Requisitos funcionais](2%20-%20requisitos-funcionais-cadastro-usuario.md). A participação do usuário nas organizações (pedido de acesso por código, convite, decisão do administrador, papel e remoção) está documentada no módulo [Cadastro de organização](../cadastro-organizacao/README.md).

## Diagrama de casos de uso

![Diagrama de casos de uso do cadastro de usuário](diagrama-casos-uso-cadastro-usuario.svg)

Figura 1 — Diagrama de casos de uso do cadastro e da autenticação de usuário. Fonte: equipe Conecta+.

Cada linha entre um ator e um caso de uso leva o código do requisito funcional que o descreve.

## Atores

**Visitante.** É qualquer pessoa que abre o aplicativo sem estar conectada. Pode criar uma conta com nome, e-mail e senha e confirmá-la com o código recebido por e-mail, entrar com e-mail e senha e entrar com a conta Google. Não vê organizações, comissões nem reuniões antes de entrar. O administrador do sistema também é um visitante até entrar: ele usa o mesmo login com e-mail e senha, mas a conta dele não é criada pelo cadastro: o servidor a cria ao iniciar, com os dados configurados no ambiente, quando ainda não existe nenhum administrador do sistema nem outra conta com o e-mail configurado.

**Servidor de e-mail.** É o sistema externo que entrega a mensagem com o código de confirmação do cadastro. A API não espera a entrega para responder; se ela falhar, a conta ainda não confirmada é apagada.

**Google.** É o sistema externo que confirma a identidade da pessoa e devolve à API o token com o e-mail, o nome e o identificador da conta Google.

## Relacionamentos entre casos de uso

Não há generalização entre atores nem `«include»` ou `«extend»` entre casos de uso: cada caso de uso é iniciado por uma ação própria do visitante, e os dois sistemas externos participam como atores secundários.

## Requisitos funcionais de usuário

| Caso de uso | Requisito | Nome                              | Ator principal |
| ----------- | --------- | --------------------------------- | -------------- |
| UC-USR-01   | RF-USR-01 | Cadastrar e confirmar conta local | Visitante      |
| UC-USR-02   | RF-USR-02 | Autenticar com senha              | Visitante      |
| UC-USR-03   | RF-USR-03 | Autenticar com Google             | Visitante      |

**Requisito RF-USR-01 — Cadastrar e confirmar conta local.** O sistema deve permitir ao visitante criar uma conta com nome, e-mail e uma senha forte e, em seguida, confirmar o e-mail digitando o código de 7 caracteres enviado para ele. A conta só pode ser usada para entrar depois dessa confirmação.

**Requisito RF-USR-02 — Autenticar com senha.** O sistema deve permitir ao visitante entrar no aplicativo com o e-mail e a senha de uma conta ativa e já confirmada, mantendo a pessoa conectada até a sessão expirar ou a conta deixar de estar disponível.

**Requisito RF-USR-03 — Autenticar com Google.** O sistema deve permitir ao visitante entrar com a conta Google, criando a conta no primeiro acesso ou ligando a conta Google à conta que já existe com o mesmo e-mail.

## Não implementado

- Entrar pela biometria do aparelho depois do primeiro acesso, sugerido na avaliação do Incremento 1: não implementado. Enquanto o token da sessão é válido (1 hora), o aplicativo reabre já conectado, sem pedir nada.
- Pedir um novo código de confirmação do cadastro: não implementado. Se o código expirar, a conta fica sem confirmação e o mesmo e-mail não pode ser usado num novo cadastro; ela só é confirmada se a pessoa entrar com a conta Google do mesmo e-mail (RF-USR-03).
