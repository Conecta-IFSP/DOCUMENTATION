# Documentação - Manual de Utilização

Esta pasta reúne a documentação do requisito R4 do Incremento 3 do projeto Conecta+: o manual digital que orienta os usuários sobre como utilizar o sistema.

## Documentos

| Item | Documento | Descrição |
|---|---|---|
| 1 | [Diagrama de Casos de Uso](1%20-%20casos-uso-manual-de-utilizacao.md) | Apresenta o ator e os 2 casos de uso, com a relação «extend» entre eles. |
| 2 | [Requisitos Funcionais](2%20-%20requisitos-funcionais-manual-de-utilizacao.md) | Descreve RF-MAN-01 (Consultar Ajuda) e RF-MAN-02 (Abrir manual em PDF) no formulário padrão. |
| 3 | [Diagrama de Classes](3%20-%20diagrama-classes-manual-de-utilizacao.md) | Representa a estrutura do conteúdo da Ajuda e as classes do aplicativo que o exibem. |
| 3 | [Dicionário de Dados - Classes](3%20-%20dicionario-dados-classes-manual-de-utilizacao.md) | Descreve os tipos, os atributos e as regras do conteúdo e das classes da Ajuda. |
| 4 | [Diagrama Relacional](4%20-%20diagrama-relacional-manual-de-utilizacao.md) | Não se aplica: o R4 não usa banco de dados. Contém a justificativa. |
| 4 | [Dicionário de Dados - Modelo Relacional](4%20-%20dicionario-dados-relacional-manual-de-utilizacao.md) | Não se aplica, pelo mesmo motivo. Contém a justificativa. |
| 5 | [Requisitos Não Funcionais](5%20-%20requisitos-nao-funcionais-manual-de-utilizacao.md) | Define RNF-MAN-01 a RNF-MAN-09, com critério de aceitação e forma de verificação. |
| 6 | [Diagramas de Sequência](6%20-%20diagramas-sequencia-manual-de-utilizacao.md) | Contém SD-MAN-01 e SD-MAN-02. |
| 7 | [Testes](7%20-%20testes-manual-de-utilizacao.md) | Registra os casos de teste CT-MAN-01 a CT-MAN-13 e seus resultados. |
| — | [Manual do Usuário (PDF)](manual-usuario/Manual-do-Usuario-Conecta.pdf) | Manual completo do aplicativo, aberto pelo botão "Ver manual completo (PDF)" da tela Ajuda. |

## Escopo do Incremento

A documentação contempla os seguintes fluxos:

- consulta da tela Ajuda, com perguntas frequentes organizadas por assunto, com ou sem login;
- busca de perguntas por palavra;
- abertura do manual completo em PDF a partir da tela Ajuda.

## Rastreabilidade

| Caso de uso | Requisito | Diagrama de sequência | Requisitos não funcionais | Casos de teste |
|---|---|---|---|---|
| UC-MAN-01 - Consultar Ajuda | RF-MAN-01 | SD-MAN-01 | RNF-MAN-01, 02, 05, 06, 07, 08, 09 | CT-MAN-01 a CT-MAN-08, CT-MAN-12, CT-MAN-13 |
| UC-MAN-02 - Abrir manual em PDF | RF-MAN-02 | SD-MAN-02 | RNF-MAN-01, 03, 04, 05, 06, 07, 08 | CT-MAN-09 a CT-MAN-12 |

## Repositórios

- [Backend](https://github.com/Conecta-IFSP/BACKEND)
- [Frontend](https://github.com/Conecta-IFSP/FRONTEND)
