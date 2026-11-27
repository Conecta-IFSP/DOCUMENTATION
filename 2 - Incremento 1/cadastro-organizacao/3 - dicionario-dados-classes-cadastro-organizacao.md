# Dicionário de classes — Cadastro de organização

> Padrão da disciplina · Módulo ORG · Complementa o diagrama de classes · Revisado em 27/11/2026

## Resumo

| Classe                      | Camada               | O que representa                                                                                              |
| --------------------------- | -------------------- | ------------------------------------------------------------------------------------------------------------- |
| `TelaCadastroOrganizacao`   | Aplicativo           | Tela "Criar organização", com prévia e envio do cadastro.                                                     |
| `TelaEntrarComCodigo`       | Aplicativo           | Tela "Entrar com código", que envia o pedido de entrada.                                                      |
| `TelaOrganizacoes`          | Aplicativo           | Aba "Organizações": lista as organizações do usuário e responde convites.                                     |
| `TelaOrganizacao`           | Aplicativo           | Tela de uma organização: comissões, membros, convites, código e ações sobre vínculos.                         |
| `TelaGerenciarOrganizacoes` | Aplicativo           | Tela "Gerenciar organizações" do administrador do sistema: busca, autoriza, revoga, edita e exclui.          |
| `TelaInicio`                | Aplicativo           | Tela inicial: contadores de convites e solicitações e atalho do administrador do sistema.                    |
| `Confirmacao`               | Aplicativo           | Diálogo que pede confirmação antes de uma ação; "Cancelar" fecha sem executar nada.                          |
| `api`                       | Aplicativo (serviço) | Envia as requisições HTTP com a chave do aplicativo e o token, e devolve `Resultado` com os dados ou o erro.  |
| `validacao`                 | Aplicativo (serviço) | Confere os campos no aparelho antes do envio e formata o código de acesso.                                    |
| `OrganizacoesController`    | API                  | Recebe as rotas HTTP de `/organizacoes` e repassa ao serviço com o usuário do token.                          |
| `OrganizacoesService`       | Negócio              | Aplica as regras de situação, vínculo, papel, código e gravação protegida.                                    |
| `JwtAuthGuard`              | API                  | Exige um token JWT válido de uma conta ativa, emitido depois da última troca de senha.                       |
| `UsuarioAtivoGuard`         | API                  | Relê a conta no banco e substitui o tipo do token pelo tipo gravado.                                          |
| `UsuarioComumGuard`         | API                  | Recusa a conta `ADMIN_SISTEMA` nas rotas de usuário comum.                                                    |
| `AdminSistemaGuard`         | API                  | Aceita só a conta `ADMIN_SISTEMA` nas rotas de moderação.                                                     |
| DTOs (`*Dto`)               | API                  | Formato e validação de cada corpo de requisição.                                                              |
| `Organizacao`               | Documento            | Uma organização cadastrada, com os seus vínculos.                                                             |
| `MembroOrganizacao`         | Subdocumento         | O vínculo de uma pessoa com a organização: papel, situação e datas.                                           |
| `Usuario`                   | Documento            | Conta de usuário, usada para identificar criador, convidado e membros.                                        |
| `Comissao`                  | Documento            | Comissão da organização, apagada na exclusão e alterada na remoção de membro.                                 |
| `OrganizacaoModel`          | Dados (model)        | O model do Mongoose da coleção `organizacoes`.                                                                |
| `UsuarioModel`              | Dados (model)        | O model do Mongoose da coleção `usuarios`, usado para conferir contas.                                        |
| `ComissaoModel`             | Dados (model)        | O model do Mongoose da coleção `comissoes`, usado na exclusão da organização e na remoção de membro.          |
| `organizacaoSchema`         | Dados (módulo)       | As funções exportadas por `organizacao.schema.ts`; aqui, o sorteio do código de acesso.                      |

