# Conecta+ — documentação do projeto

Este repositório reúne a especificação funcional, técnica e de testes do **Conecta+**, aplicação acadêmica para gestão de organizações, comissões e reuniões.

## Base usada nesta revisão

| Repositório     | Referência auditada                                     |
| --------------- | --------------------------------------------------------- |
| `FRONTEND`      | branch `homolog`, commit `8fc0a5a`                         |
| `BACKEND`       | branch `homolog`, commit `811cf02`                         |
| `DOCUMENTATION` | documentação publicada                                    |

Esta documentação descreve o comportamento implementado nessa base, conferido em 26/11/2026. Alterações no código devem atualizar, na mesma entrega, os requisitos, diagramas, dicionários e testes afetados.

## Leitura recomendada

1. [Padrão da documentação](PADRAO-DOCUMENTACAO.md): nomenclatura, conteúdo obrigatório e critérios de qualidade.
2. [Arquitetura e tecnologias](ARQUITETURA-E-TECNOLOGIAS.md): componentes, segurança, bibliotecas e versões exatas.
3. Documentos do incremento correspondente.
4. [Catálogo de softwares e licenças](CATALOGO-SOFTWARES-E-LICENCAS.md): bibliotecas, serviços e ferramentas usados e suas licenças.

## Estrutura

```text
DOCUMENTATION/
├── PADRAO-DOCUMENTACAO.md
├── ARQUITETURA-E-TECNOLOGIAS.md
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

Cada módulo deve possuir: visão geral, casos de uso, requisitos funcionais, diagrama e dicionário de classes, diagrama e dicionário relacional, requisitos não funcionais, diagramas de sequência e testes.

### Incremento 1

- [Cadastro de comissão](2%20-%20Incremento%201/cadastro-comissao/README.md)
- [Cadastro de organização](2%20-%20Incremento%201/cadastro-organizacao/README.md)
- [Cadastro de usuário](2%20-%20Incremento%201/cadastro-usuario/README.md)
- [Configurações de usuário](2%20-%20Incremento%201/configuracoes-usuario/README.md)
- [Informações do sistema](2%20-%20Incremento%201/informacao-sistemas/README.md)

Cada pasta do Incremento 1 contém exatamente os mesmos dez artefatos técnicos e segue o modelo definido em [Padrão da documentação](PADRAO-DOCUMENTACAO.md).

### Incremento 2

- [Reuniões](3%20-%20Incremento%202/reunioes/README.md): agendamento, convites, entrada e saída, lista de presença e anexos da pauta.

### Incremento 3

- [Atas de reunião](4%20-%20Incremento%203/atas/README.md): redação, envio com prazo, aprovação (automática no fim do prazo para quem não respondeu) e reprovação, versões e PDF.
- [Manual de utilização](4%20-%20Incremento%203/manual-de-utilizacao/README.md): tela Ajuda com perguntas frequentes e manual completo em PDF.
- [Relatório individual](4%20-%20Incremento%203/relatorio-individual/README.md): reuniões com presença confirmada por período, vínculos atuais com organizações e comissões, e exportação em PDF e CSV.
- [Votação na reunião](4%20-%20Incremento%203/votacao/README.md): abertura e encerramento pelo presidente, voto único e sigiloso dos presentes e resultado.

## Repositórios relacionados

- [Frontend](https://github.com/Conecta-IFSP/FRONTEND)
- [Backend](https://github.com/Conecta-IFSP/BACKEND)
- [Organização no GitHub](https://github.com/Conecta-IFSP)

## Projeto acadêmico

Projeto desenvolvido no **Instituto Federal de São Paulo — IFSP**.
