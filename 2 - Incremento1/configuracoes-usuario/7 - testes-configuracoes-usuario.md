# Testes — Configurações de usuário

## Ambiente da revisão

| Item | Valor |
|---|---|
| Data | 23/10/2026 |
| Backend | branch `homolog`, commit `6f2ffc4` |
| Frontend | branch `homolog`, commit `792fab4` |
| Node.js / npm | 24.11.0 / 11.6.1 |
| Backend | `npm run build`: aprovado; `npm test`: 13 arquivos, 182 testes aprovados |
| Frontend | 9 testes Node aprovados; lint sem diagnóstico; typecheck pendente de regeneração das typed routes do Expo |

## Casos automatizados cobertos pelo código atual

| Código | Requisito | Cenário | Resultado esperado |
|---|---|---|---|
| CT-CFG-01 | RF-CFG-01 | `GET /usuarios/me` com JWT válido | retorna somente o próprio perfil |
| CT-CFG-02 | RF-CFG-01 | atualizar nome/tema | persiste campos permitidos; e-mail direto é recusado |
| CT-CFG-03 | RF-CFG-02 | solicitar e confirmar e-mail | grava campos temporários, valida código e efetiva o novo endereço |
| CT-CFG-04 | RF-CFG-03 | senha atual errada/igual/nova válida | dois erros `400`; cenário válido grava novo hash |
| CT-CFG-05 | RF-CFG-04 | recuperar e-mail existente e inexistente | mesma mensagem externa nos dois cenários |
| CT-CFG-06 | RF-CFG-04 | código correto, incorreto e expirado | somente correto produz token e consome código |
| CT-CFG-07 | RF-CFG-04 | redefinir com token válido/expirado | válido altera senha e consome token; expirado falha |
| CT-CFG-08 | RNF-CFG-12 | `401` em rota protegida | remove sessão e notifica navegação |
| CT-CFG-09 | RNF-CFG-12 | `403` ou `500` | mantém sessão local |
| CT-CFG-10 | RNF-CFG-12 | resposta `401` de requisição antiga | não remove token de uma sessão mais nova |

## Roteiro manual obrigatório

| Código | Ambiente | Passos | Critério de aprovação | Resultado desta revisão |
|---|---|---|---|---|
| CT-CFG-11 | Android com biometria | abrir E-mail e Senha pelo perfil | sistema solicita biometria/PIN antes de navegar | Não executado |
| CT-CFG-12 | Android sem biometria | repetir fluxo | ação continua usando a sessão JWT, sem travar | Não executado |
| CT-CFG-13 | Android/web | escolher cada tema e simular erro da API | tema muda imediatamente; no erro volta ao anterior | Não executado |
| CT-CFG-14 | Android | recuperação completa pelo código | não usa link/localhost; código leva à nova senha | Não executado |
| CT-CFG-15 | Android | focar todos os campos com teclado aberto | campo e ação atual permanecem alcançáveis | Não executado |
| CT-CFG-16 | sessão autenticada | trocar senha | senha muda, mas sessão permanece ativa | Comportamento conhecido; recomendação de logout não implementada |

## Pendência de qualidade

O `npm run typecheck` do frontend encontrou referências de rota ausentes no arquivo gerado de typed routes. O código não foi alterado nesta revisão; antes de declarar o frontend aprovado, deve-se regenerar `.expo/types` pela ferramenta Expo e repetir o comando.
