# Conecta+

O CONECTA+ é um aplicativo mobile voltado ao gerenciamento do ciclo de uma reunião. A proposta é reunir, em uma experiência única e simples, calendário, participantes, pauta, anotações, tarefas e histórico. O sistema não foi pensado para competir diretamente com plataformas de videoconferência ou suítes corporativas completas, mas para organizar o que acontece antes, durante e depois de uma reunião.

A ideia central é reduzir a dispersão de informações. Em uma situação comum, a data fica no calendário, o link da chamada em outra ferramenta, a pauta em uma mensagem, as anotações em um arquivo e as tarefas em um aplicativo separado. O CONECTA+ pretende concentrar esse fluxo em um único ambiente mobile, mantendo um escopo enxuto e adequado ao projeto acadêmico.

O CONECTA+ não busca superar as plataformas existentes em quantidade de recursos. Enquanto o aplicativo cuida da pauta, das anotações, das tarefas e do histórico, o Google Meet será utilizado para a realização das reuniões.

## Objetivo

Desenvolver um aplicativo mobile para Android e iOS que permita organizar reuniões e seus principais elementos em um só lugar, com foco em facilidade de uso, interface agradável, persistência dos dados e acompanhamento do histórico.

## Google Calendar

O Google Calendar tem como principal foco a organização da agenda. É uma ferramenta eficiente para visualizar compromissos e disponibilidade, oferecendo recursos como agendamento de reuniões e eventos, tarefas, compartilhamento de agenda, múltiplos calendários e páginas de agendamento.

O Calendar organiza muito bem quando uma reunião acontece, porém não foi desenvolvido para concentrar, como função principal, a pauta, as anotações, as tarefas decorrentes do encontro e todo o histórico da reunião.

**Pontos fortes:** eventos e tarefas no mesmo ambiente, compartilhamento de calendários entre equipes, controle de permissões e possibilidade de organizar e priorizar tarefas.

**Pontos fracos:** o acesso offline é limitado e o uso completo da ferramenta depende de conexão com a internet. Comparado a algumas outras soluções de calendário, também oferece menos possibilidades de personalização. Por utilizar a infraestrutura em nuvem do Google, informações sensíveis ou corporativas exigem atenção quanto à forma como são armazenadas e compartilhadas.

## Google Meet

O Google Meet é voltado principalmente à realização de chamadas e videoconferências. Entre suas funcionalidades básicas estão a criação e participação em reuniões, gerenciamento de participantes e recursos de comunicação durante os encontros.

**Pontos fortes:** possui uma interface direta e fácil de utilizar, além de boa integração com o ecossistema Google. Permite criar e acessar reuniões de forma simples e pode ser utilizado em diferentes contextos, como reuniões profissionais, acadêmicas ou pessoais.

**Pontos fracos:** seu foco principal está na videoconferência, e não na organização completa do ciclo da reunião. Recursos adicionais podem variar conforme o tipo de conta utilizada, e algumas funcionalidades possuem limitações relacionadas ao número de participantes ou ao plano contratado.

## Teams

O Microsoft Teams é uma plataforma que combina calendário, reuniões, chats, canais e colaboração. Também possui recursos relacionados a agendas, anotações e tarefas de reunião, além de permitir comunicação antes, durante e após os encontros.

**Pontos fortes:** forte integração com o ecossistema Microsoft 365 e centralização de chats, arquivos, reuniões e ferramentas de colaboração.

**Pontos fracos:** pode apresentar maior consumo de recursos do dispositivo e possui uma interface mais complexa para usuários que ainda não estão familiarizados com a plataforma.

## Identidade Visual

A definição do nome, ícone, logotipo, padrões de fontes e paleta de cores do CONECTA+ está documentada no arquivo abaixo.

📄 [Visualizar documentação de Identidade Visual](Topico3.pdf)

## Funcionalidades Previstas

