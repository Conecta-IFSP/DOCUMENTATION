# Dicionário de classes — Cadastro de comissão

> Padrão da disciplina · Módulo COM · Complementa o diagrama de classes · Revisado em 26/11/2026

## Resumo

| Classe                  | Camada               | O que representa                                                                                                 |
| ----------------------- | -------------------- | ---------------------------------------------------------------------------------------------------------------- |
| `TelaOrganizacao`       | Aplicativo           | Tela de uma organização; a aba "Comissões" lista as comissões com busca e filtro.                                |
| `TelaNovaComissao`      | Aplicativo           | Tela "Nova comissão": formulário de cadastro com a escolha do responsável.                                       |
| `TelaComissao`          | Aplicativo           | Tela de uma comissão, com as abas "Reuniões" e "Equipe" e as folhas de edição, adição, troca e confirmação.     |
| `SeletorPessoaComissao` | Aplicativo (componente) | Folha com a lista de pessoas, com busca, usada para escolher o responsável.                                  |
| `validacao`             | Aplicativo (módulo)  | Funções que conferem os campos antes do envio.                                                                   |
| `api`                   | Aplicativo (módulo)  | Função única de acesso à API: monta a requisição com `X-API-Key` e o token e devolve um `Resultado`.             |
| `ComissoesController`   | API                  | Recebe as rotas `/comissoes`, já protegidas pelos guards, e repassa ao serviço com o identificador do usuário.   |
| `ComissoesService`      | Negócio              | Aplica as regras: quem pode ver e gerenciar, quem pode entrar na equipe, responsável único e gravação condicionada. |
| DTOs                    | API                  | Objetos de entrada validados pelo `ValidationPipe`.                                                              |
| `Comissao`              | Documento            | Uma comissão guardada na coleção `comissoes`.                                                                    |
| `MembroComissao`        | Subdocumento         | Um integrante da equipe e o seu papel.                                                                           |
| `PapelComissao`         | Enumeração           | Papel do integrante: responsável ou membro.                                                                      |
| `Organizacao`           | Documento            | Organização dona da comissão; usada para conferir vínculos.                                                      |
| `MembroOrganizacao`     | Subdocumento         | Vínculo de uma pessoa com a organização.                                                                         |
| `Usuario`               | Documento            | Conta de uma pessoa; usada para conferir se está ativa e para mostrar nome e e-mail.                             |

## `Comissao`

Documento da coleção `comissoes`, definido em `BACKEND/src/comissoes/schemas/comissao.schema.ts`. O schema não grava o campo de versão do Mongoose (`versionKey: false`).

| Atributo         | Tipo TypeScript/Mongoose | Tipo BSON  | Obrigatório | Regra                                                                                                                         |
| ---------------- | ------------------------ | ---------- | ----------: | ----------------------------------------------------------------------------------------------------------------------------- |
| `_id`            | `Types.ObjectId`         | `objectId` |         Sim | Identificador gerado na inserção (ver o [dicionário relacional](4%20-%20dicionario-dados-relacional-cadastro-comissao.md)).   |
| `nome`           | `string`                 | `string`   |         Sim | De 2 a 100 caracteres (regra do DTO); o schema retira os espaços das pontas. Pode repetir o nome de outra comissão.           |
| `descricao`      | `string`                 | `string`   |         Não | Até 1.000 caracteres na API e 200 no aplicativo. O aplicativo envia texto vazio quando o campo fica em branco.                 |
| `organizacao_id` | `Types.ObjectId`         | `objectId` |         Sim | Referência à organização em `organizacoes`. Não é o objeto da organização, é o identificador dela. Não muda depois do cadastro. |
| `ativo`          | `boolean`                | `bool`     |         Sim | Padrão verdadeiro. Falso indica comissão desativada; o documento não é apagado.                                              |
| `membros`        | `MembroComissao[]`       | `array`    |         Sim | Padrão: lista vazia. O validador do schema exige exatamente um item `RESPONSAVEL` e nenhum `usuario_id` repetido.             |
| `criado_em`      | `Date`                   | `date`     |         Sim | Instante da inserção, em UTC, gravado pelo Mongoose (`timestamps`).                                                           |
| `atualizado_em`  | `Date`                   | `date`     |         Sim | Instante da última alteração, em UTC, gravado pelo Mongoose a cada atualização.                                              |

