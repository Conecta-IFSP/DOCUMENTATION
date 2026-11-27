# Dicionário de classes — Informações do sistema

> Padrão da disciplina · Módulo SIS · Complementa o diagrama de classes · Revisado em 27/11/2026

## Resumo

| Classe              | Camada                  | O que representa                                                                                              |
| ------------------- | ----------------------- | ------------------------------------------------------------------------------------------------------------- |
| `TelaInicio`        | Aplicativo              | Tela inicial. Para o administrador do sistema, mostra o item "Sistema", que abre a `TelaSistema`.               |
| `TelaSistema`       | Aplicativo              | Tela "Sistema": mostra as informações, o status e os relatórios e aciona popular dados de teste e resetar.      |
| `ItemMenu`          | Aplicativo (componente) | Item de menu da tela inicial com título, descrição e ícone; o item "Sistema" abre a `TelaSistema`.             |
| `Metrica`           | Aplicativo (componente) | Par rótulo e valor dos cartões "Sobre o sistema", "Servidor" e "Banco de dados".                               |
| `LinhaTecnologia`   | Aplicativo (componente) | Linha do cartão "Tecnologias": nome, uso e versão de uma biblioteca.                                           |
| `ColunaRelatorio`   | Aplicativo (componente) | Coluna do cartão "Relatórios": título e uma ou duas contagens.                                                 |
| `Confirmacao`       | Aplicativo (componente) | Diálogo com uma mensagem e os botões "Cancelar" e "Confirmar", usado antes de popular os dados de teste.      |
| `CampoTexto`        | Aplicativo (componente) | Campo de texto com rótulo e mensagem de erro, usado para digitar "RESET" e a senha.                            |
| `api`               | Aplicativo (serviço)    | Função que envia as requisições à API com a chave do aplicativo e o token da sessão.                           |
| `ToastContext`      | Aplicativo (serviço)    | Aviso flutuante no topo da tela, usado no fim de popular e de resetar.                                         |
| `versao`            | Aplicativo (serviço)    | Leitura da versão do aplicativo na configuração do Expo.                                                       |
| `ApiKeyGuard`       | API                     | Verificação da chave do aplicativo (`X-API-Key`), aplicada a todas as rotas.                                   |
| `JwtAuthGuard`      | API                     | Verificação do token da sessão e da conta no banco.                                                            |
| `AdminSistemaGuard` | API                     | Verificação de que a conta é de administrador do sistema.                                                      |
| `SistemaController` | API                     | Recebe as cinco rotas `/sistema` e repassa ao serviço.                                                         |
| `ResetarSistemaDto` | API                     | Corpo de `POST /sistema/reset`, com a senha do administrador e a palavra de confirmação.                       |
| `SistemaService`    | Negócio                 | Lê as informações e as métricas do servidor, conta documentos, cria os dados de teste, confere a senha e apaga os dados no reset. |
| `Usuario`           | Documento               | Conta de acesso, na coleção `usuarios`.                                                                        |
| `Organizacao`       | Documento               | Organização, na coleção `organizacoes`.                                                                        |
| `MembroOrganizacao` | Subdocumento            | Vínculo de uma conta com uma organização, dentro de `Organizacao`.                                              |
| `Comissao`          | Documento               | Comissão de uma organização, na coleção `comissoes`.                                                            |
| `MembroComissao`    | Subdocumento            | Participação de uma conta numa comissão, dentro de `Comissao`.                                                  |
| `Reuniao`           | Documento               | Reunião de uma comissão, na coleção `reunioes`. Este módulo a conta e a apaga.                                  |
| `Ata`               | Subdocumento            | Ata atual da reunião, dentro de `Reuniao`. Este módulo só conta pela `situacao`.                                |
| `Contador`          | Documento               | Último número de ata usado por comissão e ano, na coleção `contadores`. Este módulo só o apaga.                 |
| `Notificacao`       | Documento               | Aviso mostrado a uma conta, na coleção `notificacoes`. Este módulo só o apaga.                                  |
| `UsuarioModel`      | Dados (model)           | Model do Mongoose da coleção `usuarios`.                                                                        |
| `OrganizacaoModel`  | Dados (model)           | Model do Mongoose da coleção `organizacoes`.                                                                    |
| `ComissaoModel`     | Dados (model)           | Model do Mongoose da coleção `comissoes`.                                                                       |
| `ReuniaoModel`      | Dados (model)           | Model do Mongoose da coleção `reunioes`.                                                                        |
| `Router`            | Biblioteca              | Roteador do `expo-router`, que abre a `TelaSistema`.                                                            |
| `os`                | Biblioteca              | Módulo `node:os` do Node.js: memória, processadores, tempo ativo e sistema operacional do computador da API.   |
| `bcrypt`            | Biblioteca              | Gera o hash da senha das contas de teste e confere a senha do administrador no reset.                          |
| `Connection`        | Biblioteca              | Conexão do Mongoose com o MongoDB, injetada com `@InjectConnection`.                                           |
| `Db`                | Biblioteca              | Banco do driver do MongoDB (`conexao.db`): envia comandos e dá acesso às coleções sem model.                  |
| `Collection`        | Biblioteca              | Coleção do driver do MongoDB, usada no reset para apagar as coleções sem model.                                |

As classes de dados são definidas e descritas campo a campo nos módulos que as criam: `Usuario` em [Cadastro de usuário](../cadastro-usuario/3%20-%20dicionario-dados-classes-cadastro-usuario.md), `Organizacao` e `MembroOrganizacao` em [Cadastro de organização](../cadastro-organizacao/3%20-%20dicionario-dados-classes-cadastro-organizacao.md), `Comissao` e `MembroComissao` em [Cadastro de comissão](../cadastro-comissao/3%20-%20dicionario-dados-classes-cadastro-comissao.md), `Reuniao` e `Notificacao` em [Reuniões](../../3%20-%20Incremento%202/reunioes/3%20-%20dicionario-dados-classes-reunioes.md) e `Ata` e `Contador` em [Atas](../../4%20-%20Incremento%203/atas/3%20-%20dicionario-dados-classes-atas.md). Abaixo aparecem só os campos que este módulo usa.

