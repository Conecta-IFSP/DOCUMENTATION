# Testes — Comissões

Em 23/10/2026, `npm test` no backend `6f2ffc4` aprovou 182 testes; os arquivos `comissoes.service.spec.ts`, `comissoes.controller.spec.ts` e `comissoes.regras.spec.ts` fazem parte da execução.

| Código | Regra verificada | Resultado esperado |
|---|---|---|
| CT-COM-01 | criação por administrador com responsável elegível | ativa, um `RESPONSAVEL` |
| CT-COM-02 | criação sem permissão/alvo inelegível | `400/403` sem gravação |
| CT-COM-03 | listar por vínculo | admin vê todas da organização; membro só as próprias |
| CT-COM-04 | impedir zero ou vários responsáveis | validação recusa estado inválido |
| CT-COM-05 | substituir responsável | alvo promovido e anterior rebaixado em uma gravação |
| CT-COM-06 | impedir remoção do responsável | `400`, exige substituição anterior |
| CT-COM-07 | concorrência na equipe | compare-and-swap devolve `409` |
| CT-COM-08 | desativar e reativar | equipe preservada; gestão bloqueada enquanto inativa |

## Roteiro manual pendente

Em Android e iOS: criar comissão escolhendo organização e responsável, abrir detalhe em tela pequena, adicionar/remover membro, substituir responsável, cancelar e confirmar desativação. Registrar dispositivo, versão do sistema, passos e capturas; esta revisão não executou o roteiro em aparelho.
