# Relatório individual

> Padrão da disciplina · Módulo REL · Incremento 3 · Revisado em 27/11/2026

## Resumo

O relatório individual mostra a cada usuário as reuniões em que a presença dele foi confirmada, com o tempo de participação registrado em cada uma e o total do período escolhido: todo o período, este mês ou este ano. Junto do histórico aparecem os vínculos atuais do usuário: as organizações aprovadas de que ele é membro aprovado, com o papel de administrador ou membro, e as comissões ativas dessas organizações em que ele está na equipe, com o papel de responsável ou membro. Os vínculos não dependem do período. O relatório pode ser exportado em PDF, com os vínculos e as reuniões, e em CSV, com uma linha por reunião e uma linha por vínculo. Cada pessoa vê só o próprio relatório; o servidor não grava nada nesta consulta.

## Escopo

- consultar as reuniões com presença confirmada, com entrada, saída, duração e resumo de minutos;
- filtrar o relatório por período (todo o período, este mês ou este ano), pela data de início da reunião;
- ver os vínculos atuais com organizações e comissões, com o papel em cada uma;
- exportar o relatório em PDF: no aplicativo instalado, pelo menu de compartilhamento do aparelho; no Expo Go e na web, pela tela de impressão;
- exportar o relatório em CSV, com as reuniões e os vínculos.

Ficam fora deste módulo o registro da presença e da saída, inclusive a saída automática no término da reunião, que estão em [Reuniões](../../3%20-%20Incremento%202/reunioes/README.md), e a gestão dos vínculos, que está em [Cadastro de organização](../../2%20-%20Incremento%201/cadastro-organizacao/README.md) e [Cadastro de comissão](../../2%20-%20Incremento%201/cadastro-comissao/README.md). Este módulo só lê esses dados.

## Atores

| Ator          | Quem é no sistema                                                                 | O que faz neste módulo                                                                 |
| ------------- | --------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------- |
| Usuário comum | Conta ativa do tipo `USUARIO`, autenticada no aplicativo.                          | Consulta e filtra o próprio relatório, vê os próprios vínculos e exporta em PDF e CSV. |

O administrador do sistema (`ADMIN_SISTEMA`) não é ator deste módulo: o aplicativo não mostra a aba "Relatório" para ele, e a API recusa a consulta com 403.

## Índice padronizado

|  Nº | Artefato                                                                               | Finalidade                                              |
| --: | -------------------------------------------------------------------------------------- | ------------------------------------------------------- |
|   1 | [Casos de uso](1%20-%20casos-uso-relatorio-individual.md)                              | Diagrama, ator e requisitos de usuário.                 |
|   2 | [Requisitos funcionais](2%20-%20requisitos-funcionais-relatorio-individual.md)         | Cada caso de uso no formulário padrão.                  |
|   3 | [Diagrama de classes](3%20-%20diagrama-classes-relatorio-individual.md)                | Tela, módulos do aplicativo, API e modelos lidos.       |
|   3 | [Dicionário de classes](3%20-%20dicionario-dados-classes-relatorio-individual.md)      | Atributos, tipos e operações de cada classe.            |
|   4 | [Diagrama relacional](4%20-%20diagrama-relacional-relatorio-individual.md)             | Visão lógica dos dados lidos no MongoDB.                |
|   4 | [Dicionário relacional](4%20-%20dicionario-dados-relacional-relatorio-individual.md)   | Campo a campo: tipo, nulo, chaves e geração de valores. |
|   5 | [Requisitos não funcionais](5%20-%20requisitos-nao-funcionais-relatorio-individual.md) | Critérios de qualidade com medida e verificação.        |
|   6 | [Diagramas de sequência](6%20-%20diagramas-sequencia-relatorio-individual.md)          | Mensagens entre tela, API, serviço e banco.             |
|   7 | [Testes](7%20-%20testes-relatorio-individual.md)                                       | Testes automatizados e pendências de execução manual.   |

## Rastreabilidade

| Caso de uso | Requisito | Sequência | Testes                                                                                      |
| ----------- | --------- | --------- | ------------------------------------------------------------------------------------------- |
| UC-REL-01   | RF-REL-01 | SD-REL-01 | CT-REL-01, CT-REL-02, CT-REL-03, CT-REL-07, CT-REL-17, CT-REL-23, CT-REL-27, CT-REL-34, CT-REL-36, CT-REL-38 |
| UC-REL-02   | RF-REL-02 | SD-REL-02 | CT-REL-04, CT-REL-11, CT-REL-29, CT-REL-34, CT-REL-36                                       |
| UC-REL-03   | RF-REL-03 | SD-REL-03 | CT-REL-05, CT-REL-12, CT-REL-13, CT-REL-23, CT-REL-28, CT-REL-34, CT-REL-36, CT-REL-38      |
| UC-REL-04   | RF-REL-04 | SD-REL-04 | CT-REL-08, CT-REL-12, CT-REL-13, CT-REL-14, CT-REL-18, CT-REL-19, CT-REL-21, CT-REL-22, CT-REL-26, CT-REL-28, CT-REL-30, CT-REL-34, CT-REL-35, CT-REL-36, CT-REL-38 |
| UC-REL-05   | RF-REL-05 | SD-REL-05 | CT-REL-09, CT-REL-10, CT-REL-12, CT-REL-13, CT-REL-14, CT-REL-16, CT-REL-20, CT-REL-21, CT-REL-25, CT-REL-34, CT-REL-35, CT-REL-36 |

