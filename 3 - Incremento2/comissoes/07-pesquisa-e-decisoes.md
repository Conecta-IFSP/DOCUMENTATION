# Pesquisa de mercado e decisões

## Método e limite da pesquisa

Fonte inicial do grupo: [Avaliação de Utilização Atual e Softwares Equivalentes no Mercado](https://github.com/rafaelaszro/Projeto-Conecta---Homologa-o/blob/main/PropostaInicial/Avalia%C3%A7%C3%A3o%20de%20Utiliza%C3%A7%C3%A3o%20Atual%20e%20Softwares%20Equivalentes%20no%20Mercado.docx), também presente em `1 - PropostaInicial` no repositório de documentação. Aplicativos citados: Google Calendar, Google Meet, Microsoft Teams, Notion Calendar, Calendly e Fellow.

Nesta revisão, o documento do grupo foi confrontado com páginas oficiais de funcionalidades, consultadas em 19/10/2026. É pesquisa documental; não foi realizado experimento de usabilidade com participantes. Não se afirma que os concorrentes possuem os mesmos erros observados pelo professor no Conecta+. “Complexidade” e “fragmentação” são hipóteses/percepções do grupo que precisam ser medidas em tarefas reais.

## Comparação orientada ao requisito

|Referência|Evidência/fonte|Aproveitar no Conecta+|Risco a evitar / decisão rastreável|
|---|---|---|---|
|Google Calendar|Permite responder convites e acompanhar respostas: [ajuda de convites](https://support.google.com/calendar/answer/37135?hl=en-7), [respostas dos convidados](https://support.google.com/calendar/answer/16713792?hl=en-7)|Manter aceite/recusa visíveis no encontro, já implementados pelo grupo|Não confundir ACEITO com presença CONFIRMADA; PRE-01 distingue estados|
|Microsoft Teams|Diferencia permissões de proprietário, membro e convidado: [capacidades por papel](https://support.microsoft.com/en-us/teams/teams-channels/team-owner-member-and-guest-capabilities-in-microsoft-teams)|Papéis com operações explícitas, adotados em COM-01 a COM-09|Evitar controles habilitados por rótulo visual sem verificação no servidor; não inferir que Teams exige exatamente um proprietário|
|Fellow|Notas organizam agenda, registros e encaminhamentos: [estrutura da nota](https://help.fellow.ai/en/articles/8619527-basics-of-a-fellow-meeting-note)|Vincular a ata à reunião, com orientação sobre decisões e encaminhamentos; ATA-01|Evitar guardar texto sem identificação do encontro ou aprovação rastreável; ATA-02/03|
|Notion Calendar|Integra eventos e páginas/notas: [Calendar com Notion](https://www.notion.com/en-us/help/use-notion-calendar-with-notion)|Abrir registros pelo detalhe da reunião; acesso contextual|Não obrigar usuário a procurar documento em outro sistema; não foi integrada API Notion|
|Calendly|Organiza disponibilidade a partir de horários e calendários: [availability](https://calendly.com/help/availability-overview)|Referência para evolução de agendamento, fora da alteração de comissão|Não afirmar que Conecta+ detecta conflitos de disponibilidade: não foi implementado|
|Google Meet|O documento inicial do grupo o avalia como ferramenta de videoconferência integrada ao ecossistema Google|Preservar campo local da reunião para informar modalidade/local; videoconferência não é objetivo desta entrega|Não prometer geração automática de links ou presença capturada de chamadas; não há integração Meet no código. Esta linha registra o levantamento do grupo, não um novo teste de produto|

## Problemas do próprio Conecta+ e correções

- Tela extensa: pesquisa de pessoas por nome/e-mail, páginas de oito itens para comissão/equipe/candidatos e remoção da listagem duplicada de membros da organização na gestão de comissões. As listas ainda são recebidas integralmente da API; escala de rede permanece trabalho futuro.
- Zero ou vários responsáveis: campo obrigatório no cadastro; inclusão comum não aceita RESPONSAVEL; substituição única e atômica; bloqueio de remoção/rebaixamento do responsável; regularização explícita de legado. Não atribuir essa regra aos concorrentes: é exigência do professor para este domínio.
- Ações destrutivas: confirmação antes de desativar/remover; confirmação também para trocar responsável e publicar/aprovar ata. Cancelamento não envia a escrita.
- Atualização: mantém o resultado retornado na mesma tela; lista tem atualização por gesto/foco; registros têm botão Atualizar e gesto de puxar para baixo. Novo agendamento atualiza comissões/convidados sem limpar os campos já preenchidos. Descartar rascunho não salvo exige confirmação.
- Teclado e formulários: uso da biblioteca de formulário já adotada e folha com ajuste de teclado; transferência usa seletor embutido para evitar modal dentro de modal. Teste nativo ainda necessário.

## Decisões que precisam da comunidade

1. “Responsável” corresponde ao presidente usado nos slides? É suficiente uma pessoa ou há necessidade de substituto temporário?
2. Ata deve exigir unanimidade dos presentes confirmados, maioria ou somente presidente? A versão implementada exige unanimidade e congela aprovadores ao enviar.
3. Como corrigir uma ata publicada incorreta? Nesta versão não há reabertura/rejeição; definir processo antes do uso real.
4. Pessoa removida da organização deve conservar acesso às reuniões passadas? A política herdada permite consulta a participante ativo; não foi feita mudança transversal silenciosa.
5. O novo responsável pode administrar reuniões criadas pelo anterior? Hoje a autoria de reunião permanece; preparar ata exige também ser participante. Isso precisa ser alinhado entre os módulos.
6. Confirmar tamanho máximo de campos, necessidade de anexo/PDF, quórum, assinatura e exportação. Nenhum deles é deduzido como requisito já validado.

Registrar data, participantes, tarefa, decisão e concordância da comunidade em ata de validação. Não preencher resultados fictícios.
