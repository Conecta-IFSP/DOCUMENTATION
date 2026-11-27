# Dicionário relacional — Reuniões

> Padrão da disciplina · Módulo REU · Campos do diagrama relacional · Revisado em 26/11/2026

## Resumo

Descreve cada campo das entidades do [diagrama relacional](4%20-%20diagrama-relacional-reunioes.md), com tipo, obrigatoriedade, chaves e forma de geração. As colunas seguem o dicionário de dados do documento de exemplo da disciplina: campo, tipo de dado, se pode ser nulo, chave primária, chave estrangeira, outros e descrição. Na coluna "Outros", cada restrição é explicada com a regra e o ponto do sistema que a garante: o formulário do aplicativo, a validação de entrada da API (DTO), o serviço da API antes de gravar ou o próprio comando de gravação no banco.

## Como os identificadores são criados

Os identificadores do tipo `ObjectId` têm 12 bytes, mostrados como 24 caracteres hexadecimais. Os 4 primeiros bytes são os segundos passados desde 01/01/1970 no momento da criação; os 5 seguintes são um valor sorteado uma vez por processo; os 3 últimos são um contador que começa num valor sorteado e soma 1 a cada novo identificador criado pelo mesmo processo. Por isso o `ObjectId` não tem valor inicial fixo e não cresce de 1 em 1 de forma visível: dois identificadores seguidos só diferem no contador, e identificadores de processos diferentes diferem também no valor sorteado.

| Identificador                       | Quem cria                                                                                                   | Formato e valor inicial                                                                                   | Incremento                                                                                        |
| ----------------------------------- | ----------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------- |
| `REUNIAO._id`                       | O Mongoose, no processo da API, ao montar o documento em `ReuniaoModel.create`, antes de enviá-lo ao banco. | `ObjectId`, como descrito acima.                                                                          | Contador do processo da API, mais 1 a cada identificador criado.                                  |
| `ARQUIVO_GRIDFS._id` = `REUNIAO_ANEXO._id` | O driver do MongoDB, no processo da API, ao abrir a gravação com `GridFSBucket.openUploadStream`.    | `ObjectId`. O mesmo valor é copiado para o anexo da reunião.                                              | Contador do processo da API, mais 1 a cada identificador criado.                                  |
| `PEDACO_GRIDFS._id`                 | O driver do MongoDB, ao gravar cada pedaço do arquivo.                                                      | `ObjectId`.                                                                                               | Um novo identificador por pedaço.                                                                 |
| `PEDACO_GRIDFS.n`                   | O driver do MongoDB.                                                                                        | Inteiro; o primeiro pedaço de cada arquivo é 0.                                                           | Sobe de 1 em 1 a cada pedaço do mesmo arquivo.                                                    |
| `NOTIFICACAO._id`                   | O servidor do MongoDB, quando o `updateOne` com `upsert` de `NotificacoesService.criar` insere a notificação. | `ObjectId`, com o valor sorteado e o contador do processo do servidor do banco.                         | Contador do servidor do banco, mais 1 a cada identificador criado.                                |
| Chave de `REUNIAO_PARTICIPANTE` e de `REUNIAO_PRESENCA` | Não é criada.                                                                   | A reunião mais o `usuario_id`, que já existe em `usuarios`.                                               | Não se aplica.                                                                                    |
| Chave de `LINK_PRESENCA`            | Não é criada.                                                                                               | A própria reunião: há no máximo um link por reunião.                                                      | Não se aplica. Gerar de novo substitui o link.                                                   |
| `CORRECAO_PRESENCA.posicao`         | Não é gravada: é a posição do registro na lista `historico_presencas`.                                     | Inteiro; a primeira correção de cada reunião é 0.                                                         | Sobe de 1 em 1 a cada correção, porque o registro é sempre acrescentado ao fim da lista (`$push`). |

## REUNIAO

Guarda os dados de cada reunião agendada. É a coleção `reunioes`.

