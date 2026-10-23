# Dicionário de Dados - Modelo Relacional de Reuniões

---

## Visão Geral

| Tabela | Descrição | Armazenamento no MongoDB |
|---|---|---|
| `reuniao` | Reuniões agendadas pelas comissões. | Documentos da coleção `reunioes`. |
| `participante_reuniao` | Convites de cada usuário para cada reunião, com a resposta e a presença. | Vetor `participantes` dentro de cada reunião. As colunas `presenca_*` correspondem ao subdocumento `participantes[].presenca`. |
| `comissao`, `membro_comissao` | Comissões e seus integrantes, do Incremento 1. | Coleção `comissoes` e vetor `membros`. |
| `organizacao`, `membro_organizacao` | Organizações e seus membros, do Incremento 1. | Coleção `organizacoes` e vetor `membros`. |
| `usuario` | Contas da plataforma, do Incremento 1. | Coleção `usuarios`. |

Nas restrições, o tipo **Banco** indica que a regra é garantida pelo esquema do banco, e o tipo **Sistema** indica que é garantida pelo servidor antes de gravar.

---

# 1. Tabela `reuniao`

Representa um encontro de uma comissão, agendado pelo responsável dela, com data, horário, local e pauta.

| Campo | Tipo | Chave | Nulo | Default | Descrição |
|---|---|---|---|---|---|
| `_id` | `ObjectId` | PK | Não | Automático | Identificador único da reunião. |
| `comissao_id` | `ObjectId` | FK | Não | - | Comissão que realiza a reunião. Referencia `comissao._id`. |
| `titulo` | `String(120)` | - | Não | - | Nome curto da reunião. |
| `local` | `String(120)` | - | Não | - | Lugar onde a reunião acontece. |
| `inicio` | `Date` | - | Não | - | Data e hora de início, em UTC. |
| `fim` | `Date` | - | Não | - | Data e hora de término, em UTC. |
| `pauta` | `String(2000)` | - | Não | `''` | Assuntos previstos para a reunião. |
| `criada_por` | `ObjectId` | FK | Não | - | Organizador da reunião. Referencia `usuario._id`. |
| `situacao` | `String` | - | Não | `AGENDADA` | Situação da reunião: `AGENDADA` ou `CANCELADA`. |
| `criado_em` | `Date` | - | Não | Automático | Data e hora em que a reunião foi registrada. |
| `atualizado_em` | `Date` | - | Não | Automático | Data e hora da última alteração. |

### Restrições

| Restrição | Tipo | Regra |
|---|---|---|
| `pk_reuniao` | Banco | `_id` identifica unicamente cada reunião. |
| `fk_reuniao_comissao` | Sistema | `comissao_id` deve existir em `comissao`, com `ativo` igual a `true` no momento do agendamento. |
| `fk_reuniao_organizador` | Sistema | `criada_por` deve existir em `usuario` e ser o `RESPONSAVEL` da comissão em `membro_comissao` no momento do agendamento. |
| `ck_reuniao_titulo` | Sistema | `titulo` tem de 3 a 120 caracteres, sem espaços nas extremidades. |
| `ck_reuniao_local` | Sistema | `local` tem de 2 a 120 caracteres, sem espaços nas extremidades. |
| `ck_reuniao_pauta` | Sistema | `pauta` tem no máximo 2000 caracteres. |
| `ck_reuniao_horario` | Sistema | `fim` é maior que `inicio` e, no agendamento, maior que a data e hora atuais. |
| `ck_reuniao_situacao` | Banco | `situacao` aceita apenas `AGENDADA` ou `CANCELADA`. |
| `ck_reuniao_cancelamento` | Sistema | A única alteração permitida depois do agendamento é `situacao` de `AGENDADA` para `CANCELADA`, feita pelo organizador enquanto `fim` for maior que a data e hora atuais. |
| `ix_reuniao_comissao_inicio` | Banco | Índice em (`comissao_id`, `inicio`), para listar as reuniões de uma comissão em ordem de data. |
| `ix_reuniao_participante_inicio` | Banco | Índice em (`participante_reuniao.usuario_id`, `inicio`), para montar a agenda de cada usuário. |

---

# 2. Tabela `participante_reuniao`

Representa o convite de um usuário para uma reunião, com a resposta ao convite e o registro de presença.

| Campo | Tipo | Chave | Nulo | Default | Descrição |
|---|---|---|---|---|---|
| `reuniao_id` | `ObjectId` | PK, FK | Não | - | Reunião do convite. Referencia `reuniao._id`. |
| `usuario_id` | `ObjectId` | PK, FK | Não | - | Usuário convidado. Referencia `usuario._id`. |
| `convite` | `String` | - | Não | `PENDENTE` | Resposta ao convite: `PENDENTE`, `ACEITO` ou `RECUSADO`. |
| `presenca_entrada` | `Date` | - | Sim | `NULL` | Data e hora da entrada. Fica nulo enquanto não há presença. |
| `presenca_saida` | `Date` | - | Sim | `NULL` | Data e hora da saída. |
| `presenca_situacao` | `String` | - | Sim | `NULL` | Validação da presença: `AGUARDANDO`, `CONFIRMADA` ou `RECUSADA`. |
| `presenca_marcada_por` | `String` | - | Sim | `NULL` | Quem criou a presença: `PROPRIO` ou `RESPONSAVEL`. |

