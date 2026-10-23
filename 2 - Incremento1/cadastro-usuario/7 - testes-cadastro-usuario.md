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

| ID        | Requisito    | Cenário/entrada                      | Resultado esperado                                         | Situação                          |
| --------- | ------------ | ------------------------------------ | ---------------------------------------------------------- | --------------------------------- |
| CT-USR-01 | RF-USR-01    | Cadastrar e confirmar código         | Conta só autentica depois da confirmação.                  | Coberto no backend; UI pendente.  |
| CT-USR-02 | RF-USR-02    | Login válido/inválido/não confirmado | JWT apenas no cenário válido; falhas recebem 401 genérico. | Coberto no backend; UI pendente.  |
| CT-USR-03 | RF-USR-03    | Login Google e cancelamento          | Conta criada/vinculada ou fluxo cancelado sem sessão.      | Integração manual pendente.       |
| CT-USR-04 | RF-USR-04    | Entrar com código válido/inválido    | Solicitação em análise criada ou erro sem vínculo.         | Coberto no backend; UI a validar. |
| CT-USR-05 | RF-USR-05/06 | Convidar, aceitar/recusar e analisar | Transições corretas e conflitos detectados.                | Coberto no backend; UI pendente.  |
| CT-USR-06 | RF-USR-07    | Alterar papel/remover/último admin   | Papel válido; último admin e remoção indevida recusados.   | Coberto no backend; UI pendente.  |

## Próximas validações

Executar cadastro, Google e teclado em Expo Go/Android e registrar evidência do aparelho.
