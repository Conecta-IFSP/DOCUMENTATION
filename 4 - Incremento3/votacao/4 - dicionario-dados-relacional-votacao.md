# Dicionário relacional — Votação na reunião

> Padrão da disciplina · Módulo VOT · Campos do diagrama relacional · Revisado em 26/11/2026

## Resumo

Descreve cada campo das entidades do [diagrama relacional](4%20-%20diagrama-relacional-votacao.md), com tipo, obrigatoriedade, chaves e forma de geração. As colunas seguem o dicionário de dados do documento de exemplo da disciplina: campo, tipo de dado, se pode ser nulo, chave primária, chave estrangeira, outros e descrição. Na coluna "Outros", cada restrição é explicada com a regra e o ponto do sistema que a garante: o formulário do aplicativo, a validação de entrada da API (DTO), o serviço da API antes de gravar, o comando de gravação no banco ou a montagem da resposta.

## Como os identificadores são criados

| Identificador                       | Quem cria                                                                                                    | Formato e valor inicial                                                                                                                                                                                                                                         | Incremento                                                                                                                                  |
| ----------------------------------- | ------------------------------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------- |
| `VOTACAO._id`                       | O serviço da API, em `ReunioesService.abrirVotacao`, com `new Types.ObjectId()`, antes de enviar a gravação. | `ObjectId` de 12 bytes, mostrado como 24 caracteres hexadecimais: 4 bytes com os segundos passados desde 01/01/1970 no momento da criação, 5 bytes sorteados uma vez por processo e 3 bytes de um contador que começa num valor sorteado. Não tem valor inicial fixo. | O contador soma 1 a cada novo identificador criado pelo processo da API; por isso dois identificadores seguidos diferem só no contador e não são previsíveis por quem está fora do servidor. |
| `VOTACAO_OPCAO` (chave lógica)      | Não é criada.                                                                                                | A votação mais o `indice`, que é a posição do texto na lista `opcoes`. A primeira opção tem `indice` 0.                                                                                                                                                        | Sobe de 1 em 1 na ordem informada na abertura, até 9. A ordem não muda depois.                                                              |
| `VOTO` (chave lógica)               | Não é criada.                                                                                                | A votação que contém o voto mais o `usuario_id` de quem votou.                                                                                                                                                                                                 | Não há incremento: cada usuário tem no máximo um voto por votação.                                                                          |
| `REUNIAO._id`, `USUARIO._id`, `COMISSAO._id` | Criados fora deste módulo, quando a reunião, o usuário ou a comissão são inseridos.                 | `ObjectId`, com a mesma formação de `VOTACAO._id`. A criação de `REUNIAO._id` está no [dicionário relacional das reuniões](../../3%20-%20Incremento2/reunioes/4%20-%20dicionario-dados-relacional-reunioes.md). | Igual ao de `VOTACAO._id`.                                                                                                                 |

## REUNIAO (campos usados pela votação)

É a coleção `reunioes`. A votação só lê estes campos.

| Campo         | Tipo de dado              | Pode ser nulo | Chave primária  | Chave estrangeira    | Outros                                                                                  | Descrição                                       |
| ------------- | ------------------------- | ------------- | --------------- | -------------------- | --------------------------------------------------------------------------------------- | ----------------------------------------------- |
| `_id`         | `ObjectId`                | Não           | Sim (exclusiva) | Não                  | Ver "Como os identificadores são criados".                                              | Identificador da reunião.                       |
| `comissao_id` | `ObjectId`                | Não           | Não             | Sim (`COMISSAO._id`) | O serviço procura o presidente nos membros desta comissão.                              | Comissão da reunião.                            |
| `inicio`      | Data e hora (`date`, UTC) | Não           | Não             | Não                  | Antes deste instante o serviço não abre votação.                                        | Início da reunião.                              |
| `fim`         | Data e hora (`date`, UTC) | Não           | Não             | Não                  | A partir deste instante não se abre votação nem se vota, e a resposta passa a mostrar as votações abertas como encerradas. | Término da reunião.       |
| `situacao`    | Texto com valores fixos   | Não           | Não             | Não                  | `AGENDADA` ou `CANCELADA`. Com `CANCELADA`, o serviço e o comando de gravação recusam qualquer mudança na votação. | Indica se a reunião continua valendo. |

O documento também tem a lista `votacoes`, que começa vazia; documentos gravados antes da existência desse campo são lidos como lista vazia.

## REUNIAO_PARTICIPANTE (campos usados pela votação)

Fica em `reunioes.participantes[]`.

