# Requisitos não funcionais — Votação na reunião

> Padrão da disciplina · Módulo VOT · Revisado em 13/11/2026

## Resumo

Os requisitos não funcionais aparecem em dois níveis, como os funcionais. Os **de usuário** dizem, em uma frase, a qualidade esperada. Os **de sistema** completam cada um com a medida que o aprova, a condição em que vale, como foi verificado e onde está a evidência, de modo que qualquer pessoa consiga conferir. Onde ainda não houve medição, isso está escrito.

A classificação segue a árvore de Sommerville usada na disciplina: requisitos de **produto** (facilidade de uso, eficiência, confiabilidade, portabilidade), **organizacionais** (entrega, implementação, padrões) e **externos** (interoperabilidade, éticos, legais).

## Requisitos não funcionais de usuário

**RNF-VOT-01 — Autorização para gerenciar votações.** Só o presidente da comissão pode abrir e encerrar votações.

**RNF-VOT-02 — Autorização para votar.** Só quem está presente na reunião pode votar.

**RNF-VOT-03 — Voto único.** Cada pessoa vota uma única vez em cada votação, e o voto não pode ser alterado.

**RNF-VOT-04 — Sigilo do voto.** Ninguém, nem o presidente, vê pelo aplicativo em quem cada pessoa votou.

**RNF-VOT-05 — Resultado só depois do encerramento.** O resultado parcial não pode influenciar quem ainda vai votar.

**RNF-VOT-06 — Proteção contra gravação simultânea.** Votos e encerramento enviados ao mesmo tempo não podem gerar voto duplicado nem voto depois do encerramento.

**RNF-VOT-07 — Atualização da tela.** Os participantes devem ver as votações novas sem precisar sair da tela.

**RNF-VOT-08 — Portabilidade.** A votação deve funcionar em celulares Android a partir da versão 7.0 e em iOS.

**RNF-VOT-09 — Legibilidade nos dois temas.** A aba "Votações" deve ser legível no tema claro e no tema escuro.

**RNF-VOT-10 — Testabilidade.** Os testes automatizados devem passar antes de cada entrega.

## Requisitos não funcionais de sistema

### RNF-VOT-01 — Autorização para gerenciar votações

| Campo | Conteúdo |
| --- | --- |
| **Classificação** | Externo › Legal › Segurança |
| **Critério** | Só o membro da comissão com papel `RESPONSAVEL`, em comissão ativa, abre ou encerra votações. |
| **Medida** | Qualquer outra conta recebe HTTP 403 com a mensagem "Apenas o responsável da comissão pode gerenciar votações", e nada é gravado. Na tela, os botões "Abrir votação" e "Encerrar votação" não aparecem para quem não é presidente. |
| **Condição** | Rotas `POST /reunioes/:id/votacoes` e `POST /reunioes/:id/votacoes/:votacaoId/encerrar`. |
| **Verificação** | Testes automatizados CT-VOT-02 e CT-VOT-07. |
| **Evidência** | `ReunioesService.exigirPresidente` e `BACKEND/src/reunioes/votacoes.spec.ts`. |
| **Situação** | Atendido em 13/11/2026. |

### RNF-VOT-02 — Autorização para votar

| Campo | Conteúdo |
| --- | --- |
| **Classificação** | Externo › Legal › Segurança |
| **Critério** | Vota apenas o participante com presença `AGUARDANDO` ou `CONFIRMADA` e sem saída. |
| **Medida** | Sem entrada, com entrada recusada ou com saída marcada, a API devolve 403 ("Marque sua entrada na reunião para votar" ou "Você já saiu da reunião e não pode mais votar") e nada é gravado. Pessoa de fora da reunião recebe 403 "Você não participa desta reunião". |
| **Condição** | Rota `POST /reunioes/:id/votacoes/:votacaoId/votos`. |
| **Verificação** | Teste automatizado CT-VOT-05. |
| **Evidência** | `ReunioesService.votar` (conferência antes da gravação e `$elemMatch` em `participantes` no filtro do `findOneAndUpdate`). |
| **Situação** | Atendido em 13/11/2026. |

### RNF-VOT-03 — Voto único

| Campo | Conteúdo |
| --- | --- |
| **Classificação** | Produto › Confiabilidade |
| **Critério** | Um usuário tem no máximo um item em `votos` por votação, e o item não muda. |
| **Medida** | A segunda tentativa recebe 409 "Você já votou nesta votação". Não existe rota para trocar ou apagar voto. |
| **Condição** | Qualquer sequência de chamadas à rota de voto. |
| **Verificação** | Teste automatizado CT-VOT-05; inspeção do controller (só há `POST` para votos). |
| **Evidência** | `ReunioesService.votar` e `ReunioesController`. |
| **Situação** | Atendido em 13/11/2026. |

### RNF-VOT-04 — Sigilo do voto

| Campo | Conteúdo |
| --- | --- |
| **Classificação** | Externo › Ético › Privacidade |
| **Critério** | Nenhuma resposta da API contém a lista de votos nem o `usuario_id` de quem votou. |
| **Medida** | Cada votação devolvida tem apenas `total_votos`, `meu_voto` (o voto de quem consulta) e `resultado` (contagem por opção); a propriedade `votos` não existe na resposta, para nenhum usuário, inclusive o presidente. |
| **Condição** | Todas as rotas que devolvem reunião (`GET /reunioes`, `GET /reunioes/:id` e as rotas de ação). |
| **Verificação** | Teste automatizado CT-VOT-08. |
| **Evidência** | `ReunioesService.apresentarVotacao`. |
| **Situação** | Atendido em 13/11/2026, com a limitação registrada abaixo. |

