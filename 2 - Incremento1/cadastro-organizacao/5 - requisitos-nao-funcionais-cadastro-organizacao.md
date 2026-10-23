# Requisitos não funcionais — Cadastro de organização

> Padrão do Incremento 1 · Módulo ORG

## Resumo

Os requisitos abaixo usam o mesmo formato verificável dos demais módulos.

## Requisitos

| ID         | Qualidade             | Critério e medida                                                                      | Condição                                         | Verificação/evidência                              | Situação                             |
| ---------- | --------------------- | -------------------------------------------------------------------------------------- | ------------------------------------------------ | -------------------------------------------------- | ------------------------------------ |
| RNF-ORG-01 | Segurança             | Todas as rotas exigem chave da API e JWT; tipo, conta e papel são revalidados.         | Qualquer operação.                               | Guards globais, guards do módulo e testes 401/403. | Atendido.                            |
| RNF-ORG-02 | Autorização           | Apenas `ADMIN_SISTEMA` altera situação ou exclui; apenas admin aprovado gere vínculos. | Operações administrativas.                       | Inspeção de controllers/services e testes.         | Atendido.                            |
| RNF-ORG-03 | Integridade           | Nome e código são únicos; organização mantém ao menos um admin aprovado.               | Cadastro, renovação e alteração de papel.        | Índices/consultas e testes de último admin.        | Atendido.                            |
| RNF-ORG-04 | Concorrência          | Estado alterado por outra requisição gera 409 em vez de sobrescrita.                   | Análise de vínculo e outras mutações protegidas. | Filtros de estado em `findOneAndUpdate()`.         | Atendido.                            |
| RNF-ORG-05 | Segurança operacional | Revogação e exclusão exigem confirmação explícita no app.                              | Ações destrutivas.                               | Roteiro manual CT-ORG.                             | Verificação manual pendente.         |
| RNF-ORG-06 | Desempenho            | Busca deve responder em até 2 s em condição normal de desenvolvimento.                 | Até 100 organizações.                            | Medição via Postman/DevTools.                      | Critério proposto; medição pendente. |

## Rastreabilidade

Aplica-se a RF-ORG-01 a RF-ORG-09 e CT-ORG-01 a CT-ORG-09.
