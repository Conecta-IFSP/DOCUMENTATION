# Requisitos funcionais — Reuniões

> Padrão da disciplina · Módulo REU · Formulário padrão · Revisado em 10/11/2026

## Resumo

Cada caso de uso de [Casos de uso](1%20-%20casos-uso-reunioes.md) é descrito aqui como requisito funcional de sistema, usando apenas o formulário padrão: Função, Descrição, Entradas, Origem, Saída, Destino, Ação, Pré-condição, Pós-condição e Efeitos colaterais. O diagrama de sequência de cada requisito está em [Diagramas de sequência](6%20-%20diagramas-sequencia-reunioes.md), com o mesmo número.

Todas as rotas exigem o cabeçalho `X-API-Key`, um token JWT válido e uma conta de usuário comum ativa. Os textos entre aspas são as mensagens que o aplicativo mostra. Os requisitos da ata (antigos RF-REU-10 a RF-REU-12) estão em [RF-ATA-01 a RF-ATA-08](../../4%20-%20Incremento3/atas/2%20-%20requisitos-funcionais-atas.md).

Dois passos se repetem em quase todos os requisitos e são descritos uma vez aqui:

- **Carregar a reunião** (`ReunioesService.carregar`): confere se o identificador é um `ObjectId` válido, busca o usuário por `findById` e exige `ativo = true`, busca a reunião por `findById` e procura, em `participantes`, um item cujo `usuario_id` seja igual ao do usuário. Sem reunião, responde "Reunião não encontrada"; sem o usuário na lista, "Você não participa desta reunião". `carregarComoDono` faz o mesmo e ainda exige que `criada_por` seja igual ao usuário; senão responde "Apenas quem agendou a reunião pode fazer isso".
- **Gravar com proteção** (`findOneAndUpdate`): a gravação repete, no filtro, as condições conferidas antes. Se outra pessoa alterou a reunião nesse intervalo, nenhum documento é encontrado e o sistema responde "A reunião mudou. Atualize a tela." em vez de sobrescrever.

## RF-REU-01 — Agendar reunião

| RF-REU-01              | Agendar reunião |
| ---------------------- | --------------- |
| **Função**             | Agenda uma reunião para uma comissão. |
| **Descrição**          | Insere na coleção `reunioes` uma reunião com os dados informados pelo presidente e cria um convite para cada pessoa escolhida. |
| **Entradas**           | Objeto `CreateReuniaoDto`: `comissao_id` (identificador da comissão); `titulo` (texto de 3 a 120 caracteres); `local` (opcional; vazio ou texto de 2 a 120 caracteres); `inicio` e `fim` (data e hora no formato ISO 8601); `dia_inteiro` (opcional, verdadeiro ou falso); `pauta` (opcional, texto de até 2.000 caracteres); `convidados_ids` (opcional, lista de identificadores de usuário, sem repetição). Opcionalmente, até 10 arquivos para a pauta (RF-REU-14). |
| **Origem**             | Presidente da comissão (formulário) e banco de dados (comissões que ele preside e seus integrantes). |
| **Saída**              | Objeto `Reuniao` criado, com `situacao = AGENDADA`, os participantes com nome e e-mail e o campo `sou_dono = true`. |
| **Destino**            | Tela da reunião (`TelaReuniao`), aberta logo após o agendamento, e coleção `reunioes`. |
| **Ação**               | O presidente toca em "Nova reunião" na aba Agenda (o botão só aparece para quem é responsável por alguma comissão ativa) ou na tela da comissão. O sistema abre o formulário "Nova reunião" (`FormularioReuniao`) com: a comissão, quando ele preside mais de uma; "Título" (campo de texto vazio, até 120 caracteres); "Data" (abre um calendário); a chave "Dia inteiro", inicialmente desligada; "Início" e "Término" (ao tocar, abrem o seletor de hora do próprio aparelho: o relógio no Android e a roda no iOS, esta dentro de uma folha com o botão "Confirmar"; enquanto nada foi escolhido, o campo mostra "Escolher"); "Local (opcional)" (até 120 caracteres); "Convidados" (lista dos integrantes da comissão, todos marcados no início); "Pauta (opcional)" (até 2.000 caracteres); e "Anexos (opcional)". Ao ligar "Dia inteiro", a função `alternarDiaInteiro` apaga e esconde os campos de horário. Ao tocar em "Agendar reunião", a tela valida o título ("Informe um título com pelo menos 3 caracteres."), os horários ("Horário inválido.", "O término deve ser depois do início.") e o local ("O local deve ter pelo menos 2 caracteres."). A função `calcularHorario` monta `inicio` e `fim`: com "Dia inteiro", 00:00 e 23:59:59.999 do dia escolhido; senão, o dia com os horários digitados. A tela chama `api('/reunioes', 'POST', dados)`. Na API, `ReunioesController.criar` chama `ReunioesService.criar`, que: (1) chama `autorizarResponsavel`, que busca a comissão por `findById` e exige, em `membros`, um item com o `usuario_id` do usuário e `papel = RESPONSAVEL`, exige `ativo = true` na comissão e, na organização dela, `status = APROVADA` e o vínculo do usuário com `status = APROVADO`; (2) converte `inicio` e `fim` em datas; (3) chama `validarHorario`, que exige `fim` maior que `inicio` e `fim` maior que o horário atual; (4) usa `convidados_ids` ou, se o campo não veio, todos os integrantes, e confere que cada identificador está em `membros` da comissão; (5) junta o presidente à lista, sem repetição; (6) grava com `ReuniaoModel.create`, com o convite do presidente em `ACEITO` e os demais em `PENDENTE`; (7) devolve a reunião com os nomes dos participantes. Com a reunião criada, a tela envia os anexos escolhidos, um a um, e abre a tela da reunião. |
| **Pré-condição**       | Usuário autenticado e ativo, responsável de uma comissão ativa, com vínculo aprovado em organização aprovada. |
| **Pós-condição**       | A reunião aparece na agenda de todos os participantes. Em caso de erro, o sistema mostra o motivo: campo inválido; "O término deve ser depois do início"; "Não é possível agendar uma reunião que já terminou"; "Convide apenas integrantes desta comissão"; "Apenas o responsável da comissão pode agendar reuniões"; "Reative a comissão antes de agendar reuniões"; "Comissão não encontrada". |
| **Efeitos colaterais** | Os convidados passam a ver um convite pendente na tela inicial e na agenda. |

