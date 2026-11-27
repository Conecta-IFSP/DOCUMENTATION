# Informações do sistema

> Padrão da disciplina · Módulo SIS · Incremento 1 · Revisado em 26/11/2026

## Resumo

O administrador do sistema tem uma tela própria, chamada "Sistema", aberta pelo item "Sistema" da tela inicial. Nela ele vê o estado do computador onde a API roda (memória em uso, carga da CPU, tempo ligado e se o banco de dados respondeu) e os totais de usuários, organizações e comissões. A mesma tela tem dois botões que alteram o banco: "Popular dados de teste", que cria três contas, uma organização aprovada e uma comissão para demonstração, e "Resetar sistema", que apaga os dados de uso e mantém apenas as contas de administrador do sistema. Os dados de teste só são criados depois que o administrador toca em "Confirmar" num diálogo. O reset pede mais: o administrador digita "RESETAR" numa folha que lista o que será apagado e confirma a identidade com a biometria ou o PIN do aparelho; a API também recusa o pedido que não traz essa palavra.

## Escopo

- consultar o status do servidor: memória, CPU, tempo ligado e banco de dados;
- consultar os totais de usuários, organizações e comissões, por situação;
- criar os dados de teste definidos na configuração do servidor;
- apagar os dados de uso, preservando as contas de administrador do sistema, depois de digitar "RESETAR" e confirmar a identidade no aparelho.

Não implementados e limitações conhecidas:

- **reinício de subsistemas: não implementado.** O item "Sistema" da tela inicial tem a descrição "Status, relatórios, reinício de subsistemas e reset.", mas não existe tela, botão nem rota para reiniciar qualquer parte do sistema. A API é um único processo NestJS;
- **poucas informações na tela.** A avaliação do Incremento 1 apontou "Poucas informações exibidas". A tela mostra quatro métricas do servidor e os totais de usuários, organizações e comissões; não há números de reuniões, atas, anexos ou notificações.

As demais limitações (por exemplo, o reset sem transação e, na web e em aparelho sem bloqueio de tela, o reset protegido só pela palavra digitada) estão no campo de cada requisito em [Requisitos funcionais](2%20-%20requisitos-funcionais-informacao-sistemas.md).

O cadastro e a troca de tipo das contas, inclusive a criação do primeiro administrador do sistema, estão em [Cadastro de usuário](../cadastro-usuario/README.md). A aprovação de organizações, também feita pelo administrador do sistema, está em [Cadastro de organização](../cadastro-organizacao/README.md).

## Atores

| Ator                     | Quem é no sistema                                                        | O que faz neste módulo                                                   |
| ------------------------ | ------------------------------------------------------------------------ | ------------------------------------------------------------------------ |
| Administrador do sistema | Usuário ativo cujo campo `tipo` é `ADMIN_SISTEMA` na coleção `usuarios`. | Consulta status e relatórios, cria os dados de teste e reseta o sistema. |

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

| Caso de uso | Requisito | Sequência | Testes                                                                                       |
| ----------- | --------- | --------- | -------------------------------------------------------------------------------------------- |
| UC-SIS-01   | RF-SIS-01 | SD-SIS-01 | CT-SIS-01, CT-SIS-06, CT-SIS-08, CT-SIS-09                                                   |
| UC-SIS-02   | RF-SIS-02 | SD-SIS-02 | CT-SIS-02, CT-SIS-06, CT-SIS-08, CT-SIS-09                                                   |
| UC-SIS-03   | RF-SIS-03 | SD-SIS-03 | CT-SIS-03, CT-SIS-06, CT-SIS-08, CT-SIS-14                                                   |
| UC-SIS-04   | RF-SIS-04 | SD-SIS-04 | CT-SIS-04, CT-SIS-05, CT-SIS-06, CT-SIS-07, CT-SIS-08, CT-SIS-10, CT-SIS-11, CT-SIS-12, CT-SIS-13 |

## Base de conferência

BACKEND e FRONTEND na branch `homolog`, conferidos em 26/11/2026 (backend sobre `811cf02`, que inclui o reset com confirmação, e frontend sobre `8fc0a5a`, que inclui a folha de confirmação do reset em `0666ee0`). Os arquivos principais são `BACKEND/src/sistema/sistema.controller.ts`, `BACKEND/src/sistema/sistema.service.ts`, `BACKEND/src/sistema/dto/reset-sistema.dto.ts`, `BACKEND/src/sistema/sistema.spec.ts`, `BACKEND/src/auth/admin-sistema.guard.ts`, `BACKEND/src/auth/jwt-auth.guard.ts`, `BACKEND/src/auth/api-key.guard.ts`, `FRONTEND/src/app/sistema.tsx`, `FRONTEND/src/app/(abas)/inicio.tsx`, `FRONTEND/src/app/_layout.tsx`, `FRONTEND/src/components/Confirmacao.tsx`, `FRONTEND/src/components/FolhaInferior.tsx` e `FRONTEND/src/services/autenticacao-local.ts`.