| Campo                 | Tipo de dado                   | Pode ser nulo | Chave primária  | Chave estrangeira    | Outros                                                                                                        | Descrição                                                                           |
| --------------------- | ------------------------------ | ------------- | --------------- | -------------------- | ------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------- |
| `_id`                 | `ObjectId`                     | Não           | Sim (exclusiva) | Não                  | Ver "Como os identificadores são criados".                                                                    | Identificador da reunião.                                                           |
| `comissao_id`         | `ObjectId`                     | Não           | Não             | Sim (`COMISSAO._id`) | A comissão precisa estar ativa no agendamento; o serviço confere antes de gravar.                             | Comissão que realiza a reunião.                                                     |
| `titulo`              | Texto (3 a 120)                | Não           | Não             | Não                  | O tamanho é conferido pelo formulário e pelo DTO, depois de retirados os espaços das pontas.                  | Título da reunião.                                                                  |
| `local`               | Texto (vazio ou 2 a 120)       | Não           | Não             | Não                  | Sem local, grava texto vazio. Uma única letra é recusada pelo formulário e pelo DTO.                          | Onde a reunião acontece.                                                            |
| `inicio`              | Data e hora (`date`, UTC)      | Não           | Não             | Não                  | Em reunião de dia inteiro, 00:00 do dia, calculado pelo aplicativo.                                           | Momento em que a reunião começa.                                                    |
| `fim`                 | Data e hora (`date`, UTC)      | Não           | Não             | Não                  | O serviço só aceita término depois do início e, ao agendar ou mudar o horário, depois do horário atual. Em dia inteiro, 23:59:59.999 do dia. | Momento em que a reunião termina.                   |
| `dia_inteiro`         | Lógico (`bool`)                | Não           | Não             | Não                  | Valor inicial: falso.                                                                                         | Indica reunião sem horário, que ocupa o dia todo.                                   |
| `horario_alterado_em` | Data e hora (`date`, UTC)      | Sim           | Não             | Não                  | Só é preenchido pelo serviço quando a edição muda data, horário ou dia inteiro.                               | Faz o aplicativo pedir nova confirmação ao convidado.                               |
| `pauta`               | Texto (até 2.000)              | Não           | Não             | Não                  | Valor inicial: texto vazio. Tamanho conferido pelo formulário e pelo DTO.                                     | Assuntos a tratar.                                                                  |
| `criada_por`          | `ObjectId`                     | Não           | Não             | Sim (`USUARIO._id`)  | Gravado pelo serviço com o usuário do token; nunca vem do formulário.                                         | Quem agendou a reunião.                                                             |
| `situacao`            | Texto com valores fixos        | Não           | Não             | Não                  | `AGENDADA` ou `CANCELADA`, conferidos pelo schema. Valor inicial: `AGENDADA`. Só muda para `CANCELADA`.       | Indica se a reunião continua valendo.                                               |
| `presencas_revisao`   | Inteiro (`int`)                | Não           | Não             | Não                  | Começa em 0 e soma 1 no mesmo comando que muda uma presença. A correção de horários só grava se o valor for o que a tela leu. | Contador de mudanças nas presenças.                                   |
| `criado_em`           | Data e hora (`date`, UTC)      | Não           | Não             | Não                  | Preenchido pelo Mongoose na inserção.                                                                         | Momento do agendamento.                                                             |
| `atualizado_em`       | Data e hora (`date`, UTC)      | Não           | Não             | Não                  | Atualizado pelo Mongoose a cada gravação. A edição só grava se o valor for o mesmo que ela leu.               | Última alteração.                                                                   |

Os campos `ata`, `atas_canceladas` e `votacoes`, também guardados neste documento, estão nos dicionários das [atas](../../4%20-%20Incremento3/atas/4%20-%20dicionario-dados-relacional-atas.md) e da [votação](../../4%20-%20Incremento3/votacao/4%20-%20dicionario-dados-relacional-votacao.md).

## REUNIAO_PARTICIPANTE

Guarda cada pessoa convidada e a resposta ao convite. Fica em `reunioes.participantes[]`. Uma pessoa participa de várias reuniões, e uma reunião tem vários participantes.

| Campo        | Tipo de dado            | Pode ser nulo | Chave primária  | Chave estrangeira   | Outros                                                                                                                           | Descrição                        |
| ------------ | ----------------------- | ------------- | --------------- | ------------------- | -------------------------------------------------------------------------------------------------------------------------------- | -------------------------------- |
| `reuniao_id` | `ObjectId`              | Não           | Sim (inclusiva) | Sim (`REUNIAO._id`) | Não é gravado: é o documento que contém o participante.                                                                          | Reunião do convite.              |
| `usuario_id` | `ObjectId`              | Não           | Sim (inclusiva) | Sim (`USUARIO._id`) | O serviço só convida integrantes da comissão e retira repetições antes de gravar. Faz parte do índice da agenda do usuário.     | Pessoa convidada.                |
| `convite`    | Texto com valores fixos | Não           | Não             | Não                 | `PENDENTE`, `ACEITO` ou `RECUSADO`. Valor inicial: `PENDENTE`; `ACEITO` para quem agendou. Volta a `PENDENTE` quando o horário muda. | Resposta da pessoa ao convite. |

