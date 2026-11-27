# Requisitos não funcionais — Votação na reunião

> Padrão da disciplina · Módulo VOT · Revisado em 27/11/2026

## Resumo

Os requisitos não funcionais aparecem em dois níveis, como os funcionais. Os **de usuário** dizem, em uma frase, a qualidade esperada. Os **de sistema** completam cada um com a medida que o aprova, a condição em que vale, como foi verificado e onde está a evidência, de modo que qualquer pessoa consiga conferir. Onde ainda não houve medição, isso está escrito no campo Situação.

A classificação segue a árvore de Sommerville usada na disciplina: requisitos de **produto** (facilidade de uso, eficiência, com desempenho e espaço, confiabilidade e portabilidade), **organizacionais** (entrega, implementação e padrões) e **externos** (interoperabilidade, éticos e legais, com privacidade e segurança).

## Requisitos não funcionais de usuário

**RNF-VOT-01 — Autorização para gerenciar votações.** Só o presidente da comissão pode abrir e encerrar votações.

**RNF-VOT-02 — Autorização para votar.** Só quem está presente na reunião pode votar.

**RNF-VOT-03 — Voto único.** Cada pessoa vota uma única vez em cada votação, e o voto não pode ser alterado.

**RNF-VOT-04 — Sigilo do voto conforme o tipo da votação.** Na votação do tipo fechada, ninguém, nem o presidente, vê pelo aplicativo em quem cada pessoa votou. Na do tipo aberta, todos os participantes veem quem votou em quê, mas só depois do encerramento.

**RNF-VOT-05 — Resultado só depois do encerramento.** O resultado parcial, inclusive a quantidade de votos em branco e nulos e, na votação do tipo aberta, a lista de quem votou em quê, não pode influenciar quem ainda vai votar.

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
| **Medida**        | Qualquer outra conta recebe HTTP 403 com a mensagem "Apenas o responsável da comissão pode gerenciar votações", e nada é gravado. Na tela, os botões "Abrir votação" e "Encerrar votação" não aparecem para quem não é presidente. |
| **Condição**      | Rotas `POST /reunioes/:id/votacoes` e `POST /reunioes/:id/votacoes/:votacaoId/encerrar`. |
| **Verificação**   | Testes automatizados CT-VOT-02 e CT-VOT-07. |
| **Evidência**     | `ReunioesService.exigirPresidente` e `BACKEND/src/reunioes/votacoes.spec.ts`. |
| **Situação**      | Atendido em 13/11/2026; testes aprovados de novo em 26/11/2026. |

### RNF-VOT-02 — Autorização para votar

| Campo             | Conteúdo |
| ----------------- | -------- |
| **Classificação** | Externo › Legal › Segurança |
| **Critério**      | Vota apenas o participante com presença `AGUARDANDO` ou `CONFIRMADA` e sem saída. |
| **Medida**        | Sem entrada, com entrada recusada ou com saída marcada, a API devolve 403 ("Marque sua entrada na reunião para votar" ou "Você já saiu da reunião e não pode mais votar") e nada é gravado. Pessoa de fora da reunião recebe 403 "Você não participa desta reunião". |
| **Condição**      | Rota `POST /reunioes/:id/votacoes/:votacaoId/votos`. |
| **Verificação**   | Teste automatizado CT-VOT-05. |
| **Evidência**     | `ReunioesService.votar`: conferência antes da gravação e condição sobre a presença repetida no filtro do `findOneAndUpdate`. |
| **Situação**      | Atendido em 13/11/2026; teste aprovado de novo em 26/11/2026. |

### RNF-VOT-03 — Voto único

| Campo             | Conteúdo |
| ----------------- | -------- |
| **Classificação** | Produto › Confiabilidade |
| **Critério**      | Um usuário tem no máximo um item em `votos` por votação, e o item não muda. |
| **Medida**        | A segunda tentativa recebe 409 "Você já votou nesta votação". Não existe rota para trocar ou apagar voto. |
| **Condição**      | Qualquer sequência de chamadas à rota de voto. |
| **Verificação**   | Teste automatizado CT-VOT-05; inspeção do controller (só há `POST` para votos). |
| **Evidência**     | `ReunioesService.votar` e `ReunioesController`. |
| **Situação**      | Atendido em 13/11/2026; teste aprovado de novo em 26/11/2026. |

### RNF-VOT-04 — Sigilo do voto conforme o tipo da votação

| Campo             | Conteúdo |
| ----------------- | -------- |
| **Classificação** | Externo › Legal › Privacidade |
| **Critério**      | Nenhuma resposta da API contém a lista `votos`. Na votação de `modo = FECHADA`, nenhuma resposta contém o `usuario_id` nem o nome de quem votou. Na de `modo = ABERTA`, a lista nominal só é devolvida depois do encerramento. |
| **Medida**        | Cada votação devolvida tem `total_votos`, `meu_voto` (o voto de quem consulta), `resultado` (contagem por opção), `brancos`, `nulos` e `votos_nominais`; a propriedade `votos` não existe na resposta, para nenhum usuário, inclusive o presidente, em nenhum dos dois tipos. `votos_nominais` é `null` sempre que `modo` é `FECHADA` e, com `modo` igual a `ABERTA`, enquanto a votação não está encerrada; encerrada a votação do tipo aberta, traz, para qualquer participante, um item `{ usuario_id, nome, voto }` por voto. Na tela, a lista "Quem votou em quê" só aparece quando `votos_nominais` tem itens. |
| **Condição**      | Todas as rotas que devolvem reunião (`GET /reunioes`, `GET /reunioes/:id` e as rotas de ação). |
| **Verificação**   | Testes automatizados CT-VOT-08 e CT-VOT-19; roteiros manuais CT-VOT-27 e CT-VOT-28. |
| **Evidência**     | `ReunioesService.apresentarVotacao` e `renderizarResultado` em `FRONTEND/src/components/VotacoesReuniao.tsx`. |
| **Situação**      | Para o voto sempre secreto, atendido em 13/11/2026 e teste CT-VOT-08 aprovado de novo em 26/11/2026. Desde 27/11/2026, o sigilo depende do tipo: CT-VOT-08 e CT-VOT-19 aprovados em 27/11/2026; roteiros manuais CT-VOT-27 e CT-VOT-28 não executados. O `usuario_id` do voto fica gravado no banco nos dois tipos, para garantir o voto único: quem tem acesso direto ao MongoDB consegue ligar cada voto a uma pessoa, inclusive na votação fechada. |

