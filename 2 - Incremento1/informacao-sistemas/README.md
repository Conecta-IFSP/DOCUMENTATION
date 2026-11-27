# Informações do sistema

> Padrão da disciplina · Módulo SIS · Incremento 1 · Revisado em 27/11/2026

## Resumo

O administrador do sistema tem uma tela própria, chamada "Sistema", aberta pelo item "Sistema" da tela inicial, que tem a descrição "Informações do sistema, status, relatórios e reset.". A tela reúne cinco cartões. "Sobre o sistema" mostra o nome, as versões do aplicativo e da API, o ambiente, quando a API foi iniciada e o fuso horário. "Tecnologias" lista as bibliotecas do servidor e do aplicativo, com as versões. "Servidor" mostra o sistema operacional, o processador, a CPU e a memória em uso e há quanto tempo o computador e a API estão ligados. "Banco de dados" diz se o MongoDB respondeu, em quanto tempo e quanto espaço ocupa. "Relatórios" traz os totais de usuários, organizações, comissões, reuniões e atas, por situação. A mesma tela tem dois botões que alteram o banco: "Popular dados de teste", que cria três contas, uma organização aprovada e uma comissão depois que o administrador toca em "Confirmar" num diálogo, e "Resetar sistema", que apaga os dados de uso e mantém só as contas de administrador do sistema. Para resetar, o administrador digita "RESET" e a própria senha num cartão de confirmação; a API confere as duas coisas, e a senha é comparada com o hash guardado na conta.

## Escopo

- consultar as informações sobre o sistema: versões, tecnologias e dados fixos do servidor;
- consultar o status do servidor: CPU e memória em uso, tempo ativo do computador e da API, e a situação do banco de dados;
- consultar os totais de usuários, organizações, comissões, reuniões e atas, por situação;
- criar os dados de teste definidos na configuração do servidor;
- apagar os dados de uso, preservando as contas de administrador do sistema, depois de digitar "RESET" e a senha do administrador.

Limitações conhecidas, descritas no campo de cada requisito em [Requisitos funcionais](2%20-%20requisitos-funcionais-informacao-sistemas.md): o reset não usa transação, não há limite de tentativas de senha no reset, e as versões das bibliotecas mostradas são as declaradas no `package.json`, não necessariamente as instaladas.

O cadastro e a troca de tipo das contas, inclusive a criação do primeiro administrador do sistema, estão em [Cadastro de usuário](../cadastro-usuario/README.md). A aprovação de organizações, também feita pelo administrador do sistema, está em [Cadastro de organização](../cadastro-organizacao/README.md).

## Atores

| Ator                     | Quem é no sistema                                                        | O que faz neste módulo                                                                         |
| ------------------------ | ------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------- |
| Administrador do sistema | Usuário ativo cujo campo `tipo` é `ADMIN_SISTEMA` na coleção `usuarios`. | Consulta as informações, o status e os relatórios, cria os dados de teste e reseta o sistema. |

O usuário comum (`tipo` igual a `USUARIO`) não vê o item "Sistema", não consegue abrir a tela e recebe recusa da API em todas as rotas deste módulo.

## Índice padronizado

