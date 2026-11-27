# Pesquisa de mercado — Conecta+

> Proposta Inicial · Pesquisa feita em 27/11/2026, na entrega final

## 1. Objetivo

Verificar se o problema que o Conecta+ quer resolver existe para pessoas que participam de reuniões em grupos, comissões e entidades, e como as soluções existentes no mercado o atendem. O problema é a dispersão das informações de uma reunião: a data fica no calendário, o link da chamada em outro aplicativo, a pauta em mensagens, e a presença, as votações e a ata ficam em papel ou em arquivos soltos, o que torna o histórico difícil de consultar.

## 2. Método

A pesquisa tem duas partes:

1. **Análise documental dos concorrentes** já citados no documento [Avaliação de Utilização Atual e Softwares Equivalentes no Mercado](Avalia%C3%A7%C3%A3o%20de%20Utiliza%C3%A7%C3%A3o%20Atual%20e%20Softwares%20Equivalentes%20no%20Mercado.docx), de 24/09/2026: Google Calendar, Google Meet, Microsoft Teams, Notion Calendar, Calendly e Fellow. Cada informação foi conferida no site oficial do produto, com acesso em 27/11/2026. Quando a página oficial não cita uma função, a tabela diz "não citado", o que não quer dizer que a função não exista.
2. **Validação com o público**, por meio de declarações individuais assinadas por pessoas de fora do IFSP, coletadas entre 25 e 27/11/2026.

**Quando foi feita:** esta pesquisa foi feita na entrega final, depois do desenvolvimento, para responder à avaliação do professor, que apontou "levantamento feito sem pesquisa de mercado". Ela não orientou o planejamento do Incremento 1. Serve para conferir, ao final, se o problema e as funções entregues correspondem a uma necessidade real.

## 3. Concorrentes

Legenda das colunas do ciclo da reunião: **Sim** = a página oficial cita a função; **Pago** = citada só em plano pago; **Não citado** = a página oficial consultada não cita a função.

| Ferramenta | Foco principal (página oficial) | Agenda | Link da chamada | Pauta | Presença | Votação | Ata com aprovação | Histórico | Plano gratuito | Fonte (acesso em 27/11/2026) |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Google Calendar | Agenda e agendamento de eventos ("criar uma página de agendamento direto no Google Agenda") | Sim | Sim, integra o Google Meet ("Adicione videoconferências do Google Meet a um evento") | Não citado | Não citado | Não citado | Não citado | Não citado | Não informado nas páginas consultadas | https://workspace.google.com/products/calendar/ ; https://workspace.google.com/pricing |
| Google Meet | "Videoconferência para empresas" | Sim, com o Google Calendar ("Marque reuniões de forma fácil e disponibilize a pauta, os anexos e a confirmação de presença em um só lugar") | Sim | Sim (pauta e anexos no evento) | Pago ("Video meetings with attendance tracking", no plano Plus) | Pago (enquetes, como recurso dos planos pagos) | Não citado | Não citado (há gravações e transcrições) | Sim: chamadas em grupo com "até 100 participantes e duração de até 60 minutos"; chamadas individuais sem limite de tempo | https://workspace.google.com/products/meet/ ; https://workspace.google.com/pricing |
| Microsoft Teams | Comunicação e colaboração: chat, reuniões online e comunidades ("Converse, colabore e reúna-se com qualquer pessoa"; "Inicie um chat com qualquer pessoa"; "Crie uma comunidade para colaborar") | Não citado | Sim ("Crie uma reunião rápida com um link para que outras pessoas participem, mesmo sem uma conta do Teams") | Não citado | Não citado | Não citado | Não citado | Não citado | Sim ("Microsoft Teams gratuito"): reuniões "gratuitamente por até 60 minutos"; o número máximo de participantes não é citado na página | https://www.microsoft.com/pt-br/microsoft-teams/free |
| Notion Calendar | "All of your commitments, now in one place." | Sim ("Built-in scheduling") | Sim, integra Google Meet ("Available by default with Google Calendar") e Zoom | Não citado | Não citado | Não citado | Não citado | Não citado | Sim ("Get Notion Calendar for free."), sem limites informados para o Calendar | https://www.notion.com/product/calendar ; https://www.notion.com/pricing |
| Calendly | Agendamento ("Book meetings with the world's #1 scheduling tool") | Sim | Sim, acrescenta os dados da videochamada (Zoom, Google Meet e Teams) | Pago (resumos da reunião) | Não citado | Só enquetes para escolher o horário ("Coordinate time with groups using polls"), não votação na reunião | Não citado | Pago (histórico de reuniões por contato) | Sim ("Always free"): 1 tipo de evento e 1 calendário conectado | https://calendly.com/ ; https://calendly.com/pricing |
| Fellow | Anotações e pautas de reunião ("Collaborative meeting agendas") | Sincroniza calendários ("Multi-calendar sync"); agendamento próprio não citado | Acompanha reuniões de Zoom, Google Meet e Microsoft Teams, sem gerar o link | Sim | Não citado | Não citado | Não citado | Sim ("Meeting notes history"), com limite que varia por plano | Sim (US$ 0 por usuário): 5 anotações automáticas e 5 gravações por usuário, no total | https://fellow.ai/ ; https://fellow.ai/pricing/ (o endereço fellow.app redireciona para fellow.ai) |

## 4. Validação com o público

Foram coletadas **9 declarações individuais**, todas de pessoas de fora do IFSP, de 3 estados:

| Estado | Declarações |
| --- | --: |
| São Paulo | 7 |
| Rio de Janeiro | 1 |
| Santa Catarina | 1 |
| **Total** | **9** |

Contextos em que essas pessoas participam de reuniões:

| Contexto | Declarações |
| --- | --: |
| Faculdade | 3 |
| Trabalho | 2 |
| Residência médica | 1 |
| Reunião de pais | 1 |
| Atlética de faculdade | 1 |
| Projeto social | 1 |
| **Total** | **9** |

Cada pessoa leu e assinou um texto que descreve o problema: a data fica no calendário, o link da chamada em outro aplicativo, a pauta em mensagens, e a presença, as votações e a ata em papel ou em arquivos soltos, com um histórico difícil de consultar. Ao assinar, reconheceu a necessidade de um aplicativo que reúna agendamento, pauta, anexos, presença, votações, ata com aprovação e histórico.

As declarações assinadas foram entregues no Moodle da disciplina e não ficam neste repositório, porque contêm dados pessoais. Este documento usa só os dados agregados acima.

## 5. Resultados

- **Agenda e chamada estão bem atendidas.** Google Calendar, Notion Calendar e Calendly cuidam do agendamento; esses três e o Google Meet geram ou integram o link de uma videochamada (Google Meet, Zoom ou Teams), e o Fellow acompanha as chamadas desses serviços.
- **Pauta e anotações aparecem em parte.** O Google Meet cita a pauta e os anexos no evento, e o Fellow é voltado a pautas e anotações. O Calendly cita resumos da reunião só em plano pago.
- **Presença e votação aparecem pouco, e só em planos pagos.** Entre as páginas consultadas, só o Google Meet cita o registro de presença ("attendance tracking") e as enquetes, nos planos pagos. A enquete do Calendly serve para escolher o horário, não para votar na reunião.
- **Nenhuma página consultada cita uma ata com aprovação formal dos participantes.** Também nenhuma reúne presença, votação, ata com aprovação e histórico num só lugar.
- **Os planos gratuitos têm limites** que afetam grupos e comissões: 60 minutos para chamadas em grupo no Google Meet e para reuniões no Microsoft Teams gratuito, 1 tipo de evento no Calendly e 5 anotações automáticas por usuário no Fellow.
- **As 9 pessoas reconheceram o problema e a necessidade** de um aplicativo que reúna essas etapas, em contextos variados (faculdade, trabalho, residência médica, reunião de pais, atlética e projeto social).

Isso sustenta a proposta do Conecta+ de não competir em agenda nem em videochamada, mas cobrir o que falta: presença, votação, ata com aprovação e histórico, organizados por comissão.

## 6. Requisitos derivados

| Conclusão | Função no Conecta+ | Onde está |
| --- | --- | --- |
| As reuniões acontecem em grupos e comissões | Organizações, convites, pedidos de entrada e comissões com responsável | [Cadastro de organização](../2%20-%20Incremento%201/cadastro-organizacao/README.md) e [Cadastro de comissão](../2%20-%20Incremento%201/cadastro-comissao/README.md) |
| A data e a pauta ficam espalhadas | Agendamento com pauta e anexos, agenda com calendário mensal e convites | [Reuniões](../3%20-%20Incremento%202/reunioes/README.md) (RF-REU-01 a RF-REU-05 e RF-REU-13 a RF-REU-16) |
| A presença é pouco atendida e só em planos pagos | Entrada e saída com biometria, confirmação pelo presidente, link de presença, correção de horários, saída automática e lista de presença em PDF | [Reuniões](../3%20-%20Incremento%202/reunioes/README.md) (RF-REU-06 a RF-REU-09 e RF-REU-17 a RF-REU-22) |
| A votação na reunião não aparece nas ferramentas de agenda | Votação aberta ou fechada, com voto em branco e nulo e resultado só no encerramento | [Votação na reunião](../4%20-%20Incremento%203/votacao/README.md) |
| Nenhuma ferramenta cita ata com aprovação | Ata com envio, prazo, aprovação ou reprovação pelos presentes, versões e PDF | [Atas de reunião](../4%20-%20Incremento%203/atas/README.md) |
| O histórico é difícil de consultar | Agenda com reuniões encerradas, registros da reunião (ata, versões e lista de presença) e relatório individual em PDF e CSV | [Reuniões](../3%20-%20Incremento%202/reunioes/README.md), [Atas de reunião](../4%20-%20Incremento%203/atas/README.md) e [Relatório individual](../4%20-%20Incremento%203/relatorio-individual/README.md) |
| Agenda e chamada já são bem atendidas | O Conecta+ não gera o link da chamada; o local da reunião é um campo de texto opcional | [Objetivos: situação na entrega final](objetivos.md#situação-na-entrega-final-27112026) |

## 7. Limitações

- **Declarações não são um questionário.** Não há perguntas quantitativas nem escalas; cada pessoa só confirmou, por escrito, o problema e a necessidade descritos no texto.
- **Amostra pequena e de conveniência:** 9 pessoas, escolhidas pela equipe e concentradas em São Paulo.
- **Coleta no fim do projeto:** as declarações foram coletadas na entrega final e não orientaram o planejamento.
- **Análise das páginas oficiais:** "não citado" quer dizer que a página consultada não cita a função, e não que ela não exista. Algumas páginas foram lidas em parte, e o Microsoft Teams foi analisado só pela página do plano gratuito.
- **Preços e planos podem mudar.** Os dados valem para a data de acesso, 27/11/2026.
