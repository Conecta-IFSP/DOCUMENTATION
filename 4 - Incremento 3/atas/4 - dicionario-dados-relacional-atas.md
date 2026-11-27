# Dicionário relacional — Atas de reunião

> Padrão da disciplina · Módulo ATA · Campos do diagrama relacional · Revisado em 27/11/2026

## Resumo

Descreve cada campo das entidades do [diagrama relacional](4%20-%20diagrama-relacional-atas.md), com tipo, obrigatoriedade, chaves e forma de geração. As colunas seguem o dicionário de dados do documento de exemplo da disciplina: campo, tipo de dado, se pode ser nulo, chave primária, chave estrangeira, outros e descrição.

## Como os identificadores são criados

| Identificador                    | Quem gera                             | Formato e valor inicial                                                                                                                                                                       | Incremento                                                                   |
| -------------------------------- | ------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------- |
| `REUNIAO._id`                    | Driver do MongoDB, no aplicativo servidor | `ObjectId` de 12 bytes, mostrado como 24 caracteres hexadecimais. É formado por 4 bytes com a hora da criação em segundos, 5 bytes sorteados uma vez por processo e 3 bytes de um contador interno. Não tem valor inicial fixo. | Não é sequencial. O contador interno sobe de 1 em 1 dentro do mesmo processo, mas os outros bytes mudam, então os valores não são previsíveis nem contínuos. |
| `ATA` (chave lógica)             | Não é gerado                          | A ata vigente é identificada pela reunião em que está, mais o campo `versao`.                                                                                                                  | `versao` começa em 1 e sobe de 1 a cada salvamento ou nova versão.           |
| `ATA.numero`                     | `ReunioesService.proximoNumeroAta`    | Inteiro. O primeiro número de cada comissão em cada ano é 1.                                                                                                                                  | Sobe de 1 em 1, por comissão e por ano, só quando a ata é aprovada.          |
| `CONTADOR_ATA._id`               | `ReunioesService.proximoNumeroAta`    | Texto montado pelo sistema: `atas:<id da comissão>:<ano>`, por exemplo `atas:66f3c1d2e4b07a9f2c1a4b0f:2026`.                                                                                   | Não muda. Um documento por comissão e ano.                                   |
| `CONTADOR_ATA.valor`             | MongoDB, pela operação `$inc`         | Inteiro. Na primeira ata aprovada o documento é criado com `valor = 1` (`upsert`).                                                                                                            | Sobe de 1 em 1. A operação é atômica, então duas aprovações ao mesmo tempo nunca recebem o mesmo número. |

Um número de ata consumido não volta: nenhuma operação das atas ou das reuniões devolve um número ao contador, e a API não tem rota para apagar uma reunião. Só o reset do sistema, do módulo de Informações do Sistema, apaga de uma vez as reuniões, as comissões e a coleção `contadores`. O número é reservado antes de a ata ser gravada; se essa gravação falhar (conflito 409, ou a aprovação pelo fim do prazo não encontrar a ata como foi lida), o número reservado fica sem ata e a sequência pula um valor.

## REUNIAO

Documento da coleção `reunioes`. Só aparecem os campos que as atas usam; os demais e as regras de gravação estão no [dicionário relacional das reuniões](../../3%20-%20Incremento%202/reunioes/4%20-%20dicionario-dados-relacional-reunioes.md).