Os requisitos não funcionais RNF-REL-01 a RNF-REL-13 e os testes que os verificam estão em [Requisitos não funcionais](5%20-%20requisitos-nao-funcionais-relatorio-individual.md) e [Testes](7%20-%20testes-relatorio-individual.md).

## Estudo da comunidade e comparação de relatórios

### Objetivo e método do levantamento

**Consulta e registro: 27/11/2026.** O levantamento complementa a proposta inicial, que já apresentava Microsoft Teams e Google Meet como aplicativos relacionados ao projeto. O objetivo desta análise é compreender como usuários consultam registros de participação, quais dificuldades relatam e como essas observações se relacionam com o relatório individual do Conecta+.

Foram selecionadas duas páginas oficiais sobre relatórios de presença e três discussões públicas com relatos diretamente relacionados à localização, disponibilidade e consulta do histórico. A seleção foi por pertinência ao requisito, a partir de buscas sobre *attendance report*, *recurring meeting attendance* e localização do relatório no Teams. As fontes oficiais foram usadas para confirmar funcionalidades; as publicações da comunidade, para registrar experiências de uso. As referências estão vinculadas a cada observação nas tabelas abaixo.

Trata-se de pesquisa documental exploratória, com amostra de conveniência. Neste levantamento específico do relatório, não foram realizados questionários, entrevistas ou testes comparativos com participantes. A validação com o público registrada pela equipe na [Pesquisa de mercado](../../1%20-%20Proposta%20Inicial/pesquisa-de-mercado.md) é uma iniciativa distinta; seus dados e limitações permanecem descritos naquela página. Os relatos não representam todos os usuários nem comprovam que um problema ocorre em todas as versões. Esta análise fundamenta a revisão das decisões do módulo; não atribui retrospectivamente todas as decisões iniciais a estas publicações.

### Recursos dos aplicativos usados como referência