## RF-REU-02 — Consultar agenda

| RF-REU-02              | Consultar agenda |
| ---------------------- | ---------------- |
| **Função**             | Lista as reuniões do usuário. |
| **Descrição**          | Busca na coleção `reunioes` as reuniões em que o usuário é participante e as apresenta por dia. |
| **Entradas**           | Usuário autenticado (pelo token) e, opcionalmente, na URL: `de` e `ate` (data e hora ISO 8601) e `comissao_id` (identificador da comissão). |
| **Origem**             | Banco de dados. |
| **Saída**              | Lista de objetos `Reuniao`, cada um com a comissão (nome), os participantes (nome e e-mail) e `sou_dono`, em ordem crescente de `inicio`. |
| **Destino**            | Aba Agenda (`TelaAgenda`), seção "Próximas reuniões" da tela inicial e aba de reuniões da tela da comissão. |
| **Ação**               | Ao abrir a aba Agenda, e sempre que ela volta a ser exibida, a função `carregar` chama `api('/reunioes')`. Na API, `ReunioesController.listar` chama `ReunioesService.listar`, que monta a consulta: `participantes.usuario_id` igual ao identificador do usuário; quando há `de`, `fim` maior ou igual a `de`; quando há `ate`, `inicio` menor ou igual a `ate`; quando há `comissao_id`, `comissao_id` igual ao informado. Executa `ReuniaoModel.find`, preenche o nome da comissão e o nome e e-mail dos participantes com `populate` e ordena pelo campo `inicio`, do mais antigo para o mais recente (`sort({ inicio: 1 })`). A tela agrupa as reuniões pelo dia de `inicio`, marca no calendário os dias que têm reunião e lista os sete dias a partir do dia escolhido, com título, horário (ou "Dia inteiro"), comissão e situação. Reunião com convite pendente mostra os botões "Aceitar" e "Recusar" (RF-REU-05). A tela inicial chama a mesma rota com `de` igual ao horário atual e mostra as três primeiras; a tela da comissão usa `comissao_id`. |
| **Pré-condição**       | Usuário autenticado e ativo. |
| **Pós-condição**       | As reuniões são exibidas; sem reuniões, a tela mostra o aviso de lista vazia. Data inválida responde "Período inválido"; comissão inválida, "ID inválido". |
| **Efeitos colaterais** | Nenhum. |

## RF-REU-03 — Visualizar reunião