O validador de `membros` roda na criação e nas atualizações deste módulo, que usam `runValidators`. Quando falha, a mensagem é "A comissão deve ter exatamente um responsável e integrantes únicos" e a API responde erro interno (500); no uso normal isso não acontece, porque o serviço confere as mesmas regras antes de gravar.

## `MembroComissao`

Subdocumento de `Comissao`, sem `_id` próprio (`@Schema({ _id: false })`): ele é identificado pela comissão e pelo `usuario_id`.

| Atributo        | Tipo TypeScript/Mongoose      | Tipo BSON  | Obrigatório | Regra                                                                                                   |
| --------------- | ----------------------------- | ---------- | ----------: | ------------------------------------------------------------------------------------------------------- |
| `usuario_id`    | `Types.ObjectId`              | `objectId` |         Sim | Referência ao usuário em `usuarios`. Na inclusão, a pessoa precisa estar ativa e aprovada na organização. |
| `papel`         | `PapelComissao` (enum string) | `string`   |         Sim | `RESPONSAVEL` ou `MEMBRO`.                                                                              |
| `adicionado_em` | `Date`                        | `date`     |         Não | Instante em que a pessoa entrou na equipe, em UTC. O serviço sempre o preenche; sem ele, o schema usa o horário da gravação. É mantido quando a equipe é regravada. |

## `PapelComissao`

| Valor         | Significado                                                                                            |
| ------------- | ------------------------------------------------------------------------------------------------------ |
| `RESPONSAVEL` | Preside a comissão: agenda reuniões e escreve as atas (módulos REU e ATA). Há exatamente um por comissão. |
| `MEMBRO`      | Integrante comum: consulta a comissão e participa das reuniões para as quais é convidado.              |

## `Organizacao`, `MembroOrganizacao` e `Usuario` (campos usados)

| Classe              | Atributo     | Tipo                      | Uso neste módulo                                                                 |
| ------------------- | ------------ | ------------------------- | -------------------------------------------------------------------------------- |
| `Organizacao`       | `nome`       | `string`                  | Mostrado na tela da comissão e na escolha de organização do cadastro.            |
| `Organizacao`       | `status`     | `StatusOrganizacao`       | Precisa ser `APROVADA` para qualquer operação com comissões.                     |
| `Organizacao`       | `membros`    | `MembroOrganizacao[]`     | Define quem vê, quem gerencia e quem pode entrar na equipe.                      |
| `MembroOrganizacao` | `usuario_id` | `Types.ObjectId`          | Pessoa vinculada.                                                                |
| `MembroOrganizacao` | `papel`      | `PapelOrganizacao`        | `ADMIN` gerencia as comissões; `MEMBRO` só vê as suas.                           |
| `MembroOrganizacao` | `status`     | `StatusMembroOrganizacao` | Só `APROVADO` dá acesso.                                                         |
| `Usuario`           | `nome`, `email` | `string`               | Mostrados na equipe e na escolha de pessoas.                                     |
| `Usuario`           | `tipo`       | `TipoUsuario`             | `ADMIN_SISTEMA` é recusado pelo `UsuarioComumGuard`.                             |
| `Usuario`           | `ativo`      | `boolean`                 | Conta inativa não usa o módulo nem pode entrar numa equipe.                      |

A descrição completa dessas classes está nos dicionários dos módulos de [organização](../cadastro-organizacao/3%20-%20dicionario-dados-classes-cadastro-organizacao.md) e de [usuário](../cadastro-usuario/3%20-%20dicionario-dados-classes-cadastro-usuario.md).

## Classes do aplicativo

