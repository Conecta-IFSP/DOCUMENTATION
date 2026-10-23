# Dicionário de Dados - Classes de Reuniões

---

## 1. Classe `Reuniao`

Representa um encontro de uma comissão agendado pelo seu responsável. Reúne a data, o horário, o local, a pauta e as pessoas convidadas, e guarda também a presença de cada participante. Os registros ficam na coleção `reunioes`.

| Campo | Tipo | Obrigatório | Descrição |
|---|---|---:|---|
| `_id` | `ObjectId` | Sim | Identificador único da reunião, gerado pelo MongoDB. |
| `comissao_id` | `ObjectId` | Sim | Comissão que realiza a reunião. |
| `titulo` | `String` | Sim | Nome curto da reunião, armazenado sem espaços nas extremidades. |
| `local` | `String` | Sim | Lugar onde a reunião acontece, como uma sala, um endereço ou um link. Armazenado sem espaços nas extremidades. |
| `inicio` | `Date` | Sim | Data e hora de início, armazenadas em UTC. |
| `fim` | `Date` | Sim | Data e hora de término, armazenadas em UTC. |
| `pauta` | `String` | Não | Assuntos previstos para a reunião. O valor padrão é um texto vazio. |
| `criada_por` | `ObjectId` | Sim | Usuário que agendou a reunião, chamado de organizador. |
| `situacao` | `SituacaoReuniao` | Sim | Indica se a reunião está agendada ou foi cancelada. O valor padrão é `AGENDADA`. |
| `participantes` | `Participante[]` | Sim | Pessoas convidadas, incluindo o organizador, com a resposta ao convite e a presença de cada uma. |
| `criado_em` | `Date` | Sim | Data e hora em que a reunião foi registrada, preenchida automaticamente. |
| `atualizado_em` | `Date` | Sim | Data e hora da última alteração da reunião, preenchida automaticamente. |

### Regras relacionadas à reunião

O título deve ter de 3 a 120 caracteres.

O local deve ter de 2 a 120 caracteres.

A pauta pode ter no máximo 2000 caracteres.

O término deve ser posterior ao início e, no momento do agendamento, posterior à data e hora atuais.

O organizador deve ser o responsável da comissão no momento do agendamento e consta sempre entre os participantes, com convite aceito.

Um mesmo usuário não pode aparecer mais de uma vez entre os participantes.

A situação só pode mudar de `AGENDADA` para `CANCELADA`, antes do término da reunião. Depois do agendamento, nenhum outro campo da reunião é alterado.

As situações "em andamento" e "realizada" não são armazenadas. Elas são definidas comparando `inicio` e `fim` com a data e hora atuais.

---

## 2. Classe `Participante`

Representa o convite de uma pessoa para uma reunião: quem foi convidado, qual foi a resposta e, quando a pessoa compareceu, o registro de presença. Existe apenas dentro de uma `Reuniao`.

| Campo | Tipo | Obrigatório | Descrição |
|---|---|---:|---|
| `usuario_id` | `ObjectId` | Sim | Usuário convidado. |
| `convite` | `RespostaConvite` | Sim | Resposta do usuário ao convite. O valor padrão é `PENDENTE`. |
| `presenca` | `Presenca` | Não | Registro de comparecimento. Não existe enquanto o usuário não registra entrada nem é marcado como presente. |

### Regras relacionadas ao participante

O usuário deve ser integrante da comissão da reunião.

O organizador é registrado com convite `ACEITO`, e os demais convidados com convite `PENDENTE`.

O convite só pode ser respondido enquanto a reunião estiver agendada e não tiver terminado.

O organizador não responde ao próprio convite.

Quem já registrou entrada não pode recusar o convite.

---

## 3. Classe `Presenca`

Representa o comparecimento de um participante à reunião: quando entrou, quando saiu e se a presença foi validada pelo organizador. Existe apenas dentro de um `Participante`.

