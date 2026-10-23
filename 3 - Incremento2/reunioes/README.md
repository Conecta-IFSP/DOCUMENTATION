# Documentação - Reuniões

Esta pasta reúne a documentação do incremento de reuniões das comissões do projeto Conecta+.

## Documentos

| Item | Documento | Descrição |
|---|---|---|
| 1 | [Diagrama de Casos de Uso](1%20-%20casos-uso-reunioes.md) | Apresenta os atores e os 9 casos de uso do incremento. |
| 2 | [Requisitos Funcionais](2%20-%20requisitos-funcionais-reunioes.md) | Descreve cada caso de uso como requisito funcional, de RF-01 a RF-09, no formulário padrão. |
| 3 | [Diagrama de Classes](3%20-%20diagrama-classes-reunioes.md) | Representa as classes de dados, do servidor e do aplicativo e seus relacionamentos. |
| 3 | [Dicionário de Dados - Classes](3%20-%20dicionario-dados-classes-reunioes.md) | Descreve o que cada classe representa, seus atributos, tipos e regras. |
| 4 | [Diagrama Relacional](4%20-%20diagrama-relacional-reunioes.md) | Apresenta a visão lógica relacional dos dados das reuniões. |
| 4 | [Dicionário de Dados - Modelo Relacional](4%20-%20dicionario-dados-relacional-reunioes.md) | Descreve cada tabela e campo, as chaves e as restrições. |
| 5 | [Requisitos Não Funcionais](5%20-%20requisitos-nao-funcionais-reunioes.md) | Define os requisitos RNF-01 a RNF-10, com critério de aceitação e forma de verificação. |
| 6 | [Diagramas de Sequência](6%20-%20diagramas-sequencia-reunioes.md) | Contém um diagrama de sequência para cada requisito funcional, de RF-01 a RF-09. |
| 7 | [Testes Realizados](7%20-%20testes-reunioes.md) | Registra os casos de teste CT-01 a CT-38 e seus resultados. |

## Escopo do Incremento

A documentação contempla os seguintes fluxos:

- agendamento de reunião pelo responsável da comissão, com convite a toda a comissão ou a integrantes escolhidos;
- consulta da agenda de reuniões e das reuniões de cada comissão;
- resposta ao convite;
- registro de entrada e saída pelo participante e validação da presença pelo organizador;
- geração da lista de presença;
- cancelamento de reunião.

## Rastreabilidade

| Caso de uso | Requisito | Diagrama de sequência | Casos de teste |
|---|---|---|---|
| UC01 - Agendar reunião | RF-01 | RF-01 | CT-01 a CT-09, CT-24 a CT-28, CT-30, CT-38 |
| UC02 - Consultar agenda de reuniões | RF-02 | RF-02 | CT-10, CT-31 |
| UC03 - Visualizar reunião | RF-03 | RF-03 | CT-11, CT-28 |
| UC04 - Cancelar reunião | RF-04 | RF-04 | CT-12, CT-13, CT-36 |
| UC05 - Responder convite | RF-05 | RF-05 | CT-14, CT-32 |
| UC06 - Registrar entrada | RF-06 | RF-06 | CT-15 a CT-18, CT-33 |
| UC07 - Registrar saída | RF-07 | RF-07 | CT-19, CT-33 |
| UC08 - Confirmar ou recusar presença | RF-08 | RF-08 | CT-20, CT-21, CT-34 |
| UC09 - Gerar lista de presença | RF-09 | RF-09 | CT-22, CT-23, CT-28, CT-35 |

## Repositórios

- [Backend](https://github.com/Conecta-IFSP/BACKEND)
- [Frontend](https://github.com/Conecta-IFSP/FRONTEND)
