# Manual de utilização

> Padrão da disciplina · Módulo MAN · Requisito R4 do Incremento 3 · Revisado em 13/11/2026

## Resumo

O requisito R4 do Incremento 3 pede um manual digital que oriente os usuários sobre como utilizar o sistema. Ele foi atendido em duas partes. A primeira é a tela Ajuda do aplicativo, com 42 perguntas frequentes separadas em 7 assuntos, busca por palavra e o selo "Responsável" nas perguntas que tratam de ações exclusivas do responsável pela comissão. A segunda é o manual completo em PDF, com 27 páginas, aberto pelo botão "Ver manual completo (PDF)" no fim da tela Ajuda. O R4 não usa a API nem o banco de dados: o conteúdo da Ajuda faz parte do aplicativo, e o PDF fica neste repositório.

## Escopo

- consultar a tela Ajuda, com ou sem login, com as perguntas organizadas por assunto;
- abrir e fechar a resposta de cada pergunta;
- buscar perguntas por palavra;
- abrir o manual completo em PDF a partir da tela Ajuda.

Ficam fora deste módulo a edição das perguntas pelo próprio aplicativo e a leitura do PDF dentro do aplicativo: o conteúdo só muda com uma nova versão, e o PDF é aberto no navegador do aparelho.

## Atores

| Ator | Quem é no sistema | O que faz neste módulo |
| --- | --- | --- |
| Usuário | Qualquer pessoa que usa o aplicativo, com ou sem login. | Lê as perguntas frequentes, busca por palavra e abre o manual em PDF. Não altera o conteúdo. |

## Índice padronizado

| Nº | Artefato | Finalidade |
| --: | --- | --- |
| 1 | [Casos de uso](1%20-%20casos-uso-manual-de-utilizacao.md) | Diagrama, ator e requisitos de usuário. |
| 2 | [Requisitos funcionais](2%20-%20requisitos-funcionais-manual-de-utilizacao.md) | RF-MAN-01 e RF-MAN-02 no formulário padrão. |
| 3 | [Diagrama de classes](3%20-%20diagrama-classes-manual-de-utilizacao.md) | Conteúdo da Ajuda, tela e componentes. |
| 3 | [Dicionário de classes](3%20-%20dicionario-dados-classes-manual-de-utilizacao.md) | Atributos, tipos, métodos e criação dos identificadores. |
| 4 | [Diagrama relacional](4%20-%20diagrama-relacional-manual-de-utilizacao.md) | Não se aplica: o R4 não usa banco de dados. Contém a justificativa. |
| 4 | [Dicionário relacional](4%20-%20dicionario-dados-relacional-manual-de-utilizacao.md) | Não se aplica, pelo mesmo motivo. Contém a justificativa. |
| 5 | [Requisitos não funcionais](5%20-%20requisitos-nao-funcionais-manual-de-utilizacao.md) | RNF-MAN-01 a RNF-MAN-09, com medida, verificação e situação. |
| 6 | [Diagramas de sequência](6%20-%20diagramas-sequencia-manual-de-utilizacao.md) | SD-MAN-01 e SD-MAN-02. |
| 7 | [Testes](7%20-%20testes-manual-de-utilizacao.md) | CT-MAN-01 a CT-MAN-13 e seus resultados. |
| — | [Manual do Usuário (PDF)](manual-usuario/Manual-do-Usuario-Conecta.pdf) | Manual completo do aplicativo, aberto pelo botão "Ver manual completo (PDF)" da tela Ajuda. |

## Rastreabilidade

| Caso de uso | Requisito | Sequência | Requisitos não funcionais | Testes |
| --- | --- | --- | --- | --- |
| UC-MAN-01 — Consultar Ajuda | RF-MAN-01 | SD-MAN-01 | RNF-MAN-01, 02, 05, 06, 07, 08, 09 | CT-MAN-01 a CT-MAN-08, CT-MAN-12, CT-MAN-13 |
| UC-MAN-02 — Abrir manual em PDF | RF-MAN-02 | SD-MAN-02 | RNF-MAN-01, 03, 04, 05, 06, 07, 08 | CT-MAN-09 a CT-MAN-12 |

## Base de conferência

FRONTEND na branch `homolog`, commit `66806e4`, e DOCUMENTATION na branch `main`, conferidos em 13/11/2026. Os arquivos principais são `FRONTEND/src/app/ajuda.tsx`, `FRONTEND/src/constants/ajuda.ts`, `FRONTEND/src/utils/busca-ajuda.ts`, `FRONTEND/src/components/Sanfona.tsx` e `FRONTEND/tests/busca-ajuda.test.cjs`. O BACKEND não é usado pelo R4.

## Repositórios

- [Backend](https://github.com/Conecta-IFSP/BACKEND)
- [Frontend](https://github.com/Conecta-IFSP/FRONTEND)