As classes `Usuario` e `Comissao` estão descritas nos módulos [Cadastro de usuário](../cadastro-usuario/3%20-%20dicionario-dados-classes-cadastro-usuario.md) e [Cadastro de comissão](../cadastro-comissao/3%20-%20dicionario-dados-classes-cadastro-comissao.md); abaixo aparecem só os campos que este módulo usa.

## `Organizacao`

Documento da coleção `organizacoes`, definido em `BACKEND/src/organizacoes/schemas/organizacao.schema.ts`. Representa uma instituição cadastrada no Conecta+. O schema grava `criado_em` e `atualizado_em` automaticamente (`timestamps`).

| Atributo        | Tipo TypeScript/Mongoose           | Tipo BSON  | Obrigatório | Regra                                                                                                                                                          |
| --------------- | ---------------------------------- | ---------- | ----------: | -------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `_id`           | `Types.ObjectId`                   | `objectId` |         Sim | Identificador gerado na inserção; formato descrito no [dicionário relacional](4%20-%20dicionario-dados-relacional-cadastro-organizacao.md).                     |
| `nome`          | `string`                           | `string`   |         Sim | De 2 a 100 caracteres, sem espaços nas pontas (`trim`); a API confere o tamanho antes de retirar os espaços. Único entre as organizações sem diferenciar maiúsculas, conferido pelo serviço antes de gravar, sem índice único. |
| `cidade`        | `string`                           | `string`   |         Sim | De 2 a 60 caracteres, sem espaços nas pontas.                                                                                                                  |
| `descricao`     | `string`                           | `string`   |         Não | Até 200 caracteres. Ausente quando não informada no cadastro; texto vazio quando apagada na edição.                                                            |
| `status`        | `StatusOrganizacao` (enum string)  | `string`   |         Sim | `PENDENTE`, `APROVADA` ou `REVOGADA`. Padrão: `PENDENTE`. Só o administrador do sistema muda.                                                                 |
| `criada_por`    | `Types.ObjectId`                   | `objectId` |         Sim | Identificador do usuário que cadastrou, em `usuarios`. É a referência ao usuário, não o objeto dele.                                                           |
| `codigo_acesso` | `string`                           | `string`   |         Sim | 10 caracteres sorteados por `gerarCodigoAcesso()`; único entre as organizações (`unique`); não sai nas consultas, salvo quando pedido com `+codigo_acesso` (`select: false`). |
| `membros`       | `MembroOrganizacao[]`              | `array`    |         Sim | Vínculos da organização. Padrão: lista vazia; o cadastro já grava o vínculo do criador.                                                                       |
| `criado_em`     | `Date`                             | `date`     |         Sim | Instante da inserção, em UTC. Ordena as listas do RF-ORG-02.                                                                                                   |
| `atualizado_em` | `Date`                             | `date`     |         Sim | Instante da última gravação feita pelo Mongoose, em UTC.                                                                                                       |

`gerarCodigoAcesso()` é uma função do mesmo arquivo, usada como valor padrão de `codigo_acesso` e pelo `OrganizacoesService.gerarNovoCodigo`. Ela monta o código com `TAMANHO_CODIGO` (10) caracteres do `ALFABETO_CODIGO` (`ABCDEFGHJKMNPQRSTUVWXYZ23456789`), cada um escolhido por `randomInt`, do módulo `node:crypto`.

## `MembroOrganizacao`

Subdocumento de `Organizacao`, sem `_id` próprio (`@Schema({ _id: false })`): ele é identificado pela organização em que está e pelo `usuario_id`. Representa a relação de uma pessoa com a organização, desde o pedido ou o convite até a aprovação.

