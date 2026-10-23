# Testes — Configurações de usuário

> Padrão do Incremento 1 · Módulo CFG

## Resumo

Registra evidência executada e cenários manuais no formato comum do Incremento 1.

## Evidência automatizada

| Data       | Projeto/commit     | Comando         | Resultado                          | Arquivos relacionados                              |
| ---------- | ------------------ | --------------- | ---------------------------------- | -------------------------------------------------- |
| 23/10/2026 | BACKEND `6f2ffc4`  | `npm test`      | 13 arquivos e 182 testes aprovados | `auth.service.spec.ts`, `usuarios.service.spec.ts` |
| 23/10/2026 | BACKEND `6f2ffc4`  | `npm run build` | Aprovado                           | TypeScript/NestJS                                  |
| 23/10/2026 | FRONTEND `792fab4` | testes Node     | 9 testes aprovados                 | sessão/API e horários                              |

## Matriz de testes manuais

| ID        | Requisito | Cenário/entrada                         | Resultado esperado                                        | Situação                         |
| --------- | --------- | --------------------------------------- | --------------------------------------------------------- | -------------------------------- |
| CT-CFG-01 | RF-CFG-01 | Alterar nome sem trocar de tela         | Cabeçalho e formulário atualizados imediatamente.         | Pendente em aparelho.            |
| CT-CFG-02 | RF-CFG-02 | Solicitar e confirmar novo e-mail       | Só muda após código correto; duplicado/expirado recusado. | Coberto no backend; UI pendente. |
| CT-CFG-03 | RF-CFG-03 | Trocar senha atual/correta/incorreta    | Hash muda apenas com senha atual válida e nova diferente. | Coberto no backend; UI pendente. |
| CT-CFG-04 | RF-CFG-04 | Recuperar por código no celular         | Fluxo termina sem link, `localhost` ou IP.                | Teste Expo Go pendente.          |
| CT-CFG-05 | RF-CFG-05 | Alterar tema com sucesso/falha simulada | Mantém em sucesso e restaura em falha.                    | Pendente em aparelho.            |
| CT-CFG-06 | RF-CFG-06 | Sair e tentar voltar à rota privada     | Token removido e login exibido.                           | Pendente em aparelho.            |

## Próximas validações

Executar CT-CFG-01 a CT-CFG-06 em Expo Go/Android; o typecheck do frontend ainda depende de regenerar as rotas tipadas do Expo.
