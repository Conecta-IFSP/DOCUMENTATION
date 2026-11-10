# Dicionário relacional — Reuniões

> Padrão da disciplina · Módulo REU · Revisado em 10/11/2026

## Resumo

Descreve cada campo das entidades do [diagrama relacional](4%20-%20diagrama-relacional-reunioes.md). As colunas seguem o dicionário de dados do documento de exemplo da disciplina: campo, tipo de dado, se pode ser nulo, chave primária, chave estrangeira, outros e descrição.

O banco é o MongoDB. `REUNIAO` é a coleção `reunioes`; as demais entidades são dados gravados dentro do documento da reunião, separados aqui para mostrar chaves e restrições. Nessas entidades, a reunião faz parte da chave, mas não é um campo gravado: ela é o documento que contém o dado.

## Como os identificadores são criados

| Identificador                | Quem cria                                    | Formato e valor inicial                                                                                                                                                                                              | Incremento                                                                                                            |
| ---------------------------- | -------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------- |
| `REUNIAO._id`                | Driver do MongoDB, no momento do `create`    | `ObjectId` de 12 bytes, exibido como 24 caracteres hexadecimais: 4 bytes com a hora da criação em segundos, 5 bytes sorteados quando o servidor inicia e 3 bytes de um contador. Não tem valor inicial fixo.          | Não é sequencial. O contador de 3 bytes começa em um valor sorteado e soma 1 a cada identificador criado pelo servidor. |
| `REUNIAO_ANEXO._id`          | GridFS, ao abrir a gravação do arquivo       | `ObjectId`, com o mesmo formato acima. É criado para o arquivo em `anexos.files` e copiado para o anexo.                                                                                                               | Igual ao anterior.                                                                                                    |
| Chave de `REUNIAO_PARTICIPANTE` | Não é criada                              | É a reunião mais o `usuario_id` do participante, que já existe em `usuarios`.                                                                                                                                         | Não se aplica.                                                                                                        |

Diferente de um campo identidade de banco relacional, o `ObjectId` não começa em 1 nem cresce de 1 em 1: ele é montado no servidor da aplicação, sem consulta ao banco, e traz embutida a hora em que foi criado.

## REUNIAO

Guarda os dados de cada reunião agendada. É a coleção `reunioes`.

| Campo                 | Tipo de dado                     | Pode ser nulo | Chave primária  | Chave estrangeira    | Outros                                                    | Descrição                                                                       |
| --------------------- | -------------------------------- | ------------- | --------------- | -------------------- | --------------------------------------------------------- | ------------------------------------------------------------------------------- |
| `_id`                 | `ObjectId`                       | Não           | Sim (exclusivo) | Não                  | Criado pelo driver do MongoDB; ver seção anterior.        | Identificador único da reunião.                                                 |
| `comissao_id`         | `ObjectId`                       | Não           | Não             | Sim (`COMISSAO._id`) | Índice junto com `inicio`.                                | Comissão que realiza a reunião.                                                 |
| `titulo`              | Texto (`string`, 3 a 120)        | Não           | Não             | Não                  | Espaços das pontas são retirados.                         | Título da reunião.                                                              |
| `local`               | Texto (`string`, 0 ou 2 a 120)   | Não           | Não             | Não                  | Valor padrão: texto vazio.                                | Onde a reunião acontece. Vazio quando não foi informado.                        |
| `inicio`              | Data e hora (`date`, UTC)        | Não           | Não             | Não                  | Em reunião de dia inteiro, 00:00 do dia.                  | Momento em que a reunião começa.                                                |
| `fim`                 | Data e hora (`date`, UTC)        | Não           | Não             | Não                  | Maior que `inicio`. Em dia inteiro, 23:59:59.999 do dia.  | Momento em que a reunião termina.                                               |
| `dia_inteiro`         | Lógico (`boolean`)               | Não           | Não             | Não                  | Valor padrão: falso.                                      | Indica reunião sem horário, que ocupa o dia todo.                               |
| `horario_alterado_em` | Data e hora (`date`, UTC)        | Sim           | Não             | Não                  | Preenchido só quando a data ou o horário são editados.    | Última mudança de data ou horário; faz o aplicativo pedir nova confirmação.     |
| `pauta`               | Texto (`string`, até 2.000)      | Não           | Não             | Não                  | Valor padrão: texto vazio.                                | Assuntos a tratar.                                                              |
| `criada_por`          | `ObjectId`                       | Não           | Não             | Sim (`USUARIO._id`)  | -                                                         | Presidente que agendou a reunião.                                               |
| `situacao`            | Texto com valores fixos          | Não           | Não             | Não                  | `AGENDADA` ou `CANCELADA`. Valor inicial: `AGENDADA`.     | Indica se a reunião continua valendo.                                           |
| `criado_em`           | Data e hora (`date`, UTC)        | Não           | Não             | Não                  | Preenchido pelo Mongoose na criação.                      | Momento do agendamento.                                                         |
| `atualizado_em`       | Data e hora (`date`, UTC)        | Não           | Não             | Não                  | Atualizado pelo Mongoose a cada gravação.                 | Última alteração; usado para impedir que duas edições se sobreponham.           |

