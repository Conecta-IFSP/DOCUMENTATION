# Dicionário relacional — Relatório individual

> Padrão da disciplina · Módulo REL · Campos do diagrama relacional · Revisado em 26/11/2026

## Resumo

Descreve cada campo das entidades do [diagrama relacional](4%20-%20diagrama-relacional-relatorio-individual.md), com tipo, obrigatoriedade, chaves e forma de geração. As colunas seguem o dicionário de dados do documento de exemplo da disciplina: campo, tipo de dado, se pode ser nulo, chave primária, chave estrangeira, outros e descrição. O relatório só lê estas entidades; quem cria e altera cada uma está indicado no texto de cada tabela. No fim, estão os campos que o relatório calcula e devolve sem gravar.

## Como os identificadores são criados

| Identificador                                                     | Quem gera                                                                                  | Formato e valor inicial                                                                                                                                                                                                                                       | Incremento                                                                                                                                                                       |
| ----------------------------------------------------------------- | ------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `USUARIO._id`, `ORGANIZACAO._id`, `COMISSAO._id`, `REUNIAO._id`   | Driver do MongoDB usado pelo Mongoose, dentro do processo da API, quando o documento é criado pelo módulo dono (cadastro de usuário, de organização, de comissão ou de reunião). | `ObjectId` de 12 bytes, mostrado como 24 caracteres hexadecimais: 4 bytes com os segundos desde 1/1/1970 no momento da criação, 5 bytes aleatórios sorteados uma vez por processo e 3 bytes de um contador. Não há valor inicial fixo: o contador começa num valor aleatório quando o processo inicia. | O contador soma 1 a cada novo identificador gerado pelo mesmo processo. Como os outros bytes mudam com o tempo e entre processos, os valores crescem com o tempo, mas não são contínuos nem previsíveis. |
| Chave de `ORGANIZACAO_MEMBRO`, `COMISSAO_MEMBRO`, `REUNIAO_PARTICIPANTE` e `REUNIAO_PRESENCA` | Não é gerada                                                                              | Os subdocumentos são gravados sem `_id` próprio (`_id: false` no schema). A chave lógica é o `_id` do documento pai mais o `usuario_id`.                                                                                                                          | Não se aplica.                                                                                                                                                                   |
| `reuniao_id`, `vinculos.*.id` e `vinculos.comissoes[].organizacao_id` da resposta | `RelatoriosService.individual`, por `toString()`                                       | O mesmo `ObjectId` do documento lido, escrito como texto de 24 caracteres hexadecimais.                                                                                                                                                                      | Não se aplica: o relatório não cria identificador novo.                                                                                                                          |

## USUARIO

Conta de quem usa o sistema. Coleção `usuarios`, criada e alterada pelo módulo de cadastro de usuário. O relatório lê só a conta do token.

| Campo   | Tipo de dado             | Pode ser nulo | Chave primária  | Chave estrangeira | Outros                                                         | Descrição                                                        |
| ------- | ------------------------ | ------------- | --------------- | ----------------- | -------------------------------------------------------------- | ---------------------------------------------------------------- |
| `_id`   | `ObjectId`               | Não           | Sim (exclusiva) | Não               | Gerado como descrito acima.                                    | Identifica a conta; vem no campo `sub` do token.                 |
| `nome`  | Texto (`string`)         | Não           | Não             | Não               | Espaços das pontas removidos.                                  | Nome mostrado no topo do relatório, no PDF e no CSV.             |
| `email` | Texto (`string`)         | Não           | Não             | Não               | Único na coleção; gravado em minúsculas.                       | E-mail mostrado no PDF e no CSV.                                 |
| `tipo`  | Texto com valores fixos  | Não           | Não             | Não               | `USUARIO` (padrão) ou `ADMIN_SISTEMA`.                         | Só `USUARIO` acessa o relatório.                                 |
| `ativo` | Booleano                 | Não           | Não             | Não               | Padrão verdadeiro.                                             | Conta inativa recebe 401 e não consulta.                         |

## ORGANIZACAO

Instituição que reúne comissões. Coleção `organizacoes`, criada pelo módulo de cadastro de organização e aprovada pelo administrador do sistema.

