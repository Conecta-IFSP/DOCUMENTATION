# Dicionário relacional — Informações do sistema

> Padrão da disciplina · Módulo SIS · Campos do diagrama relacional · Revisado em 26/11/2026

## Resumo

Descreve cada campo das entidades do [diagrama relacional](4%20-%20diagrama-relacional-informacao-sistemas.md), com tipo, obrigatoriedade, chaves e forma de geração, e diz o que este módulo faz com cada um. As colunas seguem o dicionário de dados do documento de exemplo da disciplina: campo, tipo de dado, se pode ser nulo, chave primária, chave estrangeira, outros e descrição. Os campos que este módulo não usa estão nos dicionários dos módulos donos de cada coleção.

## Como os identificadores são criados

| Identificador                                                                 | Quem gera                                                  | Formato e valor inicial                                                                                                                                                                                                                                                       | Incremento                                                                                                                                         |
| ----------------------------------------------------------------------------- | ---------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| `USUARIO._id`, `ORGANIZACAO._id`, `COMISSAO._id`, `REUNIAO._id`, `NOTIFICACAO._id` | Driver do MongoDB, no processo da API, no momento em que o documento é criado para inserção | `ObjectId` de 12 bytes, mostrado como 24 caracteres hexadecimais: 4 bytes com os segundos desde 1º/01/1970, 5 bytes aleatórios sorteados uma vez por processo e 3 bytes de um contador que começa num valor aleatório. Não há valor inicial fixo. | O contador soma 1 a cada novo identificador gerado pelo mesmo processo. Como os outros bytes mudam com a hora e com o processo, os valores não são contínuos nem previsíveis. |
| `ARQUIVO_GRIDFS._id`                                                          | Driver do MongoDB, ao abrir a gravação do arquivo no GridFS | `ObjectId`, no mesmo formato acima. O mesmo valor é copiado para `reunioes.anexos[]._id`.                                                                                                                                                                                     | Igual ao anterior.                                                                                                                                 |
| `PARTE_GRIDFS._id`                                                            | Driver do MongoDB, a cada parte gravada                    | `ObjectId`, no mesmo formato acima.                                                                                                                                                                                                                                           | Igual ao anterior.                                                                                                                                 |
| `PARTE_GRIDFS.n`                                                              | Driver do MongoDB (GridFS)                                 | Inteiro. A primeira parte de cada arquivo recebe 0.                                                                                                                                                                                                                           | Soma 1 a cada parte seguinte do mesmo arquivo.                                                                                                     |
| `ORGANIZACAO.codigo_acesso`                                                   | Função `gerarCodigoAcesso`, valor padrão do schema         | Texto de 10 caracteres, cada um sorteado com `randomInt` entre 31 símbolos: letras maiúsculas sem I, L e O, e os algarismos de 2 a 9.                                                                                                                                         | Não é sequencial: cada organização recebe um sorteio novo. O índice único impede repetição.                                                       |
| `CONTADOR_ATA._id`                                                            | `ReunioesService.proximoNumeroAta` (módulo de atas)        | Texto `atas:<id da comissão>:<ano>`.                                                                                                                                                                                                                                         | Não muda. Um documento por comissão e ano; o reset apaga todos, e a numeração recomeça.                                                            |
| `ORGANIZACAO_MEMBRO` e `COMISSAO_MEMBRO` (chave lógica)                       | Não é gerado                                               | O item é identificado pelo documento que contém a lista mais o `usuario_id`.                                                                                                                                                                                                  | Não há.                                                                                                                                            |

## USUARIO

Conta de acesso ao aplicativo. É a coleção `usuarios`. Este módulo conta as contas, cria as três contas de teste e, no reset, apaga as que não são de administrador do sistema.

