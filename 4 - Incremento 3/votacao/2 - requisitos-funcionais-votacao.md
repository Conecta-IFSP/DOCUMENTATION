# Requisitos funcionais — Votação na reunião

> Padrão da disciplina · Módulo VOT · Formulário padrão · Revisado em 13/11/2026

## Resumo

Cada caso de uso de [Casos de uso](1%20-%20casos-uso-votacao.md) é descrito aqui como requisito funcional de sistema, usando apenas o formulário padrão: Função, Descrição, Entradas, Origem, Saída, Destino, Ação, Pré-condição, Pós-condição e Efeitos colaterais. O diagrama de sequência de cada requisito está em [Diagramas de sequência](6%20-%20diagramas-sequencia-votacao.md), com o mesmo número.

Todas as rotas abaixo exigem o cabeçalho `X-API-Key`, um token JWT válido e uma conta de usuário comum ativa (`ApiKeyGuard`, `JwtAuthGuard` e `UsuarioComumGuard`). Os textos de erro citados são os devolvidos pela API e mostrados na faixa vermelha da tela da reunião.

Todas as operações começam pelo método `carregar` do `ReunioesService`, que recusa um identificador inválido ("ID inválido", 400), uma conta inativa ("Usuário sem acesso ativo", 403), uma reunião inexistente ("Reunião não encontrada", 404) e um usuário que não está em `participantes` ("Você não participa desta reunião", 403). Esses erros comuns não são repetidos em cada formulário.

## RF-VOT-01 — Abrir votação

| RF-VOT-01 | Abrir votação |
| --- | --- |
| **Função** | Cria uma votação aberta dentro da reunião. |
| **Descrição** | Acrescenta uma nova votação, com situação `ABERTA` e lista de votos vazia, ao vetor `votacoes` do documento da reunião na coleção `reunioes`. Uma reunião pode ter várias votações, abertas ao mesmo tempo ou não. |
| **Entradas** | Identificador da reunião (`ObjectId` em texto, na rota) e objeto `AbrirVotacaoDto` com: `titulo`, texto de 3 a 200 caracteres, sem os espaços das pontas; `descricao`, opcional, texto de até 1.000 caracteres; e `opcoes`, lista de 2 a 10 textos de 1 a 80 caracteres cada, sem os espaços das pontas e sem repetição (maiúsculas e minúsculas são consideradas iguais). |
| **Origem** | Presidente da comissão (assunto, descrição e opções) e banco de dados (reunião, comissão e presidente). |
| **Saída** | Objeto `Reuniao` atualizado, com a nova votação em `votacoes` no formato de apresentação: `_id`, `titulo`, `descricao`, `opcoes`, `situacao = ABERTA`, `aberta_em`, `encerrada_em = null`, `total_votos = 0`, `meu_voto = null` e `resultado = null`. |
| **Destino** | Aba "Votações" da tela da reunião (`TelaReuniao`, componente `VotacoesReuniao`) e campo `votacoes` do documento em `reunioes`. |
| **Ação** | Na tela da reunião em andamento, o presidente abre a aba "Votações" e toca em "Abrir votação" (o botão só aparece para o presidente e só entre o início e o fim da reunião). A função `abrirFormulario` abre a folha "Nova votação" com os campos "Assunto", "Descrição (opcional)" e as opções "Sim", "Não" e "Abstenção" já preenchidas; o presidente pode alterar, remover (mínimo de 2) e adicionar opções (máximo de 10). Ao tocar em "Abrir votação" na folha, a função `enviar` chama `validar`, que mostra na própria folha "Informe um assunto com pelo menos 3 caracteres", "Preencha todas as opções de voto" ou "Não repita opções de voto"; sem problemas, executa `api('/reunioes/{id}/votacoes', 'POST', { titulo, descricao, opcoes })`. Na API, `ReunioesController.abrirVotacao` chama `ReunioesService.abrirVotacao`, que: (1) carrega a reunião e confere o participante; (2) chama `exigirPresidente`, que busca a comissão por `findById` e exige um membro com o mesmo `usuario_id` e papel `RESPONSAVEL`, com a comissão ativa; (3) chama `exigirAgendada`; (4) exige o horário atual entre `inicio` e `fim`; (5) recusa opções repetidas; (6) grava com `findOneAndUpdate`, filtrando por `_id`, `situacao = AGENDADA` e `fim` maior que o horário atual, e usando `$push` em `votacoes` com um novo `_id`, `aberta_por`, `aberta_em` e `votos` vazio. |
| **Pré-condição** | Usuário autenticado, participante da reunião e presidente (papel `RESPONSAVEL`) de uma comissão ativa; reunião não cancelada e em andamento. |
| **Pós-condição** | O sistema fecha a folha e mostra "Votação aberta." ou o motivo da recusa: assunto, descrição ou opções fora das regras do DTO (400); "Não repita opções de voto" (400); "A reunião ainda não começou" ou "Esta reunião já terminou" (400); "Esta reunião foi cancelada" (400); "Reative a comissão antes de gerenciar votações" (400); "Apenas o responsável da comissão pode gerenciar votações" (403); "Comissão não encontrada" (404); "A reunião mudou. Atualize a tela." (409), quando a reunião foi cancelada ou terminou entre a leitura e a gravação. |
| **Efeitos colaterais** | A aba "Votações" dos participantes presentes passa a mostrar um contador com as votações abertas em que eles ainda não votaram, na próxima atualização da tela (a tela consulta a reunião a cada 30 segundos ou ao ser puxada para baixo). |

