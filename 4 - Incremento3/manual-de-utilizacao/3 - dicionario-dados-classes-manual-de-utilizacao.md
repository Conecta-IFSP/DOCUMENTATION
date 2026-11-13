# Dicionário de Dados - Classes do Manual de Utilização (R4)

O conteúdo da Ajuda é declarado no próprio aplicativo, em `src/constants/ajuda.ts`. Ele não é armazenado no banco de dados nem enviado ou recebido pela API. Para alterar uma pergunta, é preciso editar esse arquivo e publicar uma nova versão do aplicativo. Os tipos abaixo são os do TypeScript, como estão no código.

---

## 1. Tipo `TopicoAjuda`

Representa um assunto da tela Ajuda, como Conta e acesso ou Votação. Agrupa as perguntas sobre esse assunto e é exibido como um título com ícone, seguido da lista de perguntas.

| Campo | Tipo | Obrigatório | Descrição |
|---|---|---:|---|
| `id` | `string` | Sim | Identificador do tópico. É usado no endereço `/ajuda?topico=<id>`, que rola a tela até o tópico e abre a primeira pergunta dele. |
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
| `id` | `string` | Sim | Identificador da pergunta. Controla quais perguntas estão abertas na tela. |
| `pergunta` | `string` | Sim | Texto da pergunta exibido na sanfona. Também é considerado pela busca. |
| `resposta` | `string \| readonly string[]` | Sim | Resposta da pergunta. Um texto é exibido como parágrafo; uma lista é exibida como passos numerados. Também é considerada pela busca. |
| `perfil` | `PerfilAjuda` | Sim | Indica se a pergunta trata de uma ação exclusiva do responsável da comissão. |

### Regras relacionadas à pergunta

O `id` deve ser único em todo o arquivo, e não apenas dentro do tópico.

O padrão de escrita do arquivo limita a resposta em lista a 4 passos, com uma ação curta por passo, e a resposta em texto a 2 ou 3 frases. São orientações para quem escreve o conteúdo; o código não as verifica.

A resposta deve descrever apenas o que o aplicativo faz, com os nomes de botões e telas que aparecem para o usuário.

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

| Campo | Tipo | Descrição |
|---|---|---|
| `topico` | `string` (opcional) | Parâmetro do endereço que indica o tópico a abrir, por exemplo `conta`. |
| `busca` | `string` | Texto digitado no campo Buscar. Começa vazio. |
| `abertas` | `ReadonlySet<string>` | Identificadores das perguntas abertas. Começa vazio ou com a primeira pergunta do tópico recebido em `topico`. |
| `topicosVisiveis` | `readonly TopicoAjuda[]` | Tópicos exibidos: o resultado de `filtrarTopicosAjuda` aplicado a `TOPICOS_AJUDA` com o texto de `busca`. |
| `encontradas` | `number` | Quantidade de perguntas em `topicosVisiveis`, exibida durante a busca. |

O método `rolarParaTopico` rola a tela até o tópico recebido em `topico`, uma única vez por tópico. `alternar` abre ou fecha uma pergunta. `voltar` retorna à tela anterior ou, se não houver uma, abre o Início (com login) ou a tela Entrar (sem login).

### Classe `Sanfona`

Representa uma pergunta que se abre e se fecha (`src/components/Sanfona.tsx`). A tela Ajuda exibe uma sanfona para cada pergunta visível.

| Campo | Tipo | Obrigatório | Descrição |
|---|---|---:|---|
| `pergunta` | `string` | Sim | Texto exibido no cabeçalho da sanfona. |
| `resposta` | `string \| readonly string[]` | Sim | Texto corrido ou lista de passos, exibida numerada. |
| `aberta` | `boolean` | Não | Indica se a resposta está visível. A tela Ajuda sempre informa esse valor; sem ele, o próprio componente guarda o estado. |
| `abertaInicial` | `boolean` | Não | Estado inicial quando `aberta` não é informado. O valor padrão é `false`. |
| `aoAlternar` | `(aberta: boolean) => void` | Não | Função chamada quando o usuário toca na pergunta. |
| `selo` | `string \| null` | Não | Texto do selo, obtido de `SELO_PERFIL`. Com valor, o selo é exibido e o leitor de tela anuncia "Exclusivo do Responsável". |
| `primeira` | `boolean` | Não | Indica a primeira pergunta do tópico, que não tem linha divisória acima. O valor padrão é `false`. |

O método `alternar` inverte o estado e chama `aoAlternar`. A abertura e o fechamento são animados em 220 ms; quando o aparelho está com a opção de reduzir movimento ativada, a animação não é feita.

### Classes `CampoTexto`, `EstadoVazio` e `Botao`

Componentes compartilhados com outras telas. Na tela Ajuda, `CampoTexto` é o campo Buscar, com o botão "Limpar busca". `EstadoVazio` é a mensagem exibida quando a busca não encontra nenhuma pergunta, e `Botao` é o botão "Ver manual completo (PDF)".

### Módulo `buscaAjuda`

Representa o arquivo `src/utils/busca-ajuda.ts`, com as funções da busca da tela Ajuda.

| Função | Retorno | Descrição |
|---|---|---|
| `normalizarBusca(texto: string)` | `string` | Remove acentos e espaços nas pontas e converte para minúsculas, para que "Votação", "VOTACAO" e "votacao" sejam iguais. |
| `filtrarTopicosAjuda(topicos: readonly TopicoAjuda[], busca: string)` | `readonly TopicoAjuda[]` | Mantém só as perguntas que contêm todas as palavras digitadas no título do tópico, na pergunta ou na resposta, e retira os tópicos que ficarem sem perguntas. Com a busca vazia, devolve a própria lista recebida. Não altera a lista original. |

### Classe `Linking`

Classe da biblioteca React Native que entrega um endereço ao sistema operacional. O método `openURL` abre `URL_MANUAL_PDF` no navegador ou no leitor de PDF do aparelho. Se a abertura falhar, o erro é ignorado pela tela Ajuda.