| Campo              | Tipo de dado              | Pode ser nulo | Chave primária  | Chave estrangeira | Outros                                                            | Descrição                                                                                     |
| ------------------ | ------------------------- | ------------- | --------------- | ----------------- | ----------------------------------------------------------------- | --------------------------------------------------------------------------------------------- |
| `_id`              | `ObjectId`                | Não           | Sim (exclusivo) | Não               | Ver tabela de identificadores.                                    | Identifica a conta.                                                                           |
| `nome`             | Texto (`string`)          | Não           | Não             | Não               | Aparado nas pontas.                                               | Nome da pessoa; nas contas de teste, vem da configuração do servidor.                         |
| `email`            | Texto (`string`)          | Não           | Não             | Não               | Único; gravado em minúsculas e aparado.                           | E-mail de acesso. Repetido, a inserção é recusada pelo índice único.                          |
| `senha_hash`       | Texto (`string`)          | Sim           | Não             | Não               | Nunca devolvido nas consultas.                                    | Resultado do bcrypt (custo 10) aplicado à senha; nas contas de teste, à senha da configuração. |
| `tipo`             | Texto com valores fixos   | Não           | Não             | Não               | `USUARIO` (padrão) ou `ADMIN_SISTEMA`.                            | Define se a conta é de administrador do sistema; o reset preserva só `ADMIN_SISTEMA`.         |
| `ativo`            | Booleano (`bool`)         | Não           | Não             | Não               | Padrão verdadeiro.                                                | Conta liberada para entrar; contada em `usuarios.ativos`.                                     |
| `email_verificado` | Booleano (`bool`)         | Não           | Não             | Não               | Padrão verdadeiro.                                                | Indica se o e-mail foi confirmado; as contas de teste já nascem confirmadas.                  |

## ORGANIZACAO

Instituição onde as comissões funcionam. É a coleção `organizacoes`. Este módulo conta as organizações por situação, cria a organização de teste e, no reset, apaga todas.

| Campo           | Tipo de dado            | Pode ser nulo | Chave primária  | Chave estrangeira   | Outros                                       | Descrição                                                                |
| --------------- | ----------------------- | ------------- | --------------- | ------------------- | -------------------------------------------- | ------------------------------------------------------------------------ |
| `_id`           | `ObjectId`              | Não           | Sim (exclusivo) | Não                 | Ver tabela de identificadores.               | Identifica a organização.                                                |
| `nome`          | Texto (`string`)        | Não           | Não             | Não                 | Aparado.                                     | Nome; "Organização de Teste" nos dados de teste.                         |
| `cidade`        | Texto (`string`)        | Não           | Não             | Não                 | Aparado.                                     | Cidade; "São Paulo" nos dados de teste.                                  |
| `descricao`     | Texto (`string`)        | Sim           | Não             | Não                 | -                                            | Descrição; "Criada para testes." nos dados de teste.                     |
| `status`        | Texto com valores fixos | Não           | Não             | Não                 | `PENDENTE` (padrão), `APROVADA` ou `REVOGADA`. | Situação da organização; os relatórios contam `PENDENTE` e `APROVADA`.  |
| `criada_por`    | `ObjectId`              | Não           | Não             | Sim (`USUARIO._id`) | -                                            | Conta que cadastrou a organização; nos dados de teste, a de "admin de organização". |
| `codigo_acesso` | Texto (10 caracteres)   | Não           | Não             | Não                 | Único; nunca devolvido por este módulo.      | Código para entrar na organização; ver tabela de identificadores.        |

## ORGANIZACAO_MEMBRO

Vínculo de uma conta com uma organização. Fica em `organizacoes.membros[]` e é apagado junto com a organização.

| Campo            | Tipo de dado              | Pode ser nulo | Chave primária  | Chave estrangeira       | Outros                                             | Descrição                                       |
| ---------------- | ------------------------- | ------------- | --------------- | ----------------------- | -------------------------------------------------- | ----------------------------------------------- |
| `organizacao_id` | `ObjectId`                | Não           | Sim (inclusivo) | Sim (`ORGANIZACAO._id`) | Não é gravado: é o documento que contém a lista.   | Organização do vínculo.                         |
| `usuario_id`     | `ObjectId`                | Não           | Sim (inclusivo) | Sim (`USUARIO._id`)     | -                                                  | Conta vinculada.                                |
| `papel`          | Texto com valores fixos   | Não           | Não             | Não                     | `ADMIN` ou `MEMBRO`.                               | Papel da conta na organização.                  |
| `status`         | Texto com valores fixos   | Não           | Não             | Não                     | `PENDENTE` (padrão), `APROVADO`, `REJEITADO` ou `CONVIDADO`. | Situação do vínculo; `APROVADO` nos dados de teste. |
| `solicitado_em`  | Data e hora (`date`, UTC) | Não           | Não             | Não                     | Padrão: horário da gravação.                       | Quando o vínculo foi pedido.                    |
| `aprovado_em`    | Data e hora (`date`, UTC) | Sim           | Não             | Não                     | -                                                  | Quando o vínculo foi aprovado.                  |

## COMISSAO

