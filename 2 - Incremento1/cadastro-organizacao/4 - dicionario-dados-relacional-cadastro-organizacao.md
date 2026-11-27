# Dicionário relacional — Cadastro de organização

> Padrão da disciplina · Módulo ORG · Campos do diagrama relacional · Revisado em 26/11/2026

## Resumo

Descreve cada campo das entidades do [diagrama relacional](4%20-%20diagrama-relacional-cadastro-organizacao.md), com tipo, obrigatoriedade, chaves e forma de geração. As colunas seguem o dicionário de dados do documento de exemplo da disciplina: campo, tipo de dado, se pode ser nulo, chave primária, chave estrangeira, outros e descrição.

## Como os identificadores são criados

| Identificador                       | Quem gera                                                     | Formato e valor inicial                                                                                                                                                                                                                                                                 | Incremento                                                                                                                                                                  |
| ----------------------------------- | ------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `ORGANIZACAO._id`                   | Driver do MongoDB, no servidor da API, no momento da inserção | `ObjectId` de 12 bytes, mostrado como 24 caracteres hexadecimais: 4 bytes com os segundos desde 01/01/1970, 5 bytes aleatórios sorteados uma vez por processo e 3 bytes de um contador que começa num valor aleatório. Não tem valor inicial fixo.                                       | O contador soma 1 a cada novo identificador gerado pelo mesmo processo; como os outros bytes mudam com o tempo e entre processos, os valores não são previsíveis nem contínuos. |
| `ORGANIZACAO_MEMBRO` (chave lógica) | Não é gerado                                                  | O vínculo é identificado pela organização que o contém mais o `usuario_id`. O subdocumento não tem `_id` próprio.                                                                                                                                                                        | Não se aplica.                                                                                                                                                              |
| `ORGANIZACAO.codigo_acesso`         | `gerarCodigoAcesso()`, no servidor da API                     | Texto de 10 caracteres, cada um sorteado com `randomInt` (gerador criptográfico do `node:crypto`) entre os 31 símbolos `ABCDEFGHJKMNPQRSTUVWXYZ23456789`. É sorteado na inserção da organização, como valor padrão do campo.                                                             | Não é sequencial. Um novo valor substitui o anterior a cada uso do RF-ORG-10; se o valor sorteado já existe, a gravação falha no índice único e o serviço sorteia outro, até 5 vezes. |
| `USUARIO._id` e `COMISSAO._id`      | Driver do MongoDB, na criação da conta e da comissão          | `ObjectId`, com o mesmo formato de `ORGANIZACAO._id`. Criados nos módulos de usuário e de comissão.                                                                                                                                                                                      | Igual a `ORGANIZACAO._id`.                                                                                                                                                  |

## ORGANIZACAO

Representa uma instituição cadastrada, como um câmpus. Cada documento da coleção `organizacoes` guarda os dados da organização, a sua situação, o código de acesso e a lista de vínculos.

| Campo           | Tipo de dado                  | Pode ser nulo | Chave primária  | Chave estrangeira   | Outros                                                                       | Descrição                                                                                                 |
| --------------- | ----------------------------- | ------------- | --------------- | ------------------- | ---------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------- |
| `_id`           | `ObjectId`                    | Não           | Sim (exclusiva) | Não                 | Gerado na inserção.                                                          | Identifica a organização em todas as rotas e nas comissões.                                               |
| `nome`          | Texto (`string`, 2 a 100)     | Não           | Não             | Não                 | Único sem diferenciar maiúsculas; sem espaços nas pontas.                    | Nome mostrado em todas as telas e usado na busca.                                                         |
| `cidade`        | Texto (`string`, 2 a 60)      | Não           | Não             | Não                 | Sem espaços nas pontas.                                                      | Cidade da instituição, mostrada nos cartões.                                                              |
| `descricao`     | Texto (`string`, até 200)     | Sim           | Não             | Não                 | Ausente quando não informada no cadastro.                                    | Texto livre sobre a organização.                                                                          |
| `status`        | Texto com valores fixos       | Não           | Não             | Não                 | `PENDENTE`, `APROVADA` ou `REVOGADA`; começa em `PENDENTE`.                  | Situação da organização perante o administrador do sistema.                                               |
| `criada_por`    | `ObjectId`                    | Não           | Não             | Sim (`USUARIO._id`) | Vem do token de quem cadastrou, nunca do corpo da requisição.                | Usuário que cadastrou a organização.                                                                      |
| `codigo_acesso` | Texto (10 caracteres)         | Não           | Não             | Não                 | Único; oculto nas consultas comuns; só o administrador da organização o lê. | Código que outras pessoas digitam para pedir entrada.                                                     |
| `criado_em`     | Data e hora (`date`, UTC)     | Não           | Não             | Não                 | Gravado pelo Mongoose na inserção.                                           | Momento do cadastro; ordena as listas, da mais nova para a mais antiga.                                   |
| `atualizado_em` | Data e hora (`date`, UTC)     | Não           | Não             | Não                 | Atualizado pelo Mongoose a cada gravação feita por ele.                      | Momento da última alteração do documento, inclusive dos vínculos e do código.                             |

