# Dicionário de classes — Relatório individual

> Padrão da disciplina · Módulo REL · Complementa o diagrama de classes · Revisado em 26/11/2026

## Resumo

| Classe                        | Camada                   | Responsabilidade                                                                                              |
| ----------------------------- | ------------------------ | ------------------------------------------------------------------------------------------------------------- |
| `TelaRelatorioIndividual`     | Aplicativo               | Pedir o relatório, mostrar resumo, vínculos e reuniões, trocar o período e acionar as exportações.            |
| `api`                         | Aplicativo (módulo)      | Enviar a requisição HTTP com `X-API-Key` e token e devolver `Resultado` com os dados ou o erro; em resposta 401 com token salvo, apagar o token por `removerToken`. |
| `relatorioIndividual`         | Aplicativo (módulo)      | Calcular os limites do período, formatar datas e durações e montar o HTML do PDF e o texto do CSV.            |
| `exportarRelatorioIndividual` | Aplicativo (módulo)      | Entregar o PDF ou o CSV conforme a plataforma: impressão, menu de compartilhamento ou download.               |
| `pdf`                         | Aplicativo (módulo)      | Informar se o PDF sai pela tela de impressão (web e Expo Go).                                                 |
| `Print`, `Sharing`, `File`    | Aplicativo (bibliotecas) | Gerar o PDF, abrir o menu de compartilhamento e criar e ler arquivos no aparelho.                             |
| `window`                      | Aplicativo (navegador)   | Abrir a janela de impressão e baixar o CSV na web.                                                            |
| `RelatorioIndividual`         | Aplicativo (tipo)        | Formato do relatório devolvido pela API.                                                                      |
| `ParticipacaoIndividual`      | Aplicativo (tipo)        | Uma reunião do relatório, com a presença do usuário.                                                          |
| `ApiKeyGuard`, `JwtAuthGuard`, `UsuarioComumGuard` | API | Conferir a chave do aplicativo, o token e a conta, e barrar o administrador do sistema.                    |
| `RelatoriosController`        | API                      | Receber `GET /relatorios/individual` e repassar o usuário do token e os filtros ao serviço.                   |
| `RelatoriosService`           | Negócio                  | Validar o período, ler usuário, vínculos e reuniões e calcular durações e resumo.                             |
| `RelatorioIndividualDto`      | API                      | Receber e validar os limites `de` e `ate` do período.                                                         |
| `Usuario`, `Reuniao`, `Participante`, `Presenca`, `Organizacao`, `MembroOrganizacao`, `Comissao`, `MembroComissao` | Dados | Documentos e subdocumentos lidos pelo relatório. |

As classes do pacote Dados estão descritas por completo nos módulos que as gravam: [Reuniões](../../3%20-%20Incremento2/reunioes/3%20-%20dicionario-dados-classes-reunioes.md), [Cadastro de organização](../../2%20-%20Incremento1/cadastro-organizacao/3%20-%20dicionario-dados-classes-cadastro-organizacao.md), [Cadastro de comissão](../../2%20-%20Incremento1/cadastro-comissao/3%20-%20dicionario-dados-classes-cadastro-comissao.md) e [Cadastro de usuário](../../2%20-%20Incremento1/cadastro-usuario/3%20-%20dicionario-dados-classes-cadastro-usuario.md). Abaixo aparecem só os campos que o relatório usa.

## `TelaRelatorioIndividual`

Componente padrão de `FRONTEND/src/app/(abas)/relatorio.tsx`, aberto pela aba "Relatório". A aba fica dentro de `Tabs.Protected`, que só a mostra quando `usuario.tipo` é `USUARIO`.

| Atributo           | Tipo                          | Obrigatório | Regra                                                                                                     |
| ------------------ | ----------------------------- | ----------: | --------------------------------------------------------------------------------------------------------- |
| `periodo`          | `PeriodoRelatorio`            |         Sim | Começa em `todos`; muda pelo controle "Período do relatório".                                              |
| `relatorio`        | `RelatorioIndividual \| null` |         Não | Último relatório recebido; volta a nulo no início de cada consulta e permanece nulo em caso de erro.     |
| `carregando`       | `boolean`                     |         Sim | Começa verdadeiro; verdadeiro enquanto a consulta está em curso.                                          |
| `erro`             | `string`                      |         Sim | Mensagem da consulta que falhou; vazio quando não há erro.                                                |
| `erroExportacao`   | `string`                      |         Sim | Mensagem da exportação que falhou; vazio quando não há erro.                                               |
| `exportando`       | `'pdf' \| 'csv' \| null`      |         Não | Formato em exportação; enquanto não é nulo, botões e controle de período ficam inativos.                  |
| `pedido`           | `RefObject<number>`           |         Sim | Contador de consultas; a resposta só é aceita se o contador não mudou depois que ela foi pedida.          |
| `travaExportacao`  | `RefObject<boolean>`          |         Sim | Impede duas exportações ao mesmo tempo, mesmo com toques repetidos antes de a tela atualizar.             |