| Classe                  | Atributo ou método                    | Tipo                                   | O que faz                                                                                                              |
| ----------------------- | ------------------------------------- | -------------------------------------- | ---------------------------------------------------------------------------------------------------------------------- |
| `TelaOrganizacao`       | `filtro`                              | `'Todas' \| 'Ativas' \| 'Inativas'`    | Filtro de situação; começa em "Todas".                                                                                 |
| `TelaOrganizacao`       | `buscaComissao`                       | `string`                               | Texto do campo "Buscar" da aba Comissões.                                                                              |
| `TelaOrganizacao`       | `carregar()`                          | `Promise<Dados>`                       | Busca a organização, as comissões (`?organizacao_id=`) e as reuniões futuras.                                          |
| `TelaNovaComissao`      | `salvar()`                            | `Promise<void>`                        | Valida o formulário e envia `POST /comissoes`; no sucesso, abre a comissão criada.                                     |
| `TelaComissao`          | `gerencia`                            | `boolean`                              | Verdadeiro quando a organização da comissão está entre as administradas pelo usuário.                                 |
| `TelaComissao`          | `aba`                                 | `'reunioes' \| 'equipe'`               | Aba aberta.                                                                                                            |
| `TelaComissao`          | `folha`                               | `'opcoes' \| 'editar' \| 'adicionar' \| 'responsavel' \| null` | Folha aberta por cima da tela.                                                                |
| `TelaComissao`          | `buscar()` / `aplicar(carga, preencherFormulario)` | `Promise<Carga>` / `void` | Consulta a comissão, as organizações administradas e as pessoas elegíveis, e guarda o resultado.                       |
| `TelaComissao`          | `executar(acao, mensagem)`            | `Promise<boolean>`                     | Executa uma chamada de alteração, impede outra ao mesmo tempo, mostra a faixa verde ou vermelha e guarda a comissão devolvida. |
| `TelaComissao`          | `salvarDados()`                       | `Promise<void>`                        | Valida e envia a edição de nome e descrição.                                                                           |
| `TelaComissao`          | `desativar()`                         | `Promise<Resultado<Comissao>>`         | Chama `DELETE /comissoes/:id` e devolve só a comissão.                                                                 |
| `TelaComissao`          | `trocarResponsavel(alvo)`             | `void`                                 | Prepara a confirmação da troca de responsável.                                                                         |
| `SeletorPessoaComissao` | `pessoas`, `selecionados`             | `PessoaComissao[]`, `string[]`         | Pessoas mostradas e as já marcadas; com mais de seis pessoas, mostra o campo "Buscar".                                 |
| `validacao`             | `validarNomeComissao(nome)`           | `string \| null`                       | "Informe um nome com pelo menos 2 caracteres." quando o nome aparado tem menos de 2 caracteres.                        |
| `validacao`             | `validarDescricao(descricao)`         | `string \| null`                       | "A descrição deve ter no máximo 200 caracteres." quando passa de 200.                                                  |
| `api`                   | `api(caminho, metodo, corpo)`         | `Promise<Resultado<T>>`                | Devolve `{ ok: true, dados }` ou `{ ok: false, erro }`, com a primeira mensagem de erro da API.                        |

## Operações do `ComissoesService`