### Valores de `status`

| Valor      | Significado                                                                                                                       |
| ---------- | --------------------------------------------------------------------------------------------------------------------------------- |
| `PENDENTE` | Cadastrada por um usuário e aguardando a autorização do administrador do sistema (RF-ORG-01). Não aceita membros nem comissões.  |
| `APROVADA` | Autorizada (RF-ORG-03). Aceita pedidos por código, convites e comissões.                                                         |
| `REVOGADA` | Suspensa pelo administrador do sistema (RF-ORG-04). Os dados ficam guardados, mas nada funciona até nova autorização; só nesta situação a organização pode ser excluída (RF-ORG-06). |

## ORGANIZACAO_MEMBRO

Representa o vínculo de uma pessoa com uma organização, desde o pedido ou o convite. Fica em `organizacoes.membros[]`, um item por pessoa.

| Campo            | Tipo de dado              | Pode ser nulo | Chave primária  | Chave estrangeira       | Outros                                                                       | Descrição                                                                              |
| ---------------- | ------------------------- | ------------- | --------------- | ----------------------- | ---------------------------------------------------------------------------- | -------------------------------------------------------------------------------------- |
| `organizacao_id` | `ObjectId`                | Não           | Sim (inclusiva) | Sim (`ORGANIZACAO._id`) | Não é gravado: é o documento que contém a lista.                             | Organização do vínculo.                                                                |
| `usuario_id`     | `ObjectId`                | Não           | Sim (inclusiva) | Sim (`USUARIO._id`)     | Não se repete na mesma organização.                                          | Pessoa vinculada.                                                                      |
| `papel`          | Texto com valores fixos   | Não           | Não             | Não                     | `ADMIN` ou `MEMBRO`; todo vínculo novo começa `MEMBRO`, salvo o do criador.  | O que a pessoa pode fazer na organização.                                              |
| `status`         | Texto com valores fixos   | Não           | Não             | Não                     | `PENDENTE`, `APROVADO`, `REJEITADO` ou `CONVIDADO`.                          | Em que ponto está o vínculo.                                                           |
| `solicitado_em`  | Data e hora (`date`, UTC) | Não           | Não             | Não                     | Regravado ao reabrir um pedido ou ao convidar quem tinha sido recusado.      | Momento do pedido ou do convite.                                                       |
| `aprovado_em`    | Data e hora (`date`, UTC) | Sim           | Não             | Não                     | Existe só com `status = APROVADO`.                                           | Momento em que a pessoa passou a ser membro.                                           |

### Valores de `status` do vínculo

| Valor       | Significado                                                                                                     | Como chega a este valor                                                     |
| ----------- | --------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------- |
| `PENDENTE`  | Pedido de entrada aguardando um administrador.                                                                  | Entrada por código (RF-ORG-07), inclusive reabrindo um pedido recusado.     |
| `APROVADO`  | Membro da organização, com acesso às comissões; com `papel = ADMIN`, também administrador.                      | Cadastro (criador), aprovação do pedido (RF-ORG-09) ou aceite do convite (RF-ORG-11). |
| `REJEITADO` | Pedido recusado. Continua gravado para que o administrador possa aprová-lo depois, convidar a pessoa ou excluí-lo. | Recusa do pedido (RF-ORG-09).                                              |
| `CONVIDADO` | Convite aguardando a resposta da pessoa.                                                                        | Convite (RF-ORG-08).                                                        |

## USUARIO

Conta de usuário, da coleção `usuarios`. Aparece porque é o autor da organização e o dono de cada vínculo; os demais campos estão no módulo de cadastro de usuário.