## REUNIAO_PARTICIPANTE

Guarda cada pessoa convidada para a reunião e a resposta ao convite. Fica em `reunioes.participantes[]`. Uma pessoa pode participar de várias reuniões, e uma reunião tem vários participantes.

| Campo        | Tipo de dado            | Pode ser nulo | Chave primária  | Chave estrangeira    | Outros                                                           | Descrição                                          |
| ------------ | ----------------------- | ------------- | --------------- | -------------------- | ---------------------------------------------------------------- | -------------------------------------------------- |
| `reuniao_id` | `ObjectId`              | Não           | Sim (inclusivo) | Sim (`REUNIAO._id`)  | Não é gravado: é o documento que contém o participante.          | Reunião do convite.                                |
| `usuario_id` | `ObjectId`              | Não           | Sim (inclusivo) | Sim (`USUARIO._id`)  | Não se repete dentro da mesma reunião. Índice junto com `inicio`. | Pessoa convidada.                                  |
| `convite`    | Texto com valores fixos | Não           | Não             | Não                  | `PENDENTE`, `ACEITO` ou `RECUSADO`. Valor inicial: `PENDENTE`; `ACEITO` para quem agendou. | Resposta da pessoa ao convite. |

## REUNIAO_PRESENCA

Guarda a entrada e a saída de um participante. Fica em `reunioes.participantes[].presenca` e só existe depois que a entrada é marcada.

| Campo         | Tipo de dado              | Pode ser nulo | Chave primária  | Chave estrangeira                          | Outros                                                              | Descrição                                                        |
| ------------- | ------------------------- | ------------- | --------------- | ------------------------------------------ | ------------------------------------------------------------------- | ---------------------------------------------------------------- |
| `reuniao_id`  | `ObjectId`                | Não           | Sim (inclusivo) | Sim (`REUNIAO_PARTICIPANTE.reuniao_id`)    | Não é gravado: vem do participante que contém a presença.           | Reunião da presença.                                             |
| `usuario_id`  | `ObjectId`                | Não           | Sim (inclusivo) | Sim (`REUNIAO_PARTICIPANTE.usuario_id`)    | Não é gravado: vem do participante que contém a presença.           | Pessoa a quem a presença pertence.                               |
| `entrada`     | Data e hora (`date`, UTC) | Não           | Não             | Não                                        | Horário do servidor; igual a `inicio` quando marcada pelo presidente. | Momento da entrada.                                            |
| `saida`       | Data e hora (`date`, UTC) | Sim           | Não             | Não                                        | Nunca maior que o `fim` da reunião.                                 | Momento da saída. Nulo enquanto a pessoa não marcou.             |
| `situacao`    | Texto com valores fixos   | Não           | Não             | Não                                        | `AGUARDANDO`, `CONFIRMADA` ou `RECUSADA`.                           | Resultado da conferência feita pelo presidente.                  |
| `marcada_por` | Texto com valores fixos   | Não           | Não             | Não                                        | `PROPRIO` ou `RESPONSAVEL`.                                         | Diz se a própria pessoa marcou a entrada ou se foi o presidente. |

## REUNIAO_ANEXO

Guarda os dados de cada arquivo anexado à pauta. Fica em `reunioes.anexos[]`. O conteúdo do arquivo fica em `ARQUIVO_GRIDFS`.

