# Requisitos não funcionais — Cadastro de comissão

> Padrão do Incremento 1 · Módulo COM

## Resumo

Os requisitos abaixo usam o mesmo formato verificável dos demais módulos. “Atendido” indica evidência por inspeção ou teste; metas sem medição aparecem como pendentes.

## Requisitos

| ID         | Qualidade    | Critério e medida                                                          | Condição                               | Verificação/evidência                                                            | Situação                             |
| ---------- | ------------ | -------------------------------------------------------------------------- | -------------------------------------- | -------------------------------------------------------------------------------- | ------------------------------------ |
| RNF-COM-01 | Segurança    | Toda rota exige `X-API-Key`; rotas do módulo exigem JWT e conta ativa.     | Qualquer requisição.                   | Inspecionar `ApiKeyGuard`, `JwtAuthGuard` e `UsuarioComumGuard`; testar 401/403. | Atendido.                            |
| RNF-COM-02 | Autorização  | Mutação só é aceita de `ADMIN/APROVADO` na organização aprovada.           | Criar, editar, ativar ou gerir equipe. | Testes de `ComissoesService` e respostas 403.                                    | Atendido.                            |
| RNF-COM-03 | Integridade  | Cada comissão contém usuários únicos e exatamente um `RESPONSAVEL`.        | Toda gravação de equipe.               | Validador do schema e testes de substituição.                                    | Atendido.                            |
| RNF-COM-04 | Concorrência | Escrita com estado desatualizado é recusada, sem sobrescrever equipe nova. | Duas mutações simultâneas.             | Testes de compare-and-swap e resposta 409.                                       | Atendido.                            |
| RNF-COM-05 | Usabilidade  | Ações destrutivas pedem confirmação e informam sucesso/erro em português.  | Desativação e remoção.                 | Roteiro manual CT-COM.                                                           | Verificação manual pendente.         |
| RNF-COM-06 | Desempenho   | Listagem deve responder em até 2 s em condição normal de desenvolvimento.  | Até 100 comissões acessíveis.          | Medição com Postman/DevTools.                                                    | Critério proposto; medição pendente. |

## Rastreabilidade

Aplica-se a RF-COM-01 a RF-COM-09 e CT-COM-01 a CT-COM-09.