Grupo de trabalho de uma organização. É a coleção `comissoes`. Este módulo conta as comissões, cria a comissão de teste e, no reset, apaga todas.

| Campo            | Tipo de dado            | Pode ser nulo | Chave primária  | Chave estrangeira       | Outros                           | Descrição                                                |
| ---------------- | ----------------------- | ------------- | --------------- | ----------------------- | -------------------------------- | -------------------------------------------------------- |
| `_id`            | `ObjectId`              | Não           | Sim (exclusivo) | Não                     | Ver tabela de identificadores.   | Identifica a comissão.                                   |
| `nome`           | Texto (`string`)        | Não           | Não             | Não                     | Aparado.                         | Nome; "Comissão de Teste" nos dados de teste.            |
| `descricao`      | Texto (`string`)        | Sim           | Não             | Não                     | -                                | Descrição; "Criada para testes." nos dados de teste.     |
| `organizacao_id` | `ObjectId`              | Não           | Não             | Sim (`ORGANIZACAO._id`) | -                                | Organização a que a comissão pertence.                   |
| `ativo`          | Booleano (`bool`)       | Não           | Não             | Não                     | Padrão verdadeiro.               | Comissão em funcionamento; contada em `comissoes.ativas`. |

## COMISSAO_MEMBRO

Participação de uma conta numa comissão. Fica em `comissoes.membros[]` e é apagada junto com a comissão.

| Campo           | Tipo de dado              | Pode ser nulo | Chave primária  | Chave estrangeira    | Outros                                           | Descrição                            |
| --------------- | ------------------------- | ------------- | --------------- | -------------------- | ------------------------------------------------ | ------------------------------------ |
| `comissao_id`   | `ObjectId`                | Não           | Sim (inclusivo) | Sim (`COMISSAO._id`) | Não é gravado: é o documento que contém a lista. | Comissão da participação.            |
| `usuario_id`    | `ObjectId`                | Não           | Sim (inclusivo) | Sim (`USUARIO._id`)  | Não se repete na mesma comissão.                 | Conta participante.                  |
| `papel`         | Texto com valores fixos   | Não           | Não             | Não                  | `RESPONSAVEL` ou `MEMBRO`; exatamente um `RESPONSAVEL` por comissão. | Papel da conta na comissão.  |
| `adicionado_em` | Data e hora (`date`, UTC) | Não           | Não             | Não                  | Padrão: horário da gravação.                     | Quando a conta entrou na comissão.   |

## REUNIAO

Reunião de uma comissão. É a coleção `reunioes`. Este módulo só apaga todos os documentos, no reset; os demais campos, inclusive ata, votações, presenças e anexos gravados dentro da reunião, estão no [dicionário relacional das reuniões](../../3%20-%20Incremento2/reunioes/4%20-%20dicionario-dados-relacional-reunioes.md).

| Campo         | Tipo de dado | Pode ser nulo | Chave primária  | Chave estrangeira    | Outros                         | Descrição                    |
| ------------- | ------------ | ------------- | --------------- | -------------------- | ------------------------------ | ---------------------------- |
| `_id`         | `ObjectId`   | Não           | Sim (exclusivo) | Não                  | Ver tabela de identificadores. | Identifica a reunião.        |
| `comissao_id` | `ObjectId`   | Não           | Não             | Sim (`COMISSAO._id`) | -                              | Comissão que fez a reunião.  |

## ARQUIVO_GRIDFS

Dados de cada arquivo anexado a uma reunião. É a coleção `anexos.files`, criada e mantida pelo GridFS do MongoDB. O reset apaga todos os documentos.

| Campo                 | Tipo de dado              | Pode ser nulo | Chave primária  | Chave estrangeira   | Outros                                   | Descrição                                       |
| --------------------- | ------------------------- | ------------- | --------------- | ------------------- | ---------------------------------------- | ----------------------------------------------- |
| `_id`                 | `ObjectId`                | Não           | Sim (exclusivo) | Não                 | Ver tabela de identificadores.           | Identifica o arquivo.                           |
| `filename`            | Texto (`string`)          | Não           | Não             | Não                 | -                                        | Nome do arquivo.                                |
| `length`              | Inteiro (`long`)          | Não           | Não             | Não                 | Preenchido pelo GridFS.                  | Tamanho em bytes.                               |
| `metadata_reuniao_id` | `ObjectId`                | Não           | Não             | Sim (`REUNIAO._id`) | Gravado como `metadata.reuniao_id`.      | Reunião a que o arquivo pertence.               |

