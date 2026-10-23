# Especificação de Requisitos Funcionais - Reuniões

---

## RF-01 - Agendar reunião

| Campo | Descrição |
|---|---|
| **Nome do Caso de Uso** | Agendar reunião |
| **ID do Requisito (RF)** | RF-01 |
| **Ator Principal** | Responsável da comissão |
| **Objetivo** | Permitir que o responsável de uma comissão agende uma reunião e convide os integrantes da comissão a participar. |
| **Prioridade** | Alta |
| **Pré-Condições** | O responsável deve estar autenticado e com a conta ativa.<br>O responsável deve ser o integrante com papel de responsável de uma comissão ativa.<br>O responsável deve ser membro aprovado da organização da comissão, e a organização deve estar aprovada. |
| **Pós-Condições** | A reunião é registrada na coleção `reunioes` com situação `AGENDADA`.<br>O responsável fica registrado como organizador da reunião e como participante com convite aceito.<br>Cada convidado fica registrado como participante com convite pendente e passa a ver a reunião na sua agenda. |
| **Fluxo Principal (Caminho de Sucesso)** | 1. O responsável seleciona a opção "Nova reunião" na tela da comissão ou na agenda. A opção só aparece para quem é responsável de ao menos uma comissão ativa.<br>2. O sistema exibe os campos Título, Data, Início, Término, Local, Convidados e Pauta, com todos os integrantes da comissão selecionados como convidados.<br>3. O responsável preenche o título, a data, os horários de início e término, o local e, se desejar, a pauta.<br>4. O responsável seleciona a opção "Agendar reunião".<br>5. O aplicativo envia a requisição `POST /reunioes` com os campos `comissao_id`, `titulo`, `local`, `inicio`, `fim`, `pauta` e `convidados_ids`. As datas são enviadas no formato ISO 8601.<br>6. O sistema consulta a comissão na coleção `comissoes` e verifica que o usuário consta em `membros` com `papel` igual a `RESPONSAVEL` e que a comissão tem `ativo` igual a `true`.<br>7. O sistema consulta a organização da comissão na coleção `organizacoes` e verifica que ela tem `status` igual a `APROVADA` e que o usuário consta em `membros` com `status` igual a `APROVADO`.<br>8. O sistema verifica que o término é posterior ao início e à data e hora atuais.<br>9. O sistema verifica que todos os convidados constam em `membros` da comissão.<br>10. O sistema registra a reunião na coleção `reunioes` com os campos `comissao_id`, `titulo`, `local`, `inicio`, `fim`, `pauta`, `criada_por` (o usuário autenticado), `situacao` igual a `AGENDADA` e `participantes`. O responsável é incluído em `participantes` com `convite` igual a `ACEITO`, e cada convidado com `convite` igual a `PENDENTE`.<br>11. O sistema devolve a reunião criada, e o aplicativo apresenta a tela da reunião. |
| **Fluxos Alternativos** | **FA-01: Convidar apenas parte da comissão**<br>1. No passo 3, o responsável seleciona o campo Convidados.<br>2. O sistema exibe os integrantes da comissão.<br>3. O responsável desmarca os integrantes que não serão convidados.<br>4. O fluxo continua no passo 4, e apenas os integrantes marcados são registrados como participantes.<br><br>**FA-02: Escolher horário pelas sugestões**<br>1. No passo 3, o responsável abre as sugestões do campo Início ou Término.<br>2. O sistema exibe horários em intervalos de 30 minutos. Para o término, exibe opções de 30 a 180 minutos depois do início.<br>3. O responsável escolhe um horário, e o campo é preenchido.<br><br>**FA-03: Responsável por mais de uma comissão**<br>1. No passo 2, o sistema exibe também o campo Comissão, com as comissões ativas em que o usuário é responsável.<br>2. O responsável escolhe a comissão, e a lista de convidados passa a ser a dessa comissão. |
| **Fluxos de Exceção** | **FE-01: Campos inválidos**<br>1. O título tem menos de 3 ou mais de 120 caracteres, o local tem menos de 2 ou mais de 120 caracteres, a pauta tem mais de 2000 caracteres ou um horário não está no formato HH:MM.<br>2. O sistema indica o erro abaixo do campo, por exemplo "Informe um título com pelo menos 3 caracteres.".<br>3. A reunião não é registrada.<br><br>**FE-02: Horário inválido**<br>1. O término é igual ou anterior ao início, ou já passou.<br>2. O sistema informa "O término deve ser depois do início." ou "Este horário já passou.".<br>3. A reunião não é registrada.<br><br>**FE-03: Convidado fora da comissão**<br>1. Um dos convidados não consta em `membros` da comissão.<br>2. O sistema informa "Convide apenas integrantes desta comissão".<br>3. A reunião não é registrada.<br><br>**FE-04: Comissão inativa**<br>1. A comissão tem `ativo` igual a `false`.<br>2. O sistema informa "Reative a comissão antes de agendar reuniões".<br>3. A reunião não é registrada.<br><br>**FE-05: Usuário sem permissão**<br>1. O usuário não é o responsável da comissão, ou a organização ou o seu vínculo com ela não está aprovado.<br>2. O sistema informa "Apenas o responsável da comissão pode agendar reuniões" ou "Você não tem acesso a esta organização".<br>3. A reunião não é registrada. |

