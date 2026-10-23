# Documentação - Cadastro de Usuário

> **Projeto:** Conecta+  
> **Incremento:** Cadastro de Usuário, Solicitação de Acesso e Gestão de Membros

Esta pasta reúne a documentação referente ao incremento de cadastro e gestão de usuários do projeto Conecta+.

## Documentos

| Item | Documento | Descrição |
|---|---|---|
| 1 | [Diagrama de Casos de Uso](1%20-%20casos-uso-cadastro-usuario.md) | Cadastro local, autenticação, entrada, convite e gestão de vínculos. |
| 2 | [Requisitos Funcionais](2%20-%20requisitos-funcionais-cadastro-usuario.md) | Descreve RF-USR-01 a RF-USR-07 conforme a branch `homolog`. |
| 3 | [Diagrama de Classes](3%20-%20diagrama-classes-cadastro-usuario.md) | Representa as principais classes e relacionamentos envolvidos no incremento. |
| 3 | [Dicionário de Dados - Classes](3%20-%20dicionario-dados-cadastro-usuario.md) | Detalha os atributos, tipos e regras das classes apresentadas no diagrama. |
| 4 | [Diagrama Relacional](4%20-%20diagrama-relacional-cadastro-usuario.md) | Apresenta a visão lógica relacional dos dados utilizados pelo incremento. |
| 4 | [Dicionário de Dados - Modelo Relacional](4%20-%20dicionario-dados-relacional-cadastro-usuario.md) | Detalha as entidades, campos, chaves e relacionamentos do modelo lógico. |
| 5 | [Requisitos Não Funcionais](5%20-%20requisitos-nao-funcionais-cadastro-usuario.md) | Define requisitos de desempenho, segurança, confiabilidade, usabilidade, acessibilidade e manutenibilidade. |
| 6 | [Diagramas de Sequência](6%20-%20diagramas-sequencia-cadastro-usuario.md) | Contém sequências com controladores, serviços, banco e provedores reais. |
| 7 | [Testes](7%20-%20testes-cadastro-usuario.md) | Evidência automatizada e roteiro manual. |

## Escopo do Incremento

A documentação contempla os seguintes fluxos:

- cadastro de usuário na plataforma;
- confirmação de e-mail e login local ou Google;
- entrada em organização por código e resposta a convite;
- aprovação ou rejeição de solicitações;
- promoção, rebaixamento e remoção de membros.

## Rastreabilidade

Os casos de uso, requisitos funcionais e diagramas de sequência possuem correspondência direta:

```text
UC-USR-01..07 ↔ RF-USR-01..07 ↔ SD-USR-01..07 ↔ CT-USR
```

Base de conferência: FRONTEND `homolog` (`792fab4`) e BACKEND `homolog` (`6f2ffc4`), em 23/10/2026.
