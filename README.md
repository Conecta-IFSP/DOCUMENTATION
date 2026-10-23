# Conecta+ — documentação do projeto

Este repositório reúne a especificação funcional, técnica e de testes do **Conecta+**, aplicação acadêmica para gestão de organizações, comissões e reuniões.

## Base usada nesta revisão

| Repositório     | Referência auditada    |
| --------------- | ---------------------- |
| `FRONTEND`      | commit `792fab4`       |
| `BACKEND`       | commit `6f2ffc4`       |
| `DOCUMENTATION` | documentação publicada |

Esta documentação descreve o comportamento implementado nesses commits. Alterações posteriores no código devem atualizar também os requisitos, diagramas, dicionários e testes afetados.

## Leitura recomendada

1. [Padrão da documentação](PADRAO-DOCUMENTACAO.md): nomenclatura, conteúdo obrigatório e critérios de qualidade.
2. [Arquitetura e tecnologias](ARQUITETURA-E-TECNOLOGIAS.md): componentes, segurança, bibliotecas e versões exatas.
3. Documentos do incremento correspondente.

## Estrutura

```text
DOCUMENTATION/
├── PADRAO-DOCUMENTACAO.md
├── ARQUITETURA-E-TECNOLOGIAS.md
├── 1 - PropostaInicial/
├── 2 - Incremento1/
│   ├── cadastro-usuario/
│   ├── cadastro-organizacao/
│   ├── cadastro-comissao/
│   ├── configuracoes-usuario/
│   └── informacao-sistemas/
└── 3 - Incremento2/
    └── reunioes/
```

Cada módulo deve possuir: visão geral, casos de uso, requisitos funcionais, diagrama e dicionário de classes, diagrama e dicionário relacional, requisitos não funcionais, diagramas de sequência e testes.

### Incremento 1

- [Cadastro de comissão](2%20-%20Incremento1/cadastro-comissao/README.md)
- [Cadastro de organização](2%20-%20Incremento1/cadastro-organizacao/README.md)
- [Cadastro de usuário](2%20-%20Incremento1/cadastro-usuario/README.md)
- [Configurações de usuário](2%20-%20Incremento1/configuracoes-usuario/README.md)
- [Informações do sistema](2%20-%20Incremento1/informacao-sistemas/README.md)

Cada pasta do Incremento 1 contém exatamente os mesmos dez artefatos técnicos e segue o modelo definido em [Padrão da documentação](PADRAO-DOCUMENTACAO.md).

## Repositórios relacionados

- [Frontend](https://github.com/Conecta-IFSP/FRONTEND)
- [Backend](https://github.com/Conecta-IFSP/BACKEND)
- [Organização no GitHub](https://github.com/Conecta-IFSP)

## Projeto acadêmico

Projeto desenvolvido no **Instituto Federal de São Paulo — IFSP**.