## RF-VOT-02 — Votar

| RF-VOT-02 | Votar |
| --- | --- |
| **Função** | Registra o voto de um participante presente. |
| **Descrição** | Acrescenta um voto, com o usuário, o índice da opção escolhida e o horário, à lista `votos` da votação, dentro do documento da reunião. Cada usuário vota uma única vez por votação. |
| **Entradas** | Identificador da reunião e identificador da votação (`ObjectId` em texto, na rota) e objeto `VotarDto` com `opcao`, número inteiro maior ou igual a 0 que indica a posição da opção na lista `opcoes` (0 é a primeira). |
| **Origem** | Participante presente (opção escolhida) e banco de dados (votação, presença e votos já registrados). |
| **Saída** | Objeto `Reuniao` atualizado, em que a votação mostra `total_votos` aumentado em 1 e `meu_voto` igual ao índice escolhido. |
| **Destino** | Cartão da votação na aba "Votações" da `TelaReuniao` e campo `votacoes[].votos` do documento em `reunioes`. |
| **Ação** | Na aba "Votações", o participante presente vê as opções da votação aberta. Quem não pode votar vê, no lugar das opções, uma faixa com o motivo: "Marque sua entrada na reunião para votar.", "Você já saiu da reunião e não pode mais votar." ou "Esta reunião foi cancelada.". Ao tocar em uma opção, a função `pedirVoto` abre a confirmação `Votar em "<opção>"? O voto não pode ser alterado depois.`. Ao tocar em "Votar", a tela executa `api('/reunioes/{id}/votacoes/{votacaoId}/votos', 'POST', { opcao })`. Na API, `ReunioesController.votar` chama `ReunioesService.votar`, que: (1) carrega a reunião e o participante; (2) chama `exigirAgendada`; (3) chama `buscarVotacao`, que procura a votação pelo `_id` em `votacoes`; (4) exige a votação `ABERTA` e a reunião ainda não terminada; (5) exige `opcao` menor que a quantidade de opções; (6) exige presença registrada, não `RECUSADA` e sem `saida`; (7) recusa o usuário que já está em `votos`; (8) grava com `findOneAndUpdate`, cujo filtro repete as condições 4, 6 e 7 dentro do banco (`$elemMatch` em `votacoes` com `votos.usuario_id` diferente do usuário, e `$elemMatch` em `participantes` com presença `AGUARDANDO` ou `CONFIRMADA` e sem `saida`), usando `$push` em `votacoes.$[votacao].votos` com `arrayFilters` pelo `_id` da votação. |
| **Pré-condição** | Usuário participante da reunião, com entrada marcada, presença não recusada e sem saída; reunião não cancelada e não terminada; votação aberta; usuário ainda não votou nesta votação. |
| **Pós-condição** | O sistema mostra "Voto registrado." e troca as opções pela faixa verde `Você votou em "<opção>"`, ou mostra o motivo da recusa: "Escolha uma opção de voto" (400, DTO); "Opção de voto inválida" (400); "Esta votação está encerrada" (400); "Esta reunião foi cancelada" (400); "ID inválido" (400); "Marque sua entrada na reunião para votar" (403); "Você já saiu da reunião e não pode mais votar" (403); "Votação não encontrada" (404); "Você já votou nesta votação" (409); "A votação mudou. Atualize a tela." (409), quando a votação foi encerrada, a presença mudou ou outro voto do mesmo usuário chegou entre a leitura e a gravação. |
| **Efeitos colaterais** | O voto não pode ser trocado nem apagado. A votação some do contador da aba "Votações" desse usuário. A identidade de quem votou fica gravada no banco, mas nunca é devolvida pela API. |

## RF-VOT-03 — Encerrar votação

