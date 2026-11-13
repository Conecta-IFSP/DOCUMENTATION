# Dicionário relacional — Votação na reunião

> Padrão da disciplina · Módulo VOT · Campos do diagrama relacional · Revisado em 13/11/2026

## Resumo

Descreve cada campo das entidades do [diagrama relacional](4%20-%20diagrama-relacional-votacao.md), com tipo, obrigatoriedade, chaves e forma de geração. As colunas seguem o dicionário de dados do documento de exemplo da disciplina: campo, tipo de dado, se pode ser nulo, chave primária, chave estrangeira, outros e descrição.

## Como os identificadores são criados

| Identificador | Quem gera | Formato e valor inicial | Incremento |
| --- | --- | --- | --- |
| `VOTACAO._id` | `ReunioesService.abrirVotacao`, com `new Types.ObjectId()` | `ObjectId` de 12 bytes, mostrado como 24 caracteres hexadecimais: 4 bytes com a hora da criação em segundos, 5 bytes sorteados uma vez por processo e 3 bytes de um contador interno. Não tem valor inicial fixo. | Não é sequencial nem previsível. |
| `VOTACAO_OPCAO` (chave lógica) | Não é gerado | A opção é identificada pela votação e pela posição na lista `opcoes`, começando em 0. | A posição segue a ordem informada na abertura e não muda depois. |
| `VOTO` (chave lógica) | Não é gerado | O voto é identificado pela votação que o contém e pelo `usuario_id`. | Não há incremento: cada usuário tem no máximo um voto por votação. |

## VOTACAO

Uma votação aberta durante a reunião. Fica em `reunioes.votacoes[]`.

| Campo | Tipo de dado | Pode ser nulo | Chave primária | Chave estrangeira | Outros | Descrição |
| --- | --- | --- | --- | --- | --- | --- |
| `_id` | `ObjectId` | Não | Sim (exclusivo) | Não | Gerado na abertura. | Identificador usado nas rotas de voto e encerramento. |
| `reuniao_id` | `ObjectId` | Não | Não | Sim (`REUNIAO._id`) | Não é gravado: é o documento que contém a votação. | Reunião em que a votação acontece. |
| `titulo` | Texto (`string`, 3 a 200) | Não | Não | Não | Sem espaços nas pontas. | Assunto votado. |
| `descricao` | Texto (`string`, até 1.000) | Não | Não | Não | Padrão: texto vazio. | Explicação opcional do assunto. |
| `situacao` | Texto com valores fixos | Não | Não | Não | `ABERTA` ou `ENCERRADA`; padrão `ABERTA`. | Fase da votação. A API apresenta como `ENCERRADA` a votação aberta de uma reunião que já terminou. |
| `aberta_por` | `ObjectId` | Não | Não | Sim (`USUARIO._id`) | Membro `RESPONSAVEL` da comissão. | Presidente que abriu a votação. |
| `aberta_em` | Data e hora (`date`, UTC) | Não | Não | Não | Entre `REUNIAO.inicio` e `REUNIAO.fim`. | Momento da abertura. |
| `encerrada_em` | Data e hora (`date`, UTC) | Sim | Não | Não | Nulo enquanto aberta; nunca depois de `REUNIAO.fim`. | Momento do encerramento pelo presidente. |

## VOTACAO_OPCAO

Uma alternativa de voto. Fica em `reunioes.votacoes[].opcoes[]`, como uma lista de textos.

| Campo | Tipo de dado | Pode ser nulo | Chave primária | Chave estrangeira | Outros | Descrição |
| --- | --- | --- | --- | --- | --- | --- |
| `votacao_id` | `ObjectId` | Não | Sim (inclusivo) | Sim (`VOTACAO._id`) | Não é gravado: vem da votação que contém a lista. | Votação a que a opção pertence. |
| `indice` | Inteiro (`int`) | Não | Sim (inclusivo) | Não | Não é gravado: é a posição na lista, de 0 a 9. | Número usado no voto para indicar a opção. |
| `texto` | Texto (`string`, 1 a 80) | Não | Não | Não | Único na votação, ignorando maiúsculas e minúsculas. Padrão sugerido pela tela: "Sim", "Não" e "Abstenção". | Texto mostrado ao votar e no resultado. |

Uma votação tem no mínimo 2 e no máximo 10 opções.

## VOTO

A escolha de um participante presente. Fica em `reunioes.votacoes[].votos[]`.

| Campo | Tipo de dado | Pode ser nulo | Chave primária | Chave estrangeira | Outros | Descrição |
| --- | --- | --- | --- | --- | --- | --- |
| `votacao_id` | `ObjectId` | Não | Sim (inclusivo) | Sim (`VOTACAO._id`) | Não é gravado: vem da votação que contém o voto. | Votação em que se votou. |
| `usuario_id` | `ObjectId` | Não | Sim (inclusivo) | Sim (`USUARIO._id`) | Não se repete na mesma votação; nunca é devolvido pela API. | Quem votou. |
| `opcao` | Inteiro (`int`, mínimo 0) | Não | Não | Sim (`VOTACAO_OPCAO.indice`) | Menor que a quantidade de opções. | Opção escolhida. |
| `votado_em` | Data e hora (`date`, UTC) | Não | Não | Não | Antes de `REUNIAO.fim`. | Momento do voto. |

## REUNIAO (campos usados pela votação)

| Campo | Tipo de dado | Pode ser nulo | Chave primária | Chave estrangeira | Outros | Descrição |
| --- | --- | --- | --- | --- | --- | --- |
| `_id` | `ObjectId` | Não | Sim (exclusivo) | Não | - | Identificador da reunião. |
| `comissao_id` | `ObjectId` | Não | Não | Sim (`COMISSAO._id`) | - | Comissão em que se procura o presidente. |
| `inicio` | Data e hora (`date`, UTC) | Não | Não | Não | - | A votação só abre a partir deste instante. |
| `fim` | Data e hora (`date`, UTC) | Não | Não | Não | - | Depois deste instante não se abre votação nem se vota. |
| `situacao` | Texto com valores fixos | Não | Não | Não | `AGENDADA` ou `CANCELADA`. | Com `CANCELADA`, a votação não muda. |
| `votacoes` | Lista de `VOTACAO` | Não | Não | Não | Padrão: lista vazia. Documentos antigos sem o campo são lidos como lista vazia. | Votações da reunião. |

## REUNIAO_PRESENCA (campos usados pela votação)

| Campo | Tipo de dado | Pode ser nulo | Chave primária | Chave estrangeira | Outros | Descrição |
| --- | --- | --- | --- | --- | --- | --- |
| `situacao` | Texto com valores fixos | Não | Não | Não | `AGUARDANDO`, `CONFIRMADA` ou `RECUSADA`. | Vota quem está em `AGUARDANDO` ou `CONFIRMADA`. |
| `saida` | Data e hora (`date`, UTC) | Sim | Não | Não | - | Preenchida, impede o voto. |

## Índices

A votação não cria índices. As consultas usam o `_id` da reunião, que já é indexado; a votação e o voto são encontrados dentro do documento com `$elemMatch` e `arrayFilters`.

## Rastreabilidade

RF-VOT-01 a RF-VOT-04; [Diagrama relacional](4%20-%20diagrama-relacional-votacao.md) e [Dicionário de classes](3%20-%20dicionario-dados-classes-votacao.md).
