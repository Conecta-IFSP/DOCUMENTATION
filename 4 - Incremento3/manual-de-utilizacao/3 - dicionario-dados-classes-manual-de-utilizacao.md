# Dicionário de classes — Manual de utilização

> Padrão da disciplina · Módulo MAN · Revisado em 13/11/2026

## Resumo

Descreve cada classe do [Diagrama de classes](3%20-%20diagrama-classes-manual-de-utilizacao.md): atributos, tipos, obrigatoriedade, regras e métodos. O conteúdo da Ajuda é declarado no próprio aplicativo, em `src/constants/ajuda.ts`. Ele não é armazenado no banco de dados nem enviado ou recebido pela API. Para alterar uma pergunta, é preciso editar esse arquivo e publicar uma nova versão do aplicativo. Os tipos abaixo são os do TypeScript, como estão no código.

---

## 1. Tipo `TopicoAjuda`

Representa um assunto da tela Ajuda, como Conta e acesso ou Votação. Agrupa as perguntas sobre esse assunto e é exibido como um título com ícone, seguido da lista de perguntas.

| Campo | Tipo | Obrigatório | Descrição |
|---|---|---:|---|
| `id` | `string` | Sim | Identificador do tópico. É usado no endereço `/ajuda?topico=<id>`, que rola a tela até o tópico e abre a primeira pergunta dele. Como é criado: ver "Como os identificadores são criados". |
| `titulo` | `string` | Sim | Nome do assunto exibido na tela. Também é considerado pela busca. |
| `icone` | `IoniconsIconName` | Não | Nome de um ícone da biblioteca Ionicons, exibido ao lado do título. Sem ele, o título aparece sozinho. |
| `perguntas` | `readonly PerguntaAjuda[]` | Sim | Perguntas do tópico, na ordem em que são exibidas. |

### Regras relacionadas ao tópico

O `id` deve usar letras minúsculas, sem acento nem espaço.

O `id` do tópico `conta` não pode ser alterado: o link "Precisa de ajuda?" da tela Entrar abre a Ajuda com `topico=conta`.

Os tópicos são exibidos na ordem da lista `TOPICOS_AJUDA`.

O padrão de escrita do arquivo limita cada tópico a 7 perguntas. Esse limite é uma orientação para quem escreve o conteúdo; o código não o verifica.

Durante uma busca, o tópico que não tem nenhuma pergunta encontrada deixa de ser exibido.

---

## 2. Tipo `PerguntaAjuda`

Representa uma pergunta frequente e a sua resposta. Existe apenas dentro da lista `perguntas` de um `TopicoAjuda`.

| Campo | Tipo | Obrigatório | Descrição |
|---|---|---:|---|
| `id` | `string` | Sim | Identificador da pergunta. Controla quais perguntas estão abertas na tela. Como é criado: ver "Como os identificadores são criados". |
| `pergunta` | `string` | Sim | Texto da pergunta exibido na sanfona. Também é considerado pela busca. |
| `resposta` | `string \| readonly string[]` | Sim | Resposta da pergunta. Um texto é exibido como parágrafo; uma lista é exibida como passos numerados. Também é considerada pela busca. |
| `perfil` | `PerfilAjuda` | Sim | Indica se a pergunta trata de uma ação exclusiva do responsável da comissão. |

### Regras relacionadas à pergunta

O `id` deve ser único em todo o arquivo, e não apenas dentro do tópico.

O padrão de escrita do arquivo limita a resposta em lista a 4 passos, com uma ação curta por passo, e a resposta em texto a 2 ou 3 frases. São orientações para quem escreve o conteúdo; o código não as verifica.

A resposta deve descrever apenas o que o aplicativo faz, com os nomes de botões e telas que aparecem para o usuário.

---

## Como os identificadores são criados

Os dois identificadores deste módulo são textos fixos, escritos à mão no arquivo `src/constants/ajuda.ts` por quem redige o conteúdo. Nenhum deles é gerado pelo aplicativo, pelo servidor ou pelo banco.

| Identificador | Quem cria | Formato | Valor inicial | Incremento | Unicidade |
|---|---|---|---|---|---|
| `TopicoAjuda.id` | A pessoa que escreve o tópico, no próprio arquivo. | Uma palavra em letras minúsculas, sem acento e sem espaço. Valores atuais: `conta`, `organizacoes`, `reunioes`, `presenca`, `ata`, `votacao` e `relatorio`. | Não há: não é um número nem uma sequência. | Não há. Um tópico novo recebe uma palavra nova, escolhida por quem escreve. | Único entre os tópicos. O código não verifica; a conferência é feita na revisão do arquivo. |
| `PerguntaAjuda.id` | A pessoa que escreve a pergunta, no próprio arquivo. | Palavras em letras minúsculas, sem acento, separadas por hífen: um prefixo de assunto e um resumo da pergunta, como `ata-aprovar` e `conta-esqueci-senha`. | Não há: não é um número nem uma sequência. | Não há. Uma pergunta nova recebe um texto novo. | Único no arquivo inteiro. O código não verifica; em 13/11/2026 as 42 perguntas tinham 42 identificadores diferentes. |

