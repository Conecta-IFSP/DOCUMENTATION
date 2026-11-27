# Dicionário relacional — Manual de utilização

> Padrão da disciplina · Módulo MAN · Campos do diagrama relacional · Revisado em 26/11/2026

## Resumo

Descreve cada campo das entidades do [diagrama relacional](4%20-%20diagrama-relacional-manual-de-utilizacao.md), com as sete informações do dicionário de dados do documento de exemplo da disciplina: campo, tipo de dado, se pode ser nulo, chave primária, chave estrangeira, outros e descrição. Nenhuma dessas entidades fica em banco de dados: elas descrevem o conteúdo fixo que vai dentro do aplicativo. Na coluna "Outros", cada restrição é explicada com a regra e quem a garante: o compilador TypeScript, o código da tela ou a revisão do arquivo por quem escreve o conteúdo.

## Como os identificadores são criados

| Identificador          | Quem cria                                                                | Formato                                                                                                                                                                      | Valor inicial                                   | Incremento                                                                                          | Unicidade                                                                                                       |
| ---------------------- | ------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------- | --------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------- |
| `TOPICO_AJUDA.id`      | A pessoa que escreve o tópico, à mão, em `src/constants/ajuda.ts`.       | Uma palavra em letras minúsculas, sem acento e sem espaço. Valores atuais: `conta`, `organizacoes`, `reunioes`, `presenca`, `ata`, `votacao` e `relatorio`.                  | Não há: não é número nem sequência.             | Não há. Um tópico novo recebe uma palavra nova, escolhida por quem escreve.                         | Única entre os tópicos. O código não confere; a conferência é feita na revisão do arquivo.                     |
| `PERGUNTA_AJUDA.id`    | A pessoa que escreve a pergunta, à mão, no mesmo arquivo.                | Palavras em letras minúsculas, sem acento, separadas por hífen: um prefixo de assunto e um resumo da pergunta, como `ata-aprovar` e `conta-esqueci-senha`.                  | Não há: não é número nem sequência.             | Não há. Uma pergunta nova recebe um texto novo.                                                     | Única no arquivo inteiro. O código não confere; em 26/11/2026 as 42 perguntas tinham 42 identificadores diferentes. |
| `posicao` (tópico e pergunta) | Não é escrita: é a posição do objeto na lista.                    | Inteiro.                                                                                                                                                                     | 0 para o primeiro tópico e para a primeira pergunta de cada tópico. | Sobe de 1 em 1, na ordem em que os objetos estão escritos no arquivo. | Única dentro da mesma lista.                                                                                    |
| `ARQUIVO_MANUAL.caminho` | A equipe, ao colocar o arquivo no projeto.                             | Caminho relativo ao FRONTEND: `assets/manual/manual.pdf`.                                                                                                                    | Não há.                                         | Não há. Uma nova versão do manual substitui o arquivo no mesmo caminho.                             | Único: o aplicativo tem um só manual.                                                                           |

O prefixo do identificador da pergunta nem sempre é o `id` do tópico em que ela está: três perguntas do tópico `presenca` têm o prefixo `reunioes-` (`reunioes-presenca`, `reunioes-confirmar-presenca` e `reunioes-lista-presenca`). Isso não afeta o funcionamento, porque o identificador só é comparado por igualdade.

## TOPICO_AJUDA

Um assunto da tela Ajuda. É cada objeto da lista `TOPICOS_AJUDA`.

| Campo     | Tipo de dado               | Pode ser nulo | Chave primária  | Chave estrangeira | Outros                                                                                                                                | Descrição                                                         |
| --------- | -------------------------- | ------------- | --------------- | ----------------- | ------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------- |
| `id`      | Texto                      | Não           | Sim (exclusiva) | Não               | Ver "Como os identificadores são criados". O valor `conta` é usado pelo link da tela Entrar e não pode mudar.                        | Identifica o tópico no endereço `/ajuda?topico=<id>`.             |
| `posicao` | Inteiro                    | Não           | Não             | Não               | Não é escrita; define a ordem de exibição, porque a tela mostra os tópicos na ordem da lista, sem ordenar por campo.                  | Ordem do tópico na tela.                                          |
| `titulo`  | Texto                      | Não           | Não             | Não               | Obrigatório pelo tipo `TopicoAjuda`, conferido pelo compilador. Entra na busca.                                                       | Nome do assunto exibido na tela.                                  |
| `icone`   | Texto (nome de ícone)      | Sim           | Não             | Não               | Opcional; o compilador só aceita nomes da biblioteca Ionicons.                                                                        | Ícone exibido ao lado do título.                                  |