| RF-REU-03              | Visualizar reunião |
| ---------------------- | ------------------ |
| **Função**             | Mostra os dados completos de uma reunião. |
| **Descrição**          | Busca a reunião e a exibe somente a quem participa dela. |
| **Entradas**           | Identificador da reunião. |
| **Origem**             | Banco de dados. |
| **Saída**              | Objeto `Reuniao` com comissão, participantes, presenças, pauta, anexos, ata e `sou_dono`. |
| **Destino**            | Tela da reunião (`TelaReuniao`). |
| **Ação**               | O usuário toca em uma reunião na agenda, na tela inicial ou na comissão. A função `buscar` chama `api('/reunioes/{id}')`. Na API, `ReunioesController.buscarPorId` chama `ReunioesService.buscarPorId`, que carrega a reunião (ver Resumo) e preenche comissão e participantes com `populate`. A tela mostra título, comissão, data, horário ou "Dia inteiro", local (só quando preenchido) e a situação; o cartão de convite, enquanto o usuário não respondeu; os botões "Marcar entrada" e "Marcar saída" durante a reunião; o cartão da ata, depois do término; e as abas "Presença" e "Pauta", esta com o texto da pauta e os anexos. Enquanto a tela está aberta, `buscar` é chamada de novo a cada 30 segundos e quando o usuário puxa a tela para baixo. |
| **Pré-condição**       | Usuário autenticado, ativo e participante da reunião. |
| **Pós-condição**       | Os dados são exibidos, ou o sistema mostra "Reunião não encontrada" ou "Você não participa desta reunião". |
| **Efeitos colaterais** | Nenhum. |

## RF-REU-04 — Cancelar reunião

| RF-REU-04              | Cancelar reunião |
| ---------------------- | ---------------- |
| **Função**             | Cancela uma reunião que ainda não terminou. |
| **Descrição**          | Muda o campo `situacao` da reunião para `CANCELADA`, sem apagar o documento. |
| **Entradas**           | Identificador da reunião e objeto `UpdateReuniaoDto` contendo apenas `situacao = CANCELADA`. |
| **Origem**             | Presidente da comissão que agendou a reunião. |
| **Saída**              | Objeto `Reuniao` com `situacao = CANCELADA`. |
| **Destino**            | Tela da reunião e campo `situacao` em `reunioes`. |
| **Ação**               | Na tela da reunião, o presidente toca em "Mais opções" e em "Cancelar reunião". O sistema pergunta "Cancelar a reunião <título>?" e, se ele confirmar, a função `executar` chama `api('/reunioes/{id}', 'PATCH', { situacao: 'CANCELADA' })`. Na API, `ReunioesController.atualizar` chama `ReunioesService.atualizar`, que separa `situacao` dos demais campos: se vierem os dois, recusa com "Cancele ou edite a reunião, não as duas coisas juntas"; se vier só `situacao`, chama `cancelar`. O método `cancelar` carrega a reunião com `carregarComoDono`, chama `exigirAgendada` (recusa se `situacao` já for `CANCELADA`) e compara `fim` com o horário atual. Grava com `findOneAndUpdate`, filtrando por `_id`, `criada_por` igual ao usuário, `situacao = AGENDADA` e `fim` maior que o horário atual, e alterando somente `situacao`. |
| **Pré-condição**       | Usuário é quem agendou a reunião; reunião `AGENDADA` e com término no futuro. |
| **Pós-condição**       | O sistema mostra "Reunião cancelada." ou o motivo da recusa: "Esta reunião foi cancelada"; "Esta reunião já terminou"; "Apenas quem agendou a reunião pode fazer isso"; "A reunião mudou. Atualize a tela.". Desistir da confirmação não altera nada. |
| **Efeitos colaterais** | Convite, entrada, saída, anexos e ata ficam bloqueados para essa reunião. |

## RF-REU-05 — Responder convite

