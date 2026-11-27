# Conecta+ — documentação do projeto

Este repositório reúne a especificação funcional, técnica e de testes do **Conecta+**, aplicação acadêmica para gestão de organizações, comissões e reuniões.

## Base usada nesta revisão

| Repositório     | Referência auditada                                     |
| --------------- | --------------------------------------------------------- |
| `FRONTEND`      | branch `homolog`, commit `4f53a25`                         |
| `BACKEND`       | branch `homolog`, commit `6e0d3f5`                         |
| `DOCUMENTATION` | documentação publicada                                    |

Esta documentação descreve o comportamento implementado nessa base, conferido em 27/11/2026. Alterações no código devem atualizar, na mesma entrega, os requisitos, diagramas, dicionários e testes afetados.

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

## Repositórios relacionados

- [Frontend](https://github.com/Conecta-IFSP/FRONTEND)
- [Backend](https://github.com/Conecta-IFSP/BACKEND)
- [Organização no GitHub](https://github.com/Conecta-IFSP)

## Projeto acadêmico

Projeto desenvolvido no **Instituto Federal de São Paulo — IFSP**.