| Campo   | Tipo de dado            | Pode ser nulo | Chave primária  | Chave estrangeira | Outros                             | Descrição                                                        |
| ------- | ----------------------- | ------------- | --------------- | ----------------- | ---------------------------------- | ---------------------------------------------------------------- |
| `_id`   | `ObjectId`              | Não           | Sim (exclusiva) | Não               | Criado no cadastro da conta.       | Identifica a conta.                                              |
| `nome`  | Texto                   | Não           | Não             | Não               | -                                  | Nome mostrado nas listas de membros e em "Criada por".           |
| `email` | Texto                   | Não           | Não             | Não               | Único, em minúsculas.              | Endereço usado para encontrar a pessoa convidada.                |
| `tipo`  | Texto com valores fixos | Não           | Não             | Não               | `USUARIO` ou `ADMIN_SISTEMA`.      | Define se a conta participa de organizações ou só as modera.     |
| `ativo` | Booleano                | Não           | Não             | Não               | -                                  | Conta desativada não acessa a API nem pode ser convidada.        |

## COMISSAO

Comissão de uma organização, da coleção `comissoes`. Aparece porque é apagada junto com a organização.

| Campo            | Tipo de dado | Pode ser nulo | Chave primária  | Chave estrangeira       | Outros                       | Descrição                                       |
| ---------------- | ------------ | ------------- | --------------- | ----------------------- | ---------------------------- | ----------------------------------------------- |
| `_id`            | `ObjectId`   | Não           | Sim (exclusiva) | Não                     | Criado no módulo de comissões. | Identifica a comissão.                        |
| `organizacao_id` | `ObjectId`   | Não           | Não             | Sim (`ORGANIZACAO._id`) | -                            | Organização dona da comissão.                   |
| `nome`           | Texto        | Não           | Não             | Não                     | -                            | Nome da comissão.                               |
| `ativo`          | Booleano     | Não           | Não             | Não                     | -                            | Indica se a comissão está ativa.                |

## COMISSAO_MEMBRO

Integrante da equipe de uma comissão, em `comissoes.membros[]`. Aparece porque a remoção de um membro da organização retira a pessoa de todas as comissões dela.

| Campo         | Tipo de dado            | Pode ser nulo | Chave primária  | Chave estrangeira    | Outros                                       | Descrição                                   |
| ------------- | ----------------------- | ------------- | --------------- | -------------------- | -------------------------------------------- | ------------------------------------------- |
| `comissao_id` | `ObjectId`              | Não           | Sim (inclusiva) | Sim (`COMISSAO._id`) | Não é gravado: é o documento que contém a lista. | Comissão da equipe.                      |
| `usuario_id`  | `ObjectId`              | Não           | Sim (inclusiva) | Sim (`USUARIO._id`)  | Não se repete na mesma comissão.             | Integrante.                                 |
| `papel`       | Texto com valores fixos | Não           | Não             | Não                  | `RESPONSAVEL` ou `MEMBRO`.                   | Papel na comissão.                          |

## Índices e restrições

- `organizacoes._id` é a chave primária da coleção e já é indexada pelo MongoDB.
- `codigo_acesso` tem índice único, criado pelo Mongoose a partir de `unique: true` no schema. É ele que faz a gravação falhar quando um código sorteado já existe; o serviço reconhece essa falha e sorteia outro.
- O nome único não tem índice: o serviço consulta, antes de gravar, se existe outra organização com o mesmo nome completo, sem diferenciar maiúsculas. Por não ser uma operação única, dois cadastros com o mesmo nome no mesmo instante podem passar os dois (limitação conhecida, não testada).
- A lista de organizações do usuário filtra por `membros.usuario_id`, que não tem índice; convém criar `{ "membros.usuario_id": 1 }` quando o volume crescer (ainda não criado).
- Um usuário não aparece duas vezes na mesma organização: o pedido e o convite só acrescentam o vínculo com o filtro "usuário ainda fora de `membros`".
- A organização não fica sem administrador: o rebaixamento só grava se, no mesmo instante, houver outro vínculo `ADMIN` e `APROVADO`, e a remoção recusa vínculos `ADMIN`.
- Toda alteração de vínculo é gravada com `updateOne` cujo filtro repete a situação lida; se outra gravação chegou antes, nada muda e a API responde 409.
- A exclusão só apaga a organização se, no momento da operação, ela estiver `REVOGADA`; depois apaga as comissões dela. As reuniões dessas comissões não são apagadas.
- A remoção de um membro tira a pessoa das equipes das comissões com uma atualização direta no banco, que não passa pela regra do schema da comissão "exatamente um responsável"; se a pessoa era a responsável, a comissão fica sem responsável.

## Rastreabilidade

RF-ORG-01 a RF-ORG-14; [Diagrama relacional](4%20-%20diagrama-relacional-cadastro-organizacao.md) e [Testes](7%20-%20testes-cadastro-organizacao.md).