**Limitação conhecida.** O `usuario_id` do voto fica gravado no banco, para garantir o voto único. Quem tem acesso direto ao MongoDB consegue ligar cada voto a uma pessoa; o sigilo vale para o aplicativo e para a API.

### RNF-VOT-05 — Resultado só depois do encerramento

| Campo | Conteúdo |
| --- | --- |
| **Classificação** | Externo › Ético |
| **Critério** | A contagem por opção só é devolvida com a votação encerrada. |
| **Medida** | Com `situacao = ABERTA`, `resultado` é `null`; com `ENCERRADA` (pelo presidente ou pelo fim da reunião), `resultado` traz uma quantidade por opção. |
| **Condição** | Qualquer consulta à reunião. |
| **Verificação** | Testes automatizados CT-VOT-08 e CT-VOT-09. |
| **Evidência** | `ReunioesService.apresentarVotacao`. |
| **Situação** | Atendido em 13/11/2026. |

### RNF-VOT-06 — Proteção contra gravação simultânea

| Campo | Conteúdo |
| --- | --- |
| **Classificação** | Produto › Confiabilidade |
| **Critério** | As condições de voto e encerramento são conferidas de novo pelo MongoDB no mesmo comando que grava. |
| **Medida** | O voto só é gravado se, no instante da gravação, a votação ainda está `ABERTA`, o usuário ainda não está em `votos` e a presença ainda permite votar; o encerramento só é gravado se a votação ainda está `ABERTA`. Caso contrário, a API devolve 409 "A votação mudou. Atualize a tela." e nada é gravado. |
| **Condição** | Dois votos do mesmo usuário, ou um voto e um encerramento, chegando ao mesmo tempo. |
| **Verificação** | Teste automatizado CT-VOT-04 (confere o filtro e o `arrayFilters` enviados ao banco) e CT-VOT-07. Teste de carga com requisições paralelas: critério proposto; medição pendente. |
| **Evidência** | Filtros de `findOneAndUpdate` em `ReunioesService.votar` e `ReunioesService.encerrarVotacao`. |
| **Situação** | Atendido por inspeção e teste unitário em 13/11/2026; medição paralela pendente. |

### RNF-VOT-07 — Atualização da tela

| Campo | Conteúdo |
| --- | --- |
| **Classificação** | Produto › Facilidade de uso |
| **Critério** | A tela da reunião busca os dados de novo sem ação do usuário. |
| **Medida** | Enquanto a tela está aberta, a reunião é consultada a cada 30 segundos; puxar a tela para baixo consulta na hora. Uma votação aberta aparece para os demais em até 30 segundos. |
| **Condição** | Tela da reunião em primeiro plano, com conexão com a API. |
| **Verificação** | Inspeção do código; roteiro manual CT-VOT-13. |
| **Evidência** | `useFocusEffect` com `setInterval(..., 30000)` em `FRONTEND/src/app/reunioes/[id].tsx`. |
| **Situação** | Verificado por inspeção em 13/11/2026; roteiro manual pendente. |

### RNF-VOT-08 — Portabilidade

| Campo | Conteúdo |
| --- | --- |
| **Classificação** | Produto › Portabilidade |
| **Critério** | A aba "Votações" funciona nos mesmos aparelhos que o restante do aplicativo. |
| **Medida** | Abrir, votar, encerrar e ver o resultado sem erro e sem texto cortado em Android 7.0 ou superior e em iOS. |
| **Condição** | APK de release e aplicativo iOS apontando para a API. |
| **Verificação** | Roteiros manuais CT-VOT-10 a CT-VOT-14. |
| **Evidência** | Componentes do React Native usados também nas demais telas (`Botao`, `CampoTexto`, `FolhaInferior`, `Confirmacao`). |
| **Situação** | Critério proposto; execução nos aparelhos pendente. |

### RNF-VOT-09 — Legibilidade nos dois temas

| Campo | Conteúdo |
| --- | --- |
| **Classificação** | Produto › Facilidade de uso |
| **Critério** | Todas as cores da aba vêm do tema ativo. |
| **Medida** | Nenhuma cor fixa no componente; textos, faixas, barras do resultado e etiquetas usam `useTema().tema.cores`. |
| **Condição** | Tema claro e tema escuro. |
| **Verificação** | Inspeção do código; roteiro manual CT-VOT-14. |
| **Evidência** | `FRONTEND/src/components/VotacoesReuniao.tsx`. |
| **Situação** | Verificado por inspeção em 13/11/2026; roteiro manual pendente. |

### RNF-VOT-10 — Testabilidade

| Campo | Conteúdo |
| --- | --- |
| **Classificação** | Organizacional › Implementação |
| **Critério** | As regras da votação têm testes automatizados que rodam com `npm test`. |
| **Medida** | Os 9 testes de `votacoes.spec.ts` e o restante da suíte passam sem falha. |
| **Condição** | BACKEND na branch `homolog`, comando `npm test`. |
| **Verificação** | Execução registrada em [Testes](7%20-%20testes-votacao.md). |
| **Evidência** | `BACKEND/src/reunioes/votacoes.spec.ts`. |
| **Situação** | Atendido em 13/11/2026: 15 arquivos e 213 testes aprovados. |

## Limitações conhecidas

- Uma presença em `AGUARDANDO` já permite votar. Se o organizador recusar essa presença depois, o voto já registrado continua valendo.
- Um presidente de comissão que não foi incluído entre os participantes da reunião não consegue abrir votação nela ("Você não participa desta reunião").
- Não há limite para a quantidade de votações por reunião além do tamanho máximo de um documento do MongoDB (16 MB).

## Rastreabilidade

RNF-VOT-01 a RNF-VOT-10 ↔ RF-VOT-01 a RF-VOT-04 ↔ CT-VOT-01 a CT-VOT-14.