---

## RF-02 - Consultar agenda de reuniões

| Campo | Descrição |
|---|---|
| **Nome do Caso de Uso** | Consultar agenda de reuniões |
| **ID do Requisito (RF)** | RF-02 |
| **Ator Principal** | Participante |
| **Objetivo** | Permitir que o participante consulte as reuniões para as quais foi convidado, organizadas por dia. |
| **Prioridade** | Alta |
| **Pré-Condições** | O participante deve estar autenticado e com a conta ativa. |
| **Pós-Condições** | As reuniões do participante são apresentadas no calendário e na lista do dia selecionado. Nenhum dado é alterado. |
| **Fluxo Principal (Caminho de Sucesso)** | 1. O participante acessa a aba Agenda.<br>2. O aplicativo envia a requisição `GET /reunioes`.<br>3. O sistema consulta a coleção `reunioes` e seleciona as reuniões em que o usuário consta em `participantes.usuario_id`.<br>4. O sistema completa cada reunião com o `nome` e a `organizacao_id` da comissão, lidos da coleção `comissoes`, e com o `nome` e o `email` de cada participante, lidos da coleção `usuarios`.<br>5. O sistema ordena as reuniões por `inicio`, do mais antigo para o mais recente, e devolve a lista. Cada reunião contém `_id`, `comissao_id`, `titulo`, `local`, `inicio`, `fim`, `pauta`, `criada_por`, `situacao`, `participantes` e `sou_dono`, que indica se o usuário é o organizador.<br>6. O sistema marca no calendário os dias que possuem reunião e apresenta as reuniões do dia selecionado, com horário, título, comissão e situação. |
| **Fluxos Alternativos** | **FA-01: Trocar o dia**<br>1. O participante seleciona outro dia no calendário.<br>2. O sistema apresenta as reuniões desse dia.<br><br>**FA-02: Ver o mês inteiro**<br>1. O participante expande o calendário.<br>2. O sistema passa a exibir o mês em vez da semana.<br><br>**FA-03: Reuniões de uma comissão**<br>1. Na tela de uma comissão, o participante acessa a aba Reuniões.<br>2. O aplicativo envia a requisição `GET /reunioes?comissao_id=<id da comissão>`.<br>3. O sistema aplica também o filtro `comissao_id`.<br>4. O sistema separa as reuniões em Próximas, com situação `AGENDADA` e término futuro, e Anteriores.<br><br>**FA-04: Filtrar por período**<br>1. A requisição informa os parâmetros `de` e `ate`, em ISO 8601.<br>2. O sistema seleciona apenas as reuniões com `fim` maior ou igual a `de` e `inicio` menor ou igual a `ate`. |
| **Fluxos de Exceção** | **FE-01: Nenhuma reunião no dia**<br>1. O dia selecionado não possui reuniões.<br>2. O sistema informa "Nenhuma reunião".<br><br>**FE-02: Filtro inválido**<br>1. O parâmetro `de` ou `ate` não é uma data, ou `comissao_id` não é um identificador válido.<br>2. O sistema informa "Período inválido" ou "ID inválido".<br><br>**FE-03: Falha de conexão**<br>1. O aplicativo não consegue concluir a requisição.<br>2. O sistema apresenta a mensagem de erro e permite tentar novamente puxando a tela para baixo. |

---

## RF-03 - Visualizar reunião