| Método     | Parâmetros                 | Retorno         | O que faz                                                                                                                                    |
| ---------- | -------------------------- | --------------- | -------------------------------------------------------------------------------------------------------------------------------------------- |
| `carregar` | —                          | `Promise<void>` | Soma 1 a `pedido`, limpa a tela, chama `api` com a consulta do período e guarda o relatório ou o erro, se a resposta ainda for a mais recente. |
| `exportar` | `formato: 'pdf' \| 'csv'`  | `Promise<void>` | Liga a trava, chama `exportarRelatorioIndividual` e mostra a mensagem do erro, se houver; no fim, solta a trava.                             |
| `definirPeriodo`, `definirCarregando`, `definirRelatorio`, `definirErro`, `definirErroExportacao`, `definirExportando` | o novo valor do atributo correspondente | `void` | Funções de atualização de estado criadas por `useState`; cada uma troca um atributo e faz a tela ser desenhada de novo. |

## `relatorioIndividual`

Módulo `FRONTEND/src/utils/relatorio-individual.ts`. Não acessa a rede nem arquivos; por isso é testado diretamente pelos testes `node --test`.

| Função                    | Parâmetros                                        | Retorno  | O que faz                                                                                                                                             |
| ------------------------- | ------------------------------------------------- | -------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| `consultaRelatorio`       | `periodo: PeriodoRelatorio`, `agora: Date` (padrão: agora) | `string` | `todos` → texto vazio; `mes` e `ano` → `?de=<início do mês ou do ano>&ate=<início de amanhã>`, calculados no fuso do aparelho e enviados em UTC. |
| `horasRelatorio`          | `minutos: number`                                 | `string` | Escreve horas e minutos com dois dígitos, por exemplo `0h 59min`, `24h 00min`.                                                                         |
| `dataRelatorio`           | `data: string \| null`                            | `string` | Data e hora em `pt-BR` no fuso do aparelho; nulo vira "Não registrada".                                                                                |
| `periodoRelatorio`        | `relatorio: RelatorioIndividual`                  | `string` | "Todo o período" sem limites; senão "<de> a <ate − 1 ms>" em `dd/mm/aaaa`.                                                                             |
| `escapar`                 | `texto: string`                                   | `string` | Troca `&`, `<`, `>`, `"` e `'` por entidades HTML. Interna ao módulo.                                                                                  |
| `htmlRelatorioIndividual` | `r: RelatorioIndividual`, `logo: string` (padrão: vazio) | `string` | Monta o documento HTML A4 do PDF, com identificação, vínculos, resumo, reuniões e rodapé. A logo só é usada se for `data:image/png;base64`.     |
| `celulaCsv`               | `valor: string \| number \| null`                 | `string` | Põe o valor entre aspas, duplica aspas internas e acrescenta um apóstrofo antes de texto que começa com `=`, `+`, `-`, `@`, tabulação ou quebra de linha. |
| `csvRelatorioIndividual`  | `r: RelatorioIndividual`                          | `string` | Monta o CSV com BOM, 19 colunas, separador `;` e quebra CR LF: cabeçalho, reuniões, organizações e comissões, ou a linha "Sem participação ou vínculo atual informado". |

## `exportarRelatorioIndividual`

Módulo `FRONTEND/src/services/exportar-relatorio-individual.ts`.

| Função                        | Parâmetros                                              | Retorno           | O que faz                                                                                                                                                             |
| ----------------------------- | ------------------------------------------------------- | ----------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `exportarRelatorioIndividual` | `relatorio: RelatorioIndividual`, `formato: 'pdf' \| 'csv'` | `Promise<void>` | Web: PDF por `imprimirRelatorioWeb` e CSV por download. Celular: PDF por `Print.printAsync` (Expo Go) ou `Print.printToFileAsync` mais `Sharing.shareAsync` (aplicativo instalado); CSV por `File` mais `Sharing.shareAsync`. |
| `logoDoRelatorio`             | —                                                       | `Promise<string>` | Lê `assets/images/logo-titulo.png` e devolve `data:image/png;base64,…`: no celular, por `Asset.downloadAsync` e `File.base64`; na web, por `fetch` e `FileReader`.    |
| `imprimirRelatorioWeb`        | `relatorio: RelatorioIndividual`                        | `Promise<void>`   | Abre a janela, escreve o HTML, espera imagem e fontes e chama `print()`; em erro, fecha a janela e repassa o erro.                                                     |

