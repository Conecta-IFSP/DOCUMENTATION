# Dicionário relacional — Reuniões

> Padrão do Incremento 2 · Tipos lógicos e correspondência com MongoDB/Mongoose

## Resumo

| Estrutura lógica       | Estrutura física                         | Finalidade                  |
| ---------------------- | ---------------------------------------- | --------------------------- |
| `REUNIAO`              | `reunioes`                               | Dados do encontro.          |
| `REUNIAO_PARTICIPANTE` | `reunioes.participantes[]`               | Convites e usuários.        |
| `REUNIAO_PRESENCA`     | `reunioes.participantes[].presenca`      | Entrada, saída e validação. |
| `REUNIAO_ATA`          | `reunioes.ata`                           | Conteúdo e versão.          |
| `ATA_APROVADOR`        | `ata.aprovadores[]` e `ata.aprovacoes[]` | Elegibilidade e votos.      |

## REUNIAO

| Campo           | Tipo       | Chave/Nulo   | Restrição                              |
| --------------- | ---------- | ------------ | -------------------------------------- |
| `_id`           | `ObjectId` | PK, não nulo | Gerado pelo MongoDB.                   |
| `comissao_id`   | `ObjectId` | FK, não nulo | Comissão autorizada.                   |
| `titulo`        | `string`   | não nulo     | 3 a 120 caracteres no DTO.             |
| `local`         | `string`   | não nulo     | 2 a 120 caracteres no DTO.             |
| `inicio`        | `Date`     | não nulo     | UTC.                                   |
| `fim`           | `Date`     | não nulo     | Posterior a `inicio`.                  |
| `pauta`         | `string`   | não nulo     | Padrão vazio; até 2.000 caracteres.    |
| `criada_por`    | `ObjectId` | FK, não nulo | Organizador.                           |
| `situacao`      | `enum`     | não nulo     | `AGENDADA` ou `CANCELADA`.             |
| `participantes` | `array`    | não nulo     | Inclui o organizador; usuários únicos. |
| `ata`           | `object`   | nulo         | Disponível após o encerramento.        |
| `criado_em`     | `Date`     | não nulo     | Automático.                            |
| `atualizado_em` | `Date`     | não nulo     | Automático.                            |

## REUNIAO_PARTICIPANTE

| Campo        | Tipo       | Chave/Nulo   | Restrição                              |
| ------------ | ---------- | ------------ | -------------------------------------- |
| `reuniao_id` | `ObjectId` | PK/FK lógica | Implícito no documento pai.            |
| `usuario_id` | `ObjectId` | PK/FK lógica | Não se repete na reunião.              |
| `convite`    | `enum`     | não nulo     | `PENDENTE`, `ACEITO` ou `RECUSADO`.    |
| `presenca`   | `object`   | nulo         | Criada na entrada ou pelo organizador. |

## REUNIAO_PRESENCA

| Campo         | Tipo   | Chave/Nulo | Restrição                                                                            |
| ------------- | ------ | ---------- | ------------------------------------------------------------------------------------ |
| `entrada`     | `Date` | não nulo   | Uma única vez; dentro do período ou igual ao início quando marcada pelo responsável. |
| `saida`       | `Date` | nulo       | Maior ou igual à entrada e limitada ao término.                                      |
| `situacao`    | `enum` | não nulo   | `AGUARDANDO`, `CONFIRMADA` ou `RECUSADA`.                                            |
| `marcada_por` | `enum` | não nulo   | `PROPRIO` ou `RESPONSAVEL`.                                                          |

## REUNIAO_ATA

| Campo           | Tipo         | Chave/Nulo   | Restrição                                       |
| --------------- | ------------ | ------------ | ----------------------------------------------- |
| `conteudo`      | `string`     | não nulo     | 20 a 20.000 caracteres.                         |
| `versao`        | `number`     | não nulo     | Inteiro crescente, mínimo 1.                    |
| `situacao`      | `enum`       | não nulo     | `RASCUNHO`, `EM_APROVACAO` ou `APROVADA`.       |
| `autor_id`      | `ObjectId`   | FK, não nulo | Responsável autor.                              |
| `atualizada_em` | `Date`       | não nulo     | Instante da última versão.                      |
| `aprovadores`   | `ObjectId[]` | não nulo     | Presentes confirmados no momento da publicação. |
| `aprovacoes`    | `ObjectId[]` | não nulo     | IDs únicos, todos contidos em `aprovadores`.    |

## Índices e restrições

- índice `{ comissao_id: 1, inicio: 1 }` para agenda da comissão;
- índice `{ participantes.usuario_id: 1, inicio: 1 }` para agenda do usuário;
- alteração de reunião aceita somente cancelamento;
- entrada, saída, convite, presença e ata usam filtros condicionais para detectar concorrência;
- ata publicada não volta a rascunho e não aceita alteração de texto;
- ata só se torna aprovada quando todo aprovador consta em `aprovacoes`.

## Rastreabilidade

RF-REU-01 a RF-REU-12; diagrama relacional e testes do módulo.
