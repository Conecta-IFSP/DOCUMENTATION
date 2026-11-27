# Cadastro de usuário

> Padrão da disciplina · Módulo USR · Incremento 1 · Revisado em 26/11/2026

## Resumo

Qualquer pessoa pode criar uma conta no Conecta+ com nome, e-mail e uma senha forte; a conta só passa a entrar depois que a pessoa digita o código de 7 caracteres enviado ao e-mail. Também é possível entrar com a conta Google, que cria a conta no primeiro acesso ou se liga à conta que já existe com o mesmo e-mail. Depois de entrar, a sessão vale por 1 hora e é conferida a cada chamada: se a conta for desativada ou a senha trocada, o aplicativo volta à tela de entrada.

## Escopo

- criar a conta local e confirmar o e-mail com o código enviado;
- entrar com e-mail e senha e manter a sessão enquanto o token for válido;
- entrar com a conta Google, no aplicativo instalado e na web.

Ficam fora deste módulo:

- a participação do usuário nas organizações (pedido de acesso por código, convite e resposta, decisão do administrador, papel e remoção de membros), documentada no módulo [Cadastro de organização](../cadastro-organizacao/README.md), em RF-ORG-07 a RF-ORG-14;
- o perfil, a troca de e-mail e de senha, a recuperação de conta, o tema e a saída da sessão, documentados em [Configurações de usuário](../configuracoes-usuario/README.md).

Não estão implementados: entrar pela biometria do aparelho depois do primeiro acesso e pedir um novo código de confirmação do cadastro.

## Atores

| Ator               | Quem é no sistema                              | O que faz neste módulo                                   |
| ------------------ | ---------------------------------------------- | -------------------------------------------------------- |
| Visitante          | Pessoa sem sessão no aplicativo.               | Cria e confirma a conta; entra com senha ou com Google.  |
| Servidor de e-mail | Sistema externo de envio de e-mail.            | Entrega o código de confirmação.                         |
| Google             | Sistema externo de identidade.                 | Confirma a conta Google e devolve os dados dela.         |

## Índice padronizado

|  Nº | Artefato                                                                           | Finalidade                                              |
| --: | ---------------------------------------------------------------------------------- | ------------------------------------------------------- |
|   1 | [Casos de uso](1%20-%20casos-uso-cadastro-usuario.md)                              | Diagrama, atores e requisitos de usuário.               |
|   2 | [Requisitos funcionais](2%20-%20requisitos-funcionais-cadastro-usuario.md)         | Cada caso de uso no formulário padrão.                  |
|   3 | [Diagrama de classes](3%20-%20diagrama-classes-cadastro-usuario.md)                | Telas, API, serviços, guards e documento envolvidos.    |
|   3 | [Dicionário de classes](3%20-%20dicionario-dados-classes-cadastro-usuario.md)      | O que cada classe representa, atributos e operações.    |
|   4 | [Diagrama relacional](4%20-%20diagrama-relacional-cadastro-usuario.md)             | Visão lógica da conta de usuário no MongoDB.            |
|   4 | [Dicionário relacional](4%20-%20dicionario-dados-relacional-cadastro-usuario.md)   | Campo a campo: tipo, nulo, chaves e geração de valores. |
|   5 | [Requisitos não funcionais](5%20-%20requisitos-nao-funcionais-cadastro-usuario.md) | Critérios de qualidade com medida e verificação.        |
|   6 | [Diagramas de sequência](6%20-%20diagramas-sequencia-cadastro-usuario.md)          | Mensagens entre telas, API, serviços e banco.           |
|   7 | [Testes](7%20-%20testes-cadastro-usuario.md)                                       | Testes automatizados e roteiros executados.             |

## Rastreabilidade

| Caso de uso | Requisito | Sequência | Testes                                                                      |
| ----------- | --------- | --------- | --------------------------------------------------------------------------- |
| UC-USR-01   | RF-USR-01 | SD-USR-01 | CT-USR-01, CT-USR-08, CT-USR-09, CT-USR-11, CT-USR-12, CT-USR-13, CT-USR-24 |
| UC-USR-02   | RF-USR-02 | SD-USR-02 | CT-USR-02, CT-USR-07, CT-USR-14, CT-USR-15, CT-USR-16, CT-USR-24, CT-USR-25 |
| UC-USR-03   | RF-USR-03 | SD-USR-03 | CT-USR-03, CT-USR-17, CT-USR-24                                             |

CT-USR-04 a CT-USR-06 e CT-USR-18 a CT-USR-23 foram registrados neste módulo, mas verificam os vínculos com organizações; eles apontam para RF-ORG-07 a RF-ORG-14, documentados no módulo [Cadastro de organização](../cadastro-organizacao/README.md). CT-USR-10 verifica a recuperação de conta (RF-CFG-04) e serve de medida para RNF-USR-06; CT-USR-08, CT-USR-11 e CT-USR-12 também cobrem requisitos do módulo CFG.

## Base de conferência

BACKEND e FRONTEND na branch `homolog`, conferidos em 26/11/2026 (backend sobre `99b6b4f`, frontend sobre `a53acce`, ambos com alterações ainda não registradas em commit). Os arquivos principais são `BACKEND/src/usuarios/usuarios.controller.ts`, `BACKEND/src/usuarios/usuarios.service.ts`, `BACKEND/src/usuarios/schemas/usuario.schema.ts`, `BACKEND/src/usuarios/dto/senha-forte.ts`, `BACKEND/src/auth/auth.controller.ts`, `BACKEND/src/auth/auth.service.ts`, `BACKEND/src/auth/jwt-auth.guard.ts`, `BACKEND/src/auth/api-key.guard.ts`, `FRONTEND/src/app/login/index.tsx`, `FRONTEND/src/app/login/cadastro.tsx`, `FRONTEND/src/app/login/verificar-cadastro.tsx`, `FRONTEND/src/app/login/google-callback/index.tsx`, `FRONTEND/src/contexts/SessaoContext.tsx`, `FRONTEND/src/services/api.ts`, `FRONTEND/src/services/google.ts` e `FRONTEND/src/utils/validacao.ts`.