| Campo               | Tipo de dado              | Pode ser nulo | Chave primária  | Chave estrangeira    | Outros                                                                                          | Descrição                                                                    |
| ------------------- | ------------------------- | ------------- | --------------- | -------------------- | ----------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------- |
| `_id`               | `ObjectId`                | Não           | Sim (exclusivo) | Não                  | Ver "Como os identificadores são criados".                                                     | Identificador da reunião; a ata e as versões canceladas são identificadas por ele. |
| `comissao_id`       | `ObjectId`                | Não           | Não             | Sim (`COMISSAO._id`) | -                                                                                               | Comissão em que se confere o presidente e cuja sequência numera a ata.       |
| `titulo`            | Texto                     | Não           | Não             | Não                  | Gravado sem os espaços das pontas.                                                              | Título mostrado na tela da ata, nas pendências e no PDF.                     |
| `fim`               | Data e hora (`date`, UTC) | Não           | Não             | Não                  | A ata só é escrita, enviada, avaliada ou refeita a partir desse instante.                       | Término da reunião.                                                          |
| `situacao`          | Texto com valores fixos   | Não           | Não             | Não                  | `AGENDADA` ou `CANCELADA`; valor inicial `AGENDADA`.                                            | A ata só muda enquanto a reunião está `AGENDADA`.                            |
| `presencas_revisao` | Inteiro (`int`)           | Sim           | Não             | Não                  | Padrão 0; soma 1 a cada mudança de presença. Documentos antigos podem não ter o campo, e o valor 0 também aceita a ausência. | Revisão das presenças; `gravarAta` só grava se ela não mudou desde a leitura. |

## REUNIAO_PARTICIPANTE

Convidado da reunião. Fica em `reunioes.participantes[]`.

| Campo        | Tipo de dado            | Pode ser nulo | Chave primária  | Chave estrangeira   | Outros                                                    | Descrição                                                   |
| ------------ | ----------------------- | ------------- | --------------- | ------------------- | --------------------------------------------------------- | ----------------------------------------------------------- |
| `reuniao_id` | `ObjectId`              | Não           | Sim (inclusivo) | Sim (`REUNIAO._id`) | Não é gravado: é o documento que contém o participante.   | Reunião do convite.                                         |
| `usuario_id` | `ObjectId`              | Não           | Sim (inclusivo) | Sim (`USUARIO._id`) | Não se repete na mesma reunião.                           | Pessoa convidada; só os participantes consultam a ata.      |
| `convite`    | Texto com valores fixos | Não           | Não             | Não                 | `PENDENTE`, `ACEITO` ou `RECUSADO`; padrão `PENDENTE`.    | Resposta ao convite. As atas não leem este campo.           |

## REUNIAO_PRESENCA

Presença de um participante. Fica em `reunioes.participantes[].presenca`.

| Campo        | Tipo de dado              | Pode ser nulo | Chave primária  | Chave estrangeira                       | Outros                                       | Descrição                                                                 |
| ------------ | ------------------------- | ------------- | --------------- | --------------------------------------- | -------------------------------------------- | ------------------------------------------------------------------------- |
| `reuniao_id` | `ObjectId`                | Não           | Sim (inclusivo) | Sim (`REUNIAO_PARTICIPANTE.reuniao_id`) | Não é gravado: vem do participante.          | Reunião da presença.                                                      |
| `usuario_id` | `ObjectId`                | Não           | Sim (inclusivo) | Sim (`REUNIAO_PARTICIPANTE.usuario_id`) | Não é gravado: vem do participante.          | Pessoa da presença.                                                       |
| `entrada`    | Data e hora (`date`, UTC) | Não           | Não             | Não                                     | -                                            | Momento da entrada.                                                       |
| `saida`      | Data e hora (`date`, UTC) | Sim           | Não             | Não                                     | Ausente enquanto a pessoa não sai.           | Momento da saída.                                                         |
| `situacao`   | Texto com valores fixos   | Não           | Não             | Não                                     | `AGUARDANDO`, `CONFIRMADA` ou `RECUSADA`.    | Só quem está `CONFIRMADA` no envio entra em `ATA_APROVADOR`.              |

## ATA

Guarda a versão vigente da ata de uma reunião. Fica em `reunioes.ata`.