## REUNIAO_PRESENCA

Guarda a entrada e a saída de um participante. Fica em `reunioes.participantes[].presenca` e só existe depois que a entrada é registrada.

| Campo              | Tipo de dado              | Pode ser nulo | Chave primária  | Chave estrangeira                       | Outros                                                                                                                                          | Descrição                                                     |
| ------------------ | ------------------------- | ------------- | --------------- | --------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------- |
| `reuniao_id`       | `ObjectId`                | Não           | Sim (inclusiva) | Sim (`REUNIAO_PARTICIPANTE.reuniao_id`) | Não é gravado: vem do participante que contém a presença.                                                                                       | Reunião da presença.                                          |
| `usuario_id`       | `ObjectId`                | Não           | Sim (inclusiva) | Sim (`REUNIAO_PARTICIPANTE.usuario_id`) | Não é gravado: vem do participante. O comando de gravação da entrada só encontra o participante se ele ainda não tiver presença, o que impede duas entradas. | Pessoa a quem a presença pertence.                |
| `entrada`          | Data e hora (`date`, UTC) | Não           | Não             | Não                                     | Relógio do servidor na marcação e no link; `inicio` da reunião quando quem agendou marca presente; valor informado na correção, entre o início e o fim. | Momento da entrada.                                    |
| `saida`            | Data e hora (`date`, UTC) | Sim           | Não             | Não                                     | O serviço grava o menor entre o horário atual e o término; na correção, exige saída não anterior à entrada e até o término.                          | Momento da saída. Nulo enquanto a pessoa não saiu.            |
| `saida_automatica` | Lógico (`bool`)           | Sim           | Não             | Não                                     | Só existe com valor verdadeiro, gravado pela rotina do servidor no término.                                                                     | Indica saída registrada automaticamente.                      |
| `situacao`         | Texto com valores fixos   | Não           | Não             | Não                                     | `AGUARDANDO`, `CONFIRMADA` ou `RECUSADA`. Nasce `AGUARDANDO` na marcação de um convidado e `CONFIRMADA` na marcação de quem agendou, quando ele marca alguém como presente, pelo link e na correção. | Resultado da conferência da presença.                    |
| `marcada_por`      | Texto com valores fixos   | Não           | Não             | Não                                     | `PROPRIO`, `RESPONSAVEL` ou `LINK`, gravado pelo serviço conforme o caminho usado.                                                              | Diz quem registrou a entrada.                                 |

## REUNIAO_ANEXO

Guarda os dados de cada arquivo da pauta. Fica em `reunioes.anexos[]`. O conteúdo fica em `ARQUIVO_GRIDFS`.

| Campo        | Tipo de dado              | Pode ser nulo | Chave primária  | Chave estrangeira          | Outros                                                                                                      | Descrição                                  |
| ------------ | ------------------------- | ------------- | --------------- | -------------------------- | ----------------------------------------------------------------------------------------------------------- | ------------------------------------------ |
| `reuniao_id` | `ObjectId`                | Não           | Não             | Sim (`REUNIAO._id`)        | Não é gravado: é o documento que contém o anexo.                                                            | Reunião a que o anexo pertence.            |
| `_id`        | `ObjectId`                | Não           | Sim (exclusiva) | Sim (`ARQUIVO_GRIDFS._id`) | Copiado do arquivo gravado; ver "Como os identificadores são criados".                                      | Identifica o anexo e o arquivo guardado.   |
| `nome`       | Texto (até 200)           | Não           | Não             | Não                        | O serviço converte o nome recebido para utf8, para manter os acentos, retira os espaços das pontas e corta em 200 caracteres.            | Nome original do arquivo.                  |
| `tipo`       | Texto                     | Não           | Não             | Não                        | O serviço só aceita os 11 tipos da lista `TIPOS_ANEXO`: PDF, JPEG, PNG, texto, CSV, Office e LibreOffice.   | Tipo do arquivo, usado para abri-lo.       |
| `tamanho`    | Inteiro (`int`)           | Não           | Não             | Não                        | De 1 a 10.485.760 bytes; acima disso, a recepção do arquivo é interrompida antes de chegar ao serviço.     | Tamanho em bytes.                          |
| `enviado_em` | Data e hora (`date`, UTC) | Não           | Não             | Não                        | Relógio do servidor no envio.                                                                               | Momento do envio.                          |