| Método                     | Parâmetros                              | Retorno                         | O que faz                                                                                                   |
| -------------------------- | --------------------------------------- | ------------------------------- | ----------------------------------------------------------------------------------------------------------- |
| `listar`                   | `usuarioId`, `organizacaoId` (opcional) | lista de comissões              | Com organização: todas para o administrador, as da equipe para os demais. Sem organização: junta todas as organizações do usuário. |
| `buscarPorId`              | `id`, `usuarioId`                       | comissão                        | Exige administrador da organização ou integrante da comissão.                                              |
| `listarOrganizacoes`       | `usuarioId`                             | lista de organizações           | Organizações `APROVADA` que o usuário administra, só com `nome`, em ordem alfabética.                       |
| `listarMembrosDisponiveis` | `organizacaoId`, `usuarioId`            | lista de usuários               | Pessoas ativas com vínculo `APROVADO` na organização, com `nome` e `email`, em ordem alfabética.            |
| `listarMinhas`             | `organizacaoId`, `usuarioId`            | `{ organizacao, comissoes }`    | Comissões da organização de cuja equipe o usuário faz parte, com `meu_papel`. Não usada pelo aplicativo.     |
| `criar`                    | `CreateComissaoDto`, `usuarioId`        | comissão criada                 | Confere a organização e o responsável e cria a comissão ativa com ele como único integrante.               |
| `atualizar`                | `id`, `UpdateComissaoDto`, `usuarioId`  | comissão atualizada             | Grava os campos recebidos; para reativar, exige responsável único.                                         |
| `remover`                  | `id`, `usuarioId`                       | `{ mensagem, comissao }`        | Desativa a comissão chamando `atualizar` com `ativo = false`.                                              |
| `adicionarMembro`          | `id`, `AddMembroComissaoDto`, `usuarioId` | comissão atualizada           | Inclui um `MEMBRO` elegível numa comissão ativa.                                                            |
| `definirResponsavel`       | `id`, `membroId`, `usuarioId`           | comissão atualizada             | Promove o escolhido e rebaixa os demais numa só gravação.                                                   |
| `atualizarMembro`          | `id`, `membroId`, `papel`, `usuarioId`  | comissão                        | Rota não usada pelo aplicativo; ver o [diagrama de classes](3%20-%20diagrama-classes-cadastro-comissao.md). |
| `removerMembro`            | `id`, `membroId`, `usuarioId`           | comissão atualizada             | Retira um `MEMBRO` de uma comissão ativa.                                                                   |
| `gravarEquipe`             | `comissao`, `membros`                   | comissão atualizada             | Grava a nova equipe só se a equipe e a situação lidas não mudaram; senão, 409.                              |
| `exigirResponsavelUnico`   | `comissao`                              | —                               | Recusa quando a equipe não tem exatamente um responsável.                                                  |
| `exigirElegivel`           | `organizacaoId`, `membroId`, `usuarioId` | —                              | Recusa quem não está entre as pessoas elegíveis da organização.                                            |
| `autorizarLeitura`         | `organizacaoId`, `usuarioId`            | organização                     | Exige conta ativa e vínculo `APROVADO` numa organização `APROVADA`.                                         |
| `autorizarOrganizacao`     | `organizacaoId`, `usuarioId`            | organização                     | Como `autorizarLeitura`, mas exige papel `ADMIN`.                                                           |
| `autorizarComissao`        | `id`, `usuarioId`                       | comissão                        | Busca a comissão e chama `autorizarOrganizacao` para a organização dela.                                    |
| `validarEquipe`            | `id`, `membroId`, `usuarioId`           | comissão                        | Exige comissão ativa, administrada pelo usuário, e a pessoa na equipe.                                      |

## DTOs e validação

| DTO                       | Campo            | Validação                                                  | Mensagem de erro                                                                        |
| ------------------------- | ---------------- | ---------------------------------------------------------- | --------------------------------------------------------------------------------------- |
| `CreateComissaoDto`       | `nome`           | texto, aparado, de 2 a 100 caracteres                      | "Informe o nome da comissão", "Informe um nome com pelo menos 2 caracteres", "O nome deve ter até 100 caracteres" |
|                           | `descricao`      | opcional; texto, aparado, até 1.000 caracteres             | "A descrição deve ser um texto", "A descrição deve ter até 1000 caracteres"             |
|                           | `organizacao_id` | `ObjectId` em texto                                        | "Selecione uma organização válida"                                                     |
|                           | `responsavel_id` | `ObjectId` em texto                                        | "Selecione um responsável válido"                                                      |
| `UpdateComissaoDto`       | `nome`           | só quando enviado; as mesmas regras do cadastro            | as mesmas do cadastro                                                                   |
|                           | `descricao`      | só quando enviado; as mesmas regras do cadastro            | as mesmas do cadastro                                                                   |
|                           | `ativo`          | só quando enviado; booleano (nulo é recusado)              | "Informe se a comissão está ativa"                                                     |
| `AddMembroComissaoDto`    | `usuario_id`     | `ObjectId` em texto                                        | "Selecione um usuário válido"                                                          |
|                           | `papel`          | opcional; `MEMBRO` ou `RESPONSAVEL`                        | "Papel deve ser MEMBRO ou RESPONSAVEL"                                                 |
| `DefinirResponsavelDto`   | `usuario_id`     | `ObjectId` em texto                                        | "Selecione um responsável válido"                                                      |
| `UpdateMembroComissaoDto` | `papel`          | `MEMBRO` ou `RESPONSAVEL`                                  | "Papel deve ser MEMBRO ou RESPONSAVEL"                                                 |

## Rastreabilidade

RF-COM-01 a RF-COM-09; [Diagrama de classes](3%20-%20diagrama-classes-cadastro-comissao.md) e [Dicionário relacional](4%20-%20dicionario-dados-relacional-cadastro-comissao.md).