| RF-REU-05              | Responder convite |
| ---------------------- | ----------------- |
| **Função**             | Registra o aceite ou a recusa do convite. |
| **Descrição**          | Altera o campo `convite` do participante dentro do documento da reunião. |
| **Entradas**           | Identificador da reunião e objeto `ResponderConviteReuniaoDto` com `aceito` (verdadeiro ou falso). |
| **Origem**             | Participante da reunião. |
| **Saída**              | Objeto `Reuniao` com o convite do usuário em `ACEITO` ou `RECUSADO`. |
| **Destino**            | Aba Agenda, folha "Convites" da tela inicial, tela da reunião e campo `participantes.convite` em `reunioes`. |
| **Ação**               | O participante toca em "Aceitar" ou "Recusar" no cartão do convite. A tela chama `api('/reunioes/{id}/convite', 'PATCH', { aceito })`. Na API, `ReunioesController.responderConvite` chama `ReunioesService.responderConvite`, que carrega a reunião (ver Resumo); recusa quem a agendou, porque o convite dele já nasce aceito; chama `exigirAgendada`; compara `fim` com o horário atual; e, na recusa, confere se o participante ainda não tem `presenca`. Grava com `findOneAndUpdate`, filtrando por `_id`, `situacao = AGENDADA` e por um participante com o `usuario_id` do usuário (na recusa, também sem `presenca`), e altera `participantes.$.convite`. Quem recusou pode aceitar depois, enquanto a reunião não terminar. O cartão do convite continua na tela depois do aceite, agora com o botão "Não vou mais": a função `recusarConvite` abre a confirmação "Avisar que você não vai mais participar desta reunião?" e só envia a recusa depois do "Confirmar", para que a desistência não aconteça por toque acidental; recusar um convite ainda pendente continua sem confirmação. Quem já marcou entrada deixa de ver o cartão, porque a API negaria a recusa. Quando a data ou o horário da reunião foram alterados (RF-REU-13), o cartão mostra "Horário alterado · Confirme se vai participar no novo horário". |
| **Pré-condição**       | Usuário participa da reunião e não foi quem a agendou; reunião `AGENDADA` e não encerrada; para recusar, sem entrada marcada. |
| **Pós-condição**       | O cartão passa a mostrar a resposta, ou o sistema mostra: "Quem agendou a reunião não responde ao convite"; "Esta reunião foi cancelada"; "Esta reunião já terminou"; "Você já marcou entrada nesta reunião"; "A reunião mudou. Atualize a tela.". |
| **Efeitos colaterais** | Quem recusou deixa de ver a reunião entre as próximas e não pode marcar entrada até aceitar. |

## RF-REU-06 — Registrar entrada

| RF-REU-06              | Registrar entrada |
| ---------------------- | ----------------- |
| **Função**             | Registra a entrada do participante na reunião. |
| **Descrição**          | Cria o subdocumento `presenca` do participante, com o horário da entrada. |
| **Entradas**           | Identificador da reunião. O horário não é informado: o servidor usa o seu relógio. |
| **Origem**             | Participante da reunião. |
| **Saída**              | Objeto `Reuniao` com `presenca = { entrada, situacao, marcada_por = PROPRIO }` no participante. |
| **Destino**            | Tela da reunião, cartão "Agora" da tela inicial e campo `participantes.presenca` em `reunioes`. |
| **Ação**               | Durante o horário da reunião, a tela mostra o botão "Marcar entrada" a quem não recusou o convite. Ao tocar, a função `registrar` chama `autenticarComDispositivo`, que abre o pedido de identidade do próprio aparelho (biometria, Face ID ou PIN) sem nenhum texto do aplicativo, e devolve verdadeiro em aparelho sem bloqueio configurado; se a confirmação falhar, a tela mostra "Não foi possível confirmar sua identidade." e nada é enviado. Confirmada a identidade, `executar` chama `api('/reunioes/{id}/entrada', 'POST')`. Na API, `ReunioesController.marcarEntrada` chama `ReunioesService.marcarEntrada`, que carrega a reunião (ver Resumo), chama `exigirAgendada` e compara o horário atual com `inicio` e `fim`: antes de `inicio` responde "A reunião ainda não começou"; depois de `fim`, "Esta reunião já terminou". Confere que o convite não é `RECUSADO` e que não existe `presenca`. Grava com `findOneAndUpdate`, filtrando por um participante com o `usuario_id` do usuário, convite diferente de `RECUSADO` e sem `presenca`, e define `participantes.$.presenca` com `entrada` igual ao horário atual. A `situacao` nasce `CONFIRMADA` para quem agendou a reunião e `AGUARDANDO` para os demais, que dependem da confirmação do presidente (RF-REU-08). |
| **Pré-condição**       | Usuário participa da reunião; reunião `AGENDADA` e em andamento; convite não recusado; entrada ainda não marcada. |
| **Pós-condição**       | O sistema mostra "Entrada marcada." ou: "Não foi possível confirmar sua identidade."; "A reunião ainda não começou"; "Esta reunião já terminou"; "Aceite o convite antes de marcar a entrada"; "Sua entrada já foi registrada". |
| **Efeitos colaterais** | O participante aparece para o presidente como aguardando confirmação. |