| Atributo        | Tipo TypeScript/Mongoose                | Tipo BSON  | Obrigatório | Regra                                                                                                                         |
| --------------- | --------------------------------------- | ---------- | ----------: | ----------------------------------------------------------------------------------------------------------------------------- |
| `usuario_id`    | `Types.ObjectId`                        | `objectId` |         Sim | Referência ao usuário em `usuarios`. Não se repete dentro da mesma organização.                                               |
| `papel`         | `PapelOrganizacao` (enum string)        | `string`   |         Sim | `ADMIN` ou `MEMBRO`. Todo vínculo novo, por pedido ou convite, começa como `MEMBRO`; só o do criador começa como `ADMIN`.     |
| `status`        | `StatusMembroOrganizacao` (enum string) | `string`   |         Sim | `PENDENTE`, `APROVADO`, `REJEITADO` ou `CONVIDADO`. Padrão: `PENDENTE`.                                                      |
| `solicitado_em` | `Date`                                  | `date`     |         Sim | Instante do pedido, do convite ou da reabertura do pedido, em UTC. Padrão: horário da gravação.                               |
| `aprovado_em`   | `Date`                                  | `date`     |         Não | Instante em que o vínculo passou a `APROVADO`; é removido quando o vínculo é recusado, reaberto ou transformado em convite.   |

## `Usuario` (campos usados por este módulo)

| Atributo | Tipo                       | Obrigatório | Regra                                                                                       |
| -------- | -------------------------- | ----------: | ------------------------------------------------------------------------------------------- |
| `_id`    | `Types.ObjectId`           |         Sim | Referenciado por `criada_por` e `membros.usuario_id`.                                       |
| `nome`   | `string`                   |         Sim | Mostrado nas listas de membros, convites e "Criada por".                                    |
| `email`  | `string`                   |         Sim | Único e em minúsculas; usado para localizar a pessoa convidada (RF-ORG-08).                 |
| `tipo`   | `TipoUsuario` (enum string)|         Sim | `USUARIO` participa de organizações; `ADMIN_SISTEMA` só as modera.                          |
| `ativo`  | `boolean`                  |         Sim | Só contas ativas acessam a API, pedem entrada ou recebem convite.                           |

## `Comissao` (campos usados por este módulo)

| Atributo         | Tipo               | Obrigatório | Regra                                                                                                    |
| ---------------- | ------------------ | ----------: | -------------------------------------------------------------------------------------------------------- |
| `organizacao_id` | `Types.ObjectId`   |         Sim | Organização dona da comissão; as comissões com o `_id` da organização excluída são apagadas (RF-ORG-06). |
| `membros`        | `MembroComissao[]` |         Sim | Equipe da comissão; a remoção de um membro da organização retira o mesmo `usuario_id` daqui (RF-ORG-13). |

## Enumerações

| Enumeração                | Valores                                           | Onde aparece                |
| ------------------------- | ------------------------------------------------- | --------------------------- |
| `StatusOrganizacao`       | `PENDENTE`, `APROVADA`, `REVOGADA`                | `Organizacao.status`        |
| `PapelOrganizacao`        | `ADMIN`, `MEMBRO`                                 | `MembroOrganizacao.papel`   |
| `StatusMembroOrganizacao` | `PENDENTE`, `APROVADO`, `REJEITADO`, `CONVIDADO`  | `MembroOrganizacao.status`  |
| `TipoUsuario`             | `USUARIO`, `ADMIN_SISTEMA`                        | `Usuario.tipo`              |

O significado de cada valor de situação está no [dicionário relacional](4%20-%20dicionario-dados-relacional-cadastro-organizacao.md).

## Operações do `OrganizacoesController`