| Campo        | Tipo de dado            | Pode ser nulo | Chave primária  | Chave estrangeira   | Outros                                                                   | Descrição                         |
| ------------ | ----------------------- | ------------- | --------------- | ------------------- | ------------------------------------------------------------------------ | --------------------------------- |
| `reuniao_id` | `ObjectId`              | Não           | Sim (inclusiva) | Sim (`REUNIAO._id`) | Não é gravado: é o documento que contém o participante.                  | Reunião.                          |
| `usuario_id` | `ObjectId`              | Não           | Sim (inclusiva) | Sim (`USUARIO._id`) | Quem não está nesta lista não consulta a reunião nem vota.               | Participante.                     |
| `convite`    | Texto com valores fixos | Não           | Não             | Não                 | `PENDENTE`, `ACEITO` ou `RECUSADO`. A votação não usa este campo.        | Resposta ao convite.              |

## REUNIAO_PRESENCA (campos usados pela votação)

Fica em `reunioes.participantes[].presenca`.

| Campo        | Tipo de dado              | Pode ser nulo | Chave primária  | Chave estrangeira                       | Outros                                                                                                  | Descrição                       |
| ------------ | ------------------------- | ------------- | --------------- | --------------------------------------- | ------------------------------------------------------------------------------------------------------- | ------------------------------- |
| `reuniao_id` | `ObjectId`                | Não           | Sim (inclusiva) | Sim (`REUNIAO_PARTICIPANTE.reuniao_id`) | Não é gravado: vem do participante.                                                                     | Reunião.                        |
| `usuario_id` | `ObjectId`                | Não           | Sim (inclusiva) | Sim (`REUNIAO_PARTICIPANTE.usuario_id`) | Não é gravado: vem do participante. Sem presença, o serviço recusa o voto.                              | Participante.                   |
| `entrada`    | Data e hora (`date`, UTC) | Não           | Não             | Não                                     | A existência da entrada é o que permite votar.                                                          | Momento da entrada.             |
| `saida`      | Data e hora (`date`, UTC) | Sim           | Não             | Não                                     | Preenchida, impede o voto; o serviço e o comando de gravação conferem.                                  | Momento da saída.               |
| `situacao`   | Texto com valores fixos   | Não           | Não             | Não                                     | `AGUARDANDO`, `CONFIRMADA` ou `RECUSADA`. Vota quem está em `AGUARDANDO` ou `CONFIRMADA`.               | Situação da presença.           |

## VOTACAO

Uma votação aberta durante a reunião. Fica em `reunioes.votacoes[]`.

| Campo          | Tipo de dado              | Pode ser nulo | Chave primária  | Chave estrangeira   | Outros                                                                                                                                       | Descrição                                               |
| -------------- | ------------------------- | ------------- | --------------- | ------------------- | -------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------- |
| `_id`          | `ObjectId`                | Não           | Sim (exclusiva) | Não                 | Ver "Como os identificadores são criados".                                                                                                   | Identificador usado nas rotas de voto e encerramento.   |
| `reuniao_id`   | `ObjectId`                | Não           | Não             | Sim (`REUNIAO._id`) | Não é gravado: é o documento que contém a votação.                                                                                           | Reunião em que a votação acontece.                      |
| `titulo`       | Texto (3 a 200)           | Não           | Não             | Não                 | O formulário exige ao menos 3 caracteres; o DTO exige de 3 a 200, depois de retirar os espaços das pontas; o schema repete os limites.       | Assunto votado.                                         |
| `descricao`    | Texto (até 1.000)         | Não           | Não             | Não                 | Opcional na entrada; sem descrição, grava texto vazio.                                                                                       | Explicação do assunto.                                  |
| `situacao`     | Texto com valores fixos   | Não           | Não             | Não                 | `ABERTA` ou `ENCERRADA`. Nasce `ABERTA` e só muda uma vez, pelo encerramento. Na resposta, uma votação aberta de reunião terminada aparece como `ENCERRADA`, sem que o banco seja alterado. | Fase da votação.    |
| `aberta_por`   | `ObjectId`                | Não           | Não             | Sim (`USUARIO._id`) | Gravado pelo serviço com o usuário do token, depois de conferir que ele é o `RESPONSAVEL` da comissão.                                      | Presidente que abriu a votação.                         |
| `aberta_em`    | Data e hora (`date`, UTC) | Não           | Não             | Não                 | Relógio do servidor; o serviço só abre entre o início e o término da reunião.                                                               | Momento da abertura.                                    |
| `encerrada_em` | Data e hora (`date`, UTC) | Sim           | Não             | Não                 | Nulo enquanto aberta. No encerramento, o serviço grava o menor entre o horário atual e o término da reunião.                                | Momento do encerramento.                                |

