# Testes — Votação na reunião

> Padrão da disciplina · Módulo VOT · Revisado em 13/11/2026

## Resumo

Registra o que foi executado, quando, em qual ambiente e qual foi o resultado. Os testes automatizados cobrem as regras do servidor; os roteiros manuais cobrem as telas. Nada aqui está marcado como aprovado sem ter sido executado: os roteiros manuais ainda não executados estão marcados como pendentes.

## Ambiente

| Item | Valor |
| --- | --- |
| Data da execução | 13/11/2026 |
| Node.js / npm | 24.11.0 / 11.6.1 |
| Backend | branch `homolog`, commit `08a5b3e` (votação incluída em `c2ea38c`) |
| Aplicativo | branch `homolog`, commit `66806e4` (tela de votação incluída em `83ccd12`) |
| Sistema | Windows 11 |
| Aparelhos | Roteiros manuais ainda não executados em aparelho |

## Evidência automatizada

| Data | Projeto | Comando | Resultado |
| --- | --- | --- | --- |
| 13/11/2026 | BACKEND | `npm test` | 15 arquivos e 213 testes aprovados, nenhuma falha |
| 13/11/2026 | BACKEND | `npx vitest run src/reunioes/votacoes.spec.ts` | 9 testes aprovados |
| 13/11/2026 | BACKEND | `npx tsc --noEmit -p tsconfig.build.json` | Sem erro |
| 13/11/2026 | FRONTEND | `npx eslint` em `VotacoesReuniao.tsx`, `reunioes/[id].tsx` e `services/api.ts` | Sem diagnóstico |
| 13/11/2026 | FRONTEND | `npx tsc --noEmit` | Nenhum erro nos arquivos da votação. O projeto acusou 14 erros em outros módulos (`anexos.ts`, `pdfAta.ts`, `exportar-relatorio-individual.ts`), causados por dependências (`expo-print`, `expo-sharing`) não instaladas na máquina da execução. |

Os 9 testes do arquivo `BACKEND/src/reunioes/votacoes.spec.ts` estão descritos abaixo. Eles usam models simulados, então não dependem de banco de dados.

## Matriz de testes automatizados

| ID | Requisito | Cenário e entrada | Resultado esperado | Resultado obtido | Teste |
| --- | --- | --- | --- | --- | --- |
| CT-VOT-01 | RF-VOT-01 | Presidente abre a votação "Escolher data do evento" com as opções "Sábado" e "Domingo", com a reunião em andamento. | `findOneAndUpdate` com filtro `AGENDADA` e `$push` de uma votação `ABERTA`, com as opções informadas e `votos` vazia. | Aprovado | "permite ao responsável abrir uma votação durante a reunião" |
| CT-VOT-02 | RF-VOT-01, RNF-VOT-01 | Membro comum tenta abrir uma votação. | 403. | Aprovado | "recusa abertura por quem não é responsável da comissão" |
| CT-VOT-03 | RF-VOT-01 | Abrir com opções "Sim" e "sim"; antes do início; depois do fim; com a reunião cancelada. | 400 nos quatro casos. | Aprovado | "recusa abertura fora do horário, em reunião cancelada ou com opções repetidas" |
| CT-VOT-04 | RF-VOT-02, RNF-VOT-06 | Participante presente vota na opção 1. | Filtro com `votos.usuario_id` diferente do usuário; `$push` em `votacoes.$[votacao].votos` com o usuário e a opção 1; `arrayFilters` pelo `_id` da votação. | Aprovado | "registra o voto de um participante presente" |
| CT-VOT-05 | RF-VOT-02, RNF-VOT-02, RNF-VOT-03 | Votar de novo; votar sem entrada; votar depois da saída; votar com a entrada recusada. | 409; 403; 403; 403. | Aprovado | "impede voto duplicado, sem presença e após a saída" |
| CT-VOT-06 | RF-VOT-02 | Votar na opção 5 de uma votação com 3 opções; em votação inexistente; em votação encerrada; depois do fim da reunião. | 400; 404; 400; 400. | Aprovado | "impede voto em opção inválida, votação inexistente ou encerrada" |
| CT-VOT-07 | RF-VOT-03, RNF-VOT-01, RNF-VOT-06 | Membro comum tenta encerrar; presidente encerra; presidente tenta encerrar de novo. | 403; `findOneAndUpdate` com filtro da votação `ABERTA` e `$set` de `ENCERRADA` e `encerrada_em`; 400. | Aprovado | "encerra a votação apenas pelo responsável e uma única vez" |
| CT-VOT-08 | RF-VOT-04, RNF-VOT-04, RNF-VOT-05 | Consultar uma votação aberta com 1 voto do presidente; acrescentar o voto do membro na opção 1, encerrar e consultar como o membro. | Aberta: sem `votos`, `total_votos = 1`, `meu_voto = 0`, `resultado = null`. Encerrada: `total_votos = 2`, `meu_voto = 1`, `resultado = [1, 1, 0]`. | Aprovado | "esconde votos individuais e só mostra o resultado após o encerramento" |
| CT-VOT-09 | RF-VOT-04, RNF-VOT-05 | Consultar uma votação gravada como `ABERTA` depois do fim da reunião. | Apresentada como `ENCERRADA`, com `meu_voto = null` e `resultado = [1, 0, 0]`. | Aprovado | "considera encerrada a votação aberta quando a reunião termina" |