O prefixo do identificador da pergunta nem sempre é o `id` do tópico em que ela está hoje: três perguntas do tópico `presenca` têm o prefixo `reunioes-` (`reunioes-presenca`, `reunioes-confirmar-presenca` e `reunioes-lista-presenca`). Isso não afeta o funcionamento, porque o identificador só é comparado por igualdade.

---

## 3. Enumeração `PerfilAjuda`

Indica a quem se aplica uma pergunta. É declarada como o tipo `'responsavel' | 'todos'`.

| Valor | Descrição |
|---|---|
| `responsavel` | A pergunta trata de uma ação exclusiva do responsável da comissão. A pergunta exibe o selo "Responsável". |
| `todos` | A pergunta vale para qualquer usuário. Nenhum selo é exibido. |

### Regras relacionadas ao perfil

O perfil não restringe quem vê a pergunta. Todas as perguntas são exibidas para todos os usuários, com ou sem login. O selo apenas informa.

---

## 4. Módulo `ajuda`

Representa o arquivo `src/constants/ajuda.ts`, que declara os tipos acima e as constantes lidas pela tela Ajuda.

| Constante | Tipo | Descrição |
|---|---|---|
| `TOPICOS_AJUDA` | `readonly TopicoAjuda[]` | Todo o conteúdo da tela Ajuda, na ordem de exibição. |
| `SELO_PERFIL` | `Record<PerfilAjuda, string \| null>` | Texto do selo de cada perfil: `responsavel` → `'Responsável'`; `todos` → `null`, que significa nenhum selo. |
| `URL_MANUAL_PDF` | `string` | Endereço do manual completo em PDF, publicado na pasta `4 - Incremento3/manual-de-utilizacao/manual-usuario/` do repositório `Conecta-IFSP/DOCUMENTATION`, branch `main`. Quando vazio, o botão "Ver manual completo (PDF)" não é exibido. |

### Conteúdo atual de `TOPICOS_AJUDA`

| `id` | `titulo` | `icone` | Perguntas | Com perfil `responsavel` | Respostas em lista |
|---|---|---|---:|---:|---:|
| `conta` | Conta e acesso | `person-circle-outline` | 4 | 0 | 2 |
| `organizacoes` | Organizações e comissões | `business-outline` | 6 | 0 | 3 |
| `reunioes` | Reuniões | `calendar-outline` | 6 | 3 | 3 |
| `presenca` | Presença | `checkmark-done-outline` | 6 | 3 | 6 |
| `ata` | Ata | `document-text-outline` | 7 | 2 | 3 |
| `votacao` | Votação | `checkbox-outline` | 7 | 1 | 2 |
| `relatorio` | Relatório individual | `bar-chart-outline` | 6 | 0 | 2 |
| **Total** | | | **42** | **9** | **21** |

### Regras relacionadas ao manual

Para atualizar o manual sem publicar uma nova versão do aplicativo, o novo PDF deve substituir o anterior no mesmo caminho e com o mesmo nome de arquivo, na branch `main`; assim o endereço não muda.

---

## 5. Classes do Aplicativo

### Classe `TelaAjuda`

Representa a tela Ajuda (`src/app/ajuda.tsx`), que pode ser aberta com ou sem login. Os campos abaixo são o estado da tela. Todos são descartados quando o usuário sai dela.

| Campo | Tipo | Obrigatório | Descrição |
|---|---|---:|---|
| `topico` | `string` | Não | Parâmetro do endereço que indica o tópico a abrir, por exemplo `conta`. |
| `busca` | `string` | Sim | Texto digitado no campo Buscar. Começa vazio. |
| `abertas` | `ReadonlySet<string>` | Sim | Identificadores das perguntas abertas. Começa vazio ou com a primeira pergunta do tópico recebido em `topico`. |
| `topicosVisiveis` | `readonly TopicoAjuda[]` | Sim | Tópicos exibidos: o resultado de `filtrarTopicosAjuda` aplicado a `TOPICOS_AJUDA` com o texto de `busca`. |
| `encontradas` | `number` | Sim | Quantidade de perguntas em `topicosVisiveis`, exibida durante a busca. |

| Método | Retorno | Descrição |
|---|---|---|
| `TelaAjuda()` | `JSX.Element` | Função do componente: monta a tela com o título, o campo Buscar, os tópicos e o botão do manual. |
| `primeiraPerguntaDo(topicoId: string \| undefined)` | `string \| undefined` | Procura em `TOPICOS_AJUDA` o tópico cujo `id` é igual a `topicoId` e devolve o `id` da primeira pergunta dele. Devolve `undefined` quando não há tópico com esse `id`. É declarada no mesmo arquivo, fora do componente. |
| `definirBusca(texto: string)` | `void` | Guarda em `busca` o texto digitado. É chamada pelo `onChangeText` do campo Buscar e, com texto vazio, pelo ícone "Limpar busca". |
| `rolarParaTopico()` | `void` | Rola a tela até o tópico recebido em `topico`, uma única vez por tópico. |
| `alternar(perguntaId: string, aberta: boolean)` | `void` | Com `aberta` igual a `true`, acrescenta `perguntaId` ao conjunto `abertas`; com `false`, retira. |
| `voltar()` | `void` | Retorna à tela anterior ou, se não houver uma, abre o Início (com login) ou a tela Entrar (sem login). |

