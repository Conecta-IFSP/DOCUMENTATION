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

Siglas dos módulos: `USR` cadastro/autenticação; `ORG` organizações e vínculos; `COM` comissões; `CFG` configurações de usuário; `SIS` informações do sistema; `REU` reuniões, presença e ata.

## 3. Estrutura de cada módulo

Cada pasta do Incremento 1 possui exatamente dez documentos técnicos. O nome do módulo no arquivo é escrito em `kebab-case`:

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

| Ordem | Documento                 | Conteúdo obrigatório                                                                               |
| ----: | ------------------------- | -------------------------------------------------------------------------------------------------- |
|     — | `README.md`               | objetivo, escopo, atores, arquivos e matriz de rastreabilidade                                     |
|     1 | casos de uso              | ator, objetivo, gatilho, pré-condições, fluxo principal, alternativas, pós-condições e regras      |
|     2 | requisitos funcionais     | entrada, rota, autenticação, controller/service, coleção, consulta/gravação, resposta e erros      |
|     3 | diagrama de classes       | classes reais ou abstrações explicitamente identificadas; atributos, métodos existentes e relações |
|     3 | dicionário de classes     | finalidade, atributos, tipos, obrigatoriedade, restrições e métodos                                |
|     4 | diagrama relacional       | visão lógica das coleções e documentos embutidos do MongoDB                                        |
|     4 | dicionário relacional     | coleção, caminho do campo, tipo BSON/lógico, restrições, padrão, índice e significado              |
|     5 | requisitos não funcionais | critério mensurável, forma de verificação e evidência no código/teste                              |
|     6 | diagramas de sequência    | participantes que correspondem a componentes reais e chamadas com métodos existentes               |
|     7 | testes                    | pré-condições, entrada, passos, resultado esperado, resultado obtido e requisito coberto           |

## 4. Regras para casos de uso

Um caso de uso representa uma meta observável de um ator, e não uma tela ou uma sequência genérica de cliques. Cada descrição deve informar:

- **ator principal** e, quando existir, ator secundário;
- **gatilho** que inicia o fluxo;
- **pré-condições** verificáveis;
- **fluxo principal** numerado;
- **fluxos alternativos e exceções** com o código HTTP relevante;
- **pós-condições** persistidas e efeitos colaterais;
- referência ao requisito funcional e aos testes correspondentes.

Todos os documentos de casos de uso seguem, nesta ordem:

1. título `Casos de uso — <Módulo>`;
2. identificação do padrão e da fonte;
3. `Resumo` do escopo;
4. `Diagrama geral`;
5. `Resumo dos casos de uso`, com ID, nome, ator e resultado;
6. `Especificações`, com uma subseção por caso.

Cada especificação usa a tabela abaixo. `Atores secundários` é incluído somente quando existir.

| Campo                   | Conteúdo obrigatório                             |
| ----------------------- | ------------------------------------------------ |
| Resumo                  | Meta observável do ator.                         |
| Ator principal          | Papel que inicia o caso.                         |
| Atores secundários      | Serviço ou papel externo relevante.              |
| Gatilho                 | Evento que inicia o fluxo.                       |
| Pré-condições           | Estados verificáveis exigidos antes da execução. |
| Fluxo principal         | Passos numerados do caminho de sucesso.          |
| Alternativas e exceções | Desvios, erros e códigos HTTP relevantes.        |
| Pós-condições           | Dados e estados após a conclusão.                |
| Rastreabilidade         | RF, SD e CT relacionados.                        |

## 5. Regras para requisitos funcionais

Os requisitos são escritos no presente e descrevem o sistema implementado. Não se usa apenas “a API busca os dados”. Deve-se informar, quando aplicável:

- método e endereço HTTP;
- cabeçalhos `X-API-Key` e `Authorization`;
- DTO e validações;
- controller e método de serviço;
- coleção e campos consultados/alterados;
- filtros, ordenação e projeção;
- corpo e formato da resposta;
- códigos de erro e condições que os produzem;
- tela que inicia a operação e atualização da interface.

Melhorias ainda não implementadas devem aparecer como **pendência**, não como requisito atendido.

## 6. Regras para requisitos não funcionais

Um RNF precisa ser verificável. Evitam-se expressões vagas como “rápido”, “adequado”, “seguro” ou “alguns dados”. Cada item contém:

| Campo       | Pergunta respondida                                  |
| ----------- | ---------------------------------------------------- |
| Critério    | Qual propriedade de qualidade é exigida?             |
| Medida      | Qual limite ou condição torna o requisito aprovado?  |
| Condição    | Em qual ambiente, carga ou situação?                 |
| Verificação | Qual teste, inspeção ou ferramenta comprova?         |
| Evidência   | Onde a implementação correspondente está localizada? |

Quando ainda não existe medição, o documento declara “critério proposto; medição pendente”, sem afirmar conformidade.

## 7. Regras para diagramas

### 7.1 Classes

- Entidades persistentes usam o nome do schema (`Usuario`, `Organizacao`, `Comissao`, `Reuniao`).
- Controllers, services, DTOs e guards podem aparecer quando o diagrama é de implementação.
- “Frontend”, “Backend”, “Banco” e “JSON” não são classes.
- Todo método mostrado deve existir no código; rotas HTTP ficam em notas ou no dicionário, não como nome de método.
- Composição é usada para subdocumentos que não existem fora do documento pai.

### 7.2 Modelo relacional

O banco real é MongoDB. Os diagramas ER são **visões lógicas**, usadas para facilitar a leitura. Documentos embutidos são identificados como tal; não são descritos como tabelas físicas independentes.

### 7.3 Sequência

Participantes válidos incluem tela/componente concreto, `api()`, guards, controller, service e model Mongoose. Mensagens devem usar nomes reais, por exemplo `ComissoesController.criar()` e `ComissoesService.criar()`. Consultas MongoDB podem aparecer como `ComissaoModel.create()` ou `find()`, nunca como “seleciona o hash”.

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
- identificação `Padrão do Incremento 1 · Módulo <SIGLA>`;
- seção `Resumo` antes do conteúdo técnico;
- seção de rastreabilidade ou referência rastreável no conteúdo;
- linguagem no presente para o que está implementado;
- marcação explícita de qualquer medição ou teste ainda pendente.

O `README.md` de cada módulo usa sempre as seções `Resumo`, `Escopo`, `Índice padronizado`, `Rastreabilidade` e `Base de conferência`.