## RF-REU-07 — Registrar saída

| RF-REU-07              | Registrar saída |
| ---------------------- | --------------- |
| **Função**             | Registra a saída do participante. |
| **Descrição**          | Preenche o campo `saida` da presença do participante. |
| **Entradas**           | Identificador da reunião. O horário não é informado: o servidor usa o seu relógio. |
| **Origem**             | Participante da reunião que já marcou entrada. |
| **Saída**              | Objeto `Reuniao` com `presenca.saida` preenchido. |
| **Destino**            | Tela da reunião e campo `participantes.presenca.saida` em `reunioes`. |
| **Ação**               | Depois de marcar a entrada, a tela mostra o botão "Marcar saída". Ao tocar, a função `registrar` chama `autenticarComDispositivo`, que abre o pedido de identidade do próprio aparelho (biometria, Face ID ou PIN) sem nenhum texto do aplicativo, e devolve verdadeiro em aparelho sem bloqueio configurado; se a confirmação falhar, a tela mostra "Não foi possível confirmar sua identidade." e nada é enviado. Confirmada a identidade, `executar` chama `api('/reunioes/{id}/saida', 'POST')`. Na API, `ReunioesController.marcarSaida` chama `ReunioesService.marcarSaida`, que carrega a reunião (ver Resumo), chama `exigirAgendada`, confere que existe `presenca` com situação diferente de `RECUSADA` e que `saida` ainda está vazia. O horário gravado é o menor entre o horário atual e `fim`: quem marca a saída depois do término fica com a saída no horário de término. Grava com `findOneAndUpdate`, filtrando por um participante com o `usuario_id` do usuário, sem `presenca.saida` e com `presenca.situacao` diferente de `RECUSADA`, e define `participantes.$.presenca.saida`. |
| **Pré-condição**       | Usuário participa da reunião, marcou entrada, a presença não foi recusada e a saída ainda não foi marcada. |
| **Pós-condição**       | O sistema mostra "Saída marcada." ou: "Não foi possível confirmar sua identidade."; "Marque a entrada antes da saída"; "Sua saída já foi registrada". |
| **Efeitos colaterais** | A lista de presença passa a mostrar o tempo de permanência em minutos (RF-REU-09). |

## RF-REU-08 — Confirmar ou recusar presença

| RF-REU-08              | Confirmar ou recusar presença |
| ---------------------- | ----------------------------- |
| **Função**             | Valida a presença de um participante. |
| **Descrição**          | Altera a situação da presença de um participante ou, para quem não marcou entrada, cria a presença já confirmada. |
| **Entradas**           | Identificador da reunião, identificador do participante (na URL) e objeto `AtualizarPresencaDto` com `situacao` igual a `CONFIRMADA` ou `RECUSADA`. |
| **Origem**             | Presidente da comissão que agendou a reunião. |
| **Saída**              | Objeto `Reuniao` com a presença do participante atualizada. |
| **Destino**            | Aba "Presença" da tela da reunião e campo `participantes.presenca` em `reunioes`. |
| **Ação**               | Depois do início da reunião, a aba "Presença" mostra ao presidente, para cada pessoa que marcou entrada e aguarda, os botões de confirmar e de recusar; para quem não marcou entrada, o botão de marcar como presente. Ao tocar, a função `marcarPresenca` chama `api('/reunioes/{id}/presencas/{usuarioId}', 'PATCH', { situacao })`. Na API, `ReunioesController.atualizarPresenca` chama `ReunioesService.atualizarPresenca`, que valida o identificador do participante, carrega a reunião com `carregarComoDono`, chama `exigirAgendada`, compara o horário atual com `inicio` e procura, em `participantes`, o item com o `usuario_id` informado. Recusar alguém que não marcou entrada é negado. Grava com `findOneAndUpdate`: se o participante já tem `presenca`, altera `participantes.$.presenca.situacao`; se não tem, cria `presenca` com `entrada` igual ao `inicio` da reunião, `situacao = CONFIRMADA` e `marcada_por = RESPONSAVEL`. O filtro exige que a existência de `presenca` seja a mesma da leitura. |
| **Pré-condição**       | Usuário é quem agendou a reunião; reunião `AGENDADA` e já iniciada; a pessoa é participante. |
| **Pós-condição**       | A lista mostra a nova situação, ou o sistema mostra: "A reunião ainda não começou"; "Esta pessoa não participa da reunião"; "Esta pessoa não marcou entrada"; "Apenas quem agendou a reunião pode fazer isso"; "A presença mudou. Atualize a tela.". |
| **Efeitos colaterais** | Só participantes com presença `CONFIRMADA` aprovam a ata da reunião (RF-ATA-02). |