| Campo    | Tipo de dado            | Pode ser nulo | Chave primária  | Chave estrangeira | Outros                                         | Descrição                                                     |
| -------- | ----------------------- | ------------- | --------------- | ----------------- | ---------------------------------------------- | ------------------------------------------------------------- |
| `_id`    | `ObjectId`              | Não           | Sim (exclusiva) | Não               | Gerado como descrito acima.                    | Identifica a organização; devolvido como `vinculos.organizacoes[].id`. |
| `nome`   | Texto (`string`)        | Não           | Não             | Não               | Espaços das pontas removidos.                  | Nome mostrado nos vínculos e nas reuniões; critério de ordem. |
| `status` | Texto com valores fixos | Não           | Não             | Não               | `PENDENTE` (padrão), `APROVADA` ou `REVOGADA`. | Só `APROVADA` gera vínculo no relatório.                      |

## ORGANIZACAO_MEMBRO

Vínculo de um usuário com uma organização. Fica em `organizacoes.membros[]`. O módulo de organizações acrescenta o membro só quando o usuário ainda não está na lista, e aprova, rejeita ou convida.

| Campo            | Tipo de dado            | Pode ser nulo | Chave primária  | Chave estrangeira       | Outros                                                        | Descrição                                                   |
| ---------------- | ----------------------- | ------------- | --------------- | ----------------------- | ------------------------------------------------------------- | ----------------------------------------------------------- |
| `organizacao_id` | `ObjectId`              | Não           | Sim (inclusiva) | Sim (`ORGANIZACAO._id`) | Não é gravado: é o documento que contém a lista.              | Organização do vínculo.                                     |
| `usuario_id`     | `ObjectId`              | Não           | Sim (inclusiva) | Sim (`USUARIO._id`)     | O relatório aceita também o mesmo valor gravado como texto.   | Membro.                                                     |
| `papel`          | Texto com valores fixos | Não           | Não             | Não                     | `ADMIN` ou `MEMBRO`.                                          | Papel mostrado como "Administrador" ou "Membro".            |
| `status`         | Texto com valores fixos | Não           | Não             | Não                     | `PENDENTE` (padrão), `APROVADO`, `REJEITADO` ou `CONVIDADO`.  | Só `APROVADO` gera vínculo no relatório.                    |

## COMISSAO

Grupo de trabalho de uma organização. Coleção `comissoes`, criada e alterada pelo módulo de cadastro de comissão.

| Campo            | Tipo de dado     | Pode ser nulo | Chave primária  | Chave estrangeira       | Outros                                                         | Descrição                                                                          |
| ---------------- | ---------------- | ------------- | --------------- | ----------------------- | -------------------------------------------------------------- | ---------------------------------------------------------------------------------- |
| `_id`            | `ObjectId`       | Não           | Sim (exclusiva) | Não                     | Gerado como descrito acima.                                    | Identifica a comissão; devolvido como `vinculos.comissoes[].id`.                    |
| `nome`           | Texto (`string`) | Não           | Não             | Não                     | Espaços das pontas removidos.                                  | Nome mostrado nos vínculos e nas reuniões; critério de ordem.                       |
| `organizacao_id` | `ObjectId`       | Não           | Não             | Sim (`ORGANIZACAO._id`) | O relatório aceita também o mesmo valor gravado como texto.    | Organização da comissão; devolvido como `vinculos.comissoes[].organizacao_id`.      |
| `ativo`          | Booleano         | Não           | Não             | Não                     | Padrão verdadeiro.                                             | Só comissão ativa gera vínculo no relatório.                                        |

## COMISSAO_MEMBRO

Participação de um usuário na equipe de uma comissão. Fica em `comissoes.membros[]`.

