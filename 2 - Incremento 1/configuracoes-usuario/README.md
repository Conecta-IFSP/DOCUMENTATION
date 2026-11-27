# Configurações de usuário

> Padrão da disciplina · Módulo CFG · Incremento 1 · Revisado em 27/11/2026

## Resumo

Na tela "Perfil", quem está com a sessão aberta altera o próprio nome, escolhe o tema do aplicativo, troca o e-mail, troca a senha e sai da conta. Salvar o novo nome, abrir a troca de e-mail e abrir a troca de senha pedem antes a confirmação de identidade do aparelho (biometria, Face ID ou PIN), quando ele tem bloqueio de tela. A troca de e-mail só vale depois que a pessoa digita no aplicativo o código enviado ao novo endereço. A troca de senha exige a senha atual e encerra a sessão em todos os aparelhos: o aparelho que trocou volta ao login na hora e os demais voltam na próxima consulta ao servidor. Quem esqueceu a senha recupera a conta pela tela de login, com um código enviado ao e-mail e digitado no aplicativo, sem abrir link. A nova senha precisa ter 8 caracteres ou mais, com letra maiúscula, letra minúscula, número e símbolo. O servidor responde às solicitações de código sem esperar o envio do e-mail.

## Escopo

- consultar o perfil e alterar o nome;
- trocar o e-mail, com confirmação por código enviado ao novo endereço;
- alterar a senha, com a senha atual, e encerrar todas as sessões da conta;
- recuperar a conta por código, definindo uma nova senha;
- escolher o tema: Sistema, Claro ou Escuro;
- encerrar a sessão no aparelho.

Ficam fora deste módulo o cadastro, a confirmação do cadastro e o login, descritos em [Cadastro de usuário](../cadastro-usuario/README.md).

**Não implementado** (observações da avaliação do Incremento 1):

- pré-visualizar o tema sem salvar: o tema escolhido é aplicado e salvo no mesmo toque;
- confirmar a identidade antes de alterar o tema: a escolha do tema exige só a sessão aberta. A confirmação no aparelho (biometria, Face ID ou PIN) é pedida para salvar o nome, abrir a troca de e-mail e a troca de senha e gravar a senha recuperada.

## Atores

| Ator                | Quem é no sistema                                                                                               | O que faz neste módulo                                         |
| ------------------- | --------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------- |
| Usuário autenticado | Pessoa com a sessão aberta no aparelho, seja usuário comum ou administrador do sistema.                         | Altera o nome, troca o e-mail e a senha, escolhe o tema e sai. |
| Visitante           | Pessoa sem sessão aberta no aparelho, na tela de login.                                                         | Recupera a conta e define uma nova senha.                      |
| Serviço de e-mail   | Servidor SMTP configurado no servidor da API. É um ator secundário: só recebe as mensagens da API e as entrega. | Entrega o código da troca de e-mail e o da recuperação.        |

## Índice padronizado

|  Nº | Artefato                                                                                | Finalidade                                              |
| --: | --------------------------------------------------------------------------------------- | ------------------------------------------------------- |
|   1 | [Casos de uso](1%20-%20casos-uso-configuracoes-usuario.md)                              | Diagrama, atores e requisitos de usuário.               |
|   2 | [Requisitos funcionais](2%20-%20requisitos-funcionais-configuracoes-usuario.md)         | Cada caso de uso no formulário padrão.                  |
|   3 | [Diagrama de classes](3%20-%20diagrama-classes-configuracoes-usuario.md)                | Telas, contextos, API, serviços e modelo envolvidos.    |
|   3 | [Dicionário de classes](3%20-%20dicionario-dados-classes-configuracoes-usuario.md)      | Atributos, tipos e operações de cada classe.            |
|   4 | [Diagrama relacional](4%20-%20diagrama-relacional-configuracoes-usuario.md)             | Visão lógica dos dados da conta no MongoDB.             |
|   4 | [Dicionário relacional](4%20-%20dicionario-dados-relacional-configuracoes-usuario.md)   | Campo a campo: tipo, nulo, chaves e geração de valores. |
|   5 | [Requisitos não funcionais](5%20-%20requisitos-nao-funcionais-configuracoes-usuario.md) | Critérios de qualidade com medida e verificação.        |
|   6 | [Diagramas de sequência](6%20-%20diagramas-sequencia-configuracoes-usuario.md)          | Mensagens entre tela, API, serviço e banco.             |
|   7 | [Testes](7%20-%20testes-configuracoes-usuario.md)                                       | Testes automatizados e roteiros executados.             |