## RF-REU-09 — Gerar lista de presença

| RF-REU-09              | Gerar lista de presença |
| ---------------------- | ----------------------- |
| **Função**             | Mostra a lista de presença da reunião. |
| **Descrição**          | Monta, a partir dos participantes da reunião, a relação de quem esteve presente, com horários e tempo de permanência. Não grava nada. |
| **Entradas**           | Identificador da reunião. |
| **Origem**             | Banco de dados. |
| **Saída**              | Objeto com `reuniao_id`, `titulo`, `inicio`, `fim`, `gerada_em` e a lista `participantes`, cada item com `usuario_id`, `nome`, `convite`, `situacao`, `entrada`, `saida` e `minutos`. |
| **Destino**            | Aba "Presença" da tela de registros (`TelaRegistrosReuniao`) e menu de compartilhamento do aparelho. |
| **Ação**               | O participante abre os registros da reunião e toca na aba "Presença". A função `carregarLista` chama `api('/reunioes/{id}/lista-presenca')`. Na API, `ReunioesController.gerarListaPresenca` chama `ReunioesService.gerarListaPresenca`, que carrega a reunião (ver Resumo) e busca os nomes com `UsuarioModel.find`, filtrando por `_id` dentro da lista de participantes e trazendo só o campo `nome`. Para cada participante, a situação é a de `presenca.situacao` ou `SEM_REGISTRO`, quando não há presença; `minutos` é a diferença entre `saida` e `entrada`, em minutos inteiros arredondados para baixo, ou vazio quando falta a saída. A tela mostra o total de presentes e de ausentes e, por pessoa, "Presente", "Aguardando", "Recusada" ou "Ausente", com o período, por exemplo "14:00 – 15:30 · 90 min". Ao tocar em "Compartilhar", a função `compartilhar` monta um texto com título, data, horário e uma linha por pessoa, e abre o menu de compartilhamento do aparelho. |
| **Pré-condição**       | Usuário autenticado, ativo e participante da reunião. |
| **Pós-condição**       | A lista é exibida, ou o sistema mostra "Você não participa desta reunião" ou "Reunião não encontrada". |
| **Efeitos colaterais** | Nenhum. Quem não marcou entrada aparece como ausente, sem que uma presença seja criada. |

## RF-REU-13 — Editar reunião

| RF-REU-13              | Editar reunião |
| ---------------------- | -------------- |
| **Função**             | Altera os dados de uma reunião que ainda não terminou. |
| **Descrição**          | Atualiza, no documento da reunião, título, local, data, horário, pauta e convidados. |
| **Entradas**           | Identificador da reunião e objeto `UpdateReuniaoDto`, todos os campos opcionais e com as mesmas regras do agendamento: `titulo`, `local`, `inicio`, `fim`, `dia_inteiro`, `pauta` e `convidados_ids`. Ao menos um deve vir preenchido. |
| **Origem**             | Presidente da comissão que agendou a reunião (alterações) e banco de dados (dados atuais). |
| **Saída**              | Objeto `Reuniao` atualizado, com `horario_alterado_em` preenchido quando a data ou o horário mudaram. |
| **Destino**            | Tela da reunião e documento em `reunioes`. |
| **Ação**               | Na tela da reunião, o presidente toca em "Mais opções" e em "Editar reunião". A tela `TelaEditarReuniao` busca a reunião e abre o mesmo formulário do agendamento, preenchido com os dados atuais. Se a reunião já começou, a data, a chave "Dia inteiro" e os horários ficam travados e não abrem o seletor, com o aviso "A reunião já começou; a data e o horário não mudam.". Ao mudar a data ou o horário, o formulário avisa "Data ou horário alterado. Os convidados vão precisar confirmar de novo.". Ao tocar em "Salvar alterações", a tela chama `api('/reunioes/{id}', 'PATCH', campos)`. Na API, `ReunioesService.atualizar` encaminha para `editar`, que: (1) carrega a reunião com `carregarComoDono`, chama `exigirAgendada` e compara `fim` com o horário atual; (2) calcula o novo `inicio`, `fim` e `dia_inteiro`, mantendo os atuais quando não vieram; (3) se algum deles mudou e `inicio` atual já passou, recusa; se mudou antes do início, chama `validarHorario`; (4) com `convidados_ids`, chama `autorizarResponsavel`, confere que todos são integrantes da comissão, retira da lista quem saiu e acrescenta os novos com convite `PENDENTE`; (5) quando a data ou o horário mudaram, volta para `PENDENTE` o convite de todos, menos o do presidente, e preenche `horario_alterado_em`; (6) grava com `findOneAndUpdate`, filtrando por `_id`, `situacao = AGENDADA`, `fim` maior que o horário atual e `atualizado_em` igual ao lido. Esse último filtro impede que a edição apague uma resposta de convite ou uma entrada registrada no intervalo. Sair do formulário com alterações não salvas pede a confirmação "Descartar as alterações? O que você mudou não vai ser salvo.". |
| **Pré-condição**       | Usuário é quem agendou a reunião; reunião `AGENDADA` e com término no futuro. Para mudar data ou horário, a reunião ainda não começou. |
| **Pós-condição**       | O sistema mostra "Reunião atualizada." e volta à tela da reunião, ou mostra: "Nada para alterar"; "Esta reunião já terminou"; "A reunião já começou; a data e o horário não podem mudar"; "Convide apenas integrantes desta comissão"; "Apenas quem agendou a reunião pode fazer isso"; "A reunião mudou. Atualize a tela.". |
| **Efeitos colaterais** | Convites voltam para pendente quando o horário muda. Quem foi retirado dos convidados deixa de ver a reunião. |