| Campo        | Tipo de dado                | Pode ser nulo | Chave primária  | Chave estrangeira          | Outros                                                             | Descrição                                   |
| ------------ | --------------------------- | ------------- | --------------- | -------------------------- | ------------------------------------------------------------------ | ------------------------------------------- |
| `reuniao_id` | `ObjectId`                  | Não           | Sim (inclusivo) | Sim (`REUNIAO._id`)        | Não é gravado: é o documento que contém o anexo.                   | Reunião a que o anexo pertence.             |
| `_id`        | `ObjectId`                  | Não           | Sim (inclusivo) | Sim (`ARQUIVO_GRIDFS._id`) | Criado pelo GridFS ao gravar o arquivo; ver seção de identificadores. | Identifica o anexo e o arquivo guardado. |
| `nome`       | Texto (`string`, até 200)   | Não           | Não             | Não                        | Convertido para utf8, para manter os acentos.                      | Nome original do arquivo.                   |
| `tipo`       | Texto (`string`)            | Não           | Não             | Não                        | Um dos tipos aceitos: PDF, JPEG, PNG, texto, CSV, Office e LibreOffice. | Tipo do arquivo, usado para abri-lo.   |
| `tamanho`    | Inteiro (`int`)             | Não           | Não             | Não                        | De 1 a 10.485.760 bytes (10 MB).                                   | Tamanho do arquivo em bytes.                |
| `enviado_em` | Data e hora (`date`, UTC)   | Não           | Não             | Não                        | -                                                                  | Momento do envio.                           |

Cada reunião tem no máximo 10 anexos.

## ARQUIVO_GRIDFS

Guarda o conteúdo dos arquivos. São as coleções `anexos.files` e `anexos.chunks`, criadas e mantidas pelo GridFS do MongoDB; o sistema só as acessa pelo `GridFSBucket`.

| Campo        | Tipo de dado              | Pode ser nulo | Chave primária  | Chave estrangeira | Outros                                                | Descrição                                                   |
| ------------ | ------------------------- | ------------- | --------------- | ----------------- | ----------------------------------------------------- | ----------------------------------------------------------- |
| `_id`        | `ObjectId`                | Não           | Sim (exclusivo) | Não               | Criado pelo GridFS no início da gravação.             | Identificador do arquivo; o mesmo valor fica no anexo.      |
| `filename`   | Texto (`string`)          | Não           | Não             | Não               | -                                                     | Nome do arquivo.                                            |
| `length`     | Inteiro (`long`)          | Não           | Não             | Não               | Preenchido pelo GridFS.                               | Tamanho em bytes.                                           |
| `chunkSize`  | Inteiro (`int`)           | Não           | Não             | Não               | 261.120 bytes (255 KB), padrão do GridFS.             | Tamanho de cada pedaço em que o arquivo é dividido.         |
| `uploadDate` | Data e hora (`date`, UTC) | Não           | Não             | Não               | Preenchido pelo GridFS.                               | Momento em que a gravação terminou.                         |
| `metadata`   | Objeto                    | Não           | Não             | Não               | Contém `reuniao_id` e `tipo`.                         | Permite saber, a partir do arquivo, de qual reunião ele é.  |

## REUNIAO_ATA

A ata e suas entidades (`ATA`, `ATA_APROVADOR`, `ATA_AVALIACAO`, `ATA_CANCELADA` e `CONTADOR_ATA`) estão descritas campo a campo no [dicionário relacional das atas](../../4%20-%20Incremento3/atas/4%20-%20dicionario-dados-relacional-atas.md).

## Índices e restrições

- índice `{ comissao_id: 1, inicio: 1 }`, para a agenda da comissão;
- índice `{ participantes.usuario_id: 1, inicio: 1 }`, para a agenda do usuário;
- a alteração da reunião aceita cancelamento ou edição, nunca os dois na mesma requisição;
- a edição compara `atualizado_em` para não apagar respostas de convite nem presenças registradas entre a leitura e a gravação;
- entrada, saída, convite, presença e anexos são gravados com `findOneAndUpdate`, repetindo no filtro as condições conferidas antes;
- o anexo só entra na lista depois de o arquivo estar gravado no GridFS; se a gravação na reunião falhar, o arquivo é apagado;
- a remoção do anexo só apaga o arquivo no GridFS depois de a lista ser atualizada.

## Rastreabilidade

RF-REU-01 a RF-REU-09 e RF-REU-13 a RF-REU-16; [diagrama relacional](4%20-%20diagrama-relacional-reunioes.md) e [testes](7%20-%20testes-reunioes.md).