## VOTACAO_OPCAO

Uma alternativa de voto. Fica em `reunioes.votacoes[].opcoes[]`, como uma lista de textos.

| Campo        | Tipo de dado     | Pode ser nulo | Chave primária  | Chave estrangeira   | Outros                                                                                                                                         | Descrição                                   |
| ------------ | ---------------- | ------------- | --------------- | ------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------- |
| `votacao_id` | `ObjectId`       | Não           | Sim (inclusiva) | Sim (`VOTACAO._id`) | Não é gravado: vem da votação que contém a lista.                                                                                              | Votação a que a opção pertence.             |
| `indice`     | Inteiro (`int`)  | Não           | Sim (inclusiva) | Não                 | Não é gravado: é a posição na lista, de 0 a 9. Ver "Como os identificadores são criados".                                                     | Número usado no voto para indicar a opção.  |
| `texto`      | Texto (1 a 80)   | Não           | Não             | Não                 | O DTO exige de 1 a 80 caracteres, depois de retirar os espaços das pontas. Não se repete na votação, sem diferenciar maiúsculas de minúsculas: o formulário avisa e o serviço recusa. O formulário sugere "Sim", "Não" e "Abstenção". | Texto mostrado ao votar e no resultado. |

Uma votação tem de 2 a 10 opções: o formulário só permite remover opções enquanto houver mais de 2 e só oferece "Adicionar opção" até 10, e o DTO recusa listas fora desse intervalo.

## VOTO

A escolha de um participante presente. Fica em `reunioes.votacoes[].votos[]`.

| Campo        | Tipo de dado              | Pode ser nulo | Chave primária  | Chave estrangeira              | Outros                                                                                                                                                                                   | Descrição                   |
| ------------ | ------------------------- | ------------- | --------------- | ------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------- |
| `votacao_id` | `ObjectId`                | Não           | Sim (inclusiva) | Sim (`VOTACAO._id`)            | Não é gravado: vem da votação que contém o voto. O voto só é acrescentado se essa votação ainda estiver aberta no momento da gravação.                                                  | Votação em que se votou.    |
| `usuario_id` | `ObjectId`                | Não           | Sim (inclusiva) | Sim (`USUARIO._id`)            | Gravado pelo serviço com o usuário do token. Não se repete na mesma votação: o serviço recusa o segundo voto e o comando de gravação só insere se a pessoa ainda não votou. Nunca sai na resposta da API. | Quem votou.   |
| `opcao`      | Inteiro (`int`, mínimo 0) | Não           | Não             | Sim (`VOTACAO_OPCAO.indice`)   | O DTO exige inteiro maior ou igual a 0; o serviço exige valor menor que a quantidade de opções.                                                                                         | Opção escolhida.            |
| `votado_em`  | Data e hora (`date`, UTC) | Não           | Não             | Não                            | Relógio do servidor; o comando de gravação só aceita antes do término da reunião.                                                                                                       | Momento do voto.            |

## COMISSAO e USUARIO (campos usados)

| Entidade   | Campo    | Tipo de dado        | Pode ser nulo | Chave primária  | Chave estrangeira | Outros                                                                                  | Descrição                         |
| ---------- | -------- | ------------------- | ------------- | --------------- | ----------------- | --------------------------------------------------------------------------------------- | --------------------------------- |
| `COMISSAO` | `_id`    | `ObjectId`          | Não           | Sim (exclusiva) | Não               | Ver "Como os identificadores são criados".                                              | Identificador da comissão.        |
| `COMISSAO` | `ativo`  | Lógico (`bool`)     | Não           | Não             | Não               | Com falso, o serviço recusa abrir e encerrar votações.                                  | Indica comissão ativa.            |
| `USUARIO`  | `_id`    | `ObjectId`          | Não           | Sim (exclusiva) | Não               | Ver "Como os identificadores são criados".                                              | Identificador do usuário.         |
| `USUARIO`  | `nome`   | Texto               | Não           | Não             | Não               | Não aparece nas votações: o resultado não mostra nomes.                                 | Nome do usuário.                  |
| `USUARIO`  | `ativo`  | Lógico (`bool`)     | Não           | Não             | Não               | Conta inativa é recusada antes de qualquer operação.                                    | Indica conta ativa.               |

## Índices

A votação não cria índices. Toda operação localiza a reunião pelo `_id`, que o MongoDB já indexa, e encontra a votação e o participante dentro do próprio documento.

## Rastreabilidade

RF-VOT-01 a RF-VOT-04; [Diagrama relacional](4%20-%20diagrama-relacional-votacao.md) e [Dicionário de classes](3%20-%20dicionario-dados-classes-votacao.md).
