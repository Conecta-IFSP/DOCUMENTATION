# Manual de utilização

> Padrão da disciplina · Módulo MAN · Requisito R4 do Incremento 3 · Revisado em 26/11/2026

## Resumo

O requisito R4 do Incremento 3 pede um manual digital que oriente os usuários sobre como utilizar o sistema. O aplicativo o atende em duas partes. A primeira é a tela Ajuda, com 42 perguntas frequentes separadas em 7 assuntos, busca por palavra e o selo "Responsável" nas perguntas sobre ações exclusivas do responsável pela comissão. A segunda é o manual completo em PDF, com 27 páginas em A4, que vem dentro do aplicativo e abre na tela "Manual de utilização" pelo botão "Ver manual completo (PDF)", no fim da Ajuda. As duas partes funcionam sem login e sem internet: o conteúdo da Ajuda e o arquivo do manual fazem parte do pacote do aplicativo, e o R4 não usa a API nem o banco de dados.

## Escopo

- consultar a tela Ajuda, com ou sem login, com as perguntas organizadas por assunto;
- abrir e fechar a resposta de cada pergunta;
- buscar perguntas por palavra;
- abrir e ler o manual completo em PDF dentro do aplicativo.

Ficam fora deste módulo a edição das perguntas e do manual pelo próprio aplicativo: o conteúdo só muda com uma nova versão do aplicativo. Na versão web, o leitor do manual não é exibido, porque o componente usado para mostrá-lo só existe no Android e no iOS (limitação registrada em RF-MAN-02).

## Atores

| Ator    | Quem é no sistema                                        | O que faz neste módulo                                                                                 |
| ------- | -------------------------------------------------------- | ------------------------------------------------------------------------------------------------------ |
| Usuário | Qualquer pessoa que usa o aplicativo, com ou sem login.  | Lê as perguntas frequentes, busca por palavra e lê o manual em PDF. Não altera nenhum conteúdo.        |

## Índice padronizado

|  Nº | Artefato                                                                                  | Finalidade                                                                 |
| --: | ----------------------------------------------------------------------------------------- | -------------------------------------------------------------------------- |
|   1 | [Casos de uso](1%20-%20casos-uso-manual-de-utilizacao.md)                                 | Diagrama, ator e requisitos de usuário.                                    |
|   2 | [Requisitos funcionais](2%20-%20requisitos-funcionais-manual-de-utilizacao.md)            | Cada caso de uso no formulário padrão.                                     |
|   3 | [Diagrama de classes](3%20-%20diagrama-classes-manual-de-utilizacao.md)                   | Conteúdo da Ajuda, telas, componentes e leitor do manual.                  |
|   3 | [Dicionário de classes](3%20-%20dicionario-dados-classes-manual-de-utilizacao.md)         | Atributos, tipos e operações de cada classe.                               |
|   4 | [Diagrama relacional](4%20-%20diagrama-relacional-manual-de-utilizacao.md)                | Visão lógica do conteúdo guardado no aplicativo, sem banco de dados.       |
|   4 | [Dicionário relacional](4%20-%20dicionario-dados-relacional-manual-de-utilizacao.md)      | Campo a campo: tipo, nulo, chaves e criação dos identificadores.           |
|   5 | [Requisitos não funcionais](5%20-%20requisitos-nao-funcionais-manual-de-utilizacao.md)    | Critérios de qualidade com medida e verificação.                           |
|   6 | [Diagramas de sequência](6%20-%20diagramas-sequencia-manual-de-utilizacao.md)             | Mensagens entre telas, componentes e bibliotecas.                          |
|   7 | [Testes](7%20-%20testes-manual-de-utilizacao.md)                                          | Testes automatizados, inspeções e roteiros manuais.                        |
|   — | [Manual do Usuário (PDF)](manual-usuario/manual.pdf)                                      | Cópia do arquivo que vai dentro do aplicativo (`FRONTEND/assets/manual/manual.pdf`). |

## Rastreabilidade

| Caso de uso                       | Requisito | Sequência | Requisitos não funcionais                       | Testes                                                   |
| --------------------------------- | --------- | --------- | ----------------------------------------------- | -------------------------------------------------------- |
| UC-MAN-01 — Consultar Ajuda       | RF-MAN-01 | SD-MAN-01 | RNF-MAN-01, 02, 05, 06, 07, 08, 09              | CT-MAN-01 a CT-MAN-08, CT-MAN-12, CT-MAN-13              |
| UC-MAN-02 — Abrir manual em PDF   | RF-MAN-02 | SD-MAN-02 | RNF-MAN-01, 02, 03, 04, 05, 06, 07, 08, 10      | CT-MAN-11, CT-MAN-14 a CT-MAN-18                         |

CT-MAN-09 e CT-MAN-10 ficam registrados em [Testes](7%20-%20testes-manual-de-utilizacao.md) como execuções de 13/11/2026 sobre o FRONTEND `66806e4`, em que o botão abria o manual no navegador; eles não cobrem o RF-MAN-02 da base conferida.

## Base de conferência

FRONTEND na branch `homolog`, conferido em 26/11/2026 em `8fc0a5a`, commit que deixou em quatro passos a pergunta `conta-email-senha`. O manual entrou no pacote do aplicativo em `fa1ba7a`, e o texto do PDF foi revisado em `93fa767`. Os arquivos principais são `FRONTEND/src/app/ajuda.tsx`, `FRONTEND/src/app/manual.tsx`, `FRONTEND/src/components/ManualPdf.tsx`, `FRONTEND/src/constants/ajuda.ts`, `FRONTEND/src/utils/busca-ajuda.ts`, `FRONTEND/src/components/Sanfona.tsx`, `FRONTEND/assets/manual/manual.pdf`, `FRONTEND/assets/pdfjs/`, `FRONTEND/metro.config.js` e `FRONTEND/tests/busca-ajuda.test.cjs`. O BACKEND não é usado pelo R4.

## Repositórios

- [Backend](https://github.com/Conecta-IFSP/BACKEND)
- [Frontend](https://github.com/Conecta-IFSP/FRONTEND)