Cada reunião tem no máximo 10 anexos: o serviço confere a quantidade antes de gravar, e o comando de gravação só acrescenta o anexo se a décima posição da lista ainda estiver vazia, o que vale também para dois envios simultâneos.

## ARQUIVO_GRIDFS

Guarda a ficha de cada arquivo. É a coleção `anexos.files`, criada e mantida pelo GridFS do MongoDB.

| Campo        | Tipo de dado              | Pode ser nulo | Chave primária  | Chave estrangeira | Outros                                                       | Descrição                                               |
| ------------ | ------------------------- | ------------- | --------------- | ----------------- | ------------------------------------------------------------ | ------------------------------------------------------- |
| `_id`        | `ObjectId`                | Não           | Sim (exclusiva) | Não               | Ver "Como os identificadores são criados".                   | Identificador do arquivo; o mesmo valor fica no anexo.  |
| `filename`   | Texto                     | Não           | Não             | Não               | Igual ao `nome` do anexo.                                    | Nome do arquivo.                                        |
| `length`     | Inteiro                   | Não           | Não             | Não               | Preenchido pelo GridFS ao terminar a gravação.               | Tamanho em bytes.                                       |
| `chunkSize`  | Inteiro (`int`)           | Não           | Não             | Não               | 261.120 bytes (255 KB), o padrão do GridFS.                  | Tamanho de cada pedaço.                                 |
| `uploadDate` | Data e hora (`date`, UTC) | Não           | Não             | Não               | Preenchido pelo GridFS.                                      | Momento em que a gravação terminou.                     |
| `metadata`   | Objeto                    | Não           | Não             | Não               | Contém `reuniao_id` e `tipo`, gravados pelo serviço.         | Permite saber, pelo arquivo, a reunião e o tipo.        |

## PEDACO_GRIDFS

Guarda o conteúdo do arquivo em partes. É a coleção `anexos.chunks`, criada e mantida pelo GridFS.

| Campo      | Tipo de dado     | Pode ser nulo | Chave primária  | Chave estrangeira          | Outros                                                    | Descrição                          |
| ---------- | ---------------- | ------------- | --------------- | -------------------------- | --------------------------------------------------------- | ---------------------------------- |
| `_id`      | `ObjectId`       | Não           | Sim (exclusiva) | Não                        | Ver "Como os identificadores são criados".                | Identificador do pedaço.           |
| `files_id` | `ObjectId`       | Não           | Não             | Sim (`ARQUIVO_GRIDFS._id`) | O GridFS apaga os pedaços junto com o arquivo.            | Arquivo a que o pedaço pertence.   |
| `n`        | Inteiro (`int`)  | Não           | Não             | Não                        | Começa em 0 e sobe de 1 em 1 dentro do mesmo arquivo.     | Ordem do pedaço no arquivo.        |
| `data`     | Binário          | Não           | Não             | Não                        | Até 255 KB por pedaço.                                    | Parte do conteúdo do arquivo.      |

## LINK_PRESENCA

Guarda o link de presença ativo da reunião. Fica em `reunioes.acesso_presenca` e não aparece nas consultas comuns da reunião.

| Campo            | Tipo de dado              | Pode ser nulo | Chave primária  | Chave estrangeira   | Outros                                                                                                                    | Descrição                                                    |
| ---------------- | ------------------------- | ------------- | --------------- | ------------------- | ------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------ |
| `reuniao_id`     | `ObjectId`                | Não           | Sim (exclusiva) | Sim (`REUNIAO._id`) | Não é gravado: é o documento que contém o link.                                                                           | Reunião do link.                                             |
| `hash`           | Texto (64 hexadecimais)   | Não           | Não             | Não                 | Resumo SHA-256 do código sorteado. O código não é gravado; quem tem só o banco não consegue montar o link.               | Usado para conferir o código que o participante apresenta.   |
| `inicio`         | Data e hora (`date`, UTC) | Não           | Não             | Não                 | Cópia do início da reunião na geração; se a reunião mudar de horário, o link deixa de valer.                             | Início da validade.                                          |
| `fim`            | Data e hora (`date`, UTC) | Não           | Não             | Não                 | Cópia do término na geração; depois dele, o link é recusado.                                                              | Fim da validade.                                             |
| `responsavel_id` | Texto (24 hexadecimais)   | Não           | Não             | Sim (`USUARIO._id`) | Guardado como texto. O link só vale enquanto esse usuário for o responsável da comissão.                                 | Quem gerou o link.                                           |
| `criado_em`      | Data e hora (`date`, UTC) | Não           | Não             | Não                 | Relógio do servidor na geração.                                                                                           | Momento da geração.                                          |

