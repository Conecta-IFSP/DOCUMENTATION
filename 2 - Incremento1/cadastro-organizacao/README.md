# Cadastro e gestão de organizações

O módulo cobre cadastro sujeito à aprovação do administrador do sistema, pesquisa, status, exclusão revogada, código de acesso, convites, solicitações e administração de membros.

## Regras principais

- `nome` é único sem diferença de caixa; `cidade` é obrigatória;
- criador vira `ADMIN/APROVADO`, mas a organização nasce `PENDENTE`;
- administração global de status cabe a `ADMIN_SISTEMA`; gestão de membros cabe a `ADMIN/APROVADO` da organização;
- código de 10 caracteres substitui listagem pública para ingresso;
- a organização nunca pode ficar sem administrador aprovado;
- remoções/revogação/exclusão usam confirmação na interface;
- atualizações condicionais retornam `409` quando permissões/estado mudam concorrentemente.

Consulte `CATALOGO-API-E-REGRAS.md` na raiz para todas as rotas e campos.