| Campo         | Tipo de dado            | Pode ser nulo | Chave primária  | Chave estrangeira    | Outros                                                                   | Descrição                                          |
| ------------- | ----------------------- | ------------- | --------------- | -------------------- | ------------------------------------------------------------------------ | -------------------------------------------------- |
| `comissao_id` | `ObjectId`              | Não           | Sim (inclusiva) | Sim (`COMISSAO._id`) | Não é gravado: é o documento que contém a lista.                         | Comissão do vínculo.                               |
| `usuario_id`  | `ObjectId`              | Não           | Sim (inclusiva) | Sim (`USUARIO._id`)  | Não se repete na mesma comissão; aceito também como texto pelo relatório. | Membro da equipe.                                  |
| `papel`       | Texto com valores fixos | Não           | Não             | Não                  | `RESPONSAVEL` ou `MEMBRO`; exatamente um `RESPONSAVEL` por comissão.     | Papel mostrado como "Responsável" ou "Membro".     |

## REUNIAO

Reunião de uma comissão. Coleção `reunioes`, criada e alterada pelo módulo de reuniões.

| Campo         | Tipo de dado              | Pode ser nulo | Chave primária  | Chave estrangeira    | Outros                                          | Descrição                                                                      |
| ------------- | ------------------------- | ------------- | --------------- | -------------------- | ----------------------------------------------- | ------------------------------------------------------------------------------ |
| `_id`         | `ObjectId`                | Não           | Sim (exclusiva) | Não                  | Gerado como descrito acima.                     | Identifica a reunião; devolvido como `reuniao_id`.                             |
| `comissao_id` | `ObjectId`                | Não           | Não             | Sim (`COMISSAO._id`) | -                                               | Comissão que realiza a reunião; dá o nome da comissão e da organização.        |
| `titulo`      | Texto (`string`)          | Não           | Não             | Não                  | Espaços das pontas removidos.                   | Título do cartão e do bloco do PDF.                                            |
| `local`       | Texto (`string`)          | Não           | Não             | Não                  | Padrão: texto vazio.                            | Local; vazio vira nulo na resposta e "Local não informado" na tela.            |
| `inicio`      | Data e hora (`date`, UTC) | Não           | Não             | Não                  | -                                               | Início marcado; filtra o período e ordena o relatório.                         |
| `fim`         | Data e hora (`date`, UTC) | Não           | Não             | Não                  | -                                               | Término marcado; mostrado como "Término".                                      |
| `dia_inteiro` | Booleano                  | Não           | Não             | Não                  | Padrão falso.                                   | Reunião de dia inteiro.                                                        |
| `situacao`    | Texto com valores fixos   | Não           | Não             | Não                  | `AGENDADA` (padrão) ou `CANCELADA`.             | Só `AGENDADA` entra no relatório.                                              |

## REUNIAO_PARTICIPANTE

Pessoa convidada para a reunião. Fica em `reunioes.participantes[]`.

| Campo        | Tipo de dado            | Pode ser nulo | Chave primária  | Chave estrangeira   | Outros                                                        | Descrição                       |
| ------------ | ----------------------- | ------------- | --------------- | ------------------- | ------------------------------------------------------------- | ------------------------------- |
| `reuniao_id` | `ObjectId`              | Não           | Sim (inclusiva) | Sim (`REUNIAO._id`) | Não é gravado: é o documento que contém a lista.              | Reunião.                        |
| `usuario_id` | `ObjectId`              | Não           | Sim (inclusiva) | Sim (`USUARIO._id`) | O relatório aceita também o mesmo valor gravado como texto.   | Participante.                   |
| `convite`    | Texto com valores fixos | Não           | Não             | Não                 | `PENDENTE` (padrão), `ACEITO` ou `RECUSADO`.                  | Resposta ao convite; não é usada pelo relatório. |

## REUNIAO_PRESENCA

Registro de entrada e saída de um participante. Fica em `reunioes.participantes[].presenca`; não existe enquanto o participante não registra entrada.