## RF-REU-14 — Anexar arquivo à pauta

| RF-REU-14              | Anexar arquivo à pauta |
| ---------------------- | ---------------------- |
| **Função**             | Junta um arquivo à pauta da reunião. |
| **Descrição**          | Guarda o arquivo no GridFS do MongoDB (coleções `anexos.files` e `anexos.chunks`) e registra seus dados na lista `anexos` da reunião. |
| **Entradas**           | Identificador da reunião e um arquivo de até 10 MB (10.485.760 bytes), enviado no campo `arquivo` de um formulário `multipart/form-data`. Tipos aceitos: PDF, JPEG, PNG, texto, CSV, Word, Excel e PowerPoint (`.docx`, `.xlsx`, `.pptx`) e LibreOffice (`.odt`, `.ods`, `.odp`). |
| **Origem**             | Presidente da comissão que agendou a reunião, pelo seletor de arquivos do aparelho. |
| **Saída**              | Objeto `Reuniao` com um novo item em `anexos`: `_id` (o identificador do arquivo no GridFS), `nome` (até 200 caracteres), `tipo`, `tamanho` em bytes e `enviado_em`. |
| **Destino**            | Aba "Pauta" da tela da reunião, lista `anexos` em `reunioes` e GridFS. |
| **Ação**               | Na aba "Pauta", o presidente toca em "Adicionar anexo". A função `adicionarAnexos` chama `escolherArquivos`, que abre o seletor do aparelho e permite marcar vários arquivos, e `validarArquivos`, que confere tipo, tamanho e o limite de 10 anexos antes de enviar. Para cada arquivo, `enviarAnexo` faz o envio para `POST /reunioes/{id}/anexos`. Na API, o `FileInterceptor` recebe o arquivo com limite de 10 MB; acima disso o `ArquivoGrandeFilter` responde "O arquivo deve ter até 10 MB". `ReunioesController.adicionarAnexo` chama `ReunioesService.adicionarAnexo`, que: (1) carrega a reunião com `carregarComoDono`; (2) chama `exigirAnexosAbertos`, que exige a reunião `AGENDADA` e com `fim` no futuro; (3) confere se o tipo do arquivo está na lista `TIPOS_ANEXO`; (4) confere se a reunião tem menos de 10 anexos; (5) converte o nome do arquivo para utf8, para manter os acentos, e o corta em 200 caracteres; (6) grava o conteúdo com `GridFSBucket.openUploadStream` e espera o término; (7) acrescenta o anexo à lista com `findOneAndUpdate` e `$push`, filtrando por `_id`, `situacao = AGENDADA` e pela inexistência do décimo anexo. Se essa gravação não acontecer, apaga o arquivo recém-gravado com `GridFSBucket.delete`, para não deixar arquivo sem dono. |
| **Pré-condição**       | Usuário é quem agendou a reunião; reunião `AGENDADA` e não encerrada; menos de 10 anexos. |
| **Pós-condição**       | O sistema mostra "1 anexo adicionado." (ou a quantidade enviada) e lista o arquivo, ou mostra: "Selecione um arquivo"; "Envie PDF, imagem, texto ou documento do Office ou LibreOffice"; "A reunião já tem 10 anexos"; "O arquivo deve ter até 10 MB"; "Esta reunião já terminou"; "Apenas quem agendou a reunião pode fazer isso". Quando um arquivo falha, os seguintes não são enviados. |
| **Efeitos colaterais** | O arquivo ocupa espaço no banco até ser removido (RF-REU-16). |