- **Login:** login e cadastro de usuários.
- **Tela principal:** visão rápida das próximas reuniões e dos principais atalhos.
- **Calendário mensal:** visualização das reuniões distribuídas por data.
- **Reuniões:** cadastro, consulta e edição de título, data, horário, pauta e link gerado automaticamente para uma reunião no Google Meet.
- **Participantes:** associação de participantes às reuniões e envio da notificação com o link do encontro.
- **Anotações:** registro de observações durante ou após a reunião.
- **Tarefas:** registro de ações e atividades resultantes da reunião.
- **Histórico:** consulta às reuniões anteriores e aos seus registros.

### Navegações

| Área | Descrição |
| --- | --- |
| **Início** | Próximas reuniões, atalhos e informações relevantes. |
| **Calendário** | Visão mensal e acesso às reuniões por data. |
| **Reuniões** | Lista, cadastro, edição e detalhes das reuniões. |
| **Histórico** | Reuniões concluídas e registros associados. |

## Tecnologias para o servidor.

- **Node.js** - Responsável por executar o código do servidor fora do navegador, sendo requisito para o funcionamento de todo o backend.
- **NestJS** - Organiza o código em módulos, controllers e services, definindo com clareza a localização de cada rota e de cada regra do sistema.
- **Express** - Responsável pelo recebimento das requisições e pelo envio das respostas HTTP. O NestJS é construído sobre essa camada, aproveitando uma base já consolidada.
- **TypeScript / JavaScript** - Linguagem utilizada na construção das rotas, das regras de funcionamento do sistema e do acesso ao banco de dados.
- **MongoDB** – É um sistema de gerenciamento de banco de dados NoSQL projetado para oferecer flexibilidade, escalabilidade e desempenho para aplicações modernas.
- **REST + JSON** - O servidor disponibiliza rotas de acesso e retorna os dados em JSON.

## Tecnologias para o APP

- **React Native** - Linguagem principal para o desenvolvimento da aplicação, compatível com Android e iOS.
- **NativeWind** - Permite aplicar os estilos diretamente nos componentes por meio de classes, mantendo a padronização visual entre as telas.
- **Expo** - Disponibiliza o projeto React Native previamente configurado e permite executar e testar o aplicativo diretamente no dispositivo durante o desenvolvimento, sem necessidade de compilação manual.
- **TypeScript / JavaScript** - Linguagem utilizada na construção das telas, da navegação entre elas e das funções que se comunicam com o servidor.
- **Cliente HTTP** - Responsável por realizar as chamadas às rotas do servidor e receber as respostas que alimentam as telas do aplicativo.

## Ferramentas de apoio

- **Git** - Mantém o histórico de alterações do código ao longo do desenvolvimento.
- **Github** - Centraliza os arquivos e a documentação em um repositório remoto compartilhado pela equipe.
- **Figma** - Apoio para a elaboração das telas, do logo e do ícone antes da implementação, não constituindo dependência de execução do projeto.

## Arquitetura Proposta

A proposta adota uma arquitetura em camadas, separando a interface mobile, as regras da aplicação, a API e a persistência dos dados. Essa divisão facilita a manutenção, os testes e a evolução do projeto.

| Camada | Responsabilidade |
| --- | --- |
| **Interface** | Exibir informações, coletar dados do usuário e manter o padrão visual e de navegação. |
| **Aplicação mobile** | Organizar o estado das telas, realizar validações imediatas e executar chamadas para a API. |
| **API / regras** | Receber requisições, validar regras de negócio e controlar o acesso aos dados. |
| **Persistência** | Salvar e recuperar reuniões, participantes, pautas, anotações e tarefas. |
| **Banco** | Manter os dados armazenados e disponíveis entre diferentes execuções do sistema. |

**Ponto realista:** no protótipo acadêmico, o backend poderá ser executado localmente em um computador da equipe, sem a necessidade de contratação de hospedagem. Em um cenário de uso real pela internet, seria necessária uma infraestrutura de servidor, o que poderia gerar custos operacionais. Essa implantação externa não faz parte do escopo desta versão.

## Padrões para telas

- Título da tela no topo e ação principal claramente identificada.
- Campos agrupados por assunto, evitando formulários extensos sem divisão.
- Validação próxima ao campo que precisa de correção.
- Botão de salvar em posição previsível e com feedback após a operação.
- Confirmação antes de ações destrutivas, como exclusões.