| Campo | Tipo | Obrigatório | Descrição |
|---|---|---:|---|
| `entrada` | `Date` | Sim | Data e hora em que o participante registrou a entrada. Quando o organizador marca como presente quem não registrou entrada, recebe o início da reunião. |
| `saida` | `Date` | Não | Data e hora em que o participante registrou a saída. |
| `situacao` | `SituacaoPresenca` | Sim | Situação da validação da presença pelo organizador. |
| `marcada_por` | `OrigemPresenca` | Sim | Indica se a presença foi criada pelo próprio participante ou pelo organizador. |

### Regras relacionadas à presença

A entrada só pode ser registrada entre o início e o término da reunião, uma única vez por participante, e apenas por quem não recusou o convite.

A saída exige uma entrada que não tenha sido recusada e só pode ser registrada uma vez.

A saída registrada nunca passa do término da reunião.

A presença criada pelo próprio participante fica `AGUARDANDO`, exceto a do organizador, que fica `CONFIRMADA`.

---

## 4. Classes do Incremento 1 Utilizadas

### Classe `Comissao`

Representa um grupo de trabalho dentro de uma organização. Os registros ficam na coleção `comissoes`.

| Campo | Tipo | Obrigatório | Descrição |
|---|---|---:|---|
| `_id` | `ObjectId` | Sim | Identificador único da comissão. |
| `nome` | `String` | Sim | Nome da comissão, exibido nas reuniões. |
| `organizacao_id` | `ObjectId` | Sim | Organização à qual a comissão pertence. |
| `membros` | `MembroComissao[]` | Sim | Integrantes da comissão. Somente eles podem ser convidados para as reuniões. |
| `ativo` | `Boolean` | Sim | Indica se a comissão está ativa. Comissão inativa não agenda reuniões. |

### Classe `MembroComissao`

Representa o vínculo de um usuário com uma comissão. Existe apenas dentro de uma `Comissao`.

| Campo | Tipo | Obrigatório | Descrição |
|---|---|---:|---|
| `usuario_id` | `ObjectId` | Sim | Usuário integrante da comissão. |
| `papel` | `PapelComissao` | Sim | `RESPONSAVEL` para quem coordena a comissão, correspondente ao presidente do enunciado, ou `MEMBRO` para os demais integrantes. Cada comissão possui exatamente um `RESPONSAVEL`. |

### Classe `Organizacao`

Representa uma organização cadastrada na plataforma. Os registros ficam na coleção `organizacoes`.

| Campo | Tipo | Obrigatório | Descrição |
|---|---|---:|---|
| `_id` | `ObjectId` | Sim | Identificador único da organização. |
| `nome` | `String` | Sim | Nome da organização. |
| `status` | `StatusOrganizacao` | Sim | Situação da organização. Somente organizações `APROVADA` permitem agendar reuniões. |
| `membros` | `MembroOrganizacao[]` | Sim | Vínculos dos usuários com a organização. |

### Classe `MembroOrganizacao`

Representa o vínculo de um usuário com uma organização. Existe apenas dentro de uma `Organizacao`.

| Campo | Tipo | Obrigatório | Descrição |
|---|---|---:|---|
| `usuario_id` | `ObjectId` | Sim | Usuário vinculado. |
| `papel` | `PapelOrganizacao` | Sim | `ADMIN` ou `MEMBRO` da organização. |
| `status` | `StatusMembroOrganizacao` | Sim | Situação do vínculo. O responsável da comissão precisa ter o vínculo `APROVADO` para agendar reuniões. |

### Classe `Usuario`

Representa uma conta cadastrada na plataforma. Os registros ficam na coleção `usuarios`.

| Campo | Tipo | Obrigatório | Descrição |
|---|---|---:|---|
| `_id` | `ObjectId` | Sim | Identificador único do usuário. |
| `nome` | `String` | Sim | Nome exibido nas listas de participantes e na lista de presença. |
| `email` | `String` | Sim | E-mail exibido junto ao nome dos participantes. |
| `tipo` | `TipoUsuario` | Sim | `USUARIO` ou `ADMIN_SISTEMA`. O administrador do sistema não acessa as reuniões. |
| `ativo` | `Boolean` | Sim | Indica se a conta está ativa. Conta inativa não acessa as reuniões. |

---

## 5. Enum `SituacaoReuniao`

Define se uma reunião continua válida.