| Campo           | Tipo de dado                   | Pode ser nulo | Chave primária   | Chave estrangeira | Outros                                 | Descrição                                                                          |
| --------------- | ------------------------------ | ------------- | ---------------- | ----------------- | -------------------------------------- | ----------------------------------------------------------------------------------- |
| `reuniao_id`    | `ObjectId`                     | Não           | Sim (exclusivo)  | Sim (`REUNIAO._id`) | Não é gravado: é o documento que contém a ata. | Reunião a que a ata pertence.                                                        |
| `conteudo`      | Texto (`string`, 20 a 20.000)  | Não           | Não              | Não               | Parágrafos separados por quebra de linha. | Texto da ata escrito pelo presidente.                                                |
| `versao`        | Inteiro (`int`)                | Não           | Não              | Não               | Começa em 1; sobe de 1 em 1.           | Número da versão, usado para evitar que duas pessoas gravem por cima uma da outra.  |
| `situacao`      | Texto com valores fixos        | Não           | Não              | Não               | `RASCUNHO`, `EM_APROVACAO`, `APROVADA` ou `REPROVADA`. | Fase em que a ata está.                                                              |
| `autor_id`      | `ObjectId`                     | Não           | Não              | Sim (`USUARIO._id`) | -                                      | Presidente que escreveu esta versão.                                                 |
| `atualizada_em` | Data e hora (`date`, UTC)      | Não           | Não              | Não               | Na aprovação pelo fim do prazo, recebe o valor de `prazo`. | Momento da última gravação.                                              |
| `enviada_em`    | Data e hora (`date`, UTC)      | Sim           | Não              | Não               | Nulo enquanto for rascunho.            | Momento em que a ata foi enviada para aprovação.                                     |
| `prazo`         | Data e hora (`date`, UTC)      | Sim           | Não              | Não               | `enviada_em` + 24, 72 ou 168 horas.    | Data limite para os presentes responderem; quem não responde até ela aprova automaticamente. |
| `numero`        | Inteiro (`int`)                | Sim           | Não              | Não               | Preenchido só na aprovação.            | Número da ata dentro da comissão, no ano.                                            |
| `ano`           | Inteiro (`int`)                | Sim           | Não              | Não               | Ano no fuso `America/Sao_Paulo`.       | Ano usado junto com o número, como em "Ata nº 2/2026".                               |
| `aprovada_em`   | Data e hora (`date`, UTC)      | Sim           | Não              | Não               | -                                      | Momento em que a última aprovação chegou; na aprovação pelo fim do prazo, igual a `prazo`. |

## ATA_APROVADOR

Lista de quem precisa avaliar a ata. Fica em `reunioes.ata.aprovadores[]`, como uma lista de identificadores de usuário.

| Campo        | Tipo de dado | Pode ser nulo | Chave primária  | Chave estrangeira    | Outros                                                        | Descrição                                                                                      |
| ------------ | ------------ | ------------- | --------------- | -------------------- | ------------------------------------------------------------- | ------------------------------------------------------------------------------------------------ |
| `reuniao_id` | `ObjectId`   | Não           | Sim (inclusivo) | Sim (`REUNIAO._id`)  | Não é gravado: vem do documento que contém a ata.             | Reunião da ata.                                                                                  |
| `versao`     | Inteiro      | Não           | Sim (inclusivo) | Não                  | Não é gravado: vem de `ATA.versao`.                           | Versão da ata a que esta lista pertence.                                                        |
| `usuario_id` | `ObjectId`   | Não           | Sim (inclusivo) | Sim (`USUARIO._id`)  | É o próprio item da lista; não se repete.                     | Participante com presença `CONFIRMADA` no envio, que deve aprovar ou reprovar esta versão.      |

A lista é montada uma única vez, no envio, e não muda enquanto a versão estiver aberta. O envio só é gravado se `presencas_revisao` não mudou desde a leitura, e, depois dele, a API recusa qualquer mudança de presença na reunião. As presenças só voltam a mudar quando uma nova versão é criada; o envio dessa versão monta a lista de novo.

## ATA_AVALIACAO

Decisão de um aprovador. Fica em `reunioes.ata.avaliacoes[]`.

