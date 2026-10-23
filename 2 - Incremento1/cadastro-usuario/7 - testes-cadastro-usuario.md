# Testes — Cadastro e acesso de usuário

Backend `6f2ffc4`: 182 testes aprovados em 23/10/2026. Arquivos diretamente relacionados: `auth.service.spec.ts`, `auth.controller.spec.ts`, `usuarios.service.spec.ts`, `usuarios.controller.spec.ts`, `usuarios-acesso.spec.ts` e `organizacoes-acesso.spec.ts`.

| Código | Regra |
|---|---|
| CT-USR-01 | cadastro normaliza e-mail, gera bcrypt e exige confirmação |
| CT-USR-02 | falha de envio inicial remove conta não confirmada |
| CT-USR-03 | código correto confirma; inválido/expirado falha |
| CT-USR-04 | login recusa conta inativa/não confirmada/credencial errada |
| CT-USR-05 | Google valida token e cria/vincula provedor |
| CT-USR-06 | código de organização cria ou reabre pendência |
| CT-USR-07 | convite e resposta respeitam estados/duplicidade |
| CT-USR-08 | análise concorrente devolve conflito |
| CT-USR-09 | último administrador não pode ser rebaixado |
| CT-USR-10 | remover membro preserva conta e limpa comissões da organização |

Teste manual pendente: cadastro completo com entrega real de e-mail, login Google em Android/iOS/web e fluxos de convite/código em dois aparelhos.
