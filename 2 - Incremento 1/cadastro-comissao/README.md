# Cadastro de comissão

> Padrão da disciplina · Módulo COM · Incremento 1 · Revisado em 27/11/2026

## Resumo

Uma comissão é um grupo de trabalho dentro de uma organização aprovada. Ela tem uma equipe de integrantes e exatamente um responsável. O administrador da organização cria a comissão já escolhendo o responsável, altera o nome e a descrição, desativa e reativa a comissão sem apagar nada, adiciona e remove integrantes e troca o responsável numa única operação, que promove o novo e deixa o anterior como membro. Os demais membros da organização veem apenas as comissões de que fazem parte. A tela da comissão é dividida em duas abas, **Reuniões** e **Equipe**, e as ações de gerência ficam em folhas que abrem por cima dela.

## Escopo

- listar as comissões de uma organização, com busca pelo nome e filtro por situação;
- consultar uma comissão, com as abas Reuniões e Equipe;
- cadastrar a comissão com o seu responsável;
- editar o nome e a descrição;
- desativar e reativar a comissão, sem apagar a equipe;
- adicionar integrante;
- substituir o responsável;
- remover integrante.

Ficam fora deste módulo o cadastro da organização e o vínculo das pessoas com ela, em [Cadastro de organização](../cadastro-organizacao/README.md), e o agendamento das reuniões que aparecem na aba Reuniões, em [Reuniões](../../3%20-%20Incremento%202/reunioes/README.md). O vínculo aprovado com a organização é a entrada deste módulo: só quem tem esse vínculo pode entrar numa comissão.

## Atores

| Ator                         | Quem é no sistema                                                                                | O que faz neste módulo                                                                                                                   |
| ---------------------------- | ------------------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------- |
| Membro da organização        | Usuário comum com vínculo `APROVADO` numa organização `APROVADA`, com papel `MEMBRO` ou `ADMIN`. | Lista e consulta as comissões de cuja equipe faz parte.                                                                                  |
| Administrador da organização | Membro com papel `ADMIN` e vínculo `APROVADO`. É uma especialização do membro da organização.     | Vê todas as comissões da organização; cadastra, edita, desativa e reativa comissões; adiciona e remove integrantes; troca o responsável. |

O responsável da comissão não tem ação própria neste módulo: ele consulta a comissão como qualquer integrante. O papel de responsável é o que lhe permite agendar reuniões, escrever a ata e abrir e encerrar votações, nos módulos REU, ATA e VOT. O administrador do sistema (`ADMIN_SISTEMA`) não usa este módulo: todas as rotas de comissões recusam essa conta.

## Índice padronizado

|  Nº | Artefato                                                                            | Finalidade                                              |
| --: | ----------------------------------------------------------------------------------- | ------------------------------------------------------- |
|   1 | [Casos de uso](1%20-%20casos-uso-cadastro-comissao.md)                              | Diagrama, atores e requisitos de usuário.               |
|   2 | [Requisitos funcionais](2%20-%20requisitos-funcionais-cadastro-comissao.md)         | Cada caso de uso no formulário padrão.                  |
|   3 | [Diagrama de classes](3%20-%20diagrama-classes-cadastro-comissao.md)                | Telas, API, serviço e modelos envolvidos.               |
|   3 | [Dicionário de classes](3%20-%20dicionario-dados-classes-cadastro-comissao.md)      | Atributos, tipos e operações de cada classe.            |
|   4 | [Diagrama relacional](4%20-%20diagrama-relacional-cadastro-comissao.md)             | Visão lógica dos dados da comissão no MongoDB.          |
|   4 | [Dicionário relacional](4%20-%20dicionario-dados-relacional-cadastro-comissao.md)   | Campo a campo: tipo, nulo, chaves e geração de valores. |
|   5 | [Requisitos não funcionais](5%20-%20requisitos-nao-funcionais-cadastro-comissao.md) | Critérios de qualidade com medida e verificação.        |
|   6 | [Diagramas de sequência](6%20-%20diagramas-sequencia-cadastro-comissao.md)          | Mensagens entre tela, API, serviço e banco.             |
|   7 | [Testes](7%20-%20testes-cadastro-comissao.md)                                       | Testes automatizados e roteiros executados.             |