| Campo | Descrição |
|---|---|
| **Nome do Caso de Uso** | Visualizar reunião |
| **ID do Requisito (RF)** | RF-03 |
| **Ator Principal** | Participante |
| **Objetivo** | Permitir que o participante consulte os dados da reunião, a situação do seu convite e da sua presença, os convidados e a pauta. |
| **Prioridade** | Alta |
| **Pré-Condições** | O participante deve estar autenticado e com a conta ativa.<br>O participante deve constar entre os participantes da reunião. |
| **Pós-Condições** | Os dados da reunião são apresentados ao participante. Nenhum dado é alterado. |
| **Fluxo Principal (Caminho de Sucesso)** | 1. O participante seleciona uma reunião na agenda, no início ou na tela da comissão.<br>2. O aplicativo envia a requisição `GET /reunioes/<id>`.<br>3. O sistema consulta a reunião na coleção `reunioes` pelo `_id` e verifica que o usuário consta em `participantes.usuario_id`.<br>4. O sistema completa a reunião com o `nome` da comissão e o `nome` e o `email` de cada participante e a devolve com o campo `sou_dono`.<br>5. O sistema apresenta a comissão, o título, a situação, a data, o horário e o local, e as abas Presença e Pauta.<br>6. O sistema calcula a situação exibida a partir de `situacao`, `inicio`, `fim` e do convite do usuário: Cancelada, Recusada, Convite pendente, Agendada, Em andamento ou Realizada.<br>7. Enquanto a tela está aberta, o aplicativo repete a requisição a cada 30 segundos e atualiza os dados. |
| **Fluxos Alternativos** | **FA-01: Reunião encerrada**<br>1. O término já passou e a reunião não foi cancelada.<br>2. O sistema exibe também o cartão que dá acesso à lista de presença.<br><br>**FA-02: Atualizar manualmente**<br>1. O participante puxa a tela para baixo.<br>2. O aplicativo repete a requisição e atualiza os dados. |
| **Fluxos de Exceção** | **FE-01: Usuário não participa**<br>1. O usuário não consta em `participantes`.<br>2. O sistema informa "Você não participa desta reunião".<br><br>**FE-02: Reunião inexistente**<br>1. Não existe reunião com o identificador informado.<br>2. O sistema informa "Reunião não encontrada". |

---

## RF-04 - Cancelar reunião

| Campo | Descrição |
|---|---|
| **Nome do Caso de Uso** | Cancelar reunião |
| **ID do Requisito (RF)** | RF-04 |
| **Ator Principal** | Organizador |
| **Objetivo** | Permitir que o organizador cancele uma reunião que ainda não terminou. |
| **Prioridade** | Média |
| **Pré-Condições** | O organizador deve estar autenticado e com a conta ativa.<br>O usuário deve ser o organizador da reunião.<br>A reunião deve estar com situação `AGENDADA` e ainda não ter terminado. |
| **Pós-Condições** | A reunião passa para a situação `CANCELADA` e deixa de aceitar respostas de convite e registros de presença. |
| **Fluxo Principal (Caminho de Sucesso)** | 1. Na tela da reunião, o organizador seleciona "Mais opções" e depois "Cancelar reunião".<br>2. O sistema pede a confirmação do cancelamento, informando o título da reunião.<br>3. O organizador confirma.<br>4. O aplicativo envia a requisição `PATCH /reunioes/<id>` com o campo `situacao` igual a `CANCELADA`.<br>5. O sistema consulta a reunião na coleção `reunioes` e verifica que `criada_por` é o usuário, que `situacao` é `AGENDADA` e que `fim` é posterior à data e hora atuais.<br>6. O sistema altera o campo `situacao` da reunião para `CANCELADA`, com a condição de que ela ainda esteja `AGENDADA`, pertença ao organizador e não tenha terminado. A requisição não aceita a alteração de nenhum outro campo da reunião.<br>7. O sistema devolve a reunião atualizada, e o aplicativo informa "Reunião cancelada.". |
| **Fluxos Alternativos** | **FA-01: Desistir do cancelamento**<br>1. No passo 3, o organizador não confirma.<br>2. Nenhum dado é alterado. |
| **Fluxos de Exceção** | **FE-01: Usuário não é o organizador**<br>1. `criada_por` não é o usuário.<br>2. O sistema informa "Apenas quem agendou a reunião pode fazer isso".<br><br>**FE-02: Reunião já cancelada**<br>1. A reunião já tem situação `CANCELADA`.<br>2. O sistema informa "Esta reunião foi cancelada".<br><br>**FE-03: Reunião encerrada**<br>1. O término da reunião já passou.<br>2. O sistema informa "Esta reunião já terminou".<br><br>**FE-04: Valor inválido**<br>1. O campo `situacao` não é `CANCELADA`.<br>2. O sistema informa "A reunião só pode ser cancelada".<br><br>**FE-05: Alteração simultânea**<br>1. A reunião foi alterada por outra requisição entre a consulta e a gravação.<br>2. O sistema informa "A reunião mudou. Atualize a tela." e não grava. |

