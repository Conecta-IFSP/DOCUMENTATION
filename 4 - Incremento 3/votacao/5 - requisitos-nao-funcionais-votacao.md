# Requisitos não funcionais — Votação na reunião

> Padrão da disciplina · Módulo VOT · Revisado em 27/11/2026

## Resumo

Os requisitos não funcionais aparecem em dois níveis, como os funcionais. Os **de usuário** dizem, em uma frase, a qualidade esperada. Os **de sistema** completam cada um com a medida que o aprova, a condição em que vale, como foi verificado e onde está a evidência, de modo que qualquer pessoa consiga conferir. Onde ainda não houve medição, isso está escrito no campo Situação.

A classificação segue a árvore de Sommerville usada na disciplina: requisitos de **produto** (facilidade de uso, eficiência, com desempenho e espaço, confiabilidade e portabilidade), **organizacionais** (entrega, implementação e padrões) e **externos** (interoperabilidade, éticos e legais, com privacidade e segurança).

## Requisitos não funcionais de usuário

**RNF-VOT-01 — Autorização para gerenciar votações.** Só o presidente da comissão pode abrir e encerrar votações.

**RNF-VOT-02 — Autorização para votar.** Só quem está presente na reunião pode votar.

**RNF-VOT-03 — Integridade do voto registrado.** O voto gravado não se perde nem se duplica, mesmo que o pedido de voto seja repetido ou chegue mais de uma vez ao mesmo tempo.

**RNF-VOT-04 — Sigilo do voto e do resultado parcial.** Na votação do tipo fechada, ninguém, nem o presidente, vê pelo aplicativo em quem cada pessoa votou. Na do tipo aberta, todos os participantes veem quem votou em quê, mas só depois do encerramento. Enquanto a votação não termina, nenhuma contagem, nem a de brancos e nulos, é mostrada, para não influenciar quem ainda vai votar.

**RNF-VOT-05 — Incorporado ao RNF-VOT-04 em 27/11/2026.** A regra de só liberar o resultado depois do encerramento é funcional e está no RF-VOT-03 e no RF-VOT-04; o sigilo do resultado parcial passou para o RNF-VOT-04.

**RNF-VOT-06 — Proteção contra gravação simultânea.** Votos e encerramento enviados ao mesmo tempo não podem gerar voto duplicado nem voto depois do encerramento.

**RNF-VOT-07 — Atualização da tela.** Os participantes devem ver as votações novas sem precisar sair da tela.

**RNF-VOT-08 — Portabilidade.** A votação deve funcionar em celulares Android a partir da versão 7.0 e em iOS.

**RNF-VOT-09 — Legibilidade nos dois temas.** A aba "Votações" deve ser legível no tema claro e no tema escuro.

**RNF-VOT-10 — Testabilidade.** Os testes automatizados devem passar antes de cada entrega.

## Requisitos não funcionais de sistema

### RNF-VOT-01 — Autorização para gerenciar votações

| Campo             | Conteúdo |
| ----------------- | -------- |
| **Classificação** | Externo › Legal › Segurança |
| **Critério**      | Só o membro da comissão com papel `RESPONSAVEL`, em comissão ativa, abre ou encerra votações. |
| **Medida**        | O participante da reunião que não é `RESPONSAVEL` da comissão recebe HTTP 403 com a mensagem "Apenas o responsável da comissão pode gerenciar votações"; o `RESPONSAVEL` de comissão inativa recebe HTTP 400 com "Reative a comissão antes de gerenciar votações"; quem não está entre os participantes recebe HTTP 403 com "Você não participa desta reunião". Em todos os casos, nada é gravado. Na tela, os botões "Abrir votação" e "Encerrar votação" não aparecem para quem não é presidente. |
| **Condição**      | Rotas `POST /reunioes/:id/votacoes` e `POST /reunioes/:id/votacoes/:votacaoId/encerrar`. |
| **Verificação**   | Testes automatizados CT-VOT-02 e CT-VOT-07. |
| **Evidência**     | `ReunioesService.exigirPresidente` e `BACKEND/src/reunioes/votacoes.spec.ts`. |
| **Situação**      | Atendido em 13/11/2026; testes aprovados de novo em 26/11/2026 e em 27/11/2026. |