| RF-VOT-03 | Encerrar votação |
| --- | --- |
| **Função** | Encerra uma votação aberta. |
| **Descrição** | Muda a situação da votação para `ENCERRADA` e grava o horário do encerramento, impedindo novos votos e liberando o resultado. |
| **Entradas** | Identificador da reunião e identificador da votação (`ObjectId` em texto, na rota). Não há corpo. |
| **Origem** | Presidente da comissão (comando) e banco de dados (votação e comissão). |
| **Saída** | Objeto `Reuniao` atualizado, em que a votação mostra `situacao = ENCERRADA`, `encerrada_em` preenchido e `resultado` com a quantidade de votos de cada opção, na mesma ordem de `opcoes`. |
| **Destino** | Cartão da votação na aba "Votações" da `TelaReuniao` e campos `votacoes[].situacao` e `votacoes[].encerrada_em` do documento em `reunioes`. |
| **Ação** | No cartão de uma votação aberta, o presidente toca em "Encerrar votação". A função `pedirEncerramento` abre a confirmação `Encerrar a votação "<assunto>"? Ninguém mais poderá votar.`, com o botão "Encerrar" em vermelho. Confirmado, a tela executa `api('/reunioes/{id}/votacoes/{votacaoId}/encerrar', 'POST')`. Na API, `ReunioesController.encerrarVotacao` chama `ReunioesService.encerrarVotacao`, que: (1) carrega a reunião; (2) chama `exigirPresidente`; (3) chama `exigirAgendada`; (4) chama `buscarVotacao`; (5) recusa a votação já `ENCERRADA`; (6) grava com `findOneAndUpdate`, filtrando pela reunião `AGENDADA` e pela votação ainda `ABERTA` (`$elemMatch`), e usando `$set` em `votacoes.$.situacao` e `votacoes.$.encerrada_em`. O horário gravado é o atual ou, se a reunião já terminou, o `fim` da reunião. |
| **Pré-condição** | Usuário participante da reunião e presidente de uma comissão ativa; reunião não cancelada; votação gravada como `ABERTA`. |
| **Pós-condição** | O sistema mostra "Votação encerrada." e o cartão passa a exibir o resultado, ou o motivo da recusa: "Esta votação já foi encerrada" (400); "Esta reunião foi cancelada" (400); "Reative a comissão antes de gerenciar votações" (400); "ID inválido" (400); "Apenas o responsável da comissão pode gerenciar votações" (403); "Votação não encontrada" ou "Comissão não encontrada" (404); "A votação mudou. Atualize a tela." (409), quando outra pessoa encerrou a votação ao mesmo tempo. |
| **Efeitos colaterais** | Votos enviados depois do encerramento são recusados pelo filtro da gravação (RF-VOT-02). A votação não pode ser reaberta. |

## RF-VOT-04 — Consultar votações e resultado

| RF-VOT-04 | Consultar votações e resultado |
| --- | --- |
| **Função** | Mostra as votações da reunião, o próprio voto e o resultado das encerradas. |
| **Descrição** | Ao devolver uma reunião, a API transforma cada votação gravada em um formato de apresentação que não contém a lista de votos: só o total, o voto de quem consulta e, depois do encerramento, a contagem por opção. Uma votação gravada como `ABERTA` em uma reunião que já terminou é apresentada como `ENCERRADA`, com `encerrada_em` igual ao `fim` da reunião. |
| **Entradas** | Identificador da reunião (`GET /reunioes/:id`) ou o período e a comissão da agenda (`GET /reunioes`); usuário do token JWT. |
| **Origem** | Banco de dados (documento da reunião com `votacoes`) e sessão do usuário. |
| **Saída** | Campo `votacoes` da reunião, uma lista em que cada item tem: `_id`; `titulo`; `descricao` (texto vazio quando não informada); `opcoes`; `situacao` (`ABERTA` ou `ENCERRADA`); `aberta_em`; `encerrada_em` (ou `null`); `total_votos`; `meu_voto` (índice da opção escolhida pelo usuário ou `null`); e `resultado` (lista com a quantidade de votos de cada opção quando encerrada, ou `null` enquanto aberta). |
| **Destino** | Aba "Votações" da `TelaReuniao`, componente `VotacoesReuniao`. |
| **Ação** | Ao abrir a tela da reunião (e a cada 30 segundos, ou quando a tela é puxada para baixo), a função `buscar` executa `api('/reunioes/{id}')` e, em seguida, `api('/comissoes/{comissaoId}')` para saber se o usuário é o presidente (membro `RESPONSAVEL` de comissão ativa). Na API, `ReunioesController.buscarPorId` chama `ReunioesService.buscarPorId`, que carrega a reunião, popula comissão e participantes e chama `apresentar`; este chama `apresentarVotacao` para cada votação. Na tela, `VotacoesReuniao` ordena as votações com as abertas primeiro e, dentro de cada grupo, as mais recentes primeiro. Cada cartão mostra o assunto, a etiqueta "Aberta" ou "Encerrada", a descrição, o total de votos e os horários de abertura e encerramento. Nas encerradas, a função `renderizarResultado` mostra cada opção com quantidade, porcentagem e uma barra, destaca a(s) opção(ões) mais votada(s) e marca a opção escolhida pelo usuário com "seu voto". Sem votações, aparece "Nenhuma votação nesta reunião." (ou, para o presidente durante a reunião, "Nenhuma votação ainda. Abra uma para os participantes votarem."). |
| **Pré-condição** | Usuário autenticado e participante da reunião. |
| **Pós-condição** | As votações são exibidas, ou a tela mostra o motivo da recusa da consulta (erros comuns de `carregar`). |
| **Efeitos colaterais** | Nenhum dado é gravado. A aba "Votações" mostra um contador com as votações abertas em que o usuário ainda não votou, desde que ele possa votar. |