---

## RF-05 - Responder convite

| Campo | Descrição |
|---|---|
| **Nome do Caso de Uso** | Responder convite |
| **ID do Requisito (RF)** | RF-05 |
| **Ator Principal** | Participante |
| **Objetivo** | Permitir que o participante aceite ou recuse o convite para uma reunião. |
| **Prioridade** | Alta |
| **Pré-Condições** | O participante deve estar autenticado e com a conta ativa.<br>O participante deve constar entre os participantes da reunião e não ser o organizador.<br>A reunião deve estar com situação `AGENDADA` e ainda não ter terminado.<br>Para recusar, o participante não pode ter registrado entrada. |
| **Pós-Condições** | O convite do participante fica registrado como `ACEITO` ou `RECUSADO`. |
| **Fluxo Principal (Caminho de Sucesso)** | 1. O sistema apresenta o cartão Convite com a mensagem "Aguardando sua resposta" na tela da reunião, ou o item da reunião com as opções de resposta na agenda.<br>2. O participante seleciona "Aceitar" ou "Recusar".<br>3. O aplicativo envia a requisição `PATCH /reunioes/<id>/convite` com o campo `aceito` igual a `true` ou `false`.<br>4. O sistema consulta a reunião na coleção `reunioes` e verifica que o usuário é participante, que não é o organizador (`criada_por`), que a situação é `AGENDADA` e que `fim` é posterior à data e hora atuais. Na recusa, verifica também que o participante não possui `presenca`.<br>5. O sistema altera o campo `convite` do participante para `ACEITO`, se `aceito` for `true`, ou para `RECUSADO`, se for `false`. A recusa só é gravada se o participante continuar sem `presenca`.<br>6. O sistema devolve a reunião atualizada, e o aplicativo atualiza o cartão do convite. |
| **Fluxos Alternativos** | **FA-01: Aceitar depois de recusar**<br>1. O participante recusou o convite, e a reunião ainda não terminou.<br>2. O sistema apresenta a mensagem "Você recusou o convite" e a opção "Aceitar convite".<br>3. O participante seleciona "Aceitar convite", e o fluxo continua no passo 3 com `aceito` igual a `true`.<br><br>**FA-02: Responder pelo início**<br>1. Na aba Início, o sistema apresenta os convites pendentes com as opções "Aceitar" e "Recusar".<br>2. O participante seleciona uma das opções, e o fluxo continua no passo 3. |
| **Fluxos de Exceção** | **FE-01: Resposta ausente**<br>1. O campo `aceito` não é informado ou não é verdadeiro ou falso.<br>2. O sistema informa "Informe se o convite foi aceito".<br><br>**FE-02: Reunião cancelada ou encerrada**<br>1. A reunião tem situação `CANCELADA` ou o término já passou.<br>2. O sistema informa "Esta reunião foi cancelada" ou "Esta reunião já terminou".<br><br>**FE-03: Usuário não participa**<br>1. O usuário não consta em `participantes`.<br>2. O sistema informa "Você não participa desta reunião".<br><br>**FE-04: Usuário é o organizador**<br>1. `criada_por` é o usuário.<br>2. O sistema informa "Quem agendou a reunião não responde ao convite".<br><br>**FE-05: Recusa depois da entrada**<br>1. O participante tenta recusar o convite depois de registrar entrada.<br>2. O sistema informa "Você já marcou entrada nesta reunião". |

---

## RF-06 - Registrar entrada

