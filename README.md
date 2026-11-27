# Conecta+ — documentação do projeto

Este repositório reúne a especificação funcional, técnica e de testes do **Conecta+**, aplicação acadêmica para gestão de organizações, comissões e reuniões.

## Adendo: diagramas divididos em partes

Na revisão de 27/11/2026, os diagramas de classes e os diagramas de sequência do Incremento 1 e do Incremento 2 foram divididos em partes por causa do tamanho. Inteiros, eles ficavam com 11% a 60% do tamanho na largura da página e não dava para ler; em partes, cada um aparece com pelo menos 80% do tamanho. O conteúdo não mudou: toda classe, ligação, mensagem e nota continua no diagrama, numa das partes.

- **Diagramas de classes:** as partes seguem as camadas (telas, serviços do aplicativo, controller, serviço, schemas, models e bibliotecas). Uma classe detalhada em outra parte aparece só com o nome, e o texto abaixo da figura diz onde ela está.
- **Diagramas de sequência:** as partes seguem as camadas (no aplicativo, na API e nos dados), e os diagramas mais largos têm mais de uma parte na mesma camada. A numeração das mensagens continua de uma parte para a outra, e uma nota como "Mensagens 3 a 15 na parte 2" indica onde está o trecho que falta. Abaixo de cada parte, o texto "Ligação com as outras partes" diz quais mensagens ela tem, em que parte estão as outras e quais linhas de vida ligam as partes.

| Módulo | Diagrama de classes | Diagramas de sequência |
| --- | --- | --- |
| [Cadastro de usuário](2%20-%20Incremento%201/cadastro-usuario/README.md) | [11 partes](2%20-%20Incremento%201/cadastro-usuario/3%20-%20diagrama-classes-cadastro-usuario.md) | [4 SDs em 21 partes](2%20-%20Incremento%201/cadastro-usuario/6%20-%20diagramas-sequencia-cadastro-usuario.md) |
| [Cadastro de organização](2%20-%20Incremento%201/cadastro-organizacao/README.md) | [8 partes](2%20-%20Incremento%201/cadastro-organizacao/3%20-%20diagrama-classes-cadastro-organizacao.md) | [14 SDs em 47 partes](2%20-%20Incremento%201/cadastro-organizacao/6%20-%20diagramas-sequencia-cadastro-organizacao.md) |
| [Cadastro de comissão](2%20-%20Incremento%201/cadastro-comissao/README.md) | [7 partes](2%20-%20Incremento%201/cadastro-comissao/3%20-%20diagrama-classes-cadastro-comissao.md) | [9 SDs em 28 partes](2%20-%20Incremento%201/cadastro-comissao/6%20-%20diagramas-sequencia-cadastro-comissao.md) |
| [Configurações de usuário](2%20-%20Incremento%201/configuracoes-usuario/README.md) | [13 partes](2%20-%20Incremento%201/configuracoes-usuario/3%20-%20diagrama-classes-configuracoes-usuario.md) | [6 SDs em 25 partes](2%20-%20Incremento%201/configuracoes-usuario/6%20-%20diagramas-sequencia-configuracoes-usuario.md) |
| [Informações do sistema](2%20-%20Incremento%201/informacao-sistemas/README.md) | [13 partes](2%20-%20Incremento%201/informacao-sistemas/3%20-%20diagrama-classes-informacao-sistemas.md) | [5 SDs em 16 partes](2%20-%20Incremento%201/informacao-sistemas/6%20-%20diagramas-sequencia-informacao-sistemas.md) |
| [Reuniões](3%20-%20Incremento%202/reunioes/README.md) | [25 partes](3%20-%20Incremento%202/reunioes/3%20-%20diagrama-classes-reunioes.md) | [19 SDs em 76 partes](3%20-%20Incremento%202/reunioes/6%20-%20diagramas-sequencia-reunioes.md) |

Os diagramas do Incremento 3 (atas, votação, relatório individual e manual) não foram divididos.

## Base usada nesta revisão

| Repositório     | Referência auditada                                     |
| --------------- | --------------------------------------------------------- |
| `FRONTEND`      | branch `main`, versão de 27/11/2026                        |
| `BACKEND`       | branch `main`, versão de 27/11/2026                        |
| `DOCUMENTATION` | documentação publicada                                    |

Esta documentação descreve o comportamento implementado nessa base, conferido em 27/11/2026. Nos dois repositórios, a branch de trabalho e de entrega é a `main`. Alterações no código devem atualizar, na mesma entrega, os requisitos, diagramas, dicionários e testes afetados.

## Leitura recomendada