| Campo           | Tipo de dado                | Pode ser nulo | Chave primária  | Chave estrangeira   | Outros                                            | Descrição                                                                 |
| --------------- | --------------------------- | ------------- | --------------- | ------------------- | ------------------------------------------------- | --------------------------------------------------------------------------- |
| `reuniao_id`    | `ObjectId`                  | Não           | Sim (inclusivo) | Sim (`REUNIAO._id`) | Não é gravado: vem do documento que contém a ata. | Reunião da ata.                                                             |
| `versao`        | Inteiro                     | Não           | Sim (inclusivo) | Não                 | Não é gravado: vem de `ATA.versao`.               | Versão avaliada.                                                            |
| `usuario_id`    | `ObjectId`                  | Não           | Sim (inclusivo) | Sim (`USUARIO._id`) | Precisa constar em `ATA_APROVADOR`.               | Quem avaliou.                                                               |
| `decisao`       | Texto com valores fixos     | Não           | Não             | Não                 | `APROVADA` ou `REPROVADA`.                        | Resultado da avaliação. Não pode ser trocado depois.                       |
| `trecho`        | Texto (até 20.000)          | Sim           | Não             | Não                 | Só na reprovação; precisa existir em `ATA.conteudo`. | Parágrafo apontado como problema.                                        |
| `justificativa` | Texto (até 500)             | Sim           | Não             | Não                 | Só na reprovação; opcional.                       | Explicação do que precisa mudar.                                           |
| `em`            | Data e hora (`date`, UTC)   | Não           | Não             | Não                 | -                                                 | Momento da decisão; é o que o aplicativo mostra como "Aprovou em …". Na aprovação automática, igual a `ATA.prazo`. |
| `automatica`    | Booleano (`bool`)           | Sim           | Não             | Não                 | Só existe com o valor verdadeiro.                 | Indica que o servidor registrou a aprovação no fim do prazo porque a pessoa não respondeu; o aplicativo mostra "Aprovada pelo fim do prazo em …". |

## ATA_CANCELADA

Versão reprovada que deu lugar a outra. Fica em `reunioes.atas_canceladas[]`, com os mesmos campos de `ATA` mais os de baixo.

| Campo          | Tipo de dado              | Pode ser nulo | Chave primária  | Chave estrangeira   | Outros                                  | Descrição                                                    |
| -------------- | ------------------------- | ------------- | --------------- | ------------------- | --------------------------------------- | -------------------------------------------------------------- |
| `reuniao_id`   | `ObjectId`                | Não           | Sim (inclusivo) | Sim (`REUNIAO._id`) | Não é gravado: vem do documento pai.    | Reunião da ata.                                                |
| `versao`       | Inteiro                   | Não           | Sim (inclusivo) | Não                 | Única dentro da reunião.                | Identifica a versão cancelada.                                 |
| `situacao`     | Texto com valores fixos   | Não           | Não             | Não                 | Sempre `CANCELADA` nesta lista.         | Marca a versão como encerrada sem aprovação.                   |
| `cancelada_em` | Data e hora (`date`, UTC) | Não           | Não             | Não                 | -                                       | Momento em que o presidente criou a versão seguinte.           |

Os campos `conteudo`, `autor_id`, `aprovadores` e `avaliacoes` são copiados como estavam, para que o histórico mostre o texto reprovado e quem reprovou.

## CONTADOR_ATA

Sequência de numeração. É a coleção `contadores`.

| Campo   | Tipo de dado    | Pode ser nulo | Chave primária  | Chave estrangeira | Outros                                                              | Descrição                                                        |
| ------- | --------------- | ------------- | --------------- | ----------------- | ------------------------------------------------------------------- | ------------------------------------------------------------------ |
| `_id`   | Texto           | Não           | Sim (exclusivo) | Não               | Formato `atas:<id da comissão>:<ano>`; montado pelo sistema.        | Identifica a sequência de atas de uma comissão num ano.           |
| `comissao_id` | `ObjectId` | Não         | Não             | Sim (`COMISSAO._id`) | Não é gravado: é a parte do meio do `_id`.                       | Comissão da sequência.                                             |
| `ano`   | Inteiro (`int`) | Não           | Não             | Não               | Não é gravado: é a parte final do `_id`, no fuso `America/Sao_Paulo`. | Ano da sequência.                                                |
| `valor` | Inteiro (`int`) | Não           | Não             | Não               | Criado com 1 no primeiro uso; sobe de 1 em 1 por operação atômica.  | Último número de ata entregue para essa comissão naquele ano.     |

