# Testes — Organizações

Execução de 23/10/2026: backend `6f2ffc4`, `npm test`, 182 testes aprovados em 13 arquivos. Cobertura relacionada: `organizacoes.service.spec.ts`, `organizacoes.controller.spec.ts` e `organizacoes-acesso.spec.ts`.

| Código | Regra |
|---|---|
| CT-ORG-01 | cadastro exige nome/cidade válidos e nome disponível |
| CT-ORG-02 | pesquisa filtra nome e limita visibilidade por tipo/vínculo |
| CT-ORG-03 | somente admin do sistema altera status/exclui |
| CT-ORG-04 | exclusão exige `REVOGADA` e remove comissões |
| CT-ORG-05 | código inválido não revela organização; válido cria pendência |
| CT-ORG-06 | convite exige usuário ativo e trata duplicidades |
| CT-ORG-07 | aceitar/recusar convite altera/remove vínculo |
| CT-ORG-08 | administrador analisa pendência e pode reverter rejeição |
| CT-ORG-09 | rebaixamento do último administrador é recusado |
| CT-ORG-10 | remoção de membro também o retira das comissões |

Teste manual ainda necessário: confirmar modais destrutivos, busca em Android/iOS, cópia/renovação de código e atualização da tela ao voltar ao foco.