### RNF-VOT-05 — Resultado só depois do encerramento

| Campo             | Conteúdo |
| ----------------- | -------- |
| **Classificação** | Externo › Ético |
| **Critério**      | A contagem por opção, a de votos em branco, a de votos nulos e a lista nominal só são devolvidas com a votação encerrada. |
| **Medida**        | Com `situacao = ABERTA`, `resultado`, `brancos`, `nulos` e `votos_nominais` são `null`; com `ENCERRADA` (pelo presidente ou pelo fim da reunião), `resultado` traz uma quantidade por opção, `brancos` e `nulos` trazem números e, só com `modo = ABERTA`, `votos_nominais` traz a lista. Durante a votação, só `total_votos` e `meu_voto` informam algo sobre os votos. |
| **Condição**      | Qualquer consulta à reunião. |
| **Verificação**   | Testes automatizados CT-VOT-08, CT-VOT-09, CT-VOT-17 e CT-VOT-19. |
| **Evidência**     | `ReunioesService.apresentarVotacao`. |
| **Situação**      | Atendido em 13/11/2026; testes CT-VOT-08 e CT-VOT-09 aprovados de novo em 26/11/2026. Os quatro testes aprovados em 27/11/2026, já com os votos em branco, os nulos e a lista nominal. |

### RNF-VOT-06 — Proteção contra gravação simultânea

| Campo             | Conteúdo |
| ----------------- | -------- |
| **Classificação** | Produto › Confiabilidade |
| **Critério**      | As condições de voto e de encerramento são conferidas de novo pelo MongoDB no mesmo comando que grava. |
| **Medida**        | O voto só é gravado se, no instante da gravação, a votação ainda está `ABERTA`, o usuário ainda não está em `votos` e a presença ainda permite votar; o encerramento só é gravado se a votação ainda está `ABERTA`. Caso contrário, a API devolve 409 "A votação mudou. Atualize a tela." e nada é gravado. |
| **Condição**      | Dois votos do mesmo usuário, ou um voto e um encerramento, chegando ao mesmo tempo. |
| **Verificação**   | Testes automatizados CT-VOT-04, que confere o filtro e o `arrayFilters` enviados ao banco, e CT-VOT-07. Teste com requisições paralelas contra um MongoDB real: pendente. |
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
| **Situação**      | Verificado por inspeção em 13/11/2026; roteiro manual pendente. |

### RNF-VOT-08 — Portabilidade

| Campo             | Conteúdo |
| ----------------- | -------- |
| **Classificação** | Produto › Portabilidade |
| **Critério**      | A aba "Votações" funciona nos mesmos aparelhos que o restante do aplicativo. |
| **Medida**        | Abrir votações dos dois tipos, votar em opção, em branco e nulo, encerrar e ver o resultado e a lista "Quem votou em quê" sem erro e sem texto cortado em Android 7.0 ou superior e em iOS. |
| **Condição**      | APK de release e aplicativo iOS apontando para a API. |
| **Verificação**   | Roteiros manuais CT-VOT-10 a CT-VOT-14 e CT-VOT-23 a CT-VOT-28. |
| **Evidência**     | Componentes do React Native usados também nas demais telas (`Botao`, `CampoTexto`, `FolhaInferior`, `Confirmacao`). |
| **Situação**      | Critério definido; execução nos aparelhos pendente. |

### RNF-VOT-09 — Legibilidade nos dois temas

| Campo             | Conteúdo |
| ----------------- | -------- |
| **Classificação** | Produto › Facilidade de uso |
| **Critério**      | Todas as cores da aba vêm do tema ativo. |
| **Medida**        | Nenhuma cor fixa no componente; textos, faixas, barras do resultado, etiquetas, a seção "Tipo de votação", os botões "Votar em branco" e "Votar nulo", a linha do tipo no cartão e a lista "Quem votou em quê" usam `useTema().tema.cores`. |
| **Condição**      | Tema claro e tema escuro. |
| **Verificação**   | Inspeção do código; roteiro manual CT-VOT-14. |
| **Evidência**     | `FRONTEND/src/components/VotacoesReuniao.tsx`. |
| **Situação**      | Verificado por inspeção em 13/11/2026 e, para os elementos do voto em branco, do voto nulo e do tipo da votação, em 27/11/2026; roteiro manual pendente. |

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
- Não há limite para a quantidade de votações por reunião além do tamanho máximo de um documento do MongoDB (16 MB).
- O tipo da votação (fechada ou aberta) é escolhido na abertura e não pode ser trocado depois.
- Na votação do tipo aberta, o nome de cada votante vem da lista de participantes da reunião. Se o votante deixar de ser participante (o organizador pode tirar convidados ao editar a reunião antes do término) ou a conta dele não existir mais, o voto continua contado e a lista "Quem votou em quê" mostra "Usuário indisponível" no lugar do nome.

## Rastreabilidade

RNF-VOT-01 a RNF-VOT-10 ↔ RF-VOT-01 a RF-VOT-04 ↔ CT-VOT-01 a CT-VOT-28.
