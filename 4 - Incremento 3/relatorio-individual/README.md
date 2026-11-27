# Relatório individual

> Padrão da disciplina · Módulo REL · Incremento 3 · Revisado em 27/11/2026

## Resumo

O relatório individual mostra a cada usuário as reuniões em que a presença dele foi confirmada, com o tempo de participação registrado em cada uma e o total do período escolhido: todo o período, este mês ou este ano. Junto do histórico aparecem os vínculos atuais do usuário: as organizações aprovadas de que ele é membro aprovado, com o papel de administrador ou membro, e as comissões ativas dessas organizações em que ele está na equipe, com o papel de responsável ou membro. Os vínculos não dependem do período. O relatório pode ser exportado em PDF, com os vínculos e as reuniões, e em CSV, com uma linha por reunião e uma linha por vínculo. Cada pessoa vê só o próprio relatório; o servidor não grava nada nesta consulta.

## Escopo

- consultar as reuniões com presença confirmada, com entrada, saída, duração e resumo de minutos;
- filtrar o relatório por período (todo o período, este mês ou este ano), pela data de início da reunião;
- ver os vínculos atuais com organizações e comissões, com o papel em cada uma;
- exportar o relatório em PDF: no aplicativo instalado, pelo menu de compartilhamento do aparelho; no Expo Go e na web, pela tela de impressão;
- exportar o relatório em CSV, com as reuniões e os vínculos.

Ficam fora deste módulo o registro da presença e da saída, inclusive a saída automática no término da reunião, que estão em [Reuniões](../../3%20-%20Incremento%202/reunioes/README.md), e a gestão dos vínculos, que está em [Cadastro de organização](../../2%20-%20Incremento%201/cadastro-organizacao/README.md) e [Cadastro de comissão](../../2%20-%20Incremento%201/cadastro-comissao/README.md). Este módulo só lê esses dados.

## Atores

| Ator          | Quem é no sistema                                                                 | O que faz neste módulo                                                                 |
| ------------- | --------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------- |
| Usuário comum | Conta ativa do tipo `USUARIO`, autenticada no aplicativo.                          | Consulta e filtra o próprio relatório, vê os próprios vínculos e exporta em PDF e CSV. |

O administrador do sistema (`ADMIN_SISTEMA`) não é ator deste módulo: o aplicativo não mostra a aba "Relatório" para ele, e a API recusa a consulta com 403.

## Índice padronizado

|  Nº | Artefato                                                                               | Finalidade                                              |
| --: | -------------------------------------------------------------------------------------- | ------------------------------------------------------- |
|   1 | [Casos de uso](1%20-%20casos-uso-relatorio-individual.md)                              | Diagrama, ator e requisitos de usuário.                 |
|   2 | [Requisitos funcionais](2%20-%20requisitos-funcionais-relatorio-individual.md)         | Cada caso de uso no formulário padrão.                  |
|   3 | [Diagrama de classes](3%20-%20diagrama-classes-relatorio-individual.md)                | Tela, módulos do aplicativo, API e modelos lidos.       |
|   3 | [Dicionário de classes](3%20-%20dicionario-dados-classes-relatorio-individual.md)      | Atributos, tipos e operações de cada classe.            |
|   4 | [Diagrama relacional](4%20-%20diagrama-relacional-relatorio-individual.md)             | Visão lógica dos dados lidos no MongoDB.                |
|   4 | [Dicionário relacional](4%20-%20dicionario-dados-relacional-relatorio-individual.md)   | Campo a campo: tipo, nulo, chaves e geração de valores. |
|   5 | [Requisitos não funcionais](5%20-%20requisitos-nao-funcionais-relatorio-individual.md) | Critérios de qualidade com medida e verificação.        |
|   6 | [Diagramas de sequência](6%20-%20diagramas-sequencia-relatorio-individual.md)          | Mensagens entre tela, API, serviço e banco.             |
|   7 | [Testes](7%20-%20testes-relatorio-individual.md)                                       | Testes automatizados e pendências de execução manual.   |