## RF-REU-15 — Baixar anexo da pauta

| RF-REU-15              | Baixar anexo da pauta |
| ---------------------- | --------------------- |
| **Função**             | Abre um arquivo anexado à pauta. |
| **Descrição**          | Lê o conteúdo do arquivo no GridFS e o entrega ao aparelho do participante. |
| **Entradas**           | Identificador da reunião e identificador do anexo. |
| **Origem**             | Banco de dados. |
| **Saída**              | Conteúdo do arquivo, com o tipo, o tamanho e o nome original. |
| **Destino**            | Visualizador de arquivos do aparelho; na versão web, a pasta de downloads do navegador. |
| **Ação**               | Na aba "Pauta", o participante toca no nome do anexo. A função `abrir` chama `abrirAnexo`, do serviço `anexos`, que baixa o arquivo de `GET /reunioes/{id}/anexos/{anexoId}` para a pasta de cache do aplicativo, com `File.downloadFileAsync`, e o abre com `File.preview`. Na API, `ReunioesController.abrirAnexo` chama `ReunioesService.abrirAnexo`, que carrega a reunião (ver Resumo), chama `buscarAnexo`, que procura em `anexos` o item com o `_id` informado, e abre a leitura com `GridFSBucket.openDownloadStream`. O controller devolve um `StreamableFile` com o tipo e o tamanho do anexo e o nome no cabeçalho `Content-Disposition`. |
| **Pré-condição**       | Usuário autenticado, ativo e participante da reunião; o anexo existe. |
| **Pós-condição**       | O arquivo abre no aparelho, ou o sistema mostra "Anexo não encontrado", "Você não participa desta reunião" "Nenhum app do aparelho abre este tipo de arquivo." ou "Não foi possível baixar o anexo.". |
| **Efeitos colaterais** | Uma cópia do arquivo fica no cache do aplicativo. |

## RF-REU-16 — Remover anexo da pauta

| RF-REU-16              | Remover anexo da pauta |
| ---------------------- | ---------------------- |
| **Função**             | Retira um arquivo da pauta. |
| **Descrição**          | Remove o anexo da lista `anexos` da reunião e apaga o arquivo no GridFS. |
| **Entradas**           | Identificador da reunião e identificador do anexo. |
| **Origem**             | Presidente da comissão que agendou a reunião. |
| **Saída**              | Objeto `Reuniao` sem o anexo. |
| **Destino**            | Aba "Pauta" da tela da reunião, lista `anexos` em `reunioes` e GridFS. |
| **Ação**               | Na aba "Pauta", o presidente toca no ícone de remover ao lado do anexo. A função `confirmarRemocao` pergunta "Remover o anexo <nome>?" e, se ele confirmar, chama `removerAnexo`, que envia `DELETE /reunioes/{id}/anexos/{anexoId}`. Na API, `ReunioesController.removerAnexo` chama `ReunioesService.removerAnexo`, que carrega a reunião com `carregarComoDono`, chama `exigirAnexosAbertos` e `buscarAnexo`, e retira o item com `findOneAndUpdate` e `$pull`, filtrando por `_id`, `situacao = AGENDADA` e pela existência do anexo. Só depois de a lista ser gravada o conteúdo é apagado com `GridFSBucket.delete`. |
| **Pré-condição**       | Usuário é quem agendou a reunião; reunião `AGENDADA` e não encerrada; o anexo existe. |
| **Pós-condição**       | O sistema mostra "Anexo removido." ou: "Anexo não encontrado"; "Esta reunião já terminou"; "Apenas quem agendou a reunião pode fazer isso"; "A reunião mudou. Atualize a tela.". Desistir da confirmação não altera nada. |
| **Efeitos colaterais** | O arquivo é apagado em definitivo e o espaço é liberado. |