Pela regra de escrita do arquivo, cada tópico tem no máximo 7 perguntas; hoje, de 4 a 7. Um tópico sem perguntas apareceria só com o título, por isso a revisão não aceita tópico vazio.

## PERGUNTA_AJUDA

Uma pergunta frequente e a sua resposta. É cada objeto da lista `perguntas` de um tópico.

| Campo       | Tipo de dado                          | Pode ser nulo | Chave primária  | Chave estrangeira       | Outros                                                                                                                                                         | Descrição                                       |
| ----------- | ------------------------------------- | ------------- | --------------- | ----------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------- |
| `id`        | Texto                                 | Não           | Sim (exclusiva) | Não                     | Ver "Como os identificadores são criados".                                                                                                                     | Controla quais perguntas estão abertas na tela. |
| `topico_id` | Texto                                 | Não           | Não             | Sim (`TOPICO_AJUDA.id`) | Não é escrito: a pergunta está dentro da lista do tópico, por isso pertence a exatamente um.                                                                    | Tópico da pergunta.                             |
| `posicao`   | Inteiro                               | Não           | Não             | Não                     | Não é escrita; define a ordem dentro do tópico.                                                                                                                | Ordem da pergunta no tópico.                    |
| `pergunta`  | Texto                                 | Não           | Não             | Não                     | Obrigatório pelo tipo. Entra na busca.                                                                                                                         | Texto do cabeçalho da sanfona.                  |
| `resposta`  | Texto ou lista de textos              | Não           | Não             | Não                     | O compilador aceita os dois formatos; a tela mostra o texto como parágrafo e a lista como passos numerados. Pela regra de escrita, até 4 passos ou até 3 frases, o que vale para todas as respostas em `8fc0a5a`. | Resposta mostrada ao abrir a pergunta.          |
| `perfil`    | Texto com valores fixos               | Não           | Não             | Não                     | O compilador só aceita `responsavel` ou `todos`. Com `responsavel`, a tela mostra o selo "Responsável"; o perfil não esconde a pergunta de ninguém.             | Público a que a pergunta se refere.             |

## ARQUIVO_MANUAL

O manual completo em PDF que vai dentro do aplicativo.

| Campo            | Tipo de dado     | Pode ser nulo | Chave primária  | Chave estrangeira | Outros                                                                                                                       | Descrição                                     |
| ---------------- | ---------------- | ------------- | --------------- | ----------------- | ---------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------- |
| `caminho`        | Texto            | Não           | Sim (exclusiva) | Não               | Ver "Como os identificadores são criados". O componente `ManualPdf` carrega o arquivo por esse caminho.                     | Local do arquivo no projeto.                  |
| `formato`        | Texto            | Não           | Não             | Não               | `application/pdf`. O empacotador trata a extensão `pdf` como recurso, por causa de `metro.config.js`.                       | Tipo do arquivo.                              |
| `paginas`        | Inteiro          | Não           | Não             | Não               | 27, conferido em 26/11/2026.                                                                                                 | Quantidade de páginas.                        |
| `tamanho_bytes`  | Inteiro          | Não           | Não             | Não               | 1.220.899, conferido em 26/11/2026.                                                                                          | Tamanho do arquivo.                           |
| `tamanho_pagina` | Texto            | Não           | Não             | Não               | A4: 594,96 × 841,92 pontos em todas as páginas, conferido em 26/11/2026.                                                     | Formato do papel.                             |
| `versao`         | Texto            | Não           | Não             | Não               | "Versão 1.0", impressa na capa e no cabeçalho das outras 26 páginas.                                                                          | Versão do manual.                             |

## Rastreabilidade

RF-MAN-01 (`TOPICO_AJUDA` e `PERGUNTA_AJUDA`) e RF-MAN-02 (`ARQUIVO_MANUAL`); [Diagrama relacional](4%20-%20diagrama-relacional-manual-de-utilizacao.md) e [Dicionário de classes](3%20-%20dicionario-dados-classes-manual-de-utilizacao.md).