### RNF-VOT-02 — Autorização para votar

| Campo             | Conteúdo |
| ----------------- | -------- |
| **Classificação** | Externo › Legal › Segurança |
| **Critério**      | Vota apenas o participante com presença `AGUARDANDO` ou `CONFIRMADA` e sem saída. |
| **Medida**        | Sem entrada, com entrada recusada ou com saída marcada, a API devolve 403 ("Marque sua entrada na reunião para votar" ou "Você já saiu da reunião e não pode mais votar") e nada é gravado. Pessoa de fora da reunião recebe 403 "Você não participa desta reunião". |
| **Condição**      | Rota `POST /reunioes/:id/votacoes/:votacaoId/votos`. |
| **Verificação**   | Teste automatizado CT-VOT-05; roteiro manual CT-VOT-11. |
| **Evidência**     | `ReunioesService.votar`: conferência antes da gravação e condição sobre a presença repetida no filtro do `findOneAndUpdate`. |
| **Situação**      | Atendido em 13/11/2026; teste aprovado de novo em 26/11/2026 e em 27/11/2026. Roteiro manual CT-VOT-11 aprovado em 27/11/2026 no emulador Android 14: sem entrada, a faixa "Marque sua entrada na reunião para votar."; depois de marcar a entrada, o voto foi registrado. |

### RNF-VOT-03 — Integridade do voto registrado

| Campo             | Conteúdo |
| ----------------- | -------- |
| **Classificação** | Produto › Confiabilidade |
| **Critério**      | Pedidos de voto repetidos ou simultâneos do mesmo usuário na mesma votação deixam gravado exatamente um voto dele, e nenhuma rota altera ou apaga um voto gravado. |
| **Medida**        | Para N pedidos `POST /reunioes/:id/votacoes/:votacaoId/votos` do mesmo usuário (N ≥ 2), enviados em sequência ou ao mesmo tempo, a votação termina com 1 item desse usuário em `votos`, e os outros N − 1 pedidos recebem 409, com "Você já votou nesta votação" ou "A votação mudou. Atualize a tela.". O controller não tem rota `PATCH` nem `DELETE` para votos. |
| **Condição**      | Votação `ABERTA`, usuário com presença que permite votar; pedidos repetidos pelo aplicativo ou enviados direto à API. |
| **Verificação**   | Teste automatizado CT-VOT-05 (segundo pedido em sequência → 409); CT-VOT-04, que confere no filtro do `findOneAndUpdate` a condição `'votos.usuario_id': { $ne: usuario }`; inspeção do `ReunioesController` (só `POST` para votos); roteiros manuais CT-VOT-25, CT-VOT-26 e CT-VOT-29. Pedidos simultâneos contra um MongoDB real: o mesmo teste pendente do RNF-VOT-06. |
| **Evidência**     | `ReunioesService.votar`: conferência de `votos` antes da gravação e a mesma condição no filtro do `findOneAndUpdate`, que o MongoDB avalia e grava num só comando atômico sobre o documento da reunião; `ReunioesController`. |
| **Situação**      | Pedidos em sequência: atendido em 13/11/2026; CT-VOT-05 aprovado de novo em 26/11/2026 e em 27/11/2026; na tela, CT-VOT-29 aprovado em 27/11/2026 no emulador Pixel 6, em que o aplicativo não permitiu um segundo voto. Pedidos simultâneos: atendido por inspeção do filtro atômico; medição com pedidos paralelos pendente. |

### RNF-VOT-04 — Sigilo do voto e do resultado parcial

