# Cadastro de organização

> Padrão da disciplina · Módulo ORG · Incremento 1 · Revisado em 27/11/2026

## Resumo

Uma organização é a instituição onde as comissões funcionam, como um câmpus ou uma escola. Qualquer usuário pode cadastrá-la informando o nome e a cidade; ela fica aguardando a autorização do administrador do sistema, e quem a cadastrou já é o seu primeiro administrador. Depois de autorizada, outras pessoas entram de duas formas: pedindo acesso com o código de 10 caracteres que o administrador compartilha, ou aceitando um convite feito pelo e-mail da sua conta. O administrador da organização aprova ou recusa os pedidos (e pode aprovar depois um pedido que recusou), promove membros a administrador, deixa o próprio papel quando há outro administrador, remove membros e troca o código. O administrador do sistema busca as organizações pelo nome, autoriza, revoga, corrige os dados e exclui as revogadas. As ações que apagam dados, retiram acesso, mudam papéis ou invalidam o código pedem confirmação antes de serem enviadas.

## Escopo

- cadastrar organização com nome, cidade e descrição opcional;
- listar as organizações do usuário e, para o administrador do sistema, buscar organizações pelo nome;
- autorizar, revogar, editar e excluir organizações;
- pedir entrada com o código de acesso;
- convidar um usuário pelo e-mail da conta e responder ao convite;
- aprovar ou recusar pedidos de entrada, inclusive aprovar depois um pedido recusado;
- ver, copiar e trocar o código de acesso;
- promover membro a administrador e deixar de ser administrador, sem que a organização fique sem nenhum;
- remover o acesso de um membro, cancelar um convite e excluir um pedido recusado;
- consultar os membros da organização.

Ficam fora deste módulo o cadastro de conta e o login, em [Cadastro de usuário](../cadastro-usuario/README.md); as comissões da organização, em [Cadastro de comissão](../cadastro-comissao/README.md); e as contagens de organizações do painel do sistema, a criação da organização de teste e o reset, que apaga todas as organizações, em [Informações do sistema](../informacao-sistemas/README.md).

## Atores

| Ator                         | Quem é no sistema                                                                                                | O que faz neste módulo                                                                                   |
| ---------------------------- | ---------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------- |
| Usuário                      | Conta ativa do tipo `USUARIO`.                                                                                   | Cadastra organizações, consulta as suas, pede entrada com código e responde convites.                    |
| Administrador da organização | Usuário cujo vínculo com a organização tem papel `ADMIN` e situação `APROVADO`. É uma especialização do usuário. | Consulta membros, analisa pedidos, convida, troca o código, muda papéis e remove vínculos.               |
| Administrador do sistema     | Conta ativa do tipo `ADMIN_SISTEMA`.                                                                             | Consulta e busca todas as organizações, autoriza, revoga, edita e exclui. Não participa de organizações. |

## Índice padronizado

|  Nº | Artefato                                                                               | Finalidade                                              |
| --: | -------------------------------------------------------------------------------------- | ------------------------------------------------------- |
|   1 | [Casos de uso](1%20-%20casos-uso-cadastro-organizacao.md)                              | Diagrama, atores e requisitos de usuário.               |
|   2 | [Requisitos funcionais](2%20-%20requisitos-funcionais-cadastro-organizacao.md)         | Cada caso de uso no formulário padrão.                  |
|   3 | [Diagrama de classes](3%20-%20diagrama-classes-cadastro-organizacao.md)                | Telas, API, serviço e modelo envolvidos.                |
|   3 | [Dicionário de classes](3%20-%20dicionario-dados-classes-cadastro-organizacao.md)      | Atributos, tipos e operações de cada classe.            |
|   4 | [Diagrama relacional](4%20-%20diagrama-relacional-cadastro-organizacao.md)             | Visão lógica dos dados da organização no MongoDB.       |
|   4 | [Dicionário relacional](4%20-%20dicionario-dados-relacional-cadastro-organizacao.md)   | Campo a campo: tipo, nulo, chaves e geração de valores. |
|   5 | [Requisitos não funcionais](5%20-%20requisitos-nao-funcionais-cadastro-organizacao.md) | Critérios de qualidade com medida e verificação.        |
|   6 | [Diagramas de sequência](6%20-%20diagramas-sequencia-cadastro-organizacao.md)          | Mensagens entre tela, API, serviço e banco.             |
|   7 | [Testes](7%20-%20testes-cadastro-organizacao.md)                                       | Testes automatizados e roteiros executados.             |

## Rastreabilidade