| Valor | Descrição |
|---|---|
| `AGENDADA` | Reunião válida. Aceita respostas de convite e registros de presença, conforme o horário. |
| `CANCELADA` | Reunião cancelada pelo organizador. Não aceita mais alterações. |

---

## 6. Enum `RespostaConvite`

Define a resposta de um participante ao convite.

| Valor | Descrição |
|---|---|
| `PENDENTE` | O participante ainda não respondeu. |
| `ACEITO` | O participante vai à reunião. |
| `RECUSADO` | O participante não vai à reunião. Pode aceitar depois, enquanto a reunião não terminar. |

---

## 7. Enum `SituacaoPresenca`

Define a situação da validação de uma presença.

| Valor | Descrição |
|---|---|
| `AGUARDANDO` | O participante registrou entrada e aguarda a confirmação do organizador. |
| `CONFIRMADA` | Presença validada pelo organizador, ou registrada pelo próprio organizador. |
| `RECUSADA` | O organizador não reconheceu a entrada registrada. |

---

## 8. Enum `OrigemPresenca`

Define quem criou o registro de presença.

| Valor | Descrição |
|---|---|
| `PROPRIO` | O próprio participante registrou a entrada. |
| `RESPONSAVEL` | O organizador marcou o participante como presente. |

---

## 9. Classes do Servidor

### Classe `ReunioesController`

Representa o ponto de entrada das requisições HTTP no caminho `/reunioes`. Recebe a requisição, obtém o identificador do usuário autenticado e repassa os dados ao `ReunioesService`.

| Método | Rota | Descrição |
|---|---|---|
| `listar` | `GET /reunioes` | Lista as reuniões do usuário, com os filtros `de`, `ate` e `comissao_id`. |
| `criar` | `POST /reunioes` | Agenda uma reunião. |
| `buscarPorId` | `GET /reunioes/:id` | Devolve uma reunião. |
| `atualizar` | `PATCH /reunioes/:id` | Cancela uma reunião. |
| `responderConvite` | `PATCH /reunioes/:id/convite` | Registra a resposta ao convite. |
| `marcarEntrada` | `POST /reunioes/:id/entrada` | Registra a entrada do usuário. |
| `marcarSaida` | `POST /reunioes/:id/saida` | Registra a saída do usuário. |
| `atualizarPresenca` | `PATCH /reunioes/:id/presencas/:usuarioId` | Confirma ou recusa a presença de um participante. |
| `gerarListaPresenca` | `GET /reunioes/:id/lista-presenca` | Devolve a lista de presença. |

### Classe `ReunioesService`

Representa as regras de negócio das reuniões. Consulta e grava as coleções `reunioes`, `comissoes`, `organizacoes` e `usuarios` por meio dos atributos `reuniaoModel`, `comissaoModel`, `organizacaoModel` e `usuarioModel`, que são instâncias de `Model` para cada classe de dados.

| Método | Descrição |
|---|---|
| `criar`, `listar`, `buscarPorId`, `atualizar`, `responderConvite`, `marcarEntrada`, `marcarSaida`, `atualizarPresenca`, `gerarListaPresenca` | Executam a operação de mesmo nome do controlador. |
| `validarId` | Recusa identificadores que não são `ObjectId` válidos. |
| `validarUsuario` | Verifica que o usuário existe e está ativo. |
| `lerData` | Converte o texto ISO 8601 recebido em data e recusa valores inválidos. |
| `validarHorario` | Verifica que o término é posterior ao início e à data e hora atuais. |
| `autorizarResponsavel` | Verifica que o usuário é o responsável de uma comissão ativa e membro aprovado de uma organização aprovada. |
| `carregar` | Lê a reunião e verifica que o usuário é participante. Devolve a reunião, o participante e se o usuário é o organizador. |
| `carregarComoDono` | Faz o mesmo que `carregar` e exige que o usuário seja o organizador. |
| `exigirAgendada` | Recusa reuniões canceladas. |
| `popular` | Completa a reunião com o nome da comissão e o nome e o e-mail dos participantes. |
| `apresentar` | Acrescenta à resposta o campo `sou_dono`, que indica se o usuário é o organizador. |
| `responder` | Recusa com conflito quando a gravação não encontrou a reunião esperada e, caso contrário, monta a resposta. |