|  Nº | Artefato                                                                              | Finalidade                                               |
| --: | ------------------------------------------------------------------------------------- | -------------------------------------------------------- |
|   1 | [Casos de uso](1%20-%20casos-uso-informacao-sistemas.md)                              | Diagrama, ator e requisitos de usuário.                  |
|   2 | [Requisitos funcionais](2%20-%20requisitos-funcionais-informacao-sistemas.md)         | Cada caso de uso no formulário padrão.                   |
|   3 | [Diagrama de classes](3%20-%20diagrama-classes-informacao-sistemas.md)                | Telas, guards, controller, serviço e modelos envolvidos. |
|   3 | [Dicionário de classes](3%20-%20dicionario-dados-classes-informacao-sistemas.md)      | Atributos, tipos e operações de cada classe.             |
|   4 | [Diagrama relacional](4%20-%20diagrama-relacional-informacao-sistemas.md)             | Visão lógica das coleções contadas, criadas e apagadas.  |
|   4 | [Dicionário relacional](4%20-%20dicionario-dados-relacional-informacao-sistemas.md)   | Campo a campo: tipo, nulo, chaves e geração de valores.  |
|   5 | [Requisitos não funcionais](5%20-%20requisitos-nao-funcionais-informacao-sistemas.md) | Critérios de qualidade com medida e verificação.         |
|   6 | [Diagramas de sequência](6%20-%20diagramas-sequencia-informacao-sistemas.md)          | Mensagens entre tela, API, guards, serviço e banco.      |
|   7 | [Testes](7%20-%20testes-informacao-sistemas.md)                                       | Testes automatizados e roteiros executados.              |

## Rastreabilidade

| Caso de uso | Requisito | Sequência | Testes                                                                                                          |
| ----------- | --------- | --------- | --------------------------------------------------------------------------------------------------------------- |
| UC-SIS-01   | RF-SIS-01 | SD-SIS-01 | CT-SIS-01, CT-SIS-06, CT-SIS-08, CT-SIS-09, CT-SIS-16                                                           |
| UC-SIS-02   | RF-SIS-02 | SD-SIS-02 | CT-SIS-02, CT-SIS-06, CT-SIS-08, CT-SIS-09, CT-SIS-16                                                           |
| UC-SIS-03   | RF-SIS-03 | SD-SIS-03 | CT-SIS-03, CT-SIS-06, CT-SIS-08, CT-SIS-14, CT-SIS-18                                                           |
| UC-SIS-04   | RF-SIS-04 | SD-SIS-04 | CT-SIS-04, CT-SIS-05, CT-SIS-06, CT-SIS-07, CT-SIS-08, CT-SIS-10 a CT-SIS-13, CT-SIS-15, CT-SIS-17, CT-SIS-19 |
| UC-SIS-05   | RF-SIS-05 | SD-SIS-05 | CT-SIS-06, CT-SIS-08, CT-SIS-09, CT-SIS-16                                                                      |

CT-SIS-01 a CT-SIS-05 e CT-SIS-12 a CT-SIS-14 foram executados em versões da tela e da API anteriores à atual; o registro de cada um continua em [Testes](7%20-%20testes-informacao-sistemas.md), marcado como de versão substituída.

## Base de conferência

BACKEND e FRONTEND na branch `homolog`, conferidos em 27/11/2026, com a versão de Informações do Sistema de Ruan Frazão: backend `931c2c8` e frontend `795d49b`, integrados na `homolog` (backend `d3b9ae3`, frontend `3842cd7`), com ajustes locais ainda sem commit que mantêm exatamente essa versão. Ela substituiu a versão anterior (backend `811cf02` e frontend `0666ee0`), em que o reset pedia a palavra "RESETAR" numa folha inferior e a biometria do aparelho, e a tela não tinha o cartão "Sobre o sistema". Os arquivos principais são `BACKEND/src/sistema/sistema.controller.ts`, `BACKEND/src/sistema/sistema.service.ts`, `BACKEND/src/sistema/dto/resetar-sistema.dto.ts`, `BACKEND/src/sistema/sistema.spec.ts`, `BACKEND/src/auth/admin-sistema.guard.ts`, `BACKEND/src/auth/jwt-auth.guard.ts`, `BACKEND/src/auth/api-key.guard.ts`, `FRONTEND/src/app/sistema.tsx`, `FRONTEND/src/app/(abas)/inicio.tsx`, `FRONTEND/src/app/_layout.tsx`, `FRONTEND/src/components/Confirmacao.tsx`, `FRONTEND/src/contexts/ToastContext.tsx` e `FRONTEND/src/utils/versao.ts`.
