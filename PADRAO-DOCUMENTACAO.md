# Padrão da documentação

## 1. Finalidade

Este arquivo define o formato obrigatório dos documentos do Conecta+. O objetivo é permitir que um leitor identifique, sem consultar o código, **quem executa uma função, em quais condições, quais dados entram, onde são processados, o que é persistido e quais erros podem ocorrer**.

## 2. Nomenclatura

| Elemento                | Padrão            | Exemplo                |
| ----------------------- | ----------------- | ---------------------- |
| Caso de uso             | `UC-<MÓDULO>-NN`  | `UC-USR-01`            |
| Requisito funcional     | `RF-<MÓDULO>-NN`  | `RF-ORG-03`            |
| Requisito não funcional | `RNF-<MÓDULO>-NN` | `RNF-COM-04`           |
| Caso de teste           | `CT-<MÓDULO>-NN`  | `CT-REU-12`            |
| Diagrama de sequência   | `SD-<MÓDULO>-NN`  | `SD-CFG-02`            |
| Campos da API e banco   | `snake_case`      | `organizacao_id`       |
| Classes, DTOs e enums   | `PascalCase`      | `CreateOrganizacaoDto` |
| Métodos TypeScript      | `camelCase`       | `buscarPorId()`        |

Siglas dos módulos: `USR` cadastro/autenticação; `ORG` organizações e vínculos; `COM` comissões; `CFG` configurações de usuário; `SIS` informações do sistema; `REU` reuniões, convites e presença; `ATA` atas de reunião; `MAN` manual de utilização; `VOT` votação na reunião.

## 3. Estrutura de cada módulo

O modelo é o documento de requisitos de exemplo da disciplina (material "Aula 1 - Revisão", slide "Exemplo de documento de requisitos"). Cada pasta de módulo possui dez documentos, com o nome do módulo em `kebab-case`:

```text
README.md
1 - casos-uso-<modulo>.md
2 - requisitos-funcionais-<modulo>.md
3 - diagrama-classes-<modulo>.md
3 - dicionario-dados-classes-<modulo>.md
4 - diagrama-relacional-<modulo>.md
4 - dicionario-dados-relacional-<modulo>.md
5 - requisitos-nao-funcionais-<modulo>.md
6 - diagramas-sequencia-<modulo>.md
7 - testes-<modulo>.md
```

| Ordem | Documento                 | Conteúdo obrigatório                                                                      |
| ----: | ------------------------- | ----------------------------------------------------------------------------------------- |
|     — | `README.md`               | resumo, escopo, atores, índice e rastreabilidade                                          |
|     1 | casos de uso              | diagrama de casos de uso em UML, papel de cada ator e requisitos funcionais de usuário    |
|     2 | requisitos funcionais     | cada caso de uso como requisito funcional de sistema, apenas no formulário padrão         |
|     3 | diagrama de classes       | classes por camada, com tipos nos atributos e nos métodos, e ligações com multiplicidade  |
|     3 | dicionário de classes     | atributos, tipos, obrigatoriedade, regras e métodos                                       |
|     4 | diagrama relacional       | visão lógica das coleções e dos dados embutidos do MongoDB                                |
|     4 | dicionário relacional     | campo, tipo de dado, pode ser nulo, chave primária, chave estrangeira, outros e descrição |
|     5 | requisitos não funcionais | requisitos de usuário e de sistema, com classificação, medida e forma de verificação      |
|     6 | diagramas de sequência    | um diagrama por requisito funcional, com mensagens numeradas                              |
|     7 | testes                    | ambiente, comando, cenário, resultado esperado e resultado obtido                         |

Os módulos **REU** (Incremento 2), **ATA**, **MAN** e **VOT** (Incremento 3) já seguem esta versão do padrão, revisada em 10/11/2026. Os cinco módulos do Incremento 1 ainda estão na versão anterior, com fluxograma no lugar do diagrama de casos de uso e tabelas próprias de caso de uso.

## 4. Casos de uso e requisitos de usuário

O requisito de **usuário** é uma descrição breve; o de **sistema** é a descrição completa. O documento de casos de uso traz só o nível de usuário:

1. **Diagrama de casos de uso** em UML: atores (bonecos), casos de uso (elipses com verbo e substantivo) dentro da fronteira do sistema, e uma linha de associação entre o ator e cada caso de uso, rotulada com o código do requisito funcional. Generalização entre atores e `«extend»` ou `«include»` entre casos de uso entram quando existem. O diagrama é um arquivo SVG, porque o Mermaid não desenha esse tipo de diagrama; fluxograma não é diagrama de casos de uso.
2. **Atores:** um parágrafo por ator, dizendo quem ele é no sistema e o que pode e o que não pode fazer.
3. **Requisitos funcionais de usuário:** uma frase por requisito, no formato "Requisito RF-XXX-NN — Nome. O sistema deve permitir a … fazer …".

O texto é escrito para quem não conhece o código: sem código HTTP, sem nome de rota e sem jargão. O detalhe técnico fica no requisito de sistema.

## 5. Requisitos funcionais de sistema

Cada caso de uso é descrito **apenas** no formulário padrão, uma tabela cujo cabeçalho é o código e o nome do requisito:

| Campo              | O que escrever                                                                                                                                                                                 |
| ------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Função             | Uma frase com o que o requisito faz.                                                                                                                                                           |
| Descrição          | O que acontece com os dados: qual coleção ou campo é lido ou gravado.                                                                                                                          |
| Entradas           | Cada dado de entrada com tipo, tamanho e formato, por exemplo "texto de 3 a 120 caracteres".                                                                                                   |
| Origem             | De onde vêm as entradas: o ator, o banco de dados ou os dois.                                                                                                                                  |
| Saída              | O que o requisito devolve, com os campos relevantes.                                                                                                                                           |
| Destino            | Para onde vai a saída: a tela e o campo ou coleção gravados.                                                                                                                                   |
| Ação               | O passo a passo completo: onde o usuário toca, o que a tela mostra, com quais controles e valores iniciais, quais validações são feitas e como, e quais funções de quais classes são chamadas. |
| Pré-condição       | O que precisa ser verdade antes.                                                                                                                                                               |
| Pós-condição       | O que o sistema mostra no sucesso e em cada erro.                                                                                                                                              |
| Efeitos colaterais | O que muda além da saída, ou "Nenhum".                                                                                                                                                         |

No campo **Ação** não basta dizer "valida", "filtra" ou "ordena". É preciso dizer **como**: qual campo é comparado com qual, se a comparação é por igualdade ou por intervalo, por qual campo e em qual sentido a lista é ordenada, e qual método faz isso.

Melhorias ainda não implementadas são marcadas como **não implementadas**, nunca como requisito atendido.

## 6. Requisitos não funcionais

Também em dois níveis. O de **usuário** é uma frase com a qualidade esperada. O de **sistema** precisa ser de fácil verificação e traz:

| Campo         | Pergunta respondida                                                                                      |
| ------------- | -------------------------------------------------------------------------------------------------------- |
| Classificação | Em qual ramo da árvore de Sommerville o requisito está: produto, organizacional ou externo, e o subtipo. |
| Critério      | Qual propriedade de qualidade é exigida?                                                                 |
| Medida        | Qual limite ou condição torna o requisito aprovado?                                                      |
| Condição      | Em qual ambiente, carga ou situação?                                                                     |
| Verificação   | Qual teste, inspeção ou ferramenta comprova?                                                             |
| Evidência     | Onde a implementação correspondente está localizada?                                                     |
| Situação      | Atendido, com data, ou o que falta medir.                                                                |

Evitam-se expressões vagas como "rápido", "adequado" ou "seguro". Quando ainda não existe medição, o documento diz isso, sem afirmar conformidade.

## 7. Regras para diagramas

### 7.1 Classes

- As classes são agrupadas em pacotes, um por camada: **Aplicativo**, **API** e **Dados**.
- Todo atributo tem tipo. Todo método tem o tipo de cada parâmetro e o tipo do retorno.
- Só entram classes e métodos que existem no código; rotas HTTP ficam em tabela à parte, não como nome de método.
- "Frontend", "Backend", "Banco" e "JSON" não são classes.
- **Composição** (losango cheio) para dados gravados dentro de outro documento; **associação** (seta cheia) para referência por identificador a um documento de outra coleção; **herança** (triângulo vazio) entre classes; **dependência** (seta tracejada) para uso em uma chamada. Toda composição e associação leva multiplicidade nas duas pontas.

### 7.2 Modelo relacional e dicionário de dados

O banco real é MongoDB. O diagrama relacional é uma **visão lógica**: dados embutidos aparecem como entidades para mostrar chaves e cardinalidades, e o texto diz onde cada entidade fica de verdade.

O dicionário descreve cada campo com as sete informações do exemplo da disciplina: campo, tipo de dado, pode ser nulo, chave primária (exclusiva ou inclusiva), chave estrangeira (com a entidade e o campo de destino), outros e descrição. Toda entidade do diagrama tem a sua tabela no dicionário.

Para cada identificador, o dicionário diz **quem o cria, em que formato, com qual valor inicial e com qual incremento**. "Gerado pelo MongoDB" não é suficiente.

### 7.3 Sequência

- Um diagrama para cada requisito funcional, com o mesmo número.
- As linhas de vida são classes ou módulos do código: tela, `api`, controller, service e model.
- Toda mensagem é a chamada de um método existente, com seus parâmetros, por exemplo `criar(dto, usuarioId)` ou `findById(id)`. A ação do usuário também é a chamada do método da tela que ele dispara.
- As mensagens são numeradas (`autonumber`), e os retornos voltam por toda a cadeia, na ordem inversa.
- Uma regra que não é um método, como comparar dois horários, vai em uma nota, não em uma mensagem. A rota HTTP também vai em nota.
- O Mermaid quebra com ponto e vírgula dentro de mensagem; os argumentos são separados por vírgula.

## 8. Segurança e transparência

- A chave `X-API-Key` identifica o aplicativo, mas, por estar em variável `EXPO_PUBLIC_*`, não é um segredo forte: ela pode ser extraída do bundle. A autorização de usuário continua sendo responsabilidade do JWT e dos guards.
- Senhas, hashes, tokens, códigos e credenciais SMTP nunca são copiados para a documentação.
- Arquivos `.env` reais não são documentados nem versionados; somente nomes e finalidade das variáveis são descritos.
- Limitações e falhas conhecidas devem ser registradas de maneira explícita.

## 9. Controle de revisão

Toda revisão deve registrar os commits do frontend e backend usados como fonte, executar os testes possíveis e não transformar resultado não executado em “aprovado”. O arquivo de teste informa data, ambiente e comando utilizado.

## 10. Elementos comuns a todos os artefatos

Todo documento técnico possui:

- um único título de nível 1 no formato `<Artefato> — <Módulo>`;
- identificação `Padrão da disciplina · Módulo <SIGLA>` e a data da revisão;
- seção `Resumo` antes do conteúdo técnico;
- seção de rastreabilidade ou referência rastreável no conteúdo;
- linguagem no presente para o que está implementado;
- marcação explícita de qualquer medição ou teste ainda pendente.

O `README.md` de cada módulo usa sempre as seções `Resumo`, `Escopo`, `Índice padronizado`, `Rastreabilidade` e `Base de conferência`.