O PDF do aplicativo instalado é compartilhado com o endereço devolvido por `Print.printToFileAsync`, sem mover nem renomear o arquivo. O módulo `pdf` (`FRONTEND/src/services/pdf.ts`) é usado pelo relatório só pela função `pdfPelaImpressao`, que devolve verdadeiro na web e no Expo Go; a função `entregarPdf` do mesmo módulo, usada pelas atas, não é chamada pelo relatório.

## Bibliotecas chamadas na exportação

| Classe    | Origem                          | Métodos usados                                               | Uso no relatório                                                                                  |
| --------- | ------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------------------------------------------- |
| `Print`   | `expo-print`                    | `printAsync(opcoes)`, `printToFileAsync(opcoes)`             | Tela de impressão no Expo Go; geração do arquivo PDF no aplicativo instalado, em 595 × 842 pontos. |
| `Sharing` | `expo-sharing`                  | `isAvailableAsync()`, `shareAsync(url, opcoes)`              | Conferir e abrir o menu de compartilhamento do aparelho para o PDF e o CSV.                        |
| `File`    | `expo-file-system`              | `new File(...)`, `create()`, `write(conteudo)`, `base64()`   | Criar e gravar o CSV no cache; ler a logo em base64.                                               |
| `Model`, `Query` | `mongoose`               | `find`, `findById`, `findOne`; `select`, `populate`, `sort`, `lean`, `exec` | Leituras do `RelatoriosService` e do `JwtAuthGuard` no MongoDB. `lean` devolve objetos simples, sem os métodos de documento. |
| `window`  | navegador                       | `open`, `document.write`, `focus`, `print`, `close`, `URL.createObjectURL`, `URL.revokeObjectURL`, `document.createElement` | Janela de impressão do PDF e download do CSV na web. |

## `RelatorioIndividual` e `ParticipacaoIndividual`

Tipos de `FRONTEND/src/types/relatorio-individual.ts`. Descrevem o objeto JSON devolvido por `GET /relatorios/individual`; as datas chegam como texto ISO 8601 em UTC.

| Campo                                  | Tipo                               | Obrigatório | Regra                                                                                                    |
| -------------------------------------- | ---------------------------------- | ----------: | -------------------------------------------------------------------------------------------------------- |
| `usuario.nome`, `usuario.email`        | `string`                           |         Sim | Dados da conta do token, lidos de `usuarios`.                                                            |
| `vinculos`                             | objeto                             |         Não | Opcional no tipo; a API atual sempre envia. Sem ele, a tela e o PDF avisam que a informação está indisponível. |
| `vinculos.organizacoes[]`              | `{ id, nome, papel }`              |         Sim | `id` é o `_id` da organização em texto; `papel` é `ADMIN` ou `MEMBRO`. Ordenadas por nome e `_id`.       |
| `vinculos.comissoes[]`                 | `{ id, nome, organizacao_id, organizacao, papel }` | Sim | `id` e `organizacao_id` em texto; `organizacao` é o nome; `papel` é `RESPONSAVEL` ou `MEMBRO`.  |
| `gerada_em`                            | `string`                           |         Sim | Instante em que o servidor montou o relatório.                                                           |
| `periodo.de`, `periodo.ate`            | `string \| null`                   |         Sim | Limites recebidos; nulos sem filtro. `ate` é exclusivo.                                                  |
| `resumo.reunioes`                      | `number`                           |         Sim | Quantidade de itens em `reunioes`.                                                                       |
| `resumo.minutos_registrados`           | `number`                           |         Sim | Soma de `minutos`, contando nulo como zero.                                                              |
| `resumo.registros_sem_duracao`         | `number`                           |         Sim | Quantidade de itens com `minutos` nulo.                                                                  |
| `reunioes`                             | `ParticipacaoIndividual[]`         |         Sim | Ordenada do início mais recente para o mais antigo.                                                      |