### Classe `Sanfona`

Representa uma pergunta que se abre e se fecha (`src/components/Sanfona.tsx`). A tela Ajuda exibe uma sanfona para cada pergunta visível.

| Campo | Tipo | Obrigatório | Descrição |
|---|---|---:|---|
| `pergunta` | `string` | Sim | Texto exibido no cabeçalho da sanfona. |
| `resposta` | `string \| readonly string[]` | Sim | Texto corrido ou lista de passos, exibida numerada. |
| `aberta` | `boolean` | Não | Indica se a resposta está visível. A tela Ajuda sempre informa esse valor; sem ele, o próprio componente guarda o estado. |
| `abertaInicial` | `boolean` | Não | Estado inicial quando `aberta` não é informado. O valor padrão é `false`. |
| `aoAlternar` | `(aberta: boolean) => void` | Não | Função chamada quando o usuário toca na pergunta. No diagrama aparece como o método `aoAlternar(aberta: boolean)`. |
| `selo` | `string \| null` | Não | Texto do selo, obtido de `SELO_PERFIL`. Com valor, o selo é exibido e o leitor de tela anuncia "Exclusivo do Responsável". |
| `primeira` | `boolean` | Não | Indica a primeira pergunta do tópico, que não tem linha divisória acima. O valor padrão é `false`. |

O método `alternar` inverte o estado e chama `aoAlternar`. A abertura e o fechamento são animados em 220 ms; quando o aparelho está com a opção de reduzir movimento ativada, a animação não é feita.

### Classes `CampoTexto`, `EstadoVazio` e `Botao`

Componentes compartilhados com outras telas. As tabelas mostram só as propriedades usadas pela tela Ajuda, com o valor que ela informa.

| Classe | Campo | Tipo | Obrigatório | Valor na tela Ajuda |
|---|---|---|---:|---|
| `CampoTexto` | `rotulo` | `string` | Sim | "Buscar" |
| `CampoTexto` | `icone` | `IoniconsIconName` | Não | `search-outline` |
| `CampoTexto` | `compacto` | `boolean` | Não | `true` |
| `CampoTexto` | `value` | `string` | Não | O estado `busca`. |
| `CampoTexto` | `onChangeText` | `(texto: string) => void` | Não | `definirBusca`. |
| `EstadoVazio` | `icone` | `IoniconsIconName` | Sim | `search-outline` |
| `EstadoVazio` | `mensagem` | `string` | Sim | "Tente outra palavra, como o nome da tela ou do botão. Se não encontrar, consulte o manual completo." |
| `Botao` | `titulo` | `string` | Sim | "Ver manual completo (PDF)" |
| `Botao` | `icone` | `IoniconsIconName` | Não | `document-outline` |
| `Botao` | `variante` | `'primario' \| 'secundario' \| 'perigo'` | Não | `secundario` |
| `Botao` | `pilula` | `boolean` | Não | `true` |
| `Botao` | `aoTocar` | `() => void` | Sim | Função que chama `Linking.openURL(URL_MANUAL_PDF)` e ignora o erro. |

O campo Buscar também recebe a ação "Limpar busca", exibida no fim do campo só quando há texto.

### Módulo `buscaAjuda`

Representa o arquivo `src/utils/busca-ajuda.ts`, com as funções da busca da tela Ajuda.

| Função | Retorno | Descrição |
|---|---|---|
| `normalizarBusca(texto: string)` | `string` | Remove acentos e espaços nas pontas e converte para minúsculas, para que "Votação", "VOTACAO" e "votacao" sejam iguais. |
| `filtrarTopicosAjuda(topicos: readonly TopicoAjuda[], busca: string)` | `readonly TopicoAjuda[]` | Mantém só as perguntas que contêm todas as palavras digitadas no título do tópico, na pergunta ou na resposta, e retira os tópicos que ficarem sem perguntas. A comparação é por trecho (`includes`), não por palavra inteira: "ata" também encontra "data". Com a busca vazia, devolve a própria lista recebida. Não altera a lista original. |

### Classe `Linking`

Classe da biblioteca React Native que entrega um endereço ao sistema operacional.

| Método | Retorno | Descrição |
|---|---|---|
| `openURL(url: string)` | `Promise<any>` | Pede ao sistema operacional que abra o endereço. Na tela Ajuda, recebe `URL_MANUAL_PDF`, que é aberto no navegador padrão do aparelho. A `Promise` é rejeitada quando nenhum aplicativo abre o endereço; a tela Ajuda ignora esse erro. |

## Rastreabilidade

`TopicoAjuda`, `PerguntaAjuda`, `PerfilAjuda`, `TelaAjuda`, `Sanfona`, `CampoTexto`, `EstadoVazio` e `buscaAjuda` atendem ao RF-MAN-01. `URL_MANUAL_PDF`, `Botao` e `Linking` atendem ao RF-MAN-02.