Cada método recebe os parâmetros da rota, o corpo já validado pelo DTO e a requisição autenticada (`RequisicaoAutenticada`), de onde tira `usuario.sub` (identificador) e `usuario.tipo`. Não há regra de negócio no controller: ele só repassa ao serviço. As rotas e os guardas estão no [diagrama de classes](3%20-%20diagrama-classes-cadastro-organizacao.md#rotas-atendidas-pelo-controller).

## Operações do `OrganizacoesService`

| Método                      | Parâmetros                                         | Retorno                                       | O que faz                                                                                                                      |
| --------------------------- | -------------------------------------------------- | --------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------ |
| `criar`                     | `CreateOrganizacaoDto`, `usuarioId`                | organização sem `codigo_acesso`               | Confere o nome, grava a organização `PENDENTE` com o criador `ADMIN`/`APROVADO` e repete em caso de código repetido (até 5 vezes). |
| `listar`                    | `tipoUsuario`, `usuarioId`, `busca`                | lista de organizações                         | Administrador do sistema: todas, filtradas pelo nome. Usuário: só as com vínculo, com `meu_vinculo`.                          |
| `buscarPorId`               | `id`, `solicitanteId`                              | organização com membros e código              | Confere o administrador e chama `detalhes`.                                                                                    |
| `atualizar`                 | `id`, `UpdateOrganizacaoDto`                       | organização atualizada                        | Apara os textos, confere o nome e grava os campos recebidos, inclusive `status`.                                              |
| `remover`                   | `id`                                               | `{ mensagem }`                                | Apaga a organização só se ela estiver `REVOGADA` e depois as suas comissões.                                                   |
| `entrarComCodigo`           | `codigo`, `usuarioId`                              | `{ mensagem, membro, organizacao }`           | Normaliza o código, acha a organização `APROVADA` e chama `adicionarMembro`.                                                  |
| `adicionarMembro`           | `id`, `usuarioId`                                  | `{ mensagem, membro }`                        | Cria o vínculo `PENDENTE` ou reabre o `REJEITADO`; recusa convite e vínculo existente.                                        |
| `gerarNovoCodigo`           | `id`, `solicitanteId`                              | `{ mensagem, codigo_acesso }`                 | Sorteia e grava um novo código, repetindo em caso de colisão (até 5 vezes).                                                   |
| `atualizarStatusMembro`     | `id`, `usuarioId`, `UpdateStatusMembroDto`, `solicitanteId` | `{ mensagem, usuario_id, status }`   | Aprova ou recusa um pedido `PENDENTE` e aprova um `REJEITADO`.                                                                 |
| `removerMembro`             | `id`, `usuarioId`, `solicitanteId`                 | `{ mensagem }`                                | Retira vínculo `MEMBRO` aprovado, convidado ou recusado, e tira a pessoa das comissões.                                       |
| `alterarPapelMembro`        | `id`, `usuarioId`, `UpdatePapelMembroDto`, `solicitanteId` | `{ mensagem, usuario_id, papel }`     | Promove ou rebaixa um vínculo aprovado, mantendo ao menos um administrador.                                                    |
| `convidarMembro`            | `id`, `ConvidarMembroDto`, `solicitanteId`         | `{ mensagem, usuario }`                       | Localiza a conta pelo e-mail e grava o vínculo `CONVIDADO`.                                                                   |
| `responderConvite`          | `id`, `usuarioId`, `ResponderConviteDto`           | `{ mensagem, aceito }`                        | Aceita (vínculo `APROVADO`) ou recusa (vínculo apagado) o convite.                                                             |
| `detalhes`                  | `id`                                               | organização                                   | Lê a organização com `codigo_acesso` e com os dados dos usuários preenchidos.                                                  |
| `reabrirSolicitacao`        | `id`, `usuarioId`                                  | `{ mensagem, membro }`                        | Volta um vínculo `REJEITADO` para `PENDENTE`.                                                                                 |
| `vinculoDe`                 | `organizacao`, `usuarioId`                         | vínculo ou `undefined`                        | Procura em `membros` o item do usuário.                                                                                        |
| `administradorAprovado`     | `solicitanteId`                                    | filtro do MongoDB                             | Monta o `$elemMatch` "usuário com papel `ADMIN` e `APROVADO`" usado nas gravações protegidas.                                  |
| `codigoRepetido`            | `erro`                                             | `boolean`                                     | Diz se o erro é de chave duplicada (código 11000) no índice de `codigo_acesso`.                                               |
| `escaparRegex`              | `texto`                                            | texto                                         | Escapa os caracteres especiais de expressão regular, para que o termo seja comparado ao pé da letra.                          |
| `exigirAdministrador`       | `id`, `usuarioId`                                  | organização                                   | Busca a organização e exige o vínculo `ADMIN`/`APROVADO` do usuário (403 e 404).                                              |
| `exigirOrganizacaoAprovada` | `organizacao`                                      | —                                             | Exige `status = APROVADA` (403).                                                                                               |
| `exigirNomeDisponivel`      | `nome`, `ignorarId` (opcional)                     | —                                             | Recusa (409) se outra organização tem o mesmo nome, sem diferenciar maiúsculas.                                               |
| `validarId`                 | `id`                                               | —                                             | Recusa (400) texto que não é um `ObjectId`.                                                                                    |

## DTOs e validação

| DTO                     | Campo       | Validação                                                  | Mensagem de erro                                                                                      |
| ----------------------- | ----------- | ---------------------------------------------------------- | ----------------------------------------------------------------------------------------------------- |
| `CreateOrganizacaoDto`  | `nome`      | texto de 2 a 100 caracteres                                | "Informe o nome da organização", "O nome deve ter pelo menos 2 caracteres", "O nome deve ter até 100 caracteres" |
|                         | `cidade`    | texto de 2 a 60 caracteres                                 | "Informe a cidade", "A cidade deve ter no máximo 60 caracteres"                                       |
|                         | `descricao` | opcional, texto de até 200 caracteres                      | "A descrição deve ser um texto", "A descrição deve ter até 200 caracteres"                            |
| `UpdateOrganizacaoDto`  | `nome`, `cidade`, `descricao` | as mesmas regras, todas opcionais        | as mesmas mensagens                                                                                   |
|                         | `status`    | opcional, `PENDENTE`, `APROVADA` ou `REVOGADA`             | "Situação deve ser PENDENTE, APROVADA ou REVOGADA"                                                    |
| `EntrarComCodigoDto`    | `codigo`    | texto de até 20 caracteres                                 | "Informe o código", "Código inválido"                                                                 |
| `ConvidarMembroDto`     | `email`     | endereço de e-mail                                         | "Informe um e-mail válido"                                                                            |
| `ResponderConviteDto`   | `aceitar`   | valor lógico                                               | "Informe se o convite foi aceito"                                                                     |
| `UpdateStatusMembroDto` | `status`    | `APROVADO` ou `REJEITADO`                                  | "Situação deve ser APROVADO ou REJEITADO"                                                             |
| `UpdatePapelMembroDto`  | `papel`     | `ADMIN` ou `MEMBRO`                                        | "Papel deve ser ADMIN ou MEMBRO"                                                                      |

## Telas e módulos do aplicativo

| Classe                      | Operação                    | O que faz                                                                                                                              |
| --------------------------- | --------------------------- | -------------------------------------------------------------------------------------------------------------------------------------- |
| `TelaCadastroOrganizacao`   | `enviar()`                  | Valida o formulário e envia `POST /organizacoes`; no sucesso, guarda os dados em `enviada` e mostra "Solicitação enviada".            |
|                             | `definirErro(valor: string)`, `definirEnviada(valor: DadosOrganizacao)` | Atualizadores do `useState`, com retorno `void`: a mensagem de erro da API e os dados enviados, que trocam o formulário pela confirmação. |
| `TelaEntrarComCodigo`       | `enviar()`                  | Valida o código e envia `POST /organizacoes/entrar`; no sucesso, guarda o pedido em `enviado` e mostra "Pedido enviado".              |
|                             | `definirEnviado(valor: Pedido)` | Atualizador do `useState`, com retorno `void`: guarda o pedido devolvido pela API.                                                 |
| `TelaOrganizacoes`          | `carregar(continuar)`       | Busca organizações, comissões e reuniões e, nas administradas, os membros; `continuar` evita gravar o estado de uma tela já fechada.  |
|                             | `atualizar()`               | Recarrega ao puxar a lista para baixo.                                                                                                 |
|                             | `responder(organizacao, aceitar)` | Envia a resposta ao convite e recarrega.                                                                                         |
|                             | `definirOrganizacoes(valor: Organizacao[])`, `definirAviso(valor: Aviso)` | Atualizadores do `useState`, com retorno `void`: guardam a lista montada por `carregar` e a faixa de aviso.     |
| `TelaOrganizacao`           | `carregar()`                | Lê a organização na lista do usuário e, se ele administra, os membros e o código (`Dados`).                                           |
|                             | `aprovar(membro)`           | Aprova um pedido ou um pedido recusado, sem diálogo.                                                                                   |
|                             | `pedir(pendente)`           | Abre o diálogo `Confirmacao` com a mensagem, o rótulo e a ação guardados em `Pendente`.                                                |
|                             | `confirmar()`               | Executa a ação de `Pendente`, mostra a mensagem de sucesso ou de erro e recarrega.                                                    |
|                             | `convidar()`                | Valida o e-mail e envia o convite.                                                                                                     |
|                             | `copiarCodigo()`            | Copia o código para a área de transferência.                                                                                           |
|                             | `pedirNovoCodigo()`         | Fecha a folha "Convidar" e pede confirmação para gerar um novo código.                                                                |
|                             | `opcoesDe(membro)`          | Monta as opções da folha de cada pessoa conforme o vínculo e o número de administradores.                                             |
|                             | `caminho(usuarioId, sufixo)`| Monta a rota `/organizacoes/{id}/membros/{usuarioId}` com o sufixo (`/papel`).                                                        |
|                             | `definirDados(valor: Dados)`, `definirAviso(valor: Aviso)`, `definirMenu(valor: Membro)`, `definirConvidarAberto(valor: boolean)`, `definirEmail(valor: string)` | Atualizadores do `useState`, todos com retorno `void`: guardam o que `carregar` devolveu, a faixa de aviso (`tom` e `mensagem`), a pessoa cuja folha de opções está aberta, se a folha "Convidar" está aberta e o e-mail digitado nela. |
| `TelaGerenciarOrganizacoes` | `carregar(termoBusca)`      | Busca as organizações, descartando respostas de consultas mais antigas.                                                               |
|                             | `atualizar(id, dados)`      | Envia `PATCH` e troca o cartão pelo devolvido; retorna `true` no sucesso.                                                              |
|                             | `salvarEdicao()`            | Valida e envia nome, cidade e descrição.                                                                                               |
|                             | `excluir(organizacao)`      | Envia `DELETE` e tira o cartão da lista.                                                                                               |
|                             | `confirmarAcao()`           | Executa a ação guardada em `acaoPendente` (revogar ou excluir) e fecha o diálogo.                                                     |
|                             | `definirOrganizacoes(valor: SetStateAction<Organizacao[]>)`, `definirBusca(valor: string)`, `definirEditandoId(valor: string)`, `definirAcaoPendente(valor: AcaoPendente)`, `definirErro(valor: string)` | Atualizadores do `useState`, todos com retorno `void`. `definirOrganizacoes` recebe a lista nova ou uma função que troca ou retira um item da lista atual; `definirBusca` dispara a consulta 350 ms depois da última tecla; `definirEditandoId` abre (com o `_id`) ou fecha (com `null`) a edição; `definirAcaoPendente` abre o diálogo de revogar ou excluir (`organizacao` e `tipo`) ou o fecha com `null`. |
|                             | `reset(valores: object)`    | O `reset` do formulário de edição (`edicao`, criado com `useForm` do `react-hook-form`): preenche nome, cidade e descrição atuais. Retorno `void`. |
| `TelaInicio`                | `carregar(continuar)`       | Busca organizações, reuniões e atas pendentes; conta pedidos nas organizações administradas.                                           |
|                             | `responder(convite, aceito)`| Responde convite de organização ou de reunião.                                                                                         |
| `validacao`                 | `validarNomeOrganizacao`, `validarCidade`, `validarDescricao`, `validarCodigo`, `validarEmail` | Devolvem o texto do erro ou `null` quando o valor é válido. |
|                             | `mascararCodigo(texto)`     | Deixa só letras e números em maiúsculas, até 10, com hífen depois do quinto.                                                          |

## Models do Mongoose e módulo do schema

Os três models são `Model<...>` injetados pelo NestJS com `@InjectModel` nos atributos `organizacaoModel`, `usuarioModel` e `comissaoModel` do `OrganizacoesService`; o `UsuarioAtivoGuard` recebe o `UsuarioModel` no atributo `usuarios`. As consultas devolvem uma `Query`, concluída com `.exec()`. Os métodos encadeados na `Query` (`select`, `populate`, `sort` e `lean`) aparecem nas notas dos diagramas de sequência, e não como mensagens, porque não são métodos do model.

| Classe              | Estereótipo          | Método                                                    | Retorno                           | O que faz                                                                                                        |
| ------------------- | -------------------- | --------------------------------------------------------- | --------------------------------- | ---------------------------------------------------------------------------------------------------------------- |
| `OrganizacaoModel`  | `<<Model Mongoose>>` | `create(dados: object)`                                   | `Promise<OrganizacaoDocument>`    | Insere a organização; o schema sorteia `codigo_acesso` e preenche `criado_em` e `atualizado_em`.               |
|                     |                      | `exists(filtro: object)`                                  | `Query<object>`                   | Devolve o `_id` da primeira organização com o mesmo nome, ou `null`.                                             |
|                     |                      | `find(filtro: object)`                                    | `Query<OrganizacaoDocument[]>`    | Lista as organizações do usuário (`membros.usuario_id`) ou, para o administrador do sistema, as que têm o termo no nome. |
|                     |                      | `findById(id: string)`                                    | `Query<OrganizacaoDocument>`      | Busca a organização pelo `_id`.                                                                                  |
|                     |                      | `findOne(filtro: object)`                                 | `Query<OrganizacaoDocument>`      | Busca a organização `APROVADA` com o código de acesso informado.                                                |
|                     |                      | `findByIdAndUpdate(id: string, dados: object, opcoes: object)` | `Query<OrganizacaoDocument>` | Grava os campos recebidos e devolve o documento novo (`new: true`), conferindo o schema (`runValidators: true`). |
|                     |                      | `updateOne(filtro: object, alteracao: object, opcoes: object)` | `Query<UpdateResult>`        | Grava um vínculo ou o código só se o filtro ainda valer; `modifiedCount` diz se gravou. `opcoes` traz `arrayFilters` quando a alteração é num item de `membros`. |
|                     |                      | `deleteOne(filtro: object)`                               | `Query<DeleteResult>`             | Apaga a organização só se ela estiver `REVOGADA`; `deletedCount` diz se apagou.                                |
| `UsuarioModel`      | `<<Model Mongoose>>` | `findById(id: string)`                                    | `Query<UsuarioDocument>`          | Confere a conta de quem cadastra ou pede entrada.                                                                |
|                     |                      | `findOne(filtro: object)`                                 | `Query<UsuarioDocument>`          | Localiza a conta ativa pelo e-mail do convite.                                                                   |
| `ComissaoModel`     | `<<Model Mongoose>>` | `updateMany(filtro: object, alteracao: object)`           | `Query<UpdateResult>`             | Tira a pessoa da equipe de todas as comissões da organização (`$pull` em `membros`).                             |
|                     |                      | `deleteMany(filtro: object)`                              | `Query<DeleteResult>`             | Apaga as comissões da organização excluída.                                                                      |
| `organizacaoSchema` | `<<module>>`         | `gerarCodigoAcesso()`                                     | `string`                          | Sorteia 10 caracteres de `ALFABETO_CODIGO` com `randomInt` do `node:crypto`. É o valor padrão de `codigo_acesso` e é chamado por `gerarNovoCodigo`. |

## Rastreabilidade

RF-ORG-01 a RF-ORG-14; [Diagrama de classes](3%20-%20diagrama-classes-cadastro-organizacao.md) e [Dicionário relacional](4%20-%20dicionario-dados-relacional-cadastro-organizacao.md).
