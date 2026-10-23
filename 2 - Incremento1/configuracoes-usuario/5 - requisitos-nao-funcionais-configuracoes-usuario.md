# Requisitos não funcionais — Configurações de usuário

| Código | Qualidade | Critério verificável | Verificação/evidência | Situação |
|---|---|---|---|---|
| RNF-CFG-01 | Autorização | Rotas `/usuarios/me*` recusam ausência/invalidade de JWT com `401` e nunca aceitam ID de usuário no corpo. | Testes de guards/controllers; `JwtAuthGuard` e `req.usuario.sub`. | Implementado |
| RNF-CFG-02 | Proteção de senha | Nenhuma resposta ou consulta comum expõe `senha_hash`; hashes novos usam bcrypt custo 10. | Inspeção do schema/service e testes. | Implementado |
| RNF-CFG-03 | Recuperação segura | Código e token têm validade de 30 min, uso único e somente hash SHA-256 persistido. | Testes de `AuthService`. | Implementado |
| RNF-CFG-04 | Privacidade | Solicitação de recuperação retorna exatamente a mesma mensagem para e-mail existente e inexistente. | Teste automatizado com os dois cenários. | Implementado |
| RNF-CFG-05 | Validação | Nome, e-mail, tema e senha são validados no frontend e por DTO no backend; propriedades extras são removidas pelo `ValidationPipe`. | `validacao.ts`, DTOs e `main.ts`. | Implementado; nome mínimo diverge: 3 no app e 2 na API |
| RNF-CFG-06 | Proteção local | Ao abrir troca de e-mail/senha e antes de redefinir senha, o app solicita credencial do aparelho quando existe nível cadastrado. | Teste manual Android/iOS com e sem biometria. | Implementado; não se aplica à web e é dispensado sem credencial |
| RNF-CFG-07 | Consistência do tema | Falha no `PATCH /usuarios/me` restaura a preferência anterior local e visual. | Teste manual simulando erro HTTP. | Implementado |
| RNF-CFG-08 | Persistência | Tema fica em MongoDB e na chave local `conecta_mais_tema`; JWT fica em `conecta_mais_token`. | Inspeção de `TemaContext`/`api.ts`. | Implementado |
| RNF-CFG-09 | Usabilidade | Formulários impedem envio inválido, desabilitam ações durante requisição e exibem mensagens em português. | Inspeção e teste manual. | Implementado |
| RNF-CFG-10 | Compatibilidade | Fluxos principais devem ser testados em Android e web; autenticação local é nativa e possui fallback. | Roteiro manual em dois ambientes. | Teste completo pendente |
| RNF-CFG-11 | Desempenho | Em rede local estável, leitura/alteração de perfil deve completar em até 2 s no percentil 95, excluindo tempo do SMTP/biometria. | Medição de 30 execuções por rota. | Meta proposta; não medida |
| RNF-CFG-12 | Resiliência | Se a sessão receber `401` em rota protegida, o app remove somente o token correspondente e avisa a navegação; `403/500` não encerram sessão. | `tests/sessao-api.test.cjs`: 5 casos aprovados. | Implementado |

## Limitações que não devem ser descritas como atendidas

- Não existe revogação de JWT após troca/redefinição de senha.
- Não existe rate limit para códigos de e-mail ou recuperação.
- A ação de trocar senha não faz logout automático.
- A chave global da API é pública no bundle Expo e não substitui autorização.