## Rastreabilidade

| Caso de uso | Requisito | Sequência | Testes                                                                                           |
| ----------- | --------- | --------- | ------------------------------------------------------------------------------------------------ |
| UC-CFG-01   | RF-CFG-01 | SD-CFG-01 | CT-CFG-01, CT-CFG-14, CT-CFG-15, CT-CFG-17                                                       |
| UC-CFG-02   | RF-CFG-02 | SD-CFG-02 | CT-CFG-02, CT-CFG-12, CT-CFG-15                                                                  |
| UC-CFG-03   | RF-CFG-03 | SD-CFG-03 | CT-CFG-03, CT-CFG-07, CT-CFG-08, CT-CFG-09, CT-CFG-10, CT-CFG-11, CT-CFG-14, CT-CFG-15, CT-CFG-16 |
| UC-CFG-04   | RF-CFG-04 | SD-CFG-04 | CT-CFG-04, CT-CFG-11, CT-CFG-12, CT-CFG-15, CT-CFG-16                                            |
| UC-CFG-05   | RF-CFG-05 | SD-CFG-05 | CT-CFG-05, CT-CFG-15                                                                             |
| UC-CFG-06   | RF-CFG-06 | SD-CFG-06 | CT-CFG-06, CT-CFG-10, CT-CFG-13, CT-CFG-15, CT-CFG-18                                            |

## Base de conferência

BACKEND e FRONTEND na branch `homolog`, conferidos em 27/11/2026: backend `d1f1aeb` e frontend `10abe49`. A revisão anterior, de 26/11/2026, usou o backend `caca2d2` e o frontend `8fc0a5a`; desde então, a única mudança nos arquivos deste módulo é o subtítulo da linha "Ajuda" do perfil, que passou a "Manual de utilização". Reconferidos na mesma data sobre o backend `6e0d3f5` e o frontend `fd790db`: o backend só acrescenta ao `.env.example` as variáveis `TEST_USERS_PASSWORD` e `TEST_USER_*`, do módulo SIS, e o frontend só muda o aviso e o cartão da votação, a entrega do PDF e um estilo do cartão de "Suas organizações", fora deste módulo; o diagrama de classes, o dicionário de classes e as sequências foram revistos sobre o `8cf4839`, e o `fd790db` não muda nada do que eles mostram. Esses commits incluem a regra de senha forte, o registro de `senha_alterada_em` com recusa dos tokens anteriores, o envio de e-mail sem espera e os avisos nas telas de senha; no frontend, a confirmação de identidade antes de salvar o nome entrou em `4abc6fa`. Os arquivos principais são `BACKEND/src/usuarios/usuarios.service.ts`, `BACKEND/src/usuarios/usuarios.controller.ts`, `BACKEND/src/usuarios/schemas/usuario.schema.ts`, `BACKEND/src/usuarios/dto/senha-forte.ts`, `BACKEND/src/auth/auth.service.ts`, `BACKEND/src/auth/jwt-auth.guard.ts`, `FRONTEND/src/app/perfil.tsx`, `FRONTEND/src/app/login/alterar-senha.tsx`, `FRONTEND/src/app/login/trocar-email.tsx`, `FRONTEND/src/app/login/recuperar-conta.tsx`, `FRONTEND/src/contexts/SessaoContext.tsx`, `FRONTEND/src/contexts/TemaContext.tsx` e `FRONTEND/src/services/autenticacao-local.ts`.
