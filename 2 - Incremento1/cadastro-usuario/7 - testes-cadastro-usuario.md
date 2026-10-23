# Testes — Cadastro de usuário

> Padrão do Incremento 1 · Módulo USR

## Resumo

Registra evidência executada e cenários manuais no formato comum do Incremento 1.

## Evidência automatizada

| Data       | Projeto/commit     | Comando     | Resultado                          | Arquivos relacionados                                             |
| ---------- | ------------------ | ----------- | ---------------------------------- | ----------------------------------------------------------------- |
| 23/10/2026 | BACKEND `6f2ffc4`  | `npm test`  | 13 arquivos e 182 testes aprovados | `auth.*.spec.ts`, `usuarios.*.spec.ts`, `usuarios-acesso.spec.ts` |
| 23/10/2026 | FRONTEND `792fab4` | testes Node | 9 testes aprovados                 | `sessao-api.test.cjs`, `horarios.test.cjs`                        |

## Matriz de testes manuais

| ID        | Requisito    | Cenário/entrada                      | Resultado esperado                                         | Situação              |
| --------- | ------------ | ------------------------------------ | ---------------------------------------------------------- | --------------------- |
| CT-USR-01 | RF-USR-01    | Cadastrar e confirmar código         | Conta só autentica depois da confirmação.                  | Aprovado pela equipe. |
| CT-USR-02 | RF-USR-02    | Login válido/inválido/não confirmado | JWT apenas no cenário válido; falhas recebem 401 genérico. | Aprovado pela equipe. |
| CT-USR-03 | RF-USR-03    | Login Google e cancelamento          | Conta criada/vinculada ou fluxo cancelado sem sessão.      | Aprovado pela equipe. |
| CT-USR-04 | RF-USR-04    | Entrar com código válido/inválido    | Solicitação em análise criada ou erro sem vínculo.         | Aprovado pela equipe. |
| CT-USR-05 | RF-USR-05/06 | Convidar, aceitar/recusar e analisar | Transições corretas e conflitos detectados.                | Aprovado pela equipe. |
| CT-USR-06 | RF-USR-07    | Alterar papel/remover/último admin   | Papel válido; último admin e remoção indevida recusados.   | Aprovado pela equipe. |

## Conclusão da validação manual

A equipe informou a execução dos roteiros de cadastro, autenticação externa e teclado em ambiente móvel, com todos os cenários aprovados.