O `_id` carrega a comissão e o ano, por isso o documento não tem campos separados para eles; no diagrama, `comissao_id` e `ano` aparecem como colunas lógicas, lidas do próprio identificador.

## COMISSAO

Documento da coleção `comissoes`. Só aparecem os campos que as atas usam; os demais estão no módulo de comissões.

| Campo   | Tipo de dado      | Pode ser nulo | Chave primária  | Chave estrangeira | Outros                                               | Descrição                                                                  |
| ------- | ----------------- | ------------- | --------------- | ----------------- | ---------------------------------------------------- | -------------------------------------------------------------------------- |
| `_id`   | `ObjectId`        | Não           | Sim (exclusivo) | Não               | Gerado da mesma forma que `REUNIAO._id`.             | Identificador da comissão; entra no `_id` de `CONTADOR_ATA`.               |
| `ativo` | Booleano (`bool`) | Não           | Não             | Não               | Padrão verdadeiro.                                   | Com a comissão inativa, o presidente não escreve, não envia nem refaz a ata. |

## USUARIO

Documento da coleção `usuarios`. Só aparecem os campos que as atas usam; os demais estão no módulo de cadastro de usuário.

| Campo   | Tipo de dado      | Pode ser nulo | Chave primária  | Chave estrangeira | Outros                                   | Descrição                                                                      |
| ------- | ----------------- | ------------- | --------------- | ----------------- | ---------------------------------------- | ------------------------------------------------------------------------------ |
| `_id`   | `ObjectId`        | Não           | Sim (exclusivo) | Não               | Gerado da mesma forma que `REUNIAO._id`. | Identificador referenciado por participantes, autores, aprovadores e avaliações. |
| `nome`  | Texto             | Não           | Não             | Não               | Gravado sem os espaços das pontas.       | Nome mostrado na lista de aprovadores, nas faixas de reprovação e no PDF.      |
| `ativo` | Booleano (`bool`) | Não           | Não             | Não               | Padrão verdadeiro.                       | Conta desativada não usa as rotas das atas: a API a recusa antes.              |

## Índices e restrições

- a busca de atas pendentes usa os campos `situacao`, `ata.situacao`, `ata.aprovadores`, `ata.avaliacoes.usuario_id` e `ata.autor_id`, e ordena por `ata.prazo`; convém criar o índice `{ "ata.situacao": 1, "ata.prazo": 1 }` quando o volume de reuniões crescer (ainda não criado);
- `contadores._id` é a chave primária da coleção e já é indexada pelo MongoDB;
- toda gravação da ata feita por uma pessoa usa `findOneAndUpdate` com `_id`, `situacao = AGENDADA` e a ata lida como filtro; salvar, enviar, aprovar e reprovar (`gravarAta`) também filtram pelo `presencas_revisao` lido. Se outra pessoa gravou antes ou uma presença mudou, a operação não encontra o documento e a API devolve 409;
- a busca das atas com prazo encerrado usa `situacao`, `ata.situacao` e `ata.prazo`, e cada uma é gravada com `updateOne` usando a ata lida como filtro, para não sobrescrever uma avaliação gravada ao mesmo tempo;
- a ata não pode ser alterada depois do envio: só muda de situação por aprovação, inclusive a automática no fim do prazo, reprovação ou criação de nova versão.

## Rastreabilidade

RF-ATA-01 a RF-ATA-08; [Diagrama relacional](4%20-%20diagrama-relacional-atas.md) e [Testes](7%20-%20testes-atas.md).