| Campo             | Conteúdo |
| ----------------- | -------- |
| **Classificação** | Externo › Legal › Privacidade |
| **Critério**      | Nenhuma resposta da API contém a lista `votos`. Na votação de `modo = FECHADA`, nenhuma resposta contém o `usuario_id` nem o nome de quem votou. Na de `modo = ABERTA`, a lista nominal só é devolvida depois do encerramento. Enquanto a votação está `ABERTA`, nenhuma contagem é devolvida, nem a de brancos e nulos. |
| **Medida**        | Cada votação devolvida tem `total_votos`, `meu_voto` (o voto de quem consulta), `resultado` (contagem por opção), `brancos`, `nulos` e `votos_nominais`; a propriedade `votos` não existe na resposta, para nenhum usuário, inclusive o presidente, em nenhum dos dois tipos. `votos_nominais` é `null` sempre que `modo` é `FECHADA` e, com `modo` igual a `ABERTA`, enquanto a votação não está encerrada; encerrada a votação do tipo aberta, traz, para qualquer participante, um item `{ usuario_id, nome, voto }` por voto. Com `situacao = ABERTA`, `resultado`, `brancos`, `nulos` e `votos_nominais` são `null`, e só `total_votos` e `meu_voto` informam algo sobre os votos. Na tela, a lista "Quem votou em quê" só aparece quando `votos_nominais` tem itens. |
| **Condição**      | Todas as rotas que devolvem reunião (`GET /reunioes`, `GET /reunioes/:id` e as rotas de ação). |
| **Verificação**   | Testes automatizados CT-VOT-08, CT-VOT-09, CT-VOT-17 e CT-VOT-19; roteiros manuais CT-VOT-27 e CT-VOT-28. |
| **Evidência**     | `ReunioesService.apresentarVotacao` e `renderizarResultado` em `FRONTEND/src/components/VotacoesReuniao.tsx`. |
| **Situação**      | Para o voto sempre secreto, atendido em 13/11/2026 e teste CT-VOT-08 aprovado de novo em 26/11/2026. Desde 27/11/2026, o sigilo depende do tipo: CT-VOT-08 e CT-VOT-19 aprovados em 27/11/2026; roteiros manuais CT-VOT-27 e CT-VOT-28 aprovados em 27/11/2026 no Galaxy S24 e no emulador Android 14 e, numa 2ª execução, no emulador Pixel 6. O resultado parcial (antigo RNF-VOT-05): CT-VOT-08 e CT-VOT-09 aprovados em 13/11/2026 e em 26/11/2026, e os quatro testes aprovados em 27/11/2026, já com os votos em branco, os nulos e a lista nominal. O `usuario_id` do voto fica gravado no banco nos dois tipos, para garantir o voto único: quem tem acesso direto ao MongoDB consegue ligar cada voto a uma pessoa, inclusive na votação fechada. |

### RNF-VOT-05 — Incorporado ao RNF-VOT-04