| Aplicativo e fonte oficial | Recurso confirmado e aspecto útil | Relação com o relatório individual do Conecta+ |
| --- | --- | --- |
| [Microsoft Teams — Manage meeting attendance reports](https://support.microsoft.com/en-gb/teams/meetings/manage-meeting-attendance-reports-in-microsoft-teams) | Organizadores e coorganizadores elegíveis consultam registros de entrada e saída e baixam CSV. Reuniões recorrentes possuem relatório por ocorrência. A documentação apresenta acesso pelo chat do encontro e pela aba Attendance. Horários explícitos e exportação para planilha permitem conferir e reutilizar registros. | Mantivemos horários e exportação CSV como referências de informação útil. O nosso recorte é pessoal: o usuário consulta suas próprias participações em diferentes reuniões, com filtros de período. A comparação se refere ao relatório de presença documentado, não a todos os recursos analíticos do ecossistema Microsoft. |
| [Google Meet — Track attendance and view live stream report](https://support.google.com/meet/answer/10090454?hl=en-GB) | Em edições elegíveis do Google Workspace, o organizador recebe por e-mail um relatório em Google Sheets com identificação, horários e duração da participação. O arquivo reúne dados que podem ser conferidos e trabalhados em planilha. | A identificação do participante, os horários e a duração também são relevantes no Conecta+. Acrescentamos a visão de organizações e comissões atuais e oferecemos PDF e CSV dentro do app. Os registros vêm da nossa API; não há integração com chamadas ou planilhas do Meet. |

O ponto de referência comum é tornar a participação consultável e exportável. O Conecta+ tem escopo diferente de uma plataforma de videoconferência: consulta presenças registradas no próprio sistema e não observa quando alguém abre ou fecha uma chamada externa.

### Evidências encontradas na comunidade

| Identificação e fonte | Experiência descrita na publicação | Interpretação para o projeto |
| --- | --- | --- |
| EC-REL-01 — [New Teams (Attendance report????), r/MicrosoftTeams](https://www.reddit.com/r/MicrosoftTeams/comments/1bp4cn3/new_teams_attendance_report/) | O autor relata que conseguia obter o relatório durante anos, mas deixou de encontrá-lo no aplicativo após a mudança para New Teams. Afirma ainda conseguir acessá-lo pela visualização clássica na web. | Há um uso anterior considerado satisfatório pelo autor e uma dificuldade posterior de localização. Isso reforça a importância de um acesso identificável ao relatório e de ações de exportação visíveis. Não demonstra que a função tenha sido removida do Teams. |
| EC-REL-02 — [Webinar attendance report not populating, r/MicrosoftTeams](https://www.reddit.com/r/MicrosoftTeams/comments/1jkiclv/webinar_attendance_report_not_populating/) | O autor relata conseguir baixar a lista de inscritos, mas não a participação do webinar esperado. A discussão aborda sessões distintas e disponibilidade posterior do relatório. | Inscrição e participação precisam ser apresentadas como informações diferentes. No Conecta+, a tela informa que o histórico considera presenças confirmadas e mostra o período e o momento da consulta. O relato não comprova a causa técnica da falha no outro produto nem garante que nosso sistema nunca terá indisponibilidade. |
| EC-REL-03 — [Downloading Recurring Meeting Attendance Reports, Microsoft Q&A](https://learn.microsoft.com/en-us/answers/questions/5980379/downloading-recurring-meeting-attendance-reports) | Na pergunta publicada em 20/08/2026, a pessoa solicita acesso aos relatórios de reuniões recorrentes e afirma visualizar somente o da última reunião. A evidência utilizada é a pergunta do usuário; respostas automáticas da página não são tratadas como pesquisa com a comunidade. | A necessidade relatada vai além do encontro mais recente. Nosso histórico pessoal reúne participações de diferentes reuniões e permite consultar Todo período, Este mês ou Este ano. Essa é uma resposta de projeto a uma necessidade semelhante, sem afirmar que o Teams não permite recuperar relatórios anteriores. |

Os temas identificados foram **encontrar a função**, **compreender quais registros aparecem** e **recuperar participações anteriores**. A procura pelos relatórios e o relato de uso satisfatório anterior indicam utilidade para os autores dessas publicações. Não há base nesta amostra para afirmar que PDF, CSV ou uma interface específica sejam a preferência da maioria da comunidade.

### Decisões adotadas e comportamentos que buscamos evitar

| Decisão do relatório | Fundamento e comportamento esperado | Implementação e vínculo com os requisitos |
| --- | --- | --- |
| Acesso próprio ao relatório e exportações identificadas | EC-REL-01 reforça a necessidade de localizar a função. Evitamos exigir que o participante procure um arquivo no histórico de cada encontro para consultar seus próprios dados. | `TelaRelatorioIndividual` apresenta o título, o filtro e os botões Exportar PDF e Exportar CSV. Relaciona-se a RF-REL-01, RF-REL-04 e RF-REL-05. |
| Histórico pessoal reunido por período | EC-REL-03 reforça a necessidade de consultar encontros anteriores. Buscamos reduzir a organização manual de registros de várias reuniões pelo próprio participante. | `consultaRelatorio()` prepara os limites e `RelatoriosService.individual()` consulta as reuniões do titular, ordenando por `inicio` decrescente e `_id` decrescente. O filtro considera o início da reunião. RF-REL-01 e RF-REL-02. |
| Distinção entre convite, presença e duração | EC-REL-02 evidencia que uma lista de inscritos não responde à mesma pergunta que uma lista de participantes. Não tratamos convite aceito como presença confirmada nem horário ausente como zero minutos. | A consulta exige `presenca.situacao = CONFIRMADA`; a tela apresenta Duração não calculada quando `minutos` é `null` e identifica `saida_automatica` quando informada pelo módulo de presença. RF-REL-01. |
| PDF para leitura e CSV para planilha | A exportação tabular dos produtos consultados é uma referência funcional. A oferta adicional de PDF é uma decisão do projeto para apresentar o relatório com identidade visual; não foi identificada como preferência majoritária nos relatos. | O PDF apresenta logo, titular, período, vínculos e reuniões. `csvRelatorioIndividual()` gera 19 colunas, incluindo tipo de registro, papel no vínculo e origem da saída. RF-REL-04 e RF-REL-05. |
| Organizações e comissões atuais separadas do histórico | Esta decisão atende ao feedback do professor e ao contexto de comissões do Conecta+, não a uma solicitação encontrada nas três discussões. Evitamos apresentar um vínculo atual como se comprovasse participação em uma reunião passada. | `vinculos.organizacoes` e `vinculos.comissoes` aparecem em Meus vínculos atuais. O filtro de período restringe reuniões, mas não altera esses vínculos. RF-REL-03, RF-REL-04 e RF-REL-05. |
| Atualização e falhas visíveis | A indisponibilidade relatada em EC-REL-02 reforça a necessidade de informar o estado da consulta. Queremos evitar que um resultado anterior seja interpretado como uma atualização concluída após uma falha. | `carregar()` limpa o resultado antes da consulta e aplica somente a resposta do pedido atual. A tela apresenta erro e permite nova tentativa pelo botão Atualizar relatório; no aplicativo nativo, também há atualização ao arrastar a lista. RF-REL-01. |

As referências RF-REL seguem a numeração de [Requisitos funcionais](2%20-%20requisitos-funcionais-relatorio-individual.md). A tela e as exportações consideradas nesta análise podem ser conferidas no [frontend da homolog integrado em 48eb4f0](https://github.com/Conecta-IFSP/FRONTEND/tree/48eb4f07175416e59c886df18f4444556aab830b), especialmente em `src/app/(abas)/relatorio.tsx`, `src/services/exportar-relatorio-individual.ts`, `src/utils/relatorio-individual.ts` e `src/utils/pdf-relatorio-individual.ts`. As regras de seleção e cálculo estão em [RelatoriosService.individual()](https://github.com/Conecta-IFSP/BACKEND/blob/99b6b4f944cc0b281aa20897802f77e1b65b74e4/src/relatorios/relatorios.service.ts).

### Alcance da comparação e validação futura

A autonomia do participante, a identificação de vínculos e a apresentação de dados ausentes são escolhas adequadas ao objetivo do Conecta+. Não demonstram superioridade geral sobre Teams ou Meet. O relatório registra informações da conta no sistema, não comprova presença física, atenção durante uma chamada ou assinatura digital. O PDF e o CSV também não são certificados.

Os testes funcionais documentados em [Testes do relatório](7%20-%20testes-relatorio-individual.md) verificam regras e comportamento do software; não medem satisfação da comunidade. Como próxima avaliação de usabilidade, propomos observar participantes tentando localizar o relatório, consultar um período, distinguir vínculo de participação, interpretar uma duração ausente e exportar nos dois formatos. Devem ser registrados conclusão da tarefa, necessidade de ajuda e erros de interpretação. Essa avaliação ainda não foi executada e não é apresentada como resultado do levantamento.

## Base de conferência

BACKEND e FRONTEND na branch `homolog`, conferidos em 27/11/2026: backend no commit `d1f1aeb` e frontend no commit `10abe49`. Os diagramas de classes e de sequência foram revistos no mesmo dia, e a base foi reconferida sobre o backend `6e0d3f5` e o frontend `4f53a25`, o código final nas branches `main` e `homolog`; esses commits só mudam o `.env.example`, o cartão da votação, a função `entregarPdf` (que, no `db5b2fe`, deixou de renomear o PDF com `nomearPdf`), a presença, o desbloqueio na abertura e um estilo do cartão de "Suas organizações", nada que o relatório use. A versão atual do relatório entrou no backend `99b6b4f` e no frontend `a53acce`; nos commits seguintes, a única mudança que afeta o relatório é a do `JwtAuthGuard` (backend `543e03f`), que passou a recusar o token emitido antes da última troca de senha. No frontend, `pdf.ts` mudou só na função `entregarPdf`, que o relatório não usa, e `api.ts` ganhou só tipos da votação. Os arquivos principais são `BACKEND/src/relatorios/relatorios.controller.ts`, `relatorios.service.ts`, `relatorio-individual.dto.ts` e `relatorios.module.ts`; `FRONTEND/src/app/(abas)/relatorio.tsx`, `src/services/exportar-relatorio-individual.ts`, `src/services/pdf.ts` (só a função `pdfPelaImpressao`), `src/utils/relatorio-individual.ts` e `src/types/relatorio-individual.ts`. O registro de verificação da equipe está em `FRONTEND/tests/RELATORIO.md`.

As duas observações da avaliação do Incremento 3 (17/11/2026) sobre o relatório são atendidas no código: as organizações e comissões do usuário aparecem na tela, no PDF e no CSV (RF-REL-03); e o PDF do aplicativo instalado é compartilhado direto do arquivo gerado pela impressão, sem a chamada `FileSystemFile.move` apontada no erro da avaliação (RF-REL-04, CT-REL-18). No aparelho, a exportação do PDF foi confirmada num Galaxy S24 com Android 16 (CT-REL-38) e, em 27/11/2026, no mesmo aparelho, a troca do período e as exportações do PDF e do CSV (CT-REL-34, executado em parte: faltam salvar o arquivo e conferir o CSV numa planilha); no mesmo dia, a exportação do PDF foi aprovada também no emulador Pixel 6 com Android 15. Ainda em 27/11/2026, pelo Expo Go, o roteiro foi aprovado no emulador Pixel 6 com Android 15 (CT-REL-35) e no iPhone 14 com iOS 27.2.1 (CT-REL-36, executado em parte: o aplicativo instalado no iOS não foi executado, porque não há Mac), e a tela foi percorrida com o VoiceOver no iPhone (CT-REL-37, executado em parte: o TalkBack no Android não foi executado).
