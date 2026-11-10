# Atas de reunião

> Padrão da disciplina · Módulo ATA · Incremento 3 · Revisado em 10/11/2026

## Resumo

Depois que a reunião termina, o presidente da comissão escreve a ata e a envia aos participantes que tiveram presença confirmada. Cada um deles aprova ou reprova o texto. A ata só fica aprovada quando todos os presentes aprovam; nesse momento ela recebe um número na comissão, por exemplo "Ata nº 2/2026", e pode ser baixada em PDF. Uma reprovação devolve a ata ao presidente, que a corrige numa nova versão. A versão reprovada fica guardada como cancelada.

## Escopo

- escrever e salvar o rascunho da ata;
- enviar a ata para aprovação, com prazo de resposta;
- aprovar a ata, com confirmação pela biometria ou PIN do aparelho;
- reprovar a ata, apontando um parágrafo e uma justificativa, ambos opcionais;
- criar uma nova versão a partir da ata reprovada;
- consultar a ata, as mudanças entre versões e as versões anteriores;
- ver na tela inicial as atas que esperam uma ação do usuário;
- gerar o PDF da ata aprovada.

Ficam fora deste módulo o agendamento, os convites e a presença, que estão em [Reuniões](../../3%20-%20Incremento2/reunioes/README.md). A presença confirmada é a entrada deste módulo: ela define quem aprova a ata.

## Atores

| Ator                   | Quem é no sistema                                                                                      | O que faz neste módulo                                                       |
| ---------------------- | ------------------------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------- |
| Participante da reunião | Usuário que está na lista de participantes da reunião, com ou sem presença.                            | Consulta a ata, vê suas pendências e baixa o PDF da ata aprovada.            |
| Participante presente  | Participante cuja presença ficou `CONFIRMADA`. É uma especialização do participante da reunião.         | Aprova ou reprova a ata enviada.                                             |
| Presidente da comissão | Membro da comissão com papel `RESPONSAVEL`. É uma especialização do participante da reunião.            | Escreve, salva e envia a ata; cria a nova versão quando ela é reprovada.     |

O presidente que esteve presente também é um participante presente: ao enviar a ata, a aprovação dele já é registrada.

## Índice padronizado

|  Nº | Artefato                                                               | Finalidade                                              |
| --: | ---------------------------------------------------------------------- | ------------------------------------------------------- |
|   1 | [Casos de uso](1%20-%20casos-uso-atas.md)                              | Diagrama, atores e requisitos de usuário.               |
|   2 | [Requisitos funcionais](2%20-%20requisitos-funcionais-atas.md)         | Cada caso de uso no formulário padrão.                  |
|   3 | [Diagrama de classes](3%20-%20diagrama-classes-atas.md)                | Telas, API, serviço e modelo envolvidos.                |
|   3 | [Dicionário de classes](3%20-%20dicionario-dados-classes-atas.md)      | Atributos, tipos e operações de cada classe.            |
|   4 | [Diagrama relacional](4%20-%20diagrama-relacional-atas.md)             | Visão lógica dos dados da ata no MongoDB.               |
|   4 | [Dicionário relacional](4%20-%20dicionario-dados-relacional-atas.md)   | Campo a campo: tipo, nulo, chaves e geração de valores. |
|   5 | [Requisitos não funcionais](5%20-%20requisitos-nao-funcionais-atas.md) | Critérios de qualidade com medida e verificação.        |
|   6 | [Diagramas de sequência](6%20-%20diagramas-sequencia-atas.md)          | Mensagens entre tela, API, serviço e banco.             |
|   7 | [Testes](7%20-%20testes-atas.md)                                       | Testes automatizados e roteiros executados.             |

## Rastreabilidade

| Caso de uso | Requisito  | Sequência  | Testes                        |
| ----------- | ---------- | ---------- | ----------------------------- |
| UC-ATA-01   | RF-ATA-01  | SD-ATA-01  | CT-ATA-01, CT-ATA-02, CT-ATA-03, CT-ATA-14 |
| UC-ATA-02   | RF-ATA-02  | SD-ATA-02  | CT-ATA-04, CT-ATA-10, CT-ATA-14 |
| UC-ATA-03   | RF-ATA-03  | SD-ATA-03  | CT-ATA-05, CT-ATA-07, CT-ATA-15, CT-ATA-17 |
| UC-ATA-04   | RF-ATA-04  | SD-ATA-04  | CT-ATA-06, CT-ATA-07, CT-ATA-10, CT-ATA-15, CT-ATA-18 |
| UC-ATA-05   | RF-ATA-05  | SD-ATA-05  | CT-ATA-08, CT-ATA-16          |
| UC-ATA-06   | RF-ATA-06  | SD-ATA-06  | CT-ATA-02, CT-ATA-16, CT-ATA-19 |
| UC-ATA-07   | RF-ATA-07  | SD-ATA-07  | CT-ATA-09, CT-ATA-15          |
| UC-ATA-08   | RF-ATA-08  | SD-ATA-08  | CT-ATA-17                     |

## Base de conferência

BACKEND e FRONTEND na branch `homolog`, com as alterações do Incremento 3, conferidos em 09/11/2026 (backend sobre `6f2ffc4`, frontend sobre `9cb23ad`). Os arquivos principais são `BACKEND/src/reunioes/reunioes.service.ts`, `BACKEND/src/reunioes/schemas/reuniao.schema.ts`, `BACKEND/src/reunioes/schemas/contador.schema.ts`, `FRONTEND/src/app/reunioes/registros.tsx`, `FRONTEND/src/services/pdfAta.ts` e `FRONTEND/src/utils/diferenca.ts`.