| Campo de `ParticipacaoIndividual` | Tipo             | Obrigatório | Regra                                                                                              |
| --------------------------------- | ---------------- | ----------: | -------------------------------------------------------------------------------------------------- |
| `reuniao_id`                      | `string`         |         Sim | `_id` da reunião em texto; usado como chave do cartão na lista.                                    |
| `titulo`                          | `string`         |         Sim | Título da reunião.                                                                                 |
| `comissao`, `organizacao`         | `string`         |         Sim | Nomes atuais; "Comissão indisponível" ou "Organização indisponível" quando o documento não existe. |
| `local`                           | `string \| null` |         Sim | Nulo quando a reunião não tem local.                                                               |
| `inicio`, `fim`                   | `string`         |         Sim | Horário marcado da reunião.                                                                         |
| `dia_inteiro`                     | `boolean`        |         Sim | Verdadeiro para reunião de dia inteiro.                                                            |
| `entrada`                         | `string`         |         Sim | Entrada registrada do usuário.                                                                     |
| `saida`                           | `string \| null` |         Sim | Saída registrada; nula quando não há.                                                              |
| `saida_automatica`                | `boolean`        |         Não | Verdadeiro quando a saída foi registrada pelo servidor no término previsto da reunião.             |
| `minutos`                         | `number \| null` |         Sim | Parte inteira de (saída − entrada) em minutos; nulo se a saída falta, é anterior à entrada ou é futura. |

`PeriodoRelatorio` é o tipo de texto com os valores `todos`, `mes` e `ano`.

## Classes da API

| Classe                   | Atributos                                                                                          | Método e retorno                                                         | Regra                                                                                                                         |
| ------------------------ | -------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------- |
| `ApiKeyGuard`            | `configService: ConfigService`                                                                     | `canActivate(context)`: `boolean`                                        | Aplicado a toda a API. Compara `X-API-Key` com a chave do servidor em tempo constante; erro 401 "Chave de acesso da API inválida ou ausente". |
| `JwtAuthGuard`           | `jwtService: JwtService`, `usuarios: Model<UsuarioDocument>`                                       | `canActivate(context)`: `Promise<boolean>`                               | Verifica o token `Bearer`, exige conta com `ativo` verdadeiro e senha não trocada depois da emissão do token; preenche `req.usuario`. |
| `UsuarioComumGuard`      | —                                                                                                  | `canActivate(context)`: `boolean`                                        | Recusa `ADMIN_SISTEMA` com 403 "O administrador do sistema não pode realizar esta operação".                                  |
| `RelatoriosController`   | `relatorios: RelatoriosService`                                                                    | `individual(req, filtros)`: `Promise<RelatorioIndividual>`               | Rota `GET /relatorios/individual`; usa sempre `req.usuario.sub`.                                                              |
| `RelatoriosService`      | `reunioes: Model<ReuniaoDocument>`, `usuarios: Model<UsuarioDocument>`, `organizacoes: Model<OrganizacaoDocument>`, `comissoes: Model<ComissaoDocument>` | `individual(usuarioId, filtros)`: `Promise<RelatorioIndividual>` | Lê `usuarios`, `organizacoes`, `comissoes` e `reunioes`, esta com `populate` do nome da comissão e da organização; não grava.  |

O `RelatoriosModule` (`BACKEND/src/relatorios/relatorios.module.ts`) registra o controller e o service e disponibiliza os models `Reuniao`, `Usuario`, `Comissao` e `Organizacao`, além do `AuthModule`, que fornece os guards.

## DTO e validação

| DTO                      | Campo | Validação                                         | Mensagem de erro                    |
| ------------------------ | ----- | ------------------------------------------------- | ----------------------------------- |
| `RelatorioIndividualDto` | `de`  | opcional; data e hora ISO 8601 estrita (`IsISO8601`) | "Informe uma data inicial válida" |
|                          | `ate` | opcional; data e hora ISO 8601 estrita; limite exclusivo | "Informe uma data final válida" |
| `RelatoriosService`      | `de` e `ate` | `de` precisa ser anterior a `ate`           | "O fim do período deve ser posterior ao início" |

O `ValidationPipe` global usa `whitelist`, que remove da consulta qualquer parâmetro fora do DTO; por isso um `usuario_id` enviado na URL é ignorado.

## Classes de dados (campos usados)