## `TelaSistema`

Componente da rota `sistema`, em `FRONTEND/src/app/sistema.tsx`. Só existe para a sessão cujo `tipo` é `ADMIN_SISTEMA`. O título do cabeçalho é "Sistema", e puxar a tela para baixo chama `carregar`.

| Atributo           | Tipo TypeScript       | Obrigatório | Regra                                                                                                   |
| ------------------ | --------------------- | ----------: | ------------------------------------------------------------------------------------------------------- |
| `sobre`            | `Sobre \| null`       |         Sim | Resposta de `GET /sistema/sobre`. Começa `null`; os cartões "Sobre o sistema" e "Tecnologias" só aparecem quando preenchido, e o "Servidor" só quando `sobre` e `status` estão preenchidos. |
| `status`           | `Status \| null`      |         Sim | Resposta de `GET /sistema/status`. Começa `null`; o cartão "Banco de dados" só aparece quando preenchido. |
| `relatorios`       | `Relatorios \| null`  |         Sim | Resposta de `GET /sistema/relatorios`. Começa `null`; o cartão "Relatórios" só aparece quando preenchido. |
| `carregando`       | `boolean`             |         Sim | Começa verdadeiro e mostra "Carregando informações do sistema"; vira falso quando as três respostas da primeira carga voltam. |
| `aviso`            | `Aviso \| null`       |         Sim | Faixa de erro no topo da tela. Vira `null` a cada carga bem-sucedida.                                    |
| `acaoEmAndamento`  | `string \| null`      |         Sim | `'popular-teste'` ou `'reset'` enquanto a requisição correspondente não volta. Com `'popular-teste'`, mostra o indicador no botão "Confirmar" do diálogo e desabilita os dois botões; com `'reset'`, mostra o indicador no botão "Apagar tudo" e desabilita "Cancelar". |
| `confirmacaoTeste` | `boolean`             |         Sim | Verdadeiro enquanto o diálogo dos dados de teste está aberto.                                           |
| `resetAberto`      | `boolean`             |         Sim | Verdadeiro enquanto o cartão de confirmação do reset está aberto, no lugar do botão "Resetar sistema".   |
| `palavra`          | `string`              |         Sim | Texto digitado no campo "Digite RESET para confirmar". Começa vazio e volta a vazio em `fecharReset`.    |
| `senha`            | `string`              |         Sim | Texto digitado no campo "Sua senha de administrador". Começa vazio e volta a vazio em `fecharReset`.     |
| `erroReset`        | `string \| null`      |         Sim | Mensagem de erro do reset, mostrada abaixo do campo da senha. Vira `null` ao digitar na senha, ao enviar e em `fecharReset`. |
| `resetValido`      | `boolean`             |         Sim | Calculado a cada desenho: verdadeiro quando `palavra` é igual a "RESET", por comparação exata, e `senha` não está vazia. Habilita "Apagar tudo". |

| Método                | Parâmetros         | Retorno          | O que faz                                                                                                                                                                       |
| --------------------- | ------------------ | ---------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `carregar`            | —                  | `Promise<void>`  | Pede sobre, status e relatórios ao mesmo tempo. Se um dos três falha, mostra a faixa "Não foi possível carregar as informações do sistema" com a mensagem do primeiro que falhou e não atualiza os cartões; se os três dão certo, limpa a faixa e atualiza os três estados. |
| `popularDadosDeTeste` | —                  | `Promise<void>`  | Chama `POST /sistema/popular-teste` e fecha o diálogo. No sucesso, mostra o aviso "3 usuários de teste criados." e chama `carregar` sem esperar; na falha, mostra o aviso de erro com a mensagem da API. |
| `fecharReset`         | —                  | `void`           | Fecha o cartão do reset e apaga `palavra`, `senha` e `erroReset`.                                                                                                              |
| `resetarSistema`      | —                  | `Promise<void>`  | Chama `POST /sistema/reset` com `{ confirmacao: palavra, senha }`. Na falha, grava a mensagem em `erroReset` e mantém o cartão aberto. No sucesso, chama `fecharReset`, mostra o aviso "Sistema resetado. O administrador foi mantido." e chama `carregar` sem esperar. |
| `formatarTempoAtivo`  | `segundos: number` | `string`         | Função do mesmo arquivo. Converte segundos para "Dd Hh" quando há pelo menos um dia e para "Hh Mmin" nos demais casos, descartando as casas decimais.                          |
| `versaoPacote`        | `nome: string`     | `string`         | Função do mesmo arquivo. Devolve a versão do pacote declarada no `package.json` do aplicativo (`dependencies` têm prioridade sobre `devDependencies`), sem o `^` ou o `~` do início, ou texto vazio. |
| `definirSobre`, `definirStatus`, `definirRelatorios` | `valor: Sobre`, `valor: Status`, `valor: Relatorios` | `void` | Atualizadores do `useState`: guardam as três respostas de `carregar`. |
| `definirCarregando`, `definirConfirmacaoTeste`, `definirResetAberto` | `valor: boolean` | `void` | Atualizadores do `useState`: tiram o indicador de carregamento, abrem ou fecham o diálogo dos dados de teste e abrem ou fecham o cartão do reset. |
| `definirAviso`        | `valor: Aviso`     | `void`           | Atualizador do `useState`: mostra a faixa de erro, ou a tira com `null`.                                                                                                       |
| `definirAcaoEmAndamento` | `valor: string` | `void`           | Atualizador do `useState`: `'popular-teste'` ou `'reset'` enquanto a requisição não volta, e `null` depois.                                                                    |
| `definirPalavra`, `definirSenha`, `definirErroReset` | `valor: string` | `void` | Atualizadores do `useState`: guardam o que é digitado nos dois campos do reset e a mensagem de erro mostrada abaixo da senha (`null` para apagá-la). |

