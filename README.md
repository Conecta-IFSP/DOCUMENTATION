# Conecta+ — documentação do projeto

Este repositório reúne a especificação funcional, técnica e de testes do **Conecta+**, aplicação acadêmica para gestão de organizações, comissões e reuniões.

## Base usada nesta revisão

| Repositório     | Branch de referência                 |                     Commit auditado |
| --------------- | ------------------------------------ | ----------------------------------: |
| `FRONTEND`      | `homolog`                            |                           `792fab4` |
| `BACKEND`       | `homolog`                            |                           `6f2ffc4` |
| `DOCUMENTATION` | `main` — não existe `homolog` remota | `84cc8ec` antes desta revisão local |

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

## Repositórios relacionados

- [Frontend](https://github.com/Conecta-IFSP/FRONTEND)
- [Backend](https://github.com/Conecta-IFSP/BACKEND)
- [Organização no GitHub](https://github.com/Conecta-IFSP)

## Projeto acadêmico

Projeto desenvolvido no **Instituto Federal de São Paulo — IFSP**.