### Classe `Model`

Representa o acesso do Mongoose a uma coleção do MongoDB. Cada instância trabalha com uma classe de dados, como `Reuniao` ou `Usuario`.

| Método | Descrição |
|---|---|
| `find` | Busca os documentos que atendem ao filtro. |
| `findById` | Busca um documento pelo `_id`. |
| `findOne` | Busca o primeiro documento que atende ao filtro. |
| `create` | Grava um novo documento. |
| `findOneAndUpdate` | Altera o documento que atende ao filtro e devolve o documento alterado, ou nada quando nenhum documento atende ao filtro. |

### Classes de entrada de dados

Representam os dados aceitos no corpo de cada requisição, com as regras de validação de cada campo.

| Classe | Campos | Usada em |
|---|---|---|
| `CreateReuniaoDto` | `comissao_id`, `titulo`, `local`, `inicio`, `fim`, `pauta`, `convidados_ids` | Agendar reunião. |
| `UpdateReuniaoDto` | `situacao`, que aceita apenas `CANCELADA` | Cancelar reunião. |
| `ResponderConviteReuniaoDto` | `aceito` | Responder convite. |
| `AtualizarPresencaDto` | `situacao`, que aceita `CONFIRMADA` ou `RECUSADA` | Confirmar ou recusar presença. |

### Classe `ApiKeyGuard`

Representa a verificação da chave de acesso da API. Compara o cabeçalho `X-API-Key` da requisição com a chave configurada no servidor e recusa a requisição quando ela está ausente ou é diferente. Atua em todas as rotas.

### Classe `JwtAuthGuard`

Representa a verificação do login. Lê o token do cabeçalho `Authorization`, confere a assinatura e a validade com o `JwtService` e verifica na coleção `usuarios` que a conta existe e está ativa.

### Classe `UsuarioComumGuard`

Representa a restrição de acesso do administrador do sistema. Recusa a requisição quando o usuário autenticado é do tipo `ADMIN_SISTEMA`.

### Classe `ValidationPipe`

Classe do NestJS que valida o corpo da requisição com a classe de entrada da rota e descarta os campos que ela não declara.

### Classe `JwtService`

Classe do NestJS que confere a assinatura e a validade do token de login.

---

## 10. Classes do Aplicativo

### Classe `TelaNovaReuniao`

Representa a tela de agendamento de reunião. O método `salvar` valida os campos, monta as datas de início e término e envia a reunião.

### Classe `TelaAgenda`

Representa a aba Agenda. O método `carregar` busca as reuniões do usuário e as comissões que ele integra, `escolherDia` muda o dia exibido e `responder` envia a resposta a um convite a partir da lista. A opção "Nova reunião" só aparece quando o usuário é responsável de ao menos uma comissão ativa.

### Classe `TelaComissao`

Representa a tela de uma comissão. O método `buscarReunioes` busca as reuniões da comissão para a aba Reuniões.

### Classe `TelaReuniao`

Representa a tela de uma reunião. O método `buscar` lê a reunião, `executar` envia as ações de convite, entrada, saída e cancelamento, e `marcarPresenca` envia a decisão do organizador sobre uma presença.

### Classe `TelaRegistrosReuniao`

Representa a tela de registros de uma reunião encerrada. O método `carregar` lê a reunião, `carregarLista` busca a lista de presença e `compartilhar` abre o compartilhamento da lista.

### Módulo `reunioes`

Reúne as funções de apoio das telas de reunião. A função `comissoesQueComando` devolve, de uma lista de comissões, as ativas em que o usuário tem papel `RESPONSAVEL`.

### Módulo `api`

Representa a comunicação do aplicativo com o servidor. O método `api` envia a requisição com a chave de acesso no cabeçalho `X-API-Key` e o token de login no cabeçalho `Authorization`, e devolve os dados ou a mensagem de erro. O método `obterToken` lê o token salvo no aparelho, no armazenamento seguro sob a chave `conecta_mais_token`.