| Classe              | Campo                | Tipo                               | Obrigatório | Uso no relatório                                                                                   |
| ------------------- | -------------------- | ---------------------------------- | ----------: | -------------------------------------------------------------------------------------------------- |
| `Usuario`           | `nome`, `email`      | `string`                           |         Sim | Identificação no relatório.                                                                        |
|                     | `tipo`               | `TipoUsuario` (`USUARIO`, `ADMIN_SISTEMA`) | Sim | `ADMIN_SISTEMA` é recusado.                                                                    |
|                     | `ativo`              | `boolean`                          |         Sim | Conta inativa não consulta.                                                                        |
|                     | `senha_alterada_em`  | `Date`                             |         Não | Gravado na troca e na recuperação de senha; o `JwtAuthGuard` recusa o token emitido antes dele.    |
| `Reuniao`           | `titulo`, `local`    | `string`                           | Sim / Não (padrão vazio) | Cartão da reunião.                                                                     |
|                     | `inicio`, `fim`      | `Date`                             |         Sim | `inicio` filtra e ordena; `fim` é mostrado.                                                        |
|                     | `dia_inteiro`        | `boolean`                          |  Não (padrão falso) | Mostrado como "Dia inteiro".                                                                |
|                     | `situacao`           | `SituacaoReuniao` (`AGENDADA`, `CANCELADA`) | Sim | Só `AGENDADA` entra.                                                                       |
|                     | `comissao_id`        | `ObjectId`                         |         Sim | Nome da comissão e da organização.                                                                 |
|                     | `participantes`      | `Participante[]`                   |         Sim | Onde está a presença do usuário.                                                                   |
| `Participante`      | `usuario_id`         | `ObjectId`                         |         Sim | Comparado com o usuário do token.                                                                  |
|                     | `presenca`           | `Presenca`                         |         Não | Sem presença, a reunião não entra.                                                                 |
| `Presenca`          | `entrada`            | `Date`                             |         Sim | Precisa ser até o horário da consulta; início da duração.                                          |
|                     | `saida`              | `Date`                             |         Não | Fim da duração.                                                                                    |
|                     | `saida_automatica`   | `boolean`                          |         Não | Gravado pela rotina de saída automática do módulo de reuniões.                                     |
|                     | `situacao`           | `SituacaoPresenca` (`AGUARDANDO`, `CONFIRMADA`, `RECUSADA`) | Sim | Só `CONFIRMADA` entra.                                                       |
| `Organizacao`       | `nome`               | `string`                           |         Sim | Vínculo e ordenação.                                                                               |
|                     | `status`             | `StatusOrganizacao` (`PENDENTE`, `APROVADA`, `REVOGADA`) | Sim | Só `APROVADA` entra nos vínculos.                                                  |
|                     | `membros`            | `MembroOrganizacao[]`              |         Sim | Vínculo do usuário.                                                                                |
| `MembroOrganizacao` | `usuario_id`         | `ObjectId`                         |         Sim | Comparado com o usuário do token.                                                                  |
|                     | `papel`              | `PapelOrganizacao` (`ADMIN`, `MEMBRO`) | Sim | Mostrado como "Administrador" ou "Membro".                                                       |
|                     | `status`             | `StatusMembroOrganizacao` (`PENDENTE`, `APROVADO`, `REJEITADO`, `CONVIDADO`) | Sim | Só `APROVADO` entra.                                        |
| `Comissao`          | `nome`               | `string`                           |         Sim | Vínculo, cartão da reunião e ordenação.                                                            |
|                     | `organizacao_id`     | `ObjectId`                         |         Sim | Precisa ser de uma organização do vínculo.                                                         |
|                     | `ativo`              | `boolean`                          |  Sim (padrão verdadeiro) | Só comissão ativa entra nos vínculos.                                                    |
|                     | `membros`            | `MembroComissao[]`                 |         Sim | Exatamente um `RESPONSAVEL` e nenhum usuário repetido.                                             |
| `MembroComissao`    | `usuario_id`         | `ObjectId`                         |         Sim | Comparado com o usuário do token.                                                                  |
|                     | `papel`              | `PapelComissao` (`RESPONSAVEL`, `MEMBRO`) | Sim | Mostrado como "Responsável" ou "Membro".                                                         |

As enumerações `RespostaConvite` (`PENDENTE`, `ACEITO`, `RECUSADO`) e `OrigemPresenca` (`LINK`, `PROPRIO`, `RESPONSAVEL`) fazem parte dos subdocumentos lidos, mas não são usadas pelo relatório.

## Rastreabilidade

RF-REL-01 a RF-REL-05; [Diagrama de classes](3%20-%20diagrama-classes-relatorio-individual.md) e [Dicionário relacional](4%20-%20dicionario-dados-relacional-relatorio-individual.md).
