# Testes - Informações do Sistema

## Evidência automatizada

Em 23/10/2026, na branch `homolog` do BACKEND (`6f2ffc4`):

- `npm run build`: aprovado;
- `npm test`: 13 arquivos e 182 testes aprovados;
- ferramenta: Vitest 4.1.11, Node.js 24.11.0.

Os testes de serviço verificam as regras do sistema em conjunto com os demais módulos. Ainda é necessário manter um roteiro manual de integração para conferir banco real, credenciais e mensagens exibidas no aparelho.

## Roteiro manual mínimo

| ID | Cenário | Resultado esperado |
|---|---|---|
| CT-SIS-01 | Usuário comum consulta `/sistema/status` | HTTP 403. |
| CT-SIS-02 | Administrador consulta status e relatórios | HTTP 200 com métricas e contagens. |
| CT-SIS-03 | Requisição sem `X-API-Key` | HTTP 401 antes do controlador. |
| CT-SIS-04 | Administrador confirma `popular-teste` | Dados de demonstração são criados sem remover o administrador. |
| CT-SIS-05 | Administrador cancela a confirmação de reset no aplicativo | Nenhuma requisição destrutiva é enviada. |
| CT-SIS-06 | Administrador confirma o reset | Organizações, comissões, reuniões e usuários comuns são removidos; `ADMIN_SISTEMA` permanece. |