## Rastreabilidade

| Caso de uso | Requisito | Sequência | Testes                                                                                 |
| ----------- | --------- | --------- | -------------------------------------------------------------------------------------- |
| UC-COM-01   | RF-COM-01 | SD-COM-01 | CT-COM-01, CT-COM-07, CT-COM-08, CT-COM-09, CT-COM-10                                  |
| UC-COM-02   | RF-COM-02 | SD-COM-02 | CT-COM-01, CT-COM-10, CT-COM-11, CT-COM-12, CT-COM-13, CT-COM-27, CT-COM-31            |
| UC-COM-03   | RF-COM-03 | SD-COM-03 | CT-COM-02, CT-COM-14, CT-COM-15, CT-COM-16, CT-COM-17, CT-COM-30, CT-COM-33            |
| UC-COM-04   | RF-COM-04 | SD-COM-04 | CT-COM-03, CT-COM-13, CT-COM-17                                                        |
| UC-COM-05   | RF-COM-05 | SD-COM-05 | CT-COM-03, CT-COM-13, CT-COM-18, CT-COM-34                                             |
| UC-COM-06   | RF-COM-06 | SD-COM-06 | CT-COM-03, CT-COM-18, CT-COM-19                                                        |
| UC-COM-07   | RF-COM-07 | SD-COM-07 | CT-COM-04, CT-COM-13, CT-COM-20, CT-COM-21, CT-COM-22, CT-COM-26, CT-COM-30            |
| UC-COM-08   | RF-COM-08 | SD-COM-08 | CT-COM-05, CT-COM-13, CT-COM-19, CT-COM-23, CT-COM-24, CT-COM-30, CT-COM-31            |
| UC-COM-09   | RF-COM-09 | SD-COM-09 | CT-COM-06, CT-COM-13, CT-COM-22, CT-COM-25, CT-COM-26, CT-COM-27, CT-COM-30            |

CT-COM-28 verifica o acesso às rotas (RNF-COM-01); CT-COM-29 verifica a rota auxiliar `GET /comissoes/minhas/:organizacaoId`, que o aplicativo não usa; CT-COM-31, executado em parte em 27/11/2026, cobre também a confirmação, a tela de celular e a portabilidade (RNF-COM-05, RNF-COM-07 e RNF-COM-08); CT-COM-32 é a repetição dos roteiros em iOS, pendente.

## Base de conferência

BACKEND e FRONTEND na branch `homolog`, conferidos em 27/11/2026 (backend `d1f1aeb`, frontend `10abe49`); os arquivos do módulo não têm alterações locais além desses commits. Reconferidos na mesma data sobre o backend `6e0d3f5` e o frontend `fd790db`: o backend só acrescenta ao `.env.example` as variáveis `TEST_USERS_PASSWORD` e `TEST_USER_*`, do módulo SIS, e o frontend só muda o aviso e o cartão da votação, a entrega do PDF e um estilo do cartão de "Suas organizações", fora deste módulo; o diagrama de classes, o dicionário de classes e as sequências foram revistos sobre o `8cf4839`, e o `fd790db` não muda nada do que eles mostram. Os arquivos principais são `BACKEND/src/comissoes/comissoes.controller.ts`, `BACKEND/src/comissoes/comissoes.service.ts`, `BACKEND/src/comissoes/schemas/comissao.schema.ts`, os DTOs em `BACKEND/src/comissoes/dto/`, `FRONTEND/src/app/comissoes/nova.tsx`, `FRONTEND/src/app/comissoes/[id].tsx`, a aba Comissões de `FRONTEND/src/app/(abas)/organizacao.tsx`, `FRONTEND/src/components/SeletorPessoaComissao.tsx` e `FRONTEND/src/utils/validacao.ts`; as alterações feitas por outros módulos foram conferidas em `BACKEND/src/organizacoes/organizacoes.service.ts`, `BACKEND/src/sistema/sistema.service.ts`, `BACKEND/src/reunioes/reunioes.service.ts` e `BACKEND/src/presencas/presencas.service.ts`.