| Caso de uso | Requisito | Sequência | Testes                                     |
| ----------- | --------- | --------- | ------------------------------------------ |
| UC-ORG-01   | RF-ORG-01 | SD-ORG-01 | CT-ORG-01, CT-ORG-07, CT-ORG-08, CT-ORG-09 |
| UC-ORG-02   | RF-ORG-02 | SD-ORG-02 | CT-ORG-02, CT-ORG-10, CT-ORG-11            |
| UC-ORG-03   | RF-ORG-03 | SD-ORG-03 | CT-ORG-03, CT-ORG-12                       |
| UC-ORG-04   | RF-ORG-04 | SD-ORG-04 | CT-ORG-03, CT-ORG-12, CT-ORG-28            |
| UC-ORG-05   | RF-ORG-05 | SD-ORG-05 | CT-ORG-03, CT-ORG-12                       |
| UC-ORG-06   | RF-ORG-06 | SD-ORG-06 | CT-ORG-04, CT-ORG-13, CT-ORG-28            |
| UC-ORG-07   | RF-ORG-07 | SD-ORG-07 | CT-ORG-05, CT-ORG-14, CT-ORG-15            |
| UC-ORG-08   | RF-ORG-08 | SD-ORG-08 | CT-ORG-06, CT-ORG-16                       |
| UC-ORG-09   | RF-ORG-09 | SD-ORG-09 | CT-ORG-17, CT-ORG-18, CT-ORG-19, CT-ORG-28 |
| UC-ORG-10   | RF-ORG-10 | SD-ORG-10 | CT-ORG-05, CT-ORG-20, CT-ORG-28, CT-ORG-31 |
| UC-ORG-11   | RF-ORG-11 | SD-ORG-11 | CT-ORG-06, CT-ORG-21                       |
| UC-ORG-12   | RF-ORG-12 | SD-ORG-12 | CT-ORG-06, CT-ORG-22, CT-ORG-23, CT-ORG-28 |
| UC-ORG-13   | RF-ORG-13 | SD-ORG-13 | CT-ORG-24, CT-ORG-25, CT-ORG-28            |
| UC-ORG-14   | RF-ORG-14 | SD-ORG-14 | CT-ORG-26                                  |

CT-ORG-27 verifica a autenticação e a autorização comuns a todas as rotas (RNF-ORG-01 e RNF-ORG-02); CT-ORG-29 e CT-ORG-30 cobrem os requisitos não funcionais de tela pequena (RNF-ORG-07, executado em 27/11/2026) e iOS (RNF-ORG-08, pendente).

## Base de conferência

BACKEND e FRONTEND conferidos em 27/11/2026 na branch `main`. Os arquivos deste módulo não têm alterações locais; a última mudança em `BACKEND/src/organizacoes` é a da entrega do Incremento 2, em 23/10/2026, e as últimas de `FRONTEND/src/app/(abas)/inicio.tsx` (confirmação de identidade ao marcar a entrada numa reunião e descrição do item "Sistema") não mexem nas partes de organização. Das mudanças de 27/11/2026, as variáveis `TEST_USERS_PASSWORD` e `TEST_USER_*` do `.env.example` do backend são do módulo SIS; no frontend, as correções do João (desbloqueio da sessão salva, presença na tela da reunião, `FolhaInferior` e entrega do PDF) e o aviso de votação iniciada ficam fora deste módulo (as folhas de organização usam a `FolhaInferior` sem acompanhar o teclado, e a troca do `KeyboardAvoidingView` não as muda), e a última mudança do dia faz quebrar, quando não cabe, a linha com a etiqueta do papel e os números do cartão de "Suas organizações" (`FRONTEND/src/app/(abas)/organizacoes.tsx`, RF-ORG-02); o diagrama de classes, o dicionário de classes e as sequências foram revistos sobre essa versão e não mudam com ela, que só troca um estilo. Os arquivos principais são `BACKEND/src/organizacoes/organizacoes.controller.ts`, `BACKEND/src/organizacoes/organizacoes.service.ts`, `BACKEND/src/organizacoes/schemas/organizacao.schema.ts`, `BACKEND/src/organizacoes/dto/*`, `BACKEND/src/organizacoes/usuario-ativo.guard.ts`, `BACKEND/src/auth/admin-sistema.guard.ts`, `FRONTEND/src/app/organizacoes/cadastro.tsx`, `FRONTEND/src/app/organizacoes/participar.tsx`, `FRONTEND/src/app/organizacoes/aprovacao.tsx`, `FRONTEND/src/app/(abas)/organizacoes.tsx`, `FRONTEND/src/app/(abas)/organizacao.tsx` e `FRONTEND/src/app/(abas)/inicio.tsx`.