## Nomenclaturas

| Elemento | Padrão | Exemplo |
| --- | --- | --- |
| **Variáveis e funções** | camelCase | nomeParticipante, salvarReuniao() |
| **Componentes e telas** | PascalCase | MeetingCard, CalendarScreen |
| **Constantes** | UPPER_SNAKE_CASE | MAX_PARTICIPANTES |
| **Rotas de API** | kebab-case ou substantivos consistentes | /reunioes, /participantes |
| **Tabelas e campos do banco** | snake_case | reuniao_participante, data_reuniao |

## Testes

Os testes devem acompanhar a evolução do sistema e gerar evidências dos resultados. Nesta etapa, a prioridade é garantir que os fluxos essenciais funcionem corretamente e que seus resultados possam ser apresentados de forma objetiva.

| ID | Cenário | Resultado esperado |
| --- | --- | --- |
| **CT01** | Cadastrar reunião | Reunião é salva e passa a aparecer nas consultas/calendário. |
| **CT02** | Editar reunião | Alterações persistem após salvar e reabrir. |
| **CT03** | Excluir reunião | Sistema solicita confirmação e remove apenas após confirmação. |
| **CT04** | Associar participantes | Participantes selecionados ficam vinculados à reunião correta. |
| **CT05** | Criar tarefa da reunião | Tarefa é registrada e permanece associada à reunião. |
| **CT06** | Validar campos | Campos inválidos exibem mensagens claras e impedem gravação incorreta. |
| **CT07** | Navegar pelo menu | Cada item direciona para a tela esperada. |
| **CT08** | Exibir calendário | Reuniões aparecem na data correspondente. |
| **CT09** | Persistir dados | Dados permanecem disponíveis após fechar e abrir novamente. |
| **CT10** | Responsividade | Layout permanece utilizável em diferentes tamanhos de tela suportados. |

## Requisitos não-funcionais

| Categoria | Requisitos |
| --- | --- |
| **Usabilidade** | O usuário deve conseguir criar uma reunião sem treinamento formal. O sistema deve manter padrão visual e navegação previsível. |
| **Desempenho** | A tela inicial deve abrir em aproximadamente dois segundos em um aparelho intermediário, e o salvamento de uma reunião não deve gerar atrasos perceptíveis ao usuário. |
| **Disponibilidade dos dados** | Os dados devem permanecer disponíveis após o aplicativo ser fechado. Deve existir uma estratégia de backup manual quando esse requisito for implementado. |
| **Segurança** | Ações destrutivas devem exigir confirmação. O acesso ao banco deve ocorrer por uma camada controlada, e dados sensíveis não devem ser expostos diretamente no código. |
| **Compatibilidade** | O aplicativo deve ser executável em Android e iOS, conforme a definição da disciplina. |
| **Manutenibilidade** | O código deve ser dividido em componentes e serviços, possuir nomenclatura padronizada e manter a documentação atualizada conforme a implementação de funcionalidades relevantes. |
| **Responsividade** | A interface deve se adaptar aos tamanhos de tela suportados sem perder legibilidade ou acesso às principais ações. |
| **Custo/licença** | O projeto deve priorizar tecnologias gratuitas e preservar a proposta de software de código aberto e sem custo de licença para o cliente. |

## Conclusão

O CONECTA+ apresenta-se como uma solução intencionalmente mais simples do que as grandes plataformas existentes. O valor do projeto não está em oferecer mais funcionalidades do que Google Calendar, Google Meet ou Microsoft Teams, mas em reunir as principais funções de gerenciamento de reuniões em um fluxo mobile coerente, com código aberto e escopo controlado.

A proposta técnica baseada em React Native, Expo, NativeWind, NestJS e MongoDB é compatível com um protótipo acadêmico multiplataforma e permite separar interface, regras e dados de forma organizada. Cada funcionalidade adicionada deve contribuir para resolver um problema real relacionado às reuniões e se integrar ao fluxo de antes, durante e depois do encontro, sem transformar o CONECTA+ em uma suíte genérica.