## PARTE_GRIDFS

Pedaço do conteúdo de um arquivo. É a coleção `anexos.chunks`, criada e mantida pelo GridFS. O reset apaga todos os documentos.

| Campo      | Tipo de dado      | Pode ser nulo | Chave primária  | Chave estrangeira          | Outros                                                   | Descrição                          |
| ---------- | ----------------- | ------------- | --------------- | -------------------------- | -------------------------------------------------------- | ---------------------------------- |
| `_id`      | `ObjectId`        | Não           | Sim (exclusivo) | Não                        | Ver tabela de identificadores.                           | Identifica a parte.                |
| `files_id` | `ObjectId`        | Não           | Não             | Sim (`ARQUIVO_GRIDFS._id`) | -                                                        | Arquivo a que a parte pertence.    |
| `n`        | Inteiro (`int`)   | Não           | Não             | Não                        | Começa em 0 e soma 1 por parte; único junto com `files_id`. | Posição da parte dentro do arquivo. |
| `data`     | Binário           | Não           | Não             | Não                        | Até 261.120 bytes (255 KB), padrão do GridFS.            | Conteúdo da parte.                 |

## CONTADOR_ATA

Numeração das atas. É a coleção `contadores`, descrita campo a campo no [dicionário relacional das atas](../../4%20-%20Incremento3/atas/4%20-%20dicionario-dados-relacional-atas.md). O reset apaga todos os documentos.

| Campo   | Tipo de dado    | Pode ser nulo | Chave primária  | Chave estrangeira | Outros                                    | Descrição                                                |
| ------- | --------------- | ------------- | --------------- | ----------------- | ----------------------------------------- | -------------------------------------------------------- |
| `_id`   | Texto           | Não           | Sim (exclusivo) | Não               | Formato `atas:<id da comissão>:<ano>`.    | Identifica a sequência de uma comissão num ano.          |
| `valor` | Inteiro (`int`) | Não           | Não             | Não               | Último número usado.                      | Depois do reset, a próxima ata aprovada recebe o número 1. |

## NOTIFICACAO

Aviso mostrado a uma conta no aplicativo. É a coleção `notificacoes`. Este módulo não lê nem grava esta coleção, e o reset não a apaga.

| Campo        | Tipo de dado     | Pode ser nulo | Chave primária  | Chave estrangeira   | Outros                                             | Descrição                                                                                 |
| ------------ | ---------------- | ------------- | --------------- | ------------------- | -------------------------------------------------- | ----------------------------------------------------------------------------------------- |
| `_id`        | `ObjectId`       | Não           | Sim (exclusivo) | Não                 | Ver tabela de identificadores.                     | Identifica a notificação.                                                                 |
| `usuario_id` | `ObjectId`       | Não           | Não             | Sim (`USUARIO._id`) | Indexado; único junto com `chave`.                 | Conta que recebe o aviso. Depois do reset, pode apontar para uma conta que não existe mais. |
| `chave`      | Texto (`string`) | Não           | Não             | Não                 | Único junto com `usuario_id`.                      | Identifica o assunto do aviso, para não repetir a mesma notificação à mesma conta.       |

## Índices e restrições

- `usuarios.email` tem índice único: popular os dados de teste duas vezes, sem resetar no meio, falha na inserção das contas;
- `organizacoes.codigo_acesso` tem índice único; o código da organização de teste é sorteado na criação;
- `notificacoes` tem índice único no par `usuario_id` e `chave`, e índice em `usuario_id`;
- a regra "exatamente um `RESPONSAVEL` e nenhuma conta repetida" de `COMISSAO_MEMBRO` é uma validação do Mongoose, conferida em cada gravação da comissão, inclusive na comissão de teste;
- as contagens dos relatórios usam filtros por igualdade em `usuarios.ativo`, `organizacoes.status` e `comissoes.ativo`; nenhum desses campos tem índice próprio, então o MongoDB percorre a coleção inteira em cada contagem;
- o reset usa `deleteMany` com filtro vazio em seis coleções e com `tipo` diferente de `ADMIN_SISTEMA` em `usuarios`; o MongoDB não confere chaves estrangeiras, então nada impede que restem referências a documentos apagados, como acontece em `notificacoes`.

## Rastreabilidade

RF-SIS-01 a RF-SIS-04; [Diagrama relacional](4%20-%20diagrama-relacional-informacao-sistemas.md) e [Testes](7%20-%20testes-informacao-sistemas.md).