## Roteiros manuais

Pré-condições comuns: uma comissão ativa com duas contas, uma `RESPONSAVEL` (presidente) e uma `MEMBRO`; uma reunião em andamento com as duas contas convidadas e convite aceito.

| ID | Requisito | Aparelho | Passos | Resultado esperado | Resultado obtido |
| --- | --- | --- | --- | --- | --- |
| CT-VOT-10 | RF-VOT-01, RNF-VOT-08 | Android | Como presidente, abrir a aba "Votações", tocar em "Abrir votação", tentar enviar com o assunto "Ok", corrigir para "Aprovar orçamento", remover "Abstenção" e enviar. | Mensagem "Informe um assunto com pelo menos 3 caracteres"; depois "Votação aberta." e cartão "Aberta" com "0 votos". | Pendente |
| CT-VOT-11 | RF-VOT-02, RNF-VOT-02 | Android | Como membro, sem marcar entrada, abrir a aba "Votações". Depois marcar entrada e votar em "Sim". | Primeiro a faixa "Marque sua entrada na reunião para votar."; depois a confirmação, "Voto registrado." e a faixa `Você votou em "Sim"`. | Pendente |
| CT-VOT-12 | RF-VOT-02, RNF-VOT-03 | Android | Como membro, marcar saída e abrir outra votação aberta. | Faixa "Você já saiu da reunião e não pode mais votar." e nenhuma opção para tocar. | Pendente |
| CT-VOT-13 | RF-VOT-03, RF-VOT-04, RNF-VOT-07 | Android, dois aparelhos | Presidente encerra a votação; no aparelho do membro, aguardar até 30 segundos sem tocar na tela. | Cartão "Encerrada" nos dois aparelhos, com quantidade, porcentagem, barra e "seu voto" na opção do membro. | Pendente |
| CT-VOT-14 | RNF-VOT-08, RNF-VOT-09 | iOS e Android, tema escuro | Repetir CT-VOT-10 a CT-VOT-13 em iOS e com o aplicativo no tema escuro. | Nada cortado; textos, faixas e barras legíveis. | Pendente |

## Rastreabilidade

CT-VOT-01 a CT-VOT-09 cobrem RF-VOT-01 a RF-VOT-04 e foram executados; CT-VOT-10 a CT-VOT-14 cobrem as telas e os RNFs de portabilidade, atualização e legibilidade e ainda estão pendentes.
