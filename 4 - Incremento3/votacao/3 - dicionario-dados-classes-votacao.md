# Dicionário de classes — Votação na reunião

> Padrão da disciplina · Módulo VOT · Complementa o diagrama de classes · Revisado em 13/11/2026

## Resumo

| Classe | Camada | Responsabilidade |
| --- | --- | --- |
| `TelaReuniao` | Aplicativo | Carregar a reunião, descobrir se o usuário é o presidente, calcular se ele pode votar e mostrar a aba "Votações". |
| `VotacoesReuniao` | Aplicativo (componente) | Mostrar as votações, o formulário de abertura, as opções de voto, a confirmação e o resultado. |
| `api` | Aplicativo (serviço) | Enviar a requisição HTTP com `X-API-Key` e o token JWT e devolver o resultado ou a mensagem de erro. |
| `ReunioesController` | API | Receber as rotas HTTP da votação e repassar ao serviço. |
| `ReunioesService` | Negócio | Aplicar autorização, horário, presença, voto único e sigilo. |
| `AbrirVotacaoDto` | API (entrada) | Validar assunto, descrição e opções da nova votação. |
| `VotarDto` | API (entrada) | Validar a opção escolhida. |
| `VotacaoApresentada` | API (saída) | Formato devolvido ao aplicativo, sem a lista de votos. |
| `Votacao` | Subdocumento | Guardar assunto, opções, situação, horários e votos de uma votação. |
| `Voto` | Subdocumento | Registrar a escolha de um participante e o horário. |

As classes `Reuniao`, `Participante`, `Presenca`, `Usuario` e `Comissao` estão descritas no [dicionário de classes das reuniões](../../3%20-%20Incremento2/reunioes/3%20-%20dicionario-dados-classes-reunioes.md); abaixo aparecem só os campos que a votação usa.

## `Votacao`

Subdocumento de `Reuniao`, definido em `BACKEND/src/reunioes/schemas/reuniao.schema.ts`. Diferente da ata, tem `_id` próprio (`@Schema()`), porque uma reunião tem várias votações e cada uma precisa ser encontrada pela rota.

| Atributo | Tipo TypeScript/Mongoose | Tipo BSON | Obrigatório | Regra |
| --- | --- | --- | ---: | --- |
| `_id` | `Types.ObjectId` | `objectId` | Sim | Gerado por `ReunioesService.abrirVotacao` com `new Types.ObjectId()`. Usado na rota como `votacaoId`. |
| `titulo` | `string` | `string` | Sim | Assunto da votação, de 3 a 200 caracteres, sem espaços nas pontas. |
| `descricao` | `string` | `string` | Sim | Até 1.000 caracteres. Padrão: texto vazio. |
| `opcoes` | `string[]` | `array` de `string` | Sim | De 2 a 10 textos de 1 a 80 caracteres, sem repetição (ignorando maiúsculas e minúsculas). A ordem não muda depois da abertura. |
| `situacao` | `SituacaoVotacao` (enum string) | `string` | Sim | `ABERTA` ou `ENCERRADA`. Padrão: `ABERTA`. |
| `aberta_por` | `Types.ObjectId` | `objectId` | Sim | Referência ao presidente que abriu a votação, em `usuarios`. |
| `aberta_em` | `Date` | `date` | Sim | Instante da abertura, em UTC. |
| `encerrada_em` | `Date` | `date` | Não | Instante do encerramento pelo presidente. Se o encerramento acontece depois do fim da reunião, guarda o `fim`. |
| `votos` | `Voto[]` | `array` | Sim | Um item por pessoa que votou. Padrão: lista vazia. |

## `Voto`

Subdocumento de `Votacao`, sem `_id` próprio (`@Schema({ _id: false })`). Cada usuário tem no máximo um voto por votação.

| Atributo | Tipo TypeScript/Mongoose | Tipo BSON | Obrigatório | Regra |
| --- | --- | --- | ---: | --- |
| `usuario_id` | `Types.ObjectId` | `objectId` | Sim | Referência ao usuário em `usuarios`. Não se repete na mesma votação. Nunca é devolvido pela API. |
| `opcao` | `number` (inteiro) | `int32` | Sim | Posição da opção escolhida em `Votacao.opcoes`, de 0 até a quantidade de opções menos 1. |
| `votado_em` | `Date` | `date` | Sim | Instante do voto, em UTC. |

## `VotacaoApresentada`

Objeto montado por `ReunioesService.apresentarVotacao` para cada item de `votacoes` na resposta de qualquer rota que devolve uma reunião. Não é gravado.

| Atributo | Tipo | Regra |
| --- | --- | --- |
| `_id`, `titulo`, `descricao`, `opcoes`, `aberta_em` | iguais aos de `Votacao` | Copiados da votação gravada. |
| `situacao` | `SituacaoVotacao` | `ENCERRADA` quando gravada assim ou quando a reunião já terminou; senão `ABERTA`. |
| `encerrada_em` | `Date` ou `null` | O valor gravado; se a votação expirou com a reunião, o `fim` da reunião; senão `null`. |
| `total_votos` | `number` | Quantidade de itens em `votos`. |
| `meu_voto` | `number` ou `null` | `opcao` do voto de quem consulta, ou `null` se ele não votou. |
| `resultado` | `number[]` ou `null` | Com a votação encerrada, a quantidade de votos de cada opção, na ordem de `opcoes`; aberta, `null`. |

## `Reuniao` (campos usados pela votação)

