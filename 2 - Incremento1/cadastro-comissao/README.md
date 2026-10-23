# Cadastro e gerenciamento de comissões

## Escopo

Cadastro, consulta, edição, desativação/reativação e equipe da comissão. Cada comissão pertence a uma organização aprovada e possui exatamente um responsável.

## Regras centrais

- somente administrador aprovado da organização gerencia comissões;
- membros/responsável precisam ser usuários ativos e aprovados na organização;
- comissão nasce ativa com um responsável obrigatório;
- o responsável é substituído por `PATCH /comissoes/:id/responsavel`;
- desativação é lógica e ações destrutivas/remoções usam confirmação na interface;
- equipe usa compare-and-swap para detectar concorrência.

## Artefatos

- `CasosDeUso.md`: UC-COM-01 a UC-COM-09;
- `RequisitosFuncionais.md`: rotas e regras;
- `DiagramaDeClasses.md`: schemas e dicionário de classes;
- `DiagramaRelacional.md`: visão lógica MongoDB e dicionário;
- `RequisitosNaoFuncionais.md`: critérios de qualidade;
- `DiagramasSequencia.md`: interações detalhadas;
- `7 - testes-cadastro-comissao.md`: evidências e roteiro manual.

Consulte também o catálogo raiz para a matriz completa de rotas.