| Campo | Descrição |
|---|---|
| **Nome do Caso de Uso** | Registrar entrada |
| **ID do Requisito (RF)** | RF-06 |
| **Ator Principal** | Participante |
| **Objetivo** | Permitir que o participante informe que chegou à reunião. |
| **Prioridade** | Alta |
| **Pré-Condições** | O participante deve estar autenticado e com a conta ativa.<br>O participante deve constar entre os participantes da reunião e não ter recusado o convite.<br>A reunião deve estar com situação `AGENDADA`, e a data e hora atuais devem estar entre o início e o término.<br>O participante ainda não deve ter registrado entrada. |
| **Pós-Condições** | A presença do participante é registrada com a hora de entrada e a origem `PROPRIO`.<br>A presença fica `CONFIRMADA` quando o participante é o organizador e `AGUARDANDO` nos demais casos. |
| **Fluxo Principal (Caminho de Sucesso)** | 1. Durante a reunião, o sistema apresenta o cartão "Sua presença" com a opção "Marcar entrada".<br>2. O participante seleciona "Marcar entrada".<br>3. O aplicativo envia a requisição `POST /reunioes/<id>/entrada`.<br>4. O sistema consulta a reunião na coleção `reunioes` e verifica que o usuário é participante, que a situação é `AGENDADA`, que a data e hora atuais estão entre `inicio` e `fim`, que o `convite` do participante não é `RECUSADO` e que o participante não possui `presenca`.<br>5. O sistema registra no participante o campo `presenca` com `entrada` igual à data e hora atuais, `marcada_por` igual a `PROPRIO` e `situacao` igual a `CONFIRMADA`, se o usuário for o organizador, ou `AGUARDANDO`, nos demais casos. A gravação só ocorre se o participante continuar sem `presenca` e com o convite não recusado.<br>6. O sistema devolve a reunião atualizada, e o aplicativo informa "Entrada marcada." e apresenta a situação da presença. |
| **Fluxos Alternativos** | **FA-01: Marcar entrada pelo início**<br>1. Durante a reunião, a aba Início apresenta o cartão da reunião em andamento com a opção "Marcar entrada".<br>2. O participante seleciona a opção, e o fluxo continua no passo 3. |
| **Fluxos de Exceção** | **FE-01: Fora do horário da reunião**<br>1. A data e hora atuais são anteriores ao início ou posteriores ao término.<br>2. O sistema informa "A reunião ainda não começou" ou "Esta reunião já terminou".<br>3. A entrada não é registrada.<br><br>**FE-02: Entrada já registrada**<br>1. O participante já possui `presenca`.<br>2. O sistema informa "Sua entrada já foi registrada".<br>3. Nenhuma nova entrada é registrada.<br><br>**FE-03: Reunião cancelada**<br>1. A reunião tem situação `CANCELADA`.<br>2. O sistema informa "Esta reunião foi cancelada".<br><br>**FE-04: Convite recusado**<br>1. O `convite` do participante é `RECUSADO`.<br>2. O sistema informa "Aceite o convite antes de marcar a entrada".<br>3. A entrada não é registrada. |

---

## RF-07 - Registrar saída

| Campo | Descrição |
|---|---|
| **Nome do Caso de Uso** | Registrar saída |
| **ID do Requisito (RF)** | RF-07 |
| **Ator Principal** | Participante |
| **Objetivo** | Permitir que o participante informe que saiu da reunião. |
| **Prioridade** | Alta |
| **Pré-Condições** | O participante deve estar autenticado e com a conta ativa.<br>O participante deve ter registrado entrada, e a presença não pode ter sido recusada.<br>O participante ainda não deve ter registrado saída.<br>A reunião deve estar com situação `AGENDADA`. |
| **Pós-Condições** | A hora de saída do participante é registrada. Quando a saída é informada depois do término, a hora registrada é a do término da reunião. |
| **Fluxo Principal (Caminho de Sucesso)** | 1. O participante seleciona "Marcar saída" no cartão "Sua presença".<br>2. O aplicativo envia a requisição `POST /reunioes/<id>/saida`.<br>3. O sistema consulta a reunião na coleção `reunioes` e verifica que o participante possui `presenca`, com `situacao` diferente de `RECUSADA` e sem `saida`.<br>4. O sistema registra em `presenca.saida` a data e hora atuais ou o valor de `fim` da reunião, o que for menor. A gravação só ocorre se a presença continuar sem `saida` e não estiver `RECUSADA`.<br>5. O sistema devolve a reunião atualizada, e o aplicativo informa "Saída marcada." e apresenta o período de permanência. |
| **Fluxos Alternativos** | Não se aplica. |
| **Fluxos de Exceção** | **FE-01: Entrada não registrada**<br>1. O participante não possui `presenca`, ou a presença foi recusada.<br>2. O sistema informa "Marque a entrada antes da saída".<br><br>**FE-02: Saída já registrada**<br>1. A presença já possui `saida`.<br>2. O sistema informa "Sua saída já foi registrada". |