## CORRECAO_PRESENCA

Guarda cada correção de horário feita pelo responsável. Fica em `reunioes.historico_presencas[]` e não aparece nas consultas comuns da reunião.

| Campo             | Tipo de dado              | Pode ser nulo | Chave primária  | Chave estrangeira   | Outros                                                                                                              | Descrição                                               |
| ----------------- | ------------------------- | ------------- | --------------- | ------------------- | ------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------- |
| `reuniao_id`      | `ObjectId`                | Não           | Sim (inclusiva) | Sim (`REUNIAO._id`) | Não é gravado: é o documento que contém a lista.                                                                    | Reunião corrigida.                                      |
| `posicao`         | Inteiro                   | Não           | Sim (inclusiva) | Não                 | Não é gravada: ver "Como os identificadores são criados".                                                           | Ordem da correção.                                      |
| `participante_id` | Texto (24 hexadecimais)   | Não           | Não             | Sim (`USUARIO._id`) | Guardado como texto; precisa ser participante da reunião, o que o serviço confere antes de gravar.                  | Participante corrigido.                                 |
| `responsavel_id`  | Texto (24 hexadecimais)   | Não           | Não             | Sim (`USUARIO._id`) | Guardado como texto; é o usuário do token, responsável da comissão.                                                 | Quem corrigiu.                                          |
| `alterada_em`     | Data e hora (`date`, UTC) | Não           | Não             | Não                 | Relógio do servidor na correção.                                                                                    | Momento da correção.                                    |
| `motivo`          | Texto (5 a 500)           | Não           | Não             | Não                 | O formulário e o DTO exigem ao menos 5 caracteres, depois de retirados os espaços das pontas.                      | Justificativa da correção.                              |
| `antes`           | Objeto (presença)         | Sim           | Não             | Não                 | Cópia da presença anterior; nulo quando a pessoa não tinha presença.                                                | Valores substituídos.                                   |
| `depois`          | Objeto (presença)         | Não           | Não             | Não                 | Presença gravada, sempre `CONFIRMADA` e `marcada_por = RESPONSAVEL`.                                                | Valores novos.                                          |

O registro é acrescentado no mesmo comando que altera a presença; se o comando não encontrar a reunião no estado lido, nem a presença nem o histórico mudam.

## NOTIFICACAO

Guarda os avisos mostrados na tela "Notificações". É a coleção `notificacoes`; este módulo só grava os avisos do tipo `PRESENCA`.

| Campo        | Tipo de dado              | Pode ser nulo | Chave primária  | Chave estrangeira   | Outros                                                                                                                   | Descrição                                       |
| ------------ | ------------------------- | ------------- | --------------- | ------------------- | ------------------------------------------------------------------------------------------------------------------------ | ----------------------------------------------- |
| `_id`        | `ObjectId`                | Não           | Sim (exclusiva) | Não                 | Ver "Como os identificadores são criados".                                                                               | Identificador do aviso.                         |
| `usuario_id` | `ObjectId`                | Não           | Não             | Sim (`USUARIO._id`) | Indexado. Junto com `chave`, forma um índice único no banco.                                                             | Destinatário.                                   |
| `chave`      | Texto                     | Não           | Não             | Não                 | `PRESENCA:<reunião>:<resumo do código>`. Como o par com `usuario_id` é único e a gravação só insere quando não existe, repetir o envio não duplica o aviso. | Identifica o aviso de um link para um usuário. |
| `tipo`       | Texto                     | Não           | Não             | Não                 | `PRESENCA` para os avisos deste módulo.                                                                                  | Categoria do aviso.                             |
| `titulo`     | Texto                     | Não           | Não             | Não                 | Título da reunião.                                                                                                       | Título mostrado.                                |
| `mensagem`   | Texto                     | Não           | Não             | Não                 | Texto fixo do serviço.                                                                                                   | Explicação mostrada.                            |
| `destino`    | Texto                     | Não           | Não             | Não                 | `/presenca?id=<reunião>&codigo=<código>`; o aplicativo só abre destinos que começam com `/presenca?` ou `/reunioes/`.    | Tela aberta pelo botão "Abrir".                 |
| `expira_em`  | Data e hora (`date`, UTC) | Sim           | Não             | Não                 | Término da reunião; passa a ser o horário atual quando o link é trocado ou revogado.                                     | Validade do aviso.                              |
| `lida_em`    | Data e hora (`date`, UTC) | Sim           | Não             | Não                 | Preenchido quando o usuário abre o aviso.                                                                                | Momento da leitura.                             |
| `criada_em`  | Data e hora (`date`, UTC) | Não           | Não             | Não                 | Preenchido pelo Mongoose na inserção.                                                                                    | Momento do envio.                               |