## Rastreabilidade

| Caso de uso | Requisito | Sequência | Testes                                                                                      |
| ----------- | --------- | --------- | ------------------------------------------------------------------------------------------- |
| UC-REL-01   | RF-REL-01 | SD-REL-01 | CT-REL-01, CT-REL-02, CT-REL-03, CT-REL-07, CT-REL-17, CT-REL-23, CT-REL-27, CT-REL-34, CT-REL-36, CT-REL-38 |
| UC-REL-02   | RF-REL-02 | SD-REL-02 | CT-REL-04, CT-REL-11, CT-REL-29, CT-REL-34, CT-REL-36                                       |
| UC-REL-03   | RF-REL-03 | SD-REL-03 | CT-REL-05, CT-REL-12, CT-REL-13, CT-REL-23, CT-REL-28, CT-REL-34, CT-REL-36, CT-REL-38      |
| UC-REL-04   | RF-REL-04 | SD-REL-04 | CT-REL-08, CT-REL-12, CT-REL-13, CT-REL-14, CT-REL-18, CT-REL-19, CT-REL-21, CT-REL-22, CT-REL-26, CT-REL-28, CT-REL-30, CT-REL-34, CT-REL-35, CT-REL-36, CT-REL-38 |
| UC-REL-05   | RF-REL-05 | SD-REL-05 | CT-REL-09, CT-REL-10, CT-REL-12, CT-REL-13, CT-REL-14, CT-REL-16, CT-REL-20, CT-REL-21, CT-REL-25, CT-REL-34, CT-REL-35, CT-REL-36 |

Os requisitos não funcionais RNF-REL-01 a RNF-REL-13 e os testes que os verificam estão em [Requisitos não funcionais](5%20-%20requisitos-nao-funcionais-relatorio-individual.md) e [Testes](7%20-%20testes-relatorio-individual.md).

## Base de conferência

BACKEND e FRONTEND na branch `homolog`, conferidos em 27/11/2026: backend no commit `d1f1aeb` e frontend no commit `10abe49`. Os diagramas de classes e de sequência foram revistos no mesmo dia sobre o backend `6e0d3f5` e o frontend `8cf4839`; esses commits só mudam o `.env.example`, o cartão da votação e as funções `entregarPdf` e `nomearPdf`, que o relatório não usa. A versão atual do relatório entrou no backend `99b6b4f` e no frontend `a53acce`; nos commits seguintes, a única mudança que afeta o relatório é a do `JwtAuthGuard` (backend `543e03f`), que passou a recusar o token emitido antes da última troca de senha. No frontend, `pdf.ts` mudou só na função `entregarPdf`, que o relatório não usa, e `api.ts` ganhou só tipos da votação. Os arquivos principais são `BACKEND/src/relatorios/relatorios.controller.ts`, `relatorios.service.ts`, `relatorio-individual.dto.ts` e `relatorios.module.ts`; `FRONTEND/src/app/(abas)/relatorio.tsx`, `src/services/exportar-relatorio-individual.ts`, `src/services/pdf.ts` (só a função `pdfPelaImpressao`), `src/utils/relatorio-individual.ts` e `src/types/relatorio-individual.ts`. O registro de verificação da equipe está em `FRONTEND/tests/RELATORIO.md`.

As duas observações da avaliação do Incremento 3 (17/11/2026) sobre o relatório são atendidas no código: as organizações e comissões do usuário aparecem na tela, no PDF e no CSV (RF-REL-03); e o PDF do aplicativo instalado é compartilhado direto do arquivo gerado pela impressão, sem a chamada `FileSystemFile.move` apontada no erro da avaliação (RF-REL-04, CT-REL-18). No aparelho, a exportação do PDF foi confirmada num Galaxy S24 com Android 16 (CT-REL-38); o restante do roteiro no aparelho segue pendente (CT-REL-34).