Em 27/11/2026, este requisito, "Resultado só depois do encerramento", foi juntado ao [RNF-VOT-04](#rnf-vot-04--sigilo-do-voto-e-do-resultado-parcial): liberar o resultado só no encerramento é uma regra funcional, descrita no RF-VOT-03 e no RF-VOT-04, e o que ele tinha de qualidade, não deixar o resultado parcial vazar durante a votação, é sigilo. O número fica reservado para não mudar a numeração dos demais.

### RNF-VOT-06 — Proteção contra gravação simultânea

| Campo             | Conteúdo |
| ----------------- | -------- |
| **Classificação** | Produto › Confiabilidade |
| **Critério**      | As condições de voto e de encerramento são conferidas de novo pelo MongoDB no mesmo comando que grava. |
| **Medida**        | O voto só é gravado se, no instante da gravação, a votação ainda está `ABERTA`, o usuário ainda não está em `votos` e a presença ainda permite votar; o encerramento só é gravado se a votação ainda está `ABERTA`. Caso contrário, a API devolve 409 "A votação mudou. Atualize a tela." e nada é gravado. |
| **Condição**      | Dois votos do mesmo usuário, ou um voto e um encerramento, chegando ao mesmo tempo. |
| **Verificação**   | Testes automatizados CT-VOT-04, que confere no filtro a condição de o usuário ainda não estar em `votos` e o `arrayFilters` enviados ao banco, e CT-VOT-07, que confere no filtro a votação ainda `ABERTA`; as demais condições do filtro do voto foram conferidas por inspeção. Teste com requisições paralelas contra um MongoDB real: pendente. |
| **Evidência**     | Filtros de `findOneAndUpdate` em `ReunioesService.votar` e `ReunioesService.encerrarVotacao`. |
| **Situação**      | Atendido por inspeção e teste de unidade em 13/11/2026; medição com requisições paralelas pendente. |

### RNF-VOT-07 — Atualização da tela

| Campo             | Conteúdo |
| ----------------- | -------- |
| **Classificação** | Produto › Facilidade de uso |
| **Critério**      | A tela da reunião busca os dados de novo sem ação do usuário. |
| **Medida**        | Enquanto a tela está aberta, a reunião é consultada a cada 30.000 ms; puxar a tela para baixo consulta na hora. Uma votação aberta aparece para os demais em até 30 segundos. |
| **Condição**      | Tela da reunião em primeiro plano, com conexão com a API. |
| **Verificação**   | Inspeção do código; roteiro manual CT-VOT-13. |
| **Evidência**     | `useFocusEffect` com `setInterval` de 30.000 ms em `FRONTEND/src/app/reunioes/[id].tsx`. |
| **Situação**      | Verificado por inspeção em 13/11/2026. Roteiro manual CT-VOT-13 aprovado em 27/11/2026: o presidente encerrou a votação no Galaxy S24 às 10:29:18 e o emulador Android 14 do membro, sem toque na tela, mostrou o cartão "Encerrada" às 10:29:35, 17 segundos depois. O tempo para uma votação aberta aparecer no outro aparelho não foi medido. |

### RNF-VOT-08 — Portabilidade

| Campo             | Conteúdo |
| ----------------- | -------- |
| **Classificação** | Produto › Portabilidade |
| **Critério**      | A aba "Votações" funciona nos mesmos aparelhos que o restante do aplicativo. |
| **Medida**        | Abrir votações dos dois tipos, votar em opção, em branco e nulo, encerrar e ver o resultado e a lista "Quem votou em quê" sem erro e sem texto cortado em Android 7.0 ou superior e em iOS. |
| **Condição**      | APK de release e aplicativo iOS apontando para a API. |
| **Verificação**   | Roteiros manuais CT-VOT-10 a CT-VOT-14 e CT-VOT-23 a CT-VOT-30. |
| **Evidência**     | Componentes do React Native usados também nas demais telas (`Botao`, `CampoTexto`, `FolhaInferior`, `Confirmacao`). |
| **Situação**      | Parcial: em 27/11/2026, CT-VOT-10 a CT-VOT-13 e CT-VOT-23 a CT-VOT-28 foram aprovados no Galaxy S24 (Android 16) e no emulador Android 14 (API 34), com os APKs de release. Na mesma data, CT-VOT-23 a CT-VOT-30 foram aprovados no emulador Pixel 6 (Android 15), e a folha "Nova votação" com o teclado aberto também no iPhone, pelo Expo Go (CT-VOT-30). Android 7.0 não executado; no iPhone, os demais roteiros não foram executados, e o build nativo não foi executado, porque não há Mac (CT-VOT-14). |

### RNF-VOT-09 — Legibilidade nos dois temas

| Campo             | Conteúdo |
| ----------------- | -------- |
| **Classificação** | Produto › Facilidade de uso |
| **Critério**      | Todas as cores da aba vêm do tema ativo. |
| **Medida**        | Nenhuma cor fixa no componente; textos, faixas, barras do resultado, etiquetas, a seção "Tipo de votação", os botões "Votar em branco" e "Votar nulo", a linha do tipo no cartão e a lista "Quem votou em quê" usam `useTema().tema.cores`. |
| **Condição**      | Tema claro e tema escuro. |
| **Verificação**   | Inspeção do código; roteiro manual CT-VOT-14. |
| **Evidência**     | `FRONTEND/src/components/VotacoesReuniao.tsx`. |
| **Situação**      | Verificado por inspeção em 13/11/2026 e, para os elementos do voto em branco, do voto nulo e do tipo da votação, em 27/11/2026. Roteiro manual CT-VOT-14 executado em parte em 27/11/2026: no Galaxy S24, com o tema escuro, CT-VOT-23 a CT-VOT-28 sem texto cortado e com a seção "Tipo de votação", os botões "Votar em branco" e "Votar nulo" e a lista "Quem votou em quê" legíveis; a repetição de CT-VOT-10 a CT-VOT-13 no tema escuro não foi registrada, e o iOS não foi executado. |

### RNF-VOT-10 — Testabilidade

| Campo             | Conteúdo |
| ----------------- | -------- |
| **Classificação** | Organizacional › Entrega |
| **Critério**      | As regras da votação têm testes automatizados que rodam junto com a suíte do backend. |
| **Medida**        | Os 22 testes de `votacoes.spec.ts` e o restante da suíte passam sem falha. |
| **Condição**      | BACKEND na branch `homolog`, comando `npx vitest run`. |
| **Verificação**   | Execução registrada em [Testes](7%20-%20testes-votacao.md). |
| **Evidência**     | `BACKEND/src/reunioes/votacoes.spec.ts`. |
| **Situação**      | Atendido em 26/11/2026: 16 arquivos e 227 testes aprovados, 9 deles em `votacoes.spec.ts`. Atendido em 27/11/2026, com as alterações do voto em branco, do voto nulo e do tipo da votação: 17 arquivos e 246 testes aprovados, 22 deles em `votacoes.spec.ts`. |

## Limitações conhecidas

- Uma presença em `AGUARDANDO` já permite votar. Se quem agendou recusar essa presença depois, o voto já registrado continua valendo.
- Um presidente de comissão que não foi incluído entre os participantes da reunião não consegue abrir votação nela ("Você não participa desta reunião").
- Quando a reunião é cancelada durante uma votação aberta, a votação continua apresentada como "Em andamento", sem resultado, até o término previsto da reunião. O presidente ainda vê "Encerrar votação", mas a API recusa com "Esta reunião foi cancelada"; os demais veem a faixa "Esta reunião foi cancelada." no lugar das opções.
- Quando a API recusa a abertura de uma votação, a folha "Nova votação" continua aberta e a mensagem fica na faixa vermelha da tela da reunião, atrás da folha.
- A tela só recalcula se a reunião está em andamento quando consulta a reunião (a cada 30 segundos ou ao ser puxada para baixo). Logo depois do término, até a próxima consulta, o presidente ainda pode ver "Abrir votação" e o cartão ainda pode mostrar as opções; a API recusa com "Esta reunião já terminou" ou "Esta votação está encerrada".
- Não há limite para a quantidade de votações por reunião além do tamanho máximo de um documento do MongoDB (16 MB).
- O tipo da votação (fechada ou aberta) é escolhido na abertura e não pode ser trocado depois.
- Na votação do tipo aberta, o nome de cada votante vem da lista de participantes da reunião. Se o votante deixar de ser participante (o organizador pode tirar convidados ao editar a reunião antes do término) ou a conta dele não existir mais, o voto continua contado e a lista "Quem votou em quê" mostra "Usuário indisponível" no lugar do nome.

## Rastreabilidade

RNF-VOT-01 a RNF-VOT-10 ↔ RF-VOT-01 a RF-VOT-04 ↔ CT-VOT-01 a CT-VOT-28.