### Restrições

| Restrição | Tipo | Regra |
|---|---|---|
| `pk_participante_reuniao` | Sistema | O par (`reuniao_id`, `usuario_id`) é único: o mesmo usuário não aparece duas vezes na mesma reunião. |
| `fk_participante_membro` | Sistema | `usuario_id` deve constar em `membro_comissao` da comissão da reunião. |
| `ck_participante_organizador` | Sistema | O organizador (`reuniao.criada_por`) sempre tem uma linha, com `convite` igual a `ACEITO`, que ele não pode alterar. |
| `ck_participante_recusa` | Sistema | `convite` só passa para `RECUSADO` enquanto `presenca_entrada` for nulo. |
| `ck_participante_convite` | Banco | `convite` aceita apenas `PENDENTE`, `ACEITO` ou `RECUSADO`. |
| `ck_participante_presenca` | Banco | `presenca_entrada`, `presenca_situacao` e `presenca_marcada_por` são todos nulos ou todos preenchidos. |
| `ck_participante_entrada` | Sistema | `presenca_entrada` é gravada uma única vez, com a data e hora atuais entre `reuniao.inicio` e `reuniao.fim` e apenas quando `convite` é diferente de `RECUSADO`, ou com `reuniao.inicio` quando o organizador marca o participante como presente. |
| `ck_participante_saida` | Sistema | `presenca_saida` exige `presenca_entrada` preenchida e `presenca_situacao` diferente de `RECUSADA`, é gravada uma única vez e nunca passa de `reuniao.fim`. |
| `ck_participante_situacao` | Banco | `presenca_situacao` aceita apenas `AGUARDANDO`, `CONFIRMADA` ou `RECUSADA`, e `presenca_marcada_por` apenas `PROPRIO` ou `RESPONSAVEL`. |

---

# 3.. Tabelas do Incremento 1 Utilizadas

## Tabela `comissao`

Representa um grupo de trabalho dentro de uma organização.

| Campo | Tipo | Chave | Nulo | Default | Descrição |
|---|---|---|---|---|---|
| `_id` | `ObjectId` | PK | Não | Automático | Identificador único da comissão. |
| `nome` | `String` | - | Não | - | Nome da comissão. |
| `organizacao_id` | `ObjectId` | FK | Não | - | Organização da comissão. Referencia `organizacao._id`. |
| `ativo` | `Boolean` | - | Não | `true` | Indica se a comissão está ativa. Somente comissões ativas agendam reuniões. |

## Tabela `membro_comissao`

Representa o vínculo de um usuário com uma comissão.

| Campo | Tipo | Chave | Nulo | Default | Descrição |
|---|---|---|---|---|---|
| `comissao_id` | `ObjectId` | PK, FK | Não | - | Comissão. Referencia `comissao._id`. |
| `usuario_id` | `ObjectId` | PK, FK | Não | - | Integrante. Referencia `usuario._id`. |
| `papel` | `String` | - | Não | - | `RESPONSAVEL` ou `MEMBRO`. Cada comissão possui exatamente um `RESPONSAVEL`. |

## Tabela `organizacao`

Representa uma organização cadastrada na plataforma.

| Campo | Tipo | Chave | Nulo | Default | Descrição |
|---|---|---|---|---|---|
| `_id` | `ObjectId` | PK | Não | Automático | Identificador único da organização. |
| `nome` | `String` | - | Não | - | Nome da organização. |
| `status` | `String` | - | Não | `PENDENTE` | `PENDENTE`, `APROVADA` ou `REVOGADA`. Somente organizações `APROVADA` permitem agendar reuniões. |

## Tabela `membro_organizacao`

Representa o vínculo de um usuário com uma organização.

| Campo | Tipo | Chave | Nulo | Default | Descrição |
|---|---|---|---|---|---|
| `organizacao_id` | `ObjectId` | PK, FK | Não | - | Organização. Referencia `organizacao._id`. |
| `usuario_id` | `ObjectId` | PK, FK | Não | - | Usuário vinculado. Referencia `usuario._id`. |
| `papel` | `String` | - | Não | - | `ADMIN` ou `MEMBRO`. |
| `status` | `String` | - | Não | `PENDENTE` | `PENDENTE`, `APROVADO`, `REJEITADO` ou `CONVIDADO`. O responsável da comissão precisa de vínculo `APROVADO` para agendar reuniões. |

## Tabela `usuario`

Representa uma conta cadastrada na plataforma.

| Campo | Tipo | Chave | Nulo | Default | Descrição |
|---|---|---|---|---|---|
| `_id` | `ObjectId` | PK | Não | Automático | Identificador único do usuário. |
| `nome` | `String` | - | Não | - | Nome exibido nas reuniões e na lista de presença. |
| `email` | `String` | UK | Não | - | E-mail do usuário, único na plataforma. |
| `tipo` | `String` | - | Não | `USUARIO` | `USUARIO` ou `ADMIN_SISTEMA`. O administrador do sistema não acessa as reuniões. |
| `ativo` | `Boolean` | - | Não | `true` | Indica se a conta está ativa. Contas inativas não acessam as reuniões. |