---

## RF-08 - Confirmar ou recusar presença

| Campo | Descrição |
|---|---|
| **Nome do Caso de Uso** | Confirmar ou recusar presença |
| **ID do Requisito (RF)** | RF-08 |
| **Ator Principal** | Organizador |
| **Objetivo** | Permitir que o organizador valide a presença registrada pelos participantes e marque como presente quem não registrou entrada. |
| **Prioridade** | Alta |
| **Pré-Condições** | O organizador deve estar autenticado e com a conta ativa.<br>O usuário deve ser o organizador da reunião.<br>A reunião deve estar com situação `AGENDADA` e já ter começado. |
| **Pós-Condições** | A presença do participante fica `CONFIRMADA` ou `RECUSADA`.<br>Quando um participante sem entrada é marcado como presente, a presença é criada com entrada igual ao início da reunião e origem `RESPONSAVEL`. |
| **Fluxo Principal (Caminho de Sucesso)** | 1. Na aba Presença da tela da reunião, o sistema apresenta o grupo "Aguardando confirmação" com os participantes cuja presença está `AGUARDANDO`.<br>2. O organizador seleciona a opção de confirmar ou de recusar ao lado de um participante.<br>3. O aplicativo envia a requisição `PATCH /reunioes/<id>/presencas/<id do participante>` com o campo `situacao` igual a `CONFIRMADA` ou `RECUSADA`.<br>4. O sistema consulta a reunião na coleção `reunioes` e verifica que `criada_por` é o usuário, que a situação é `AGENDADA`, que `inicio` já passou e que o participante informado consta em `participantes`.<br>5. O sistema altera o campo `presenca.situacao` do participante para o valor informado. A gravação só ocorre se o participante continuar com `presenca`.<br>6. O sistema devolve a reunião atualizada, e o aplicativo move o participante para o grupo Presentes ou Ausentes. |
| **Fluxos Alternativos** | **FA-01: Marcar presente quem não registrou entrada**<br>1. No grupo Ausentes, o organizador seleciona "Marcar presente" ao lado de um participante sem `presenca`.<br>2. O aplicativo envia a requisição do passo 3 com `situacao` igual a `CONFIRMADA`.<br>3. O sistema registra no participante o campo `presenca` com `entrada` igual a `inicio` da reunião, `situacao` igual a `CONFIRMADA` e `marcada_por` igual a `RESPONSAVEL`. A gravação só ocorre se o participante continuar sem `presenca`.<br>4. O fluxo continua no passo 6. |
| **Fluxos de Exceção** | **FE-01: Reunião não começou**<br>1. A data e hora atuais são anteriores a `inicio`.<br>2. O sistema informa "A reunião ainda não começou".<br><br>**FE-02: Recusa sem entrada**<br>1. O organizador tenta recusar a presença de um participante sem `presenca`.<br>2. O sistema informa "Esta pessoa não marcou entrada".<br><br>**FE-03: Pessoa fora da reunião**<br>1. O participante informado não consta em `participantes`.<br>2. O sistema informa "Esta pessoa não participa da reunião".<br><br>**FE-04: Valor inválido**<br>1. O campo `situacao` não é `CONFIRMADA` nem `RECUSADA`.<br>2. O sistema informa "A presença deve ser CONFIRMADA ou RECUSADA".<br><br>**FE-05: Usuário não é o organizador**<br>1. `criada_por` não é o usuário.<br>2. O sistema informa "Apenas quem agendou a reunião pode fazer isso".<br><br>**FE-06: Alteração simultânea**<br>1. A presença foi alterada por outra requisição entre a consulta e a gravação.<br>2. O sistema informa "A presença mudou. Atualize a tela." e não grava. |

---

## RF-09 - Gerar lista de presença