A constante `TECNOLOGIAS_APP`, do mesmo arquivo, é uma lista fixa de seis itens `{ nome, versao, uso }`, montada com `versaoPacote` quando o arquivo é carregado: TypeScript, Expo SDK, React Native, React, Expo Router e React Hook Form.

Tipos usados pela tela, definidos no mesmo arquivo:

| Tipo         | Campos                                                                                                                                              |
| ------------ | --------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Sobre`      | `sistema` (`nome`, `versao_api`, `ambiente`, `iniciada_em`, `fuso_horario`), `tecnologias` (lista de `nome`, `versao`, `uso`) e `servidor` (`sistema_operacional`, `arquitetura`, `cpu_modelo`, `nucleos`, `memoria_total_mb`). |
| `Status`     | `servidor.memoria` (`total_mb`, `livre_mb`, `uso_percentual`), `servidor.cpu` (`nucleos`, `uso_percentual`), `servidor.tempo_ativo_segundos`, `api.tempo_ativo_segundos` e `banco_de_dados` (`conectado`, `latencia_ms` opcional e `armazenamento` opcional, com `dados_mb`, `ocupado_mb` e `indices_mb`). Não declara `banco_de_dados.colecoes` nem `verificado_em`, que a API devolve. |
| `Relatorios` | `usuarios` (`total`, `ativos`), `organizacoes` (`total`, `pendentes`, `aprovadas`), `comissoes` (`total`, `ativas`), `reunioes` (`total`, `agendadas`, `canceladas`) e `atas` (`total`, `em_aprovacao`, `aprovadas`). |
| `Aviso`      | `tom` (`erro`, `alerta`, `sucesso` ou `informacao`), `titulo` opcional e `mensagem`. Definido em `FRONTEND/src/components/FaixaAviso.tsx`.          |

## `Metrica`, `LinhaTecnologia` e `ColunaRelatorio`

Componentes declarados em `FRONTEND/src/app/sistema.tsx`, usados só pela `TelaSistema`. As cores vêm do tema ativo.

| Componente        | Propriedade | Tipo                                     | Regra                                                                                    |
| ----------------- | ----------- | ---------------------------------------- | ---------------------------------------------------------------------------------------- |
| `Metrica`         | `rotulo`    | `string`                                 | Texto pequeno, na cor suave do tema.                                                     |
|                   | `valor`     | `string`                                 | Texto maior, em até duas linhas.                                                         |
| `LinhaTecnologia` | `nome`      | `string`                                 | Nome da biblioteca, à esquerda.                                                          |
|                   | `uso`       | `string`                                 | Para que a biblioteca serve, abaixo do nome.                                             |
|                   | `versao`    | `string`                                 | À direita, na cor principal do tema; "—" quando é texto vazio.                           |
| `ColunaRelatorio` | `titulo`    | `string`                                 | Nome do grupo, por exemplo "Usuários".                                                   |
|                   | `itens`     | lista de `{ rotulo: string, valor: string }` | Uma ou duas contagens, cada uma com rótulo e valor.                                  |

## `TelaInicio`

Tela da aba Início, em `FRONTEND/src/app/(abas)/inicio.tsx`. Neste módulo interessa só o que ela mostra ao administrador do sistema.

| Atributo                 | Tipo      | Regra                                                                                                                                                                                                 |
| ------------------------ | --------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `administradorDoSistema` | `boolean` | Verdadeiro quando o `tipo` do usuário da sessão é `ADMIN_SISTEMA`. Nesse caso a tela mostra só os itens "Gerenciar organizações" e "Sistema", e não busca reuniões nem atas pendentes. |

O item "Sistema" é um `ItemMenu` com a descrição "Informações do sistema, status, relatórios e reset." e o ícone `hardware-chip-outline`; a função `aoTocar` que a tela passa a ele executa `router.push('/sistema')`.

## `ItemMenu`

Componente em `FRONTEND/src/components/ItemMenu.tsx`, usado pela `TelaInicio`.

| Atributo ou método | Tipo               | Obrigatório | Regra                                                                  |
| ------------------ | ------------------ | ----------: | ---------------------------------------------------------------------- |
| `titulo`           | `string`           |         Sim | Texto principal do item, por exemplo "Sistema".                        |
| `descricao`        | `string`           |         Não | Texto menor abaixo do título.                                          |
| `icone`            | `IoniconsIconName` |         Sim | Ícone mostrado à esquerda.                                             |
| `aoTocar()`        | `() => void`       |         Sim | Função recebida da tela, chamada quando a pessoa toca no item.         |

## `Confirmacao`

Componente em `FRONTEND/src/components/Confirmacao.tsx`. É um `Modal` do React Native. A `TelaSistema` passa só as propriedades da tabela; as opcionais `titulo`, `rotuloConfirmar` e `perigo` ficam com o valor padrão (sem título, "Confirmar" e falso).

| Propriedade  | Tipo         | Obrigatório | Regra                                                                                       |
| ------------ | ------------ | ----------: | ------------------------------------------------------------------------------------------- |
| `mensagem`   | `string`     |         Não | Texto do diálogo; na `TelaSistema`, "Criar 3 usuários de teste?" enquanto `confirmacaoTeste` é verdadeiro. O diálogo fica visível enquanto a mensagem existe. |
| `confirmar`  | `() => void` |         Sim | Chamada pelo botão "Confirmar"; na `TelaSistema`, `popularDadosDeTeste`.                    |
| `cancelar`   | `() => void` |         Sim | Chamada por "Cancelar" e pelo botão voltar do Android, este só quando não há carregamento.  |
| `carregando` | `boolean`    |         Sim | Quando verdadeiro, mostra um indicador no botão de confirmação e desabilita os dois botões. |

## `CampoTexto`

Componente em `FRONTEND/src/components/CampoTexto.tsx`, que aceita também as propriedades do `TextInput` do React Native. Com `erro` preenchido, a borda fica na cor de erro e a mensagem aparece abaixo do campo, com um ícone de alerta. A `TelaSistema` usa dois:

| Campo                          | Propriedades passadas                                                                                                                                  |
| ------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------ |
| "Digite RESET para confirmar"  | `value` igual a `palavra`, `onChangeText` igual a `definirPalavra`, `autoCapitalize="characters"`, `autoCorrect={false}` e `placeholder="RESET"`.        |
| "Sua senha de administrador"   | `value` igual a `senha`, `onChangeText` que grava a senha e apaga `erroReset`, `secureTextEntry`, `autoCapitalize="none"` e `erro` igual a `erroReset`. |

## `api`

Módulo `FRONTEND/src/services/api.ts`.

| Função         | Parâmetros                                                        | Retorno               | O que faz                                                                                                                                                                                                                     |
| -------------- | ----------------------------------------------------------------- | --------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `api`          | `caminho: string`, `metodo` (`GET` padrão, `POST`, `PATCH` ou `DELETE`), `corpo?: unknown` | `Promise<Resultado<T>>` | Envia a requisição com `X-API-Key` e `Authorization: Bearer`. Devolve `{ ok: true, dados }` ou `{ ok: false, erro }`, com a primeira mensagem da API. Sem rede: "Não foi possível conectar ao servidor.". Em 401 com token, apaga o token e encerra a sessão. |
| `obterToken`   | —                                                                 | `Promise<string \| null>` | Lê o token com a chave `conecta_mais_token` do `SecureStore` (no navegador, do `localStorage`).                                                                                                                       |
| `removerToken` | —                                                                 | `Promise<void>`       | Apaga o token guardado.                                                                                                                                                                                                       |

## `ToastContext`

Módulo `FRONTEND/src/contexts/ToastContext.tsx`. A tela obtém `mostrarToast` com `useToast()`.

| Função         | Parâmetros                                                                                   | Retorno | O que faz                                                                                                                                              |
| -------------- | -------------------------------------------------------------------------------------------- | ------- | ------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `mostrarToast` | `mensagem: string`, `tipo` (`sucesso`, `erro` ou `informacao`, padrão `informacao`), `duracao` (milissegundos, padrão 4.000) | `void`  | Mostra o aviso no topo da tela, com ícone e fundo da cor do tipo, por cima de qualquer conteúdo. Some depois de `duracao` ou com um toque. Um novo aviso substitui o anterior. |

## `versao`

Módulo `FRONTEND/src/utils/versao.ts`.

| Função        | Parâmetros | Retorno                                     | O que faz                                                                                                                      |
| ------------- | ---------- | ------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------ |
| `versaoDoApp` | —          | `{ versao: string \| null, build: string \| null }` | Lê `version` da configuração do Expo (`app.json`). A `TelaSistema` usa só `versao`, e mostra "—" quando é `null`.            |

## `ApiKeyGuard`

Em `BACKEND/src/auth/api-key.guard.ts`, registrado para toda a API como `APP_GUARD`.

| Método                    | Parâmetros                             | Retorno   | O que faz                                                                                                                                                                                                      |
| ------------------------- | -------------------------------------- | --------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `canActivate`             | `context: ExecutionContext`            | `boolean` | Deixa passar `OPTIONS` e os dois retornos do login com Google. Nas demais, lê `API_ACCESS_TOKEN` (sem ela: 503) e compara com o cabeçalho `X-API-Key`; diferente ou ausente: 401 "Chave de acesso da API inválida ou ausente". |
| `isPublicBrowserRedirect` | `request: Request`                     | `boolean` | Verdadeiro para `GET /auth/google/callback` e `GET /auth/google/callback-web`.                                                                                                                                 |
| `chavesIguais`            | `recebida: string`, `esperada: string` | `boolean` | Verdadeiro só se os dois textos têm o mesmo tamanho em bytes e `timingSafeEqual` os acha iguais.                                                                                                               |

## `JwtAuthGuard`

Em `BACKEND/src/auth/jwt-auth.guard.ts`.

| Atributo      | Tipo              | Regra                                         |
| ------------- | ----------------- | --------------------------------------------- |
| `jwtService`  | `JwtService`      | Confere assinatura e validade do token.       |
| `usuarios`    | `Model<UsuarioDocument>` | Busca a conta dona do token.           |

| Método         | Parâmetros                  | Retorno            | O que faz                                                                                                                                                                                                                                       |
| -------------- | --------------------------- | ------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `canActivate`  | `context: ExecutionContext` | `Promise<boolean>` | Extrai o token, confere com `verifyAsync`, exige `sub` com formato de `ObjectId`, busca a conta com `_id` igual a `sub` e `ativo` verdadeiro, recusa token anterior a `senha_alterada_em` e copia `tipo` e `email` do banco para a requisição. Falhas: 401. |
| `extrairToken` | `requisicao: Request`       | `string \| undefined` | Separa o cabeçalho `Authorization` no espaço e devolve a segunda parte só quando a primeira é `Bearer`.                                                                                                                                       |

## `AdminSistemaGuard`

Em `BACKEND/src/auth/admin-sistema.guard.ts`. O mesmo arquivo define `UsuarioComumGuard`, que faz o contrário e não é usado neste módulo.

| Método        | Parâmetros                  | Retorno   | O que faz                                                                                                                                          |
| ------------- | --------------------------- | --------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| `canActivate` | `context: ExecutionContext` | `boolean` | Compara `requisicao.usuario.tipo` com `ADMIN_SISTEMA`. Diferente: 403 "Apenas administradores do sistema podem realizar esta operação". |

## `SistemaController`

Em `BACKEND/src/sistema/sistema.controller.ts`, com `@Controller('sistema')` e `@UseGuards(JwtAuthGuard, AdminSistemaGuard)`. Os métodos `status`, `sobre`, `relatorios` e `popularDadosDeTeste` apenas chamam o método de mesmo nome do serviço e devolvem o resultado. O método `reset` recebe a requisição autenticada (`@Req()`, tipo `RequisicaoAutenticada`) e o corpo `ResetarSistemaDto` (`@Body()`), que o `ValidationPipe` já validou, e chama `reset(requisicao.usuario.sub, dto.senha)`; a `confirmacao` não é repassada. As rotas estão no [diagrama de classes](3%20-%20diagrama-classes-informacao-sistemas.md).

## `ResetarSistemaDto`

Em `BACKEND/src/sistema/dto/resetar-sistema.dto.ts`. O `ValidationPipe` da API, com `whitelist: true`, descarta qualquer outro campo do corpo.

| Atributo      | Tipo     | Obrigatório | Validação                                                                                                                                  | Mensagem de erro                         |
| ------------- | -------- | ----------: | ------------------------------------------------------------------------------------------------------------------------------------------ | ---------------------------------------- |
| `senha`       | `string` |         Sim | `@IsString`: precisa ser texto. Não há tamanho mínimo nem máximo; o texto vazio passa nesta validação e é recusado depois, na comparação com o hash. | "Informe a senha do administrador" (400) |
| `confirmacao` | `string` |         Sim | `@Equals('RESET')`: igual a "RESET", por comparação exata. Texto vazio, campo ausente, letras minúsculas e espaços nas pontas são recusados. | "Digite RESET para confirmar" (400)      |

## `SistemaService`

Em `BACKEND/src/sistema/sistema.service.ts`. No carregamento do arquivo, lê o `package.json` da API com `createRequire`, para obter `version`, `dependencies` e `devDependencies`.

| Atributo        | Tipo                  | Regra                                                                                    |
| --------------- | --------------------- | ---------------------------------------------------------------------------------------- |
| `usuarios`      | `Model<UsuarioDocument>` | Model da coleção `usuarios`.                                                          |
| `organizacoes`  | `Model<OrganizacaoDocument>` | Model da coleção `organizacoes`.                                                  |
| `comissoes`     | `Model<ComissaoDocument>` | Model da coleção `comissoes`.                                                        |
| `reunioes`      | `Model<ReuniaoDocument>` | Model da coleção `reunioes`; usado no status, nos relatórios e no reset.              |
| `conexao`       | `Connection`          | Conexão do Mongoose; usada para enviar `ping`, `dbStats` e `buildInfo` e, no reset, para apagar `anexos.files`, `anexos.chunks`, `contadores` e `notificacoes` pelo nome. |

| Método                           | Parâmetros                         | Retorno                      | O que faz                                                                                                                                                                                           |
| -------------------------------- | ---------------------------------- | ---------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `sobre`                          | —                                  | objeto sobre                 | Monta o nome, a versão da API, o ambiente, o início e o fuso da API, as sete tecnologias com a versão e os dados fixos do servidor (sistema operacional, arquitetura, processador, núcleos e memória total). |
| `status`                         | —                                  | objeto de status             | Lê memória do computador pelo `node:os`, chama ao mesmo tempo `usoCpuPercentual` e `infoBanco` e devolve também o tempo ativo do computador e o da API.                                            |
| `relatorios`                     | —                                  | objeto de relatórios         | Faz treze contagens: usuários (todos e ativos), organizações (todas, pendentes e aprovadas), comissões (todas e ativas), reuniões (todas, agendadas e canceladas) e atas (todas, em aprovação e aprovadas). |
| `reset`                          | `adminId: string`, `senha: string` | objeto de reset              | Chama `confirmarSenhaAdmin` e, se ela não recusar, apaga ao mesmo tempo e sem transação as contas que não são `ADMIN_SISTEMA`, todas as organizações, comissões e reuniões, os arquivos do GridFS, os contadores e todas as notificações. |
| `popularDadosDeTeste`            | —                                  | objeto com as contas criadas | Cria três contas, uma organização aprovada e uma comissão, com os dados das variáveis `TEST_USER_*` e `TEST_USERS_PASSWORD`.                                                                       |
| `confirmarSenhaAdmin`            | `adminId: string`, `senha: string` | `Promise<void>`              | Busca a conta por `_id` com `senha_hash`. Sem hash: 400 "Defina uma senha na sua conta para resetar o sistema". `bcrypt.compare` falso: 400 "Senha incorreta".                                      |
| `usoCpuPercentual`               | —                                  | `Promise<number>`            | Soma os tempos de todos os processadores em dois instantes separados por 200 ms e devolve a parte não ociosa em porcentagem inteira; 0 se o tempo total não avançou.                               |
| `versaoMongo`                    | —                                  | `Promise<string>`            | Envia `buildInfo` e devolve `version`; texto vazio se o comando falhar.                                                                                                                             |
| `infoBanco`                      | —                                  | objeto do banco              | Mede o `ping` em ms, lê `dbStats` e conta `usuarios`, `organizacoes`, `comissoes` e `reunioes`. Se qualquer passo falhar, devolve `{ conectado: false }`.                                          |
| `obterConfiguracaoUsuariosTeste` | —                                  | objeto de configuração       | Lê as sete variáveis de ambiente, apara os textos e converte os e-mails para minúsculas. Se alguma faltar, lança 400 "Configure as contas de teste no .env: " seguido dos nomes que faltam.         |

### Objetos devolvidos

Não têm classe no código: são montados no próprio método.

| Objeto         | Campo                                     | Tipo               | Regra                                                                                   |
| -------------- | ----------------------------------------- | ------------------ | --------------------------------------------------------------------------------------- |
| Sobre          | `sistema.nome`                            | `string`           | Sempre "Conecta+".                                                                      |
|                | `sistema.versao_api`                      | `string`           | Campo `version` do `package.json` da API.                                               |
|                | `sistema.ambiente`                        | `string`           | Variável `NODE_ENV`; sem ela, "development".                                            |
|                | `sistema.iniciada_em`                     | `Date`             | Horário atual menos `process.uptime()`: quando o processo da API começou.               |
|                | `sistema.fuso_horario`                    | `string`           | Fuso do processo, por `Intl.DateTimeFormat().resolvedOptions().timeZone`.               |
|                | `tecnologias`                             | lista de objetos   | Sete itens `{ nome, versao, uso }`: Node.js, TypeScript, NestJS, MongoDB, Mongoose, bcrypt e class-validator. A versão do Node.js vem de `process.versions.node`, a do MongoDB de `buildInfo`, e as demais do `package.json`, sem `^` ou `~`. |
|                | `servidor.sistema_operacional`            | `string`           | `os.type()` e `os.release()` separados por espaço.                                      |
|                | `servidor.arquitetura`                    | `string`           | `os.arch()`, por exemplo `x64`.                                                         |
|                | `servidor.cpu_modelo`                     | `string \| null`   | Modelo do primeiro processador de `os.cpus()`, aparado; `null` sem processadores.       |
|                | `servidor.nucleos`                        | `number` (inteiro) | Quantidade de processadores lógicos de `os.cpus()`. Não exibido.                        |
|                | `servidor.memoria_total_mb`               | `number` (inteiro) | `os.totalmem()` em MB, arredondado. Não exibido.                                        |
| Status         | `servidor.memoria.total_mb`               | `number` (inteiro) | Memória total do computador em MB (bytes ÷ 1.048.576, arredondado).                     |
|                | `servidor.memoria.livre_mb`               | `number` (inteiro) | Memória livre do computador em MB, arredondada.                                         |
|                | `servidor.memoria.uso_percentual`         | `number` (inteiro) | (total − livre) ÷ total × 100, arredondado.                                             |
|                | `servidor.cpu.nucleos`                    | `number` (inteiro) | Quantidade de processadores lógicos informada por `os.cpus()`.                          |
|                | `servidor.cpu.uso_percentual`             | `number` (inteiro) | Uso da CPU de todos os processadores nos 200 ms da medição, de 0 a 100.                 |
|                | `servidor.tempo_ativo_segundos`           | `number` (inteiro) | Segundos desde que o computador foi ligado (`os.uptime()`).                             |
|                | `api.tempo_ativo_segundos`                | `number` (inteiro) | Segundos desde que o processo da API começou (`process.uptime()`).                      |
|                | `banco_de_dados.conectado`                | `boolean`          | `true` quando `ping`, `dbStats` e as contagens funcionaram; `false` quando um deles falhou, e então o objeto só tem este campo. |
|                | `banco_de_dados.latencia_ms`              | `number` (inteiro) | Tempo do comando `ping`, em milissegundos, arredondado.                                 |
|                | `banco_de_dados.armazenamento`            | objeto             | `dados_mb` (`dataSize`), `ocupado_mb` (`storageSize`) e `indices_mb` (`indexSize`), em MB com duas casas decimais. |
|                | `banco_de_dados.colecoes`                 | objeto             | `usuarios`, `organizacoes`, `comissoes` e `reunioes`, total de documentos de cada coleção. Não exibido. |
|                | `verificado_em`                           | `Date`             | Horário da consulta. Não exibido.                                                       |
| Relatórios     | `usuarios.total`, `usuarios.ativos`       | `number` (inteiro) | Todas as contas, inclusive administradores; contas com `ativo` verdadeiro.              |
|                | `organizacoes.total`, `.pendentes`, `.aprovadas` | `number` (inteiro) | Todas; `status` `PENDENTE`; `status` `APROVADA`.                                 |
|                | `comissoes.total`, `comissoes.ativas`     | `number` (inteiro) | Todas; `ativo` verdadeiro.                                                              |
|                | `reunioes.total`, `.agendadas`, `.canceladas` | `number` (inteiro) | Todas; `situacao` `AGENDADA`, inclusive as que já terminaram; `situacao` `CANCELADA`. |
|                | `atas.total`, `.em_aprovacao`, `.aprovadas` | `number` (inteiro) | Reuniões com `ata.situacao` presente, em qualquer situação; `ata.situacao` `EM_APROVACAO`; `ata.situacao` `APROVADA`. |
| Dados de teste | `status`                                  | `string`           | Sempre `"dados_de_teste_criados"`.                                                      |
|                | `usuarios`                                | lista de objetos   | Três itens `{ nome, email, cenario }`. Não traz senha.                                  |
| Reset          | `status`                                  | `string`           | Sempre `"sistema_resetado"`.                                                            |
|                | `resetado_em`                             | `Date`             | Horário do fim das exclusões.                                                           |

## Models do Mongoose e bibliotecas

Os quatro models são `Model<...>` injetados pelo NestJS com `@InjectModel` nos atributos `usuarios`, `organizacoes`, `comissoes` e `reunioes` do `SistemaService`; o `JwtAuthGuard` recebe o `UsuarioModel` no atributo `usuarios`. As consultas devolvem uma `Query`, concluída com `.exec()`; o `select("+senha_hash")` do reset aparece numa nota do diagrama de sequência. As bibliotecas aparecem só com os métodos chamados nos diagramas de sequência.

| Classe             | Estereótipo          | Método                                 | Retorno                         | O que faz                                                                                              |
| ------------------ | -------------------- | -------------------------------------- | ------------------------------- | ------------------------------------------------------------------------------------------------------ |
| `UsuarioModel`     | `<<Model Mongoose>>` | `findOne(filtro: object)`              | `Query<UsuarioDocument>`        | No `JwtAuthGuard`, busca a conta ativa dona do token.                                                  |
|                    |                      | `findById(id: string)`                 | `Query<UsuarioDocument>`        | No reset, busca a conta do administrador para conferir a senha.                                       |
|                    |                      | `countDocuments(filtro: object)`       | `Query<number>`                 | Conta as contas (todas ou só as ativas).                                                               |
|                    |                      | `create(dados: object)`                | `Promise<UsuarioDocument>`      | Cria cada conta de teste; e-mail repetido é recusado pelo índice único.                              |
|                    |                      | `deleteMany(filtro: object)`           | `Query<DeleteResult>`           | No reset, apaga as contas cujo `tipo` não é `ADMIN_SISTEMA`.                                           |
| `OrganizacaoModel` | `<<Model Mongoose>>` | `countDocuments(filtro: object)`       | `Query<number>`                 | Conta as organizações (todas, pendentes e aprovadas).                                                  |
|                    |                      | `create(dados: object)`                | `Promise<OrganizacaoDocument>`  | Cria a "Organização de Teste".                                                                         |
|                    |                      | `deleteMany(filtro: object)`           | `Query<DeleteResult>`           | No reset, apaga todas as organizações.                                                                 |
| `ComissaoModel`    | `<<Model Mongoose>>` | `countDocuments(filtro: object)`       | `Query<number>`                 | Conta as comissões (todas e ativas).                                                                   |
|                    |                      | `create(dados: object)`                | `Promise<ComissaoDocument>`     | Cria a "Comissão de Teste".                                                                            |
|                    |                      | `deleteMany(filtro: object)`           | `Query<DeleteResult>`           | No reset, apaga todas as comissões.                                                                    |
| `ReuniaoModel`     | `<<Model Mongoose>>` | `countDocuments(filtro: object)`       | `Query<number>`                 | Conta as reuniões e as atas pela situação.                                                             |
|                    |                      | `deleteMany(filtro: object)`           | `Query<DeleteResult>`           | No reset, apaga todas as reuniões, com as atas e as votações dentro delas.                            |
| `Router`           | `<<biblioteca>>`     | `push(rota: Href)`                     | `void`                          | Abre `/sistema` por cima da tela inicial (`expo-router` 58.0.33).                                      |
| `os`               | `<<biblioteca>>`     | `totalmem()`, `freemem()`              | `number`                        | Memória total e livre do computador da API, em bytes.                                                  |
|                    |                      | `cpus()`                               | `CpuInfo[]`                     | Um item por processador lógico, com o modelo e os tempos (`times`) usados na medição do uso da CPU.  |
|                    |                      | `uptime()`                             | `number`                        | Segundos desde que o computador foi ligado.                                                            |
|                    |                      | `type()`, `release()`, `arch()`        | `string`                        | Nome e versão do sistema operacional e arquitetura do processador.                                     |
| `bcrypt`           | `<<biblioteca>>`     | `hash(dado: string, custo: number)`    | `Promise<string>`               | Gera o hash da senha das contas de teste, com custo 10.                                                |
|                    |                      | `compare(dado: string, hash: string)`  | `Promise<boolean>`              | Diz se a senha digitada no reset corresponde ao `senha_hash` do administrador.                         |
| `Connection`       | `<<biblioteca>>`     | atributo `db: Db`                      | —                               | O banco da conexão do Mongoose (`conexao.db`).                                                         |
| `Db`               | `<<biblioteca>>`     | `command(comando: object)`             | `Promise<Document>`             | Envia ao MongoDB `ping`, `dbStats` e `buildInfo`.                                                      |
|                    |                      | `collection(nome: string)`             | `Collection`                    | Dá acesso a `anexos.files`, `anexos.chunks`, `contadores` e `notificacoes`, que não têm model aqui.   |
| `Collection`       | `<<biblioteca>>`     | `deleteMany(filtro: object)`           | `Promise<DeleteResult>`         | No reset, apaga todos os documentos da coleção (filtro `{}`).                                          |

## Classes de dados (campos usados)

### `Usuario`

| Atributo            | Tipo TypeScript/Mongoose     | Tipo BSON  | Obrigatório | Uso neste módulo                                                                                         |
| ------------------- | ---------------------------- | ---------- | ----------: | -------------------------------------------------------------------------------------------------------- |
| `_id`               | `Types.ObjectId`             | `objectId` |         Sim | Comparado com o `sub` do token no `JwtAuthGuard`; no reset, identifica a conta cuja senha é conferida.   |
| `nome`              | `string`                     | `string`   |         Sim | Gravado nas contas de teste.                                                                             |
| `email`             | `string`                     | `string`   |         Sim | Único, em minúsculas. Gravado nas contas de teste; repetido, a criação falha.                            |
| `senha_hash`        | `string`                     | `string`   |         Não | Hash bcrypt de custo 10. Gravado nas contas de teste e lido no reset, com `select('+senha_hash')`, porque o schema o esconde das consultas (`select: false`). Nunca é devolvido. |
| `tipo`              | `TipoUsuario` (enum string)  | `string`   |         Sim | Padrão `USUARIO`. `ADMIN_SISTEMA` libera este módulo e preserva a conta no reset.                        |
| `ativo`             | `boolean`                    | `bool`     |         Sim | Padrão verdadeiro. Contado em `usuarios.ativos`; conta inativa não passa no `JwtAuthGuard`.              |
| `email_verificado`  | `boolean`                    | `bool`     |         Sim | Padrão verdadeiro; as contas de teste nascem com esse valor.                                             |
| `senha_alterada_em` | `Date`                       | `date`     |         Não | Token emitido antes desse instante é recusado.                                                           |

### `Organizacao` e `MembroOrganizacao`

| Classe              | Atributo        | Tipo                              | Obrigatório | Uso neste módulo                                                                            |
| ------------------- | --------------- | --------------------------------- | ----------: | ------------------------------------------------------------------------------------------- |
| `Organizacao`       | `nome`          | `string`                          |         Sim | "Organização de Teste" nos dados de teste.                                                  |
|                     | `cidade`        | `string`                          |         Sim | "São Paulo" nos dados de teste.                                                             |
|                     | `descricao`     | `string`                          |         Não | "Criada para testes." nos dados de teste.                                                   |
|                     | `status`        | `StatusOrganizacao` (enum string) |         Sim | Contado por `PENDENTE` e `APROVADA`; `APROVADA` nos dados de teste.                         |
|                     | `criada_por`    | `Types.ObjectId`                  |         Sim | Conta "admin de organização" nos dados de teste.                                            |
|                     | `codigo_acesso` | `string`                          |         Sim | Gerado pelo valor padrão do schema; não é lido nem devolvido por este módulo.               |
|                     | `membros`       | `MembroOrganizacao[]`             |         Sim | Dois membros aprovados nos dados de teste.                                                  |
| `MembroOrganizacao` | `usuario_id`    | `Types.ObjectId`                  |         Sim | Referência à conta em `usuarios`.                                                           |
|                     | `papel`         | `PapelOrganizacao` (enum string)  |         Sim | `ADMIN` ou `MEMBRO`.                                                                        |
|                     | `status`        | `StatusMembroOrganizacao` (enum string) |   Sim | `APROVADO` nos dados de teste.                                                              |
|                     | `solicitado_em` | `Date`                            |         Sim | Padrão: horário da gravação. Horário da criação nos dados de teste.                         |
|                     | `aprovado_em`   | `Date`                            |         Não | Horário da criação nos dados de teste.                                                      |

### `Comissao` e `MembroComissao`

| Classe           | Atributo         | Tipo                          | Obrigatório | Uso neste módulo                                                                       |
| ---------------- | ---------------- | ----------------------------- | ----------: | -------------------------------------------------------------------------------------- |
| `Comissao`       | `nome`           | `string`                      |         Sim | "Comissão de Teste" nos dados de teste.                                                |
|                  | `descricao`      | `string`                      |         Não | "Criada para testes." nos dados de teste.                                              |
|                  | `organizacao_id` | `Types.ObjectId`              |         Sim | A organização criada junto.                                                            |
|                  | `membros`        | `MembroComissao[]`            |         Sim | Precisa ter exatamente um `RESPONSAVEL` e nenhuma conta repetida.                      |
|                  | `ativo`          | `boolean`                     |         Sim | Padrão verdadeiro. Contado em `comissoes.ativas`.                                      |
| `MembroComissao` | `usuario_id`     | `Types.ObjectId`              |         Sim | Conta "membro de organização e comissão" nos dados de teste.                           |
|                  | `papel`          | `PapelComissao` (enum string) |         Sim | `RESPONSAVEL` nos dados de teste.                                                      |
|                  | `adicionado_em`  | `Date`                        |         Sim | Padrão: horário da gravação. Horário da criação nos dados de teste.                    |

### `Reuniao` e `Ata`

| Classe    | Atributo   | Tipo                            | Obrigatório | Uso neste módulo                                                                                                  |
| --------- | ---------- | ------------------------------- | ----------: | ----------------------------------------------------------------------------------------------------------------- |
| `Reuniao` | `situacao` | `SituacaoReuniao` (enum string) |         Não | Sem `required`, mas com o padrão `AGENDADA`, por isso sempre gravado. Contado por `AGENDADA` e `CANCELADA`. A reunião encerrada continua `AGENDADA`.                 |
|           | `ata`      | `Ata`                           |         Não | Ata atual. Só existe depois que o presidente começa a ata; a presença de `ata.situacao` define `atas.total`.      |
| `Ata`     | `situacao` | `SituacaoAta` (enum string)     |         Sim | Contado por `EM_APROVACAO` e `APROVADA`. As atas reprovadas substituídas, guardadas em `atas_canceladas`, não são contadas. |

O reset apaga a reunião inteira, com a ata, as atas canceladas, as votações, as presenças e os dados dos anexos gravados dentro dela.

### `Contador` e `Notificacao`

Este módulo não lê nem grava campos dessas classes: só apaga todos os documentos no reset. `Contador` tem `_id` (texto no formato `atas:<id da comissão>:<ano>`) e `valor` (último número de ata usado); depois do reset, a primeira ata aprovada de cada comissão volta a receber o número 1. `Notificacao` tem `usuario_id` e `chave`, que formam um par único; o reset apaga todas, inclusive as das contas de administrador do sistema.

## Enumerações

| Enumeração                | Valores                                                       | Onde aparece                 |
| ------------------------- | ------------------------------------------------------------- | ---------------------------- |
| `TipoUsuario`             | `USUARIO`, `ADMIN_SISTEMA`                                    | `Usuario.tipo`               |
| `StatusOrganizacao`       | `PENDENTE`, `APROVADA`, `REVOGADA`                            | `Organizacao.status`         |
| `PapelOrganizacao`        | `ADMIN`, `MEMBRO`                                             | `MembroOrganizacao.papel`    |
| `StatusMembroOrganizacao` | `PENDENTE`, `APROVADO`, `REJEITADO`, `CONVIDADO`              | `MembroOrganizacao.status`   |
| `PapelComissao`           | `RESPONSAVEL`, `MEMBRO`                                       | `MembroComissao.papel`       |
| `SituacaoReuniao`         | `AGENDADA`, `CANCELADA`                                       | `Reuniao.situacao`           |
| `SituacaoAta`             | `RASCUNHO`, `EM_APROVACAO`, `APROVADA`, `REPROVADA`, `CANCELADA` | `Ata.situacao`            |

## Rastreabilidade

RF-SIS-01 a RF-SIS-05; [Diagrama de classes](3%20-%20diagrama-classes-informacao-sistemas.md) e [Dicionário relacional](4%20-%20dicionario-dados-relacional-informacao-sistemas.md).