| Campo              | Tipo de dado              | Pode ser nulo | Chave primária  | Chave estrangeira   | Outros                                                       | Descrição                                                                                       |
| ------------------ | ------------------------- | ------------- | --------------- | ------------------- | ------------------------------------------------------------ | ----------------------------------------------------------------------------------------------- |
| `reuniao_id`       | `ObjectId`                | Não           | Sim (inclusiva) | Sim (`REUNIAO._id`) | Não é gravado: vem do documento da reunião.                  | Reunião.                                                                                        |
| `usuario_id`       | `ObjectId`                | Não           | Sim (inclusiva) | Sim (`USUARIO._id`) | Não é gravado aqui: vem do participante que contém a presença. | Participante.                                                                                 |
| `entrada`          | Data e hora (`date`, UTC) | Não           | Não             | Não                 | Precisa ser até o horário da consulta para entrar.           | Início da participação e da duração.                                                            |
| `saida`            | Data e hora (`date`, UTC) | Sim           | Não             | Não                 | Nula até a saída ser registrada.                             | Fim da participação; sem ela, a duração não é calculada.                                        |
| `saida_automatica` | Booleano                  | Sim           | Não             | Não                 | Só existe com valor verdadeiro.                              | Indica que a rotina do módulo de reuniões gravou a saída no término previsto (`saida = fim`).   |
| `situacao`         | Texto com valores fixos   | Não           | Não             | Não                 | `AGUARDANDO`, `CONFIRMADA` ou `RECUSADA`.                    | Só `CONFIRMADA` entra no relatório.                                                             |
| `marcada_por`      | Texto com valores fixos   | Não           | Não             | Não                 | `LINK`, `PROPRIO` ou `RESPONSAVEL`.                          | Como a presença foi registrada; não é usada pelo relatório.                                     |

## Campos calculados pelo relatório (não gravados)

| Campo da resposta                | Tipo de dado              | Pode ser nulo | Origem e regra                                                                                                                     |
| -------------------------------- | ------------------------- | ------------- | ---------------------------------------------------------------------------------------------------------------------------------- |
| `gerada_em`                      | Data e hora (UTC)         | Não           | Relógio do servidor no início da consulta; também é o limite das reuniões já iniciadas e das saídas válidas.                       |
| `reunioes[].minutos`             | Inteiro                   | Sim           | Parte inteira de (`saida` − `entrada`) em minutos; nulo se a saída não existe, é inválida, é anterior à entrada ou é posterior a `gerada_em`. |
| `reunioes[].saida_automatica`    | Booleano                  | Não           | Verdadeiro só quando `REUNIAO_PRESENCA.saida_automatica` é verdadeiro e há saída.                                                  |
| `reunioes[].comissao`, `organizacao` | Texto                 | Não           | Nome atual da comissão e da organização; "Comissão indisponível" ou "Organização indisponível" quando o documento não existe.      |
| `resumo.minutos_registrados`     | Inteiro                   | Não           | Soma de `minutos`, contando nulo como zero.                                                                                       |
| `resumo.registros_sem_duracao`   | Inteiro                   | Não           | Quantidade de reuniões com `minutos` nulo.                                                                                         |
| `vinculos.organizacoes[].papel`  | Texto com valores fixos   | Não           | `ORGANIZACAO_MEMBRO.papel` do item do usuário com `status` `APROVADO`.                                                             |
| `vinculos.comissoes[].organizacao` | Texto                   | Não           | `ORGANIZACAO.nome` da organização da comissão, lida na mesma consulta dos vínculos.                                                |

## Índices e restrições

- a busca das reuniões filtra por `participantes.usuario_id` e `inicio`, que formam o índice `{ "participantes.usuario_id": 1, "inicio": 1 }` da coleção `reunioes`;
- as buscas dos vínculos filtram `organizacoes` por `status` e `membros.usuario_id` e `comissoes` por `ativo`, `organizacao_id` e `membros.usuario_id`; essas coleções não têm índice nesses campos, então o MongoDB percorre a coleção inteira; convém criar `{ "membros.usuario_id": 1 }` nas duas quando o volume crescer (ainda não criado);
- `usuarios.email` tem índice único, o que garante um e-mail por conta;
- as referências entre coleções (`comissao_id`, `organizacao_id` e `usuario_id`) são gravadas pelos módulos donos; o MongoDB não impede que o documento apontado seja apagado depois, e o relatório trata a falta como descrito em "Campos calculados";
- registros antigos com `usuario_id` ou `organizacao_id` gravados como texto, e não como `ObjectId`, são aceitos pelas buscas do relatório, que comparam com as duas formas.

## Rastreabilidade

RF-REL-01 a RF-REL-05; [Diagrama relacional](4%20-%20diagrama-relacional-relatorio-individual.md) e [Testes](7%20-%20testes-relatorio-individual.md).