| Atributo | Tipo | Obrigatório | Regra |
| --- | --- | ---: | --- |
| `votacoes` | `Votacao[]` | Sim | Votações da reunião, na ordem de abertura. Padrão: lista vazia. |
| `participantes` | `Participante[]` | Sim | Define quem consulta as votações; a presença define quem vota. |
| `inicio` e `fim` | `Date` | Sim | A votação só é aberta entre os dois e só recebe votos antes de `fim`. |
| `situacao` | `SituacaoReuniao` | Sim | Nada da votação muda com a reunião `CANCELADA`. |
| `comissao_id` | `ObjectId` | Sim | Comissão em que se procura o presidente. |

## Componente `VotacoesReuniao`

Definido em `FRONTEND/src/components/VotacoesReuniao.tsx`. Recebe da `TelaReuniao` as propriedades abaixo.

| Propriedade | Tipo | Significado |
| --- | --- | --- |
| `reuniao` | `Reuniao` | Reunião com as votações apresentadas. |
| `presidente` | `boolean` | Verdadeiro quando o usuário é `RESPONSAVEL` da comissão ativa; mostra "Abrir votação" e "Encerrar votação". |
| `emAndamento` | `boolean` | Reunião não cancelada, já começada e não terminada; condição para mostrar "Abrir votação". |
| `motivoSemVoto` | `string` ou `null` | Texto mostrado no lugar das opções quando o usuário não pode votar. |
| `salvando` | `string` ou `null` | Chave da operação em andamento; desativa os botões. |
| `executar` | função | Executa a chamada, mostra a mensagem de sucesso ou de erro e atualiza a reunião na tela. |

| Método | O que faz |
| --- | --- |
| `abrirFormulario` | Limpa os campos, volta as opções para "Sim", "Não" e "Abstenção" e abre a folha "Nova votação". |
| `validar` | Confere assunto (mínimo 3 caracteres), opções preenchidas, pelo menos 2 opções e opções sem repetição; devolve a mensagem do primeiro problema ou texto vazio. |
| `enviar` | Chama `validar` e, sem problemas, envia `POST /reunioes/:id/votacoes`; fecha a folha quando dá certo. |
| `pedirVoto` | Abre a confirmação do voto e, confirmada, envia `POST /reunioes/:id/votacoes/:votacaoId/votos`. |
| `pedirEncerramento` | Abre a confirmação do encerramento e, confirmada, envia `POST /reunioes/:id/votacoes/:votacaoId/encerrar`. |
| `renderizarAberta` | Mostra as opções para votar, a faixa "Você votou em …" ou a faixa com o motivo de não poder votar. |
| `renderizarResultado` | Mostra cada opção com quantidade, porcentagem e barra; destaca a mais votada e o voto do usuário. |

## Enumerações

| Enumeração | Valores | Onde aparece |
| --- | --- | --- |
| `SituacaoVotacao` | `ABERTA`, `ENCERRADA` | `Votacao.situacao` e `VotacaoApresentada.situacao` |

## Operações do `ReunioesService`

| Método | Parâmetros | Retorno | O que faz |
| --- | --- | --- | --- |
| `abrirVotacao` | `id`, `AbrirVotacaoDto`, `usuarioId` | reunião atualizada | Confere presidente, reunião em andamento e opções; acrescenta a votação. |
| `votar` | `id`, `votacaoId`, `opcao`, `usuarioId` | reunião atualizada | Confere votação aberta, opção, presença e voto único; acrescenta o voto. |
| `encerrarVotacao` | `id`, `votacaoId`, `usuarioId` | reunião atualizada | Confere presidente e votação aberta; marca `ENCERRADA`. |
| `exigirPresidente` | `reuniao`, `usuarioId` | nada | Busca a comissão e exige o usuário como `RESPONSAVEL`, com a comissão ativa. |
| `buscarVotacao` | `reuniao`, `votacaoId` | votação | Valida o identificador e procura a votação na reunião. |
| `apresentarVotacao` | `votacao`, `fim`, `usuarioId` | `VotacaoApresentada` | Monta a resposta sem a lista de votos e aplica o encerramento automático. |
| `apresentar` | `reuniao`, `usuarioId` | reunião | Copia a reunião e troca `votacoes` pelo formato de apresentação. |

## DTOs e validação

| DTO | Campo | Validação | Mensagem de erro |
| --- | --- | --- | --- |
| `AbrirVotacaoDto` | `titulo` | texto aparado de 3 a 200 caracteres | "Informe o assunto da votação", "Informe um assunto com pelo menos 3 caracteres", "O assunto deve ter até 200 caracteres" |
| | `descricao` | opcional, texto aparado de até 1.000 caracteres | "A descrição deve ser um texto", "A descrição deve ter até 1000 caracteres" |
| | `opcoes` | lista de 2 a 10 textos aparados, cada um de 1 a 80 caracteres | "Informe as opções de voto", "Informe pelo menos 2 opções de voto", "Informe no máximo 10 opções de voto", "Cada opção deve ser um texto", "Preencha todas as opções de voto", "Cada opção deve ter até 80 caracteres" |
| `VotarDto` | `opcao` | inteiro, mínimo 0 | "Escolha uma opção de voto" |

A repetição de opções não é regra do DTO: é conferida por `abrirVotacao`, com a mensagem "Não repita opções de voto". O limite superior de `opcao` também é conferido no serviço, com "Opção de voto inválida".

## Rastreabilidade

RF-VOT-01 a RF-VOT-04; [Diagrama de classes](3%20-%20diagrama-classes-votacao.md) e [Dicionário relacional](4%20-%20dicionario-dados-relacional-votacao.md).