| Campo | Descrição |
|---|---|
| **Nome do Caso de Uso** | Gerar lista de presença |
| **ID do Requisito (RF)** | RF-09 |
| **Ator Principal** | Participante |
| **Objetivo** | Permitir que o participante obtenha a lista de presença da reunião, com a situação, a entrada, a saída e o tempo de permanência de cada convidado. |
| **Prioridade** | Alta |
| **Pré-Condições** | O participante deve estar autenticado e com a conta ativa.<br>O participante deve constar entre os participantes da reunião. |
| **Pós-Condições** | A lista de presença é apresentada ao participante. Nenhum dado é alterado. |
| **Fluxo Principal (Caminho de Sucesso)** | 1. Depois do término da reunião, o participante seleciona, na tela da reunião, o cartão que dá acesso à lista de presença e depois a aba Presença.<br>2. O aplicativo envia a requisição `GET /reunioes/<id>/lista-presenca`.<br>3. O sistema consulta a reunião na coleção `reunioes` e verifica que o usuário é participante.<br>4. O sistema consulta na coleção `usuarios` o `nome` de cada participante.<br>5. O sistema monta, para cada participante, os campos `usuario_id`, `nome`, `convite`, `situacao`, `entrada`, `saida` e `minutos`. O campo `situacao` recebe a situação da presença ou `SEM_REGISTRO`, quando não há entrada. O campo `minutos` recebe a diferença entre saída e entrada, em minutos inteiros, ou fica vazio quando não há saída.<br>6. O sistema devolve a lista com `reuniao_id`, `titulo`, `inicio`, `fim`, `gerada_em` e `participantes`.<br>7. O sistema apresenta o total de presentes e cada participante com a situação Presente, Aguardando, Recusada ou Ausente e o período de permanência. |
| **Fluxos Alternativos** | **FA-01: Compartilhar a lista**<br>1. O participante seleciona "Compartilhar".<br>2. O sistema monta um texto com o título, a data, o horário e a situação de cada participante.<br>3. O sistema abre o menu de compartilhamento do aparelho com esse texto. |
| **Fluxos de Exceção** | **FE-01: Usuário não participa**<br>1. O usuário não consta em `participantes`.<br>2. O sistema informa "Você não participa desta reunião".<br><br>**FE-02: Falha ao compartilhar**<br>1. O aparelho não conclui o compartilhamento.<br>2. O sistema informa "Não foi possível compartilhar.". |

---

## RF-10 - Salvar rascunho da ata

| Campo | Descrição |
|---|---|
| **Ator Principal** | Responsável da comissão |
| **Objetivo** | Registrar ou atualizar a ata de uma reunião encerrada. |
| **Entrada** | `PATCH /reunioes/:id/ata`, com `conteudo` de 20 a 20.000 caracteres e a `versao` atualmente exibida. |
| **Pré-Condições** | Usuário autenticado; reunião `AGENDADA` e encerrada; usuário responsável pela comissão; ata inexistente ou em `RASCUNHO`. |
| **Pós-Condições** | O conteúdo é salvo como `RASCUNHO`, a versão é incrementada e o autor e a data de atualização são registrados. |
| **Exceções** | Versão divergente retorna conflito; ata já enviada não pode ser editada; gravação concorrente não sobrescreve a versão mais recente. |

## RF-11 - Enviar ata para aprovação

| Campo | Descrição |
|---|---|
| **Ator Principal** | Responsável da comissão |
| **Objetivo** | Encaminhar o rascunho aos participantes com presença confirmada. |
| **Entrada** | `POST /reunioes/:id/ata/publicar`, com a `versao` atual. |
| **Pré-Condições** | Reunião encerrada; ata em `RASCUNHO`; versão atual; ao menos uma presença `CONFIRMADA`. |
| **Pós-Condições** | A ata passa para `EM_APROVACAO`; os presentes confirmados tornam-se aprovadores e a lista de aprovações começa vazia. |
| **Exceções** | Rascunho ausente, versão divergente ou nenhuma presença confirmada impedem a operação. |

## RF-12 - Aprovar ata

| Campo | Descrição |
|---|---|
| **Ator Principal** | Participante com presença confirmada |
| **Objetivo** | Registrar a aprovação individual da versão publicada. |
| **Entrada** | `POST /reunioes/:id/ata/aprovar`, com a `versao` atual. |
| **Pré-Condições** | Reunião encerrada; ata em `EM_APROVACAO`; usuário incluído em `aprovadores`. |
| **Pós-Condições** | O usuário entra em `aprovacoes`; quando todos aprovam, a situação muda para `APROVADA`. Uma repetição pelo mesmo usuário é idempotente. |
| **Exceções** | Versão divergente, usuário fora dos aprovadores ou ata fora de aprovação impedem a operação. |