## COMISSAO, ORGANIZACAO e USUARIO (campos usados)

| Entidade      | Campo            | Tipo de dado            | Pode ser nulo | Chave primária  | Chave estrangeira       | Outros                                                    | Descrição                                       |
| ------------- | ---------------- | ----------------------- | ------------- | --------------- | ----------------------- | --------------------------------------------------------- | ----------------------------------------------- |
| `COMISSAO`    | `_id`            | `ObjectId`              | Não           | Sim (exclusiva) | Não                     | Criado na inserção da comissão.                           | Identificador da comissão.                      |
| `COMISSAO`    | `organizacao_id` | `ObjectId`              | Não           | Não             | Sim (`ORGANIZACAO._id`) | -                                                         | Organização da comissão.                        |
| `COMISSAO`    | `ativo`          | Lógico (`bool`)         | Não           | Não             | Não                     | Com falso, não se agenda nem se gera link.                | Indica comissão ativa.                          |
| `ORGANIZACAO` | `_id`            | `ObjectId`              | Não           | Sim (exclusiva) | Não                     | Criado na inserção da organização.                        | Identificador da organização.                   |
| `ORGANIZACAO` | `status`         | Texto com valores fixos | Não           | Não             | Não                     | Precisa ser `APROVADA` para agendar e gerir presença.     | Situação da organização.                        |
| `USUARIO`     | `_id`            | `ObjectId`              | Não           | Sim (exclusiva) | Não                     | Criado na inserção do usuário.                            | Identificador do usuário.                       |
| `USUARIO`     | `nome`           | Texto                   | Não           | Não             | Não                     | -                                                         | Nome mostrado nas listas.                       |
| `USUARIO`     | `email`          | Texto                   | Não           | Não             | Não                     | -                                                         | Endereço usado no envio do link por e-mail.     |
| `USUARIO`     | `ativo`          | Lógico (`bool`)         | Não           | Não             | Não                     | Conta inativa não usa o módulo nem recebe o link.         | Indica conta ativa.                             |

## Índices e proteções de gravação

- índice `{ comissao_id: 1, inicio: 1 }`, usado pela lista de reuniões da comissão, e índice `{ participantes.usuario_id: 1, inicio: 1 }`, usado pela agenda do usuário;
- índice único `{ usuario_id: 1, chave: 1 }` em `notificacoes`;
- a alteração da reunião aceita cancelamento ou edição, nunca os dois no mesmo pedido;
- a edição regrava a lista inteira de participantes e, por isso, só grava se `atualizado_em` for o mesmo da leitura, para não apagar uma resposta de convite ou uma entrada registrada no intervalo;
- entrada, saída, convite, confirmação de presença, anexos, link e correção repetem no filtro da gravação as condições conferidas antes; se algo mudou, nada é gravado e a API responde com conflito (409);
- o anexo só entra na lista depois de o arquivo estar gravado no GridFS; se a gravação na reunião falhar, o arquivo é apagado; na remoção, o arquivo só é apagado depois de a lista ser atualizada.

## Rastreabilidade

RF-REU-01 a RF-REU-09 e RF-REU-13 a RF-REU-22; [Diagrama relacional](4%20-%20diagrama-relacional-reunioes.md) e [Testes](7%20-%20testes-reunioes.md).