1. [Padrão da documentação](PADRAO-DOCUMENTACAO.md): nomenclatura, conteúdo obrigatório e critérios de qualidade.
2. [Arquitetura e tecnologias](ARQUITETURA-E-TECNOLOGIAS.md): componentes, segurança, bibliotecas e versões exatas.
3. [Catálogo da API e regras](CATALOGO-API-E-REGRAS.md): rotas, guards, corpos e regras de negócio de cada controller.
4. Documentos do incremento correspondente.
5. [Tratamento das avaliações](TRATAMENTO-AVALIACAO.md): o que foi feito para cada observação do professor e o que ainda falta.
6. [Catálogo de softwares e licenças](CATALOGO-SOFTWARES-E-LICENCAS.md): bibliotecas, serviços e ferramentas usados e suas licenças.

## Estrutura

```text
DOCUMENTATION/
├── README.md
├── PADRAO-DOCUMENTACAO.md
├── ARQUITETURA-E-TECNOLOGIAS.md
├── CATALOGO-API-E-REGRAS.md
├── CATALOGO-SOFTWARES-E-LICENCAS.md
├── TRATAMENTO-AVALIACAO.md
├── licencas/
├── 1 - Proposta Inicial/
├── 2 - Incremento 1/
│   ├── cadastro-usuario/
│   ├── cadastro-organizacao/
│   ├── cadastro-comissao/
│   ├── configuracoes-usuario/
│   └── informacao-sistemas/
├── 3 - Incremento 2/
│   └── reunioes/
└── 4 - Incremento 3/
    ├── atas/
    ├── manual-de-utilizacao/
    ├── relatorio-individual/
    └── votacao/
```

Cada módulo deve possuir: visão geral, casos de uso com o diagrama em SVG, requisitos funcionais, diagrama e dicionário de classes, diagrama e dicionário relacional, requisitos não funcionais, diagramas de sequência e testes.

### Proposta Inicial

- [Objetivos da proposta](1%20-%20Proposta%20Inicial/objetivos.md), com a situação na entrega final.
- [Pesquisa de mercado](1%20-%20Proposta%20Inicial/pesquisa-de-mercado.md): concorrentes e validação com o público.

### Incremento 1

- [Cadastro de comissão](2%20-%20Incremento%201/cadastro-comissao/README.md)
- [Cadastro de organização](2%20-%20Incremento%201/cadastro-organizacao/README.md)
- [Cadastro de usuário](2%20-%20Incremento%201/cadastro-usuario/README.md)
- [Configurações de usuário](2%20-%20Incremento%201/configuracoes-usuario/README.md)
- [Informações do sistema](2%20-%20Incremento%201/informacao-sistemas/README.md)

Cada pasta do Incremento 1 contém os mesmos dez documentos e o SVG do diagrama de casos de uso, e segue o modelo definido em [Padrão da documentação](PADRAO-DOCUMENTACAO.md).

### Incremento 2

- [Reuniões](3%20-%20Incremento%202/reunioes/README.md): agendamento, edição e cancelamento, convites, entrada e saída (com saída automática no término), link de presença, correção de horários, lista de presença em PDF e anexos da pauta.

### Incremento 3

- [Atas de reunião](4%20-%20Incremento%203/atas/README.md): redação, envio com prazo, aprovação (automática no fim do prazo para quem não respondeu) e reprovação, versões e PDF.
- [Manual de utilização](4%20-%20Incremento%203/manual-de-utilizacao/README.md): tela Ajuda com perguntas frequentes e manual completo em PDF.
- [Relatório individual](4%20-%20Incremento%203/relatorio-individual/README.md): reuniões com presença confirmada por período, vínculos atuais com organizações e comissões, e exportação em PDF e CSV.
- [Votação na reunião](4%20-%20Incremento%203/votacao/README.md): abertura e encerramento pelo presidente, que escolhe se a votação é fechada (ninguém vê quem votou em quê) ou aberta (todos veem depois do encerramento), voto único dos presentes numa opção, em branco ou nulo, e resultado.

## Licença

O Conecta+ é distribuído sob a licença MIT, com o aviso `Copyright (c) 2026 Equipe Conecta+`, no código e na documentação. O texto está em [LICENSE](LICENSE), e as licenças das bibliotecas e ferramentas usadas estão no [Catálogo de softwares e licenças](CATALOGO-SOFTWARES-E-LICENCAS.md).

## Repositórios relacionados

- [Frontend](https://github.com/Conecta-IFSP/FRONTEND)
- [Backend](https://github.com/Conecta-IFSP/BACKEND)
- [Organização no GitHub](https://github.com/Conecta-IFSP)

## Projeto acadêmico

Projeto desenvolvido no **Instituto Federal de São Paulo — IFSP**.
