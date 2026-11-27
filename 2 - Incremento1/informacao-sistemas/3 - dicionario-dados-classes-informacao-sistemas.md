# Dicionário de classes — Informações do sistema

> Padrão da disciplina · Módulo SIS · Complementa o diagrama de classes · Revisado em 26/11/2026

## Resumo

| Classe              | Camada               | O que representa                                                                                         |
| ------------------- | -------------------- | -------------------------------------------------------------------------------------------------------- |
| `TelaInicio`        | Aplicativo           | Tela inicial. Para o administrador do sistema, mostra o item "Sistema", que abre a `TelaSistema`.          |
| `TelaSistema`       | Aplicativo           | Tela "Sistema": mostra status e relatórios e aciona popular dados de teste e resetar.                      |
| `Confirmacao`       | Aplicativo (componente) | Diálogo com uma mensagem e os botões "Cancelar" e "Confirmar", usado antes de popular os dados de teste. |
| `FolhaInferior`     | Aplicativo (componente) | Folha que sobe da parte de baixo da tela; no reset, mostra o aviso, o campo da palavra e o botão final.  |
| `CampoTexto`        | Aplicativo (componente) | Campo de texto com rótulo, usado para digitar "RESETAR".                                                 |
| `api`               | Aplicativo (serviço) | Função que envia as requisições à API com a chave do aplicativo e o token da sessão.                      |
| `autenticacaoLocal` | Aplicativo (serviço) | Pedido de biometria ou PIN do aparelho, feito antes de enviar o reset.                                    |
| `ApiKeyGuard`       | API                  | Verificação da chave do aplicativo (`X-API-Key`), aplicada a todas as rotas.                              |
| `JwtAuthGuard`      | API                  | Verificação do token da sessão e da conta no banco.                                                       |
| `AdminSistemaGuard` | API                  | Verificação de que a conta é de administrador do sistema.                                                 |
| `SistemaController` | API                  | Recebe as quatro rotas `/sistema` e repassa ao serviço.                                                   |
| `ResetSistemaDto`   | API                  | Corpo de `POST /sistema/reset`, com a palavra de confirmação.                                             |
| `SistemaService`    | Negócio              | Lê as métricas do servidor, conta documentos, cria os dados de teste e apaga os dados no reset.           |
| `Usuario`           | Documento            | Conta de acesso, na coleção `usuarios`.                                                                   |
| `Organizacao`       | Documento            | Organização, na coleção `organizacoes`.                                                                   |
| `MembroOrganizacao` | Subdocumento         | Vínculo de uma conta com uma organização, dentro de `Organizacao`.                                         |
| `Comissao`          | Documento            | Comissão de uma organização, na coleção `comissoes`.                                                       |
| `MembroComissao`    | Subdocumento         | Participação de uma conta numa comissão, dentro de `Comissao`.                                             |
| `Reuniao`           | Documento            | Reunião de uma comissão, na coleção `reunioes`. Este módulo só a apaga.                                    |
| `Contador`          | Documento            | Último número de ata usado por comissão e ano, na coleção `contadores`. Este módulo só o apaga.            |
| `Notificacao`       | Documento            | Aviso mostrado a uma conta, na coleção `notificacoes`. Este módulo só o apaga.                             |

As classes de dados são definidas e descritas campo a campo nos módulos que as criam: `Usuario` em [Cadastro de usuário](../cadastro-usuario/3%20-%20dicionario-dados-classes-cadastro-usuario.md), `Organizacao` e `MembroOrganizacao` em [Cadastro de organização](../cadastro-organizacao/3%20-%20dicionario-dados-classes-cadastro-organizacao.md), `Comissao` e `MembroComissao` em [Cadastro de comissão](../cadastro-comissao/3%20-%20dicionario-dados-classes-cadastro-comissao.md), `Reuniao` e `Notificacao` em [Reuniões](../../3%20-%20Incremento2/reunioes/3%20-%20dicionario-dados-classes-reunioes.md) e `Contador` em [Atas](../../4%20-%20Incremento3/atas/3%20-%20dicionario-dados-classes-atas.md). Abaixo aparecem só os campos que este módulo usa.

## `TelaSistema`

Componente da rota `sistema`, em `FRONTEND/src/app/sistema.tsx`. Só existe para a sessão cujo `tipo` é `ADMIN_SISTEMA`.

| Atributo           | Tipo TypeScript       | Obrigatório | Regra                                                                                                   |
| ------------------ | --------------------- | ----------: | ------------------------------------------------------------------------------------------------------- |
| `status`           | `Status \| null`      |         Sim | Resposta de `GET /sistema/status`. Começa `null`; o cartão "Status do servidor" só aparece quando preenchido. |
| `relatorios`       | `Relatorios \| null`  |         Sim | Resposta de `GET /sistema/relatorios`. Começa `null`; o cartão "Relatórios" só aparece quando preenchido. |
| `carregando`       | `boolean`             |         Sim | Começa verdadeiro e mostra "Carregando informações do sistema"; vira falso no fim da primeira carga.     |
| `aviso`            | `Aviso \| null`       |         Sim | Faixa de sucesso ou de erro no topo da tela. Vira `null` a cada carga bem-sucedida.                      |
| `acaoEmAndamento`  | `string \| null`      |         Sim | `'popular-teste'` ou `'reset'` enquanto a requisição correspondente não volta. Com `'popular-teste'`, desabilita o diálogo; com `'reset'`, mostra o indicador de carregamento no botão "Resetar sistema" da tela. |
| `confirmacaoReset` | `boolean`             |         Sim | Verdadeiro enquanto a folha do reset está aberta.                                                       |
| `textoReset`       | `string`              |         Sim | Texto digitado no campo "Digite RESETAR para confirmar". Começa vazio e volta a vazio a cada abertura da folha. O botão da folha só fica ativo quando é igual a `CONFIRMACAO_RESET`. |
| `confirmacaoTeste` | `boolean`             |         Sim | Verdadeiro enquanto o diálogo dos dados de teste está aberto.                                           |

| Método                | Parâmetros         | Retorno          | O que faz                                                                                                                                                                       |
| --------------------- | ------------------ | ---------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `carregar`            | —                  | `Promise<void>`  | Pede status e relatórios ao mesmo tempo. Se um dos dois falha, mostra a faixa de erro e não atualiza os cartões; se os dois dão certo, limpa a faixa e atualiza os dois cartões. |
| `popularDadosDeTeste` | —                  | `Promise<void>`  | Chama `POST /sistema/popular-teste` e fecha o diálogo. No sucesso, espera `carregar` e depois mostra a faixa verde, que por isso continua na tela; na falha, mostra a faixa vermelha. |
| `abrirReset`          | —                  | `void`           | Apaga `textoReset` e abre a folha do reset.                                                                                                                                     |
| `fecharReset`         | —                  | `void`           | Fecha a folha, exceto enquanto `acaoEmAndamento` é `'reset'`. É chamada pelo toque fora da folha e pelo botão voltar do Android.                                               |
| `resetarSistema`      | —                  | `Promise<void>`  | Não faz nada se `textoReset` não for "RESETAR". Senão, fecha a folha e chama `autenticarComDispositivo`; sem identidade confirmada, mostra "Não foi possível confirmar sua identidade.". Confirmada, chama `POST /sistema/reset` com `{ confirmacao: textoReset }`; no sucesso, espera `carregar` e depois mostra a faixa verde; na falha, a vermelha. |
| `formatarTempoAtivo`  | `segundos: number` | `string`         | Função do mesmo arquivo. Converte segundos para "Hh Mmin", descartando as casas decimais das horas e dos minutos.                                                              |

Tipos usados pela tela, definidos no mesmo arquivo:

| Tipo         | Campos                                                                                                                                              |
| ------------ | --------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Status`     | `servidor.memoria` (`total_mb`, `livre_mb`, `uso_percentual`), `servidor.cpu` (`nucleos`, `carga_media_1min`), `servidor.tempo_ativo_segundos` e `banco_de_dados` (`conectado`, `colecoes`). |
| `Relatorios` | `usuarios` (`total`, `ativos`), `organizacoes` (`total`, `pendentes`, `aprovadas`) e `comissoes` (`total`, `ativas`).                                |
| `Aviso`      | `tom` (`erro`, `alerta`, `sucesso` ou `informacao`), `titulo` opcional e `mensagem`. Definido em `FRONTEND/src/components/FaixaAviso.tsx`.          |

A constante `CONFIRMACAO_RESET`, no mesmo arquivo, vale "RESETAR". O backend tem uma constante de mesmo nome e valor em `reset-sistema.dto.ts`; as duas não são compartilhadas, e mudar uma sem a outra impede o reset.

## `TelaInicio`

Tela da aba Início, em `FRONTEND/src/app/(abas)/inicio.tsx`. Neste módulo interessa só o que ela mostra ao administrador do sistema.

| Atributo                 | Tipo      | Regra                                                                                                                                                                                                 |
| ------------------------ | --------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `administradorDoSistema` | `boolean` | Verdadeiro quando o `tipo` do usuário da sessão é `ADMIN_SISTEMA`. Nesse caso a tela mostra só os itens "Gerenciar organizações" e "Sistema", e não busca reuniões nem atas pendentes. |

O item "Sistema" tem a descrição "Status, relatórios, reinício de subsistemas e reset." e, ao ser tocado, executa `router.push('/sistema')`. O reinício de subsistemas citado nessa descrição não existe.

## `Confirmacao`

Componente em `FRONTEND/src/components/Confirmacao.tsx`. É um `Modal` do React Native.

| Propriedade       | Tipo         | Obrigatório | Regra                                                                                                  |
| ----------------- | ------------ | ----------: | ------------------------------------------------------------------------------------------------------ |
| `titulo`          | `string`     |         Não | Título acima da mensagem. A `TelaSistema` não usa.                                                     |
| `mensagem`        | `string`     |         Não | Texto do diálogo. O diálogo fica visível enquanto a mensagem existe.                                   |
| `confirmar`       | `() => void` |         Sim | Chamada pelo botão de confirmação.                                                                     |
| `cancelar`        | `() => void` |         Sim | Chamada por "Cancelar" e pelo botão voltar do Android, este só quando não há carregamento.             |
| `carregando`      | `boolean`    |         Sim | Quando verdadeiro, mostra um indicador no botão de confirmação e desabilita os dois botões.            |
| `rotuloConfirmar` | `string`     |         Não | Texto do botão de confirmação. Padrão: "Confirmar". A `TelaSistema` usa o padrão.                     |
| `perigo`          | `boolean`    |         Não | Quando verdadeiro, pinta o botão de confirmação com a cor de erro. Padrão: falso. A `TelaSistema` não liga essa opção; o reset não usa este componente. |

## `api`

Módulo `FRONTEND/src/services/api.ts`.

| Função         | Parâmetros                                                        | Retorno               | O que faz                                                                                                                                                                                                                     |
| -------------- | ----------------------------------------------------------------- | --------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `api`          | `caminho: string`, `metodo` (`GET` padrão, `POST`, `PATCH` ou `DELETE`), `corpo?: unknown` | `Promise<Resultado<T>>` | Envia a requisição com `X-API-Key` e `Authorization: Bearer`. Devolve `{ ok: true, dados }` ou `{ ok: false, erro }`, com a primeira mensagem da API. Sem rede: "Não foi possível conectar ao servidor.". Em 401 com token, apaga o token e encerra a sessão. |
| `obterToken`   | —                                                                 | `Promise<string \| null>` | Lê o token com a chave `conecta_mais_token` do `SecureStore` (no navegador, do `localStorage`).                                                                                                                       |
| `removerToken` | —                                                                 | `Promise<void>`       | Apaga o token guardado.                                                                                                                                                                                                       |

## `FolhaInferior`

Componente em `FRONTEND/src/components/FolhaInferior.tsx`. É um `Modal` do React Native que sobe da parte de baixo da tela, sobre um fundo escurecido.

| Propriedade         | Tipo         | Obrigatório | Regra                                                                                                   |
| ------------------- | ------------ | ----------: | ------------------------------------------------------------------------------------------------------- |
| `visivel`           | `boolean`    |         Sim | A folha aparece enquanto é verdadeiro. Na `TelaSistema`, recebe `confirmacaoReset`.                     |
| `titulo`            | `string`     |         Não | Título no topo da folha. Na `TelaSistema`: "Resetar sistema".                                           |
| `aoFechar`          | `() => void` |         Sim | Chamada pelo toque no fundo escurecido e pelo botão voltar do Android. Na `TelaSistema`: `fecharReset`. |
| `acompanharTeclado` | `boolean`    |         Não | Quando verdadeiro, a folha sobe junto com o teclado. Padrão: falso. A `TelaSistema` liga essa opção.    |
| `children`          | `ReactNode`  |         Sim | Conteúdo da folha: no reset, o aviso, o campo e o botão "Resetar sistema".                              |

## `CampoTexto`

Componente em `FRONTEND/src/components/CampoTexto.tsx`, que aceita também as propriedades do `TextInput` do React Native. No reset, a `TelaSistema` passa `rotulo` "Digite RESETAR para confirmar", `value` igual a `textoReset`, `onChangeText` igual a `definirTextoReset`, `autoCapitalize="characters"` (teclado em letras maiúsculas), `autoCorrect={false}` e `compacto`.

## `autenticacaoLocal`

Módulo `FRONTEND/src/services/autenticacao-local.ts`, sobre o `expo-local-authentication`.

| Função                     | Parâmetros | Retorno            | O que faz                                                                                                                                                                         |
| -------------------------- | ---------- | ------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `dispositivoProtegido`     | —          | `Promise<boolean>` | Falso na web e quando `getEnrolledLevelAsync` informa que o aparelho não tem biometria, PIN nem senha de bloqueio. Se essa consulta falhar, também devolve falso.                 |
| `autenticarComDispositivo` | —          | `Promise<boolean>` | Sem proteção, devolve `true` sem pedir nada. Com proteção, chama `authenticateAsync`, que mostra o pedido do próprio aparelho e aceita o PIN no lugar da biometria, e devolve `success`. |

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
| `usuarios`    | `Model<Usuario>`  | Busca a conta dona do token.                  |

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

Em `BACKEND/src/sistema/sistema.controller.ts`, com `@Controller('sistema')` e `@UseGuards(JwtAuthGuard, AdminSistemaGuard)`. Cada método apenas chama o método de mesmo nome do serviço e devolve o resultado. O método `reset` recebe o corpo `ResetSistemaDto` (parâmetro `_confirmacao`), que o `ValidationPipe` já validou, e não o repassa ao serviço. As rotas estão no [diagrama de classes](3%20-%20diagrama-classes-informacao-sistemas.md).

## `ResetSistemaDto`

Em `BACKEND/src/sistema/dto/reset-sistema.dto.ts`. O mesmo arquivo exporta a constante `CONFIRMACAO_RESET`, igual a "RESETAR". O `ValidationPipe` da API, com `whitelist: true`, descarta qualquer outro campo do corpo.

| Atributo      | Tipo     | Obrigatório | Validação                                                                                                                                  | Mensagem de erro                              |
| ------------- | -------- | ----------: | ------------------------------------------------------------------------------------------------------------------------------------------ | --------------------------------------------- |
| `confirmacao` | `string` |         Sim | `@Equals(CONFIRMACAO_RESET)`: igual a "RESETAR", por comparação exata. Texto vazio, campo ausente, letras minúsculas e espaços nas pontas são recusados. | "Digite RESETAR para confirmar o reset" (400) |

## `SistemaService`

Em `BACKEND/src/sistema/sistema.service.ts`.

| Atributo        | Tipo                  | Regra                                                                                    |
| --------------- | --------------------- | ---------------------------------------------------------------------------------------- |
| `usuarios`      | `Model<Usuario>`      | Model da coleção `usuarios`.                                                             |
| `organizacoes`  | `Model<Organizacao>`  | Model da coleção `organizacoes`.                                                         |
| `comissoes`     | `Model<Comissao>`     | Model da coleção `comissoes`.                                                            |
| `reunioes`      | `Model<Reuniao>`      | Model da coleção `reunioes`; usado só no reset.                                          |
| `conexao`       | `Connection`          | Conexão do Mongoose; usada no reset para apagar `anexos.files`, `anexos.chunks`, `contadores` e `notificacoes` pelo nome. |

| Método                           | Parâmetros | Retorno                   | O que faz                                                                                                                                                                                           |
| -------------------------------- | ---------- | ------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `status`                         | —          | objeto de status          | Lê memória, CPU e tempo ligado do computador pelo `node:os` e conta todos os documentos de `usuarios`, `organizacoes` e `comissoes`.                                                                |
| `relatorios`                     | —          | objeto de relatórios      | Faz sete contagens: usuários (todos e ativos), organizações (todas, pendentes e aprovadas) e comissões (todas e ativas).                                                                            |
| `popularDadosDeTeste`            | —          | objeto com as contas criadas | Cria três contas, uma organização aprovada e uma comissão, com os dados das variáveis `TEST_USER_*` e `TEST_USERS_PASSWORD`.                                                                    |
| `reset`                          | —          | objeto de reset           | Apaga, ao mesmo tempo e sem transação, as contas que não são `ADMIN_SISTEMA`, todas as organizações, comissões e reuniões, os arquivos do GridFS, os contadores e todas as notificações.            |
| `obterConfiguracaoUsuariosTeste` | —          | objeto de configuração    | Lê as sete variáveis de ambiente, apara os textos e converte os e-mails para minúsculas. Se alguma faltar, lança 400 "Configure as contas de teste no .env: " seguido dos nomes que faltam.         |

### Objetos devolvidos

Não têm classe no código: são montados no próprio método.

| Objeto                       | Campo                               | Tipo               | Regra                                                                                   |
| ---------------------------- | ----------------------------------- | ------------------ | --------------------------------------------------------------------------------------- |
| Status                       | `servidor.memoria.total_mb`         | `number` (inteiro) | Memória total do computador em MB (bytes ÷ 1.048.576, arredondado).                     |
|                              | `servidor.memoria.livre_mb`         | `number` (inteiro) | Memória livre do computador em MB, arredondada.                                         |
|                              | `servidor.memoria.uso_percentual`   | `number` (inteiro) | (total − livre) ÷ total × 100, arredondado.                                             |
|                              | `servidor.cpu.nucleos`              | `number` (inteiro) | Quantidade de processadores lógicos informada por `os.cpus()`.                          |
|                              | `servidor.cpu.carga_media_1min`     | `number`           | Carga média do último minuto, com duas casas. No Windows, sempre 0.                     |
|                              | `servidor.tempo_ativo_segundos`     | `number` (inteiro) | Segundos desde que o computador foi ligado (`os.uptime()`), não desde que a API iniciou. |
|                              | `banco_de_dados.conectado`          | `boolean`          | Sempre `true` na resposta; se o banco falha, a rota responde 500 em vez de `false`.     |
|                              | `banco_de_dados.colecoes`           | objeto             | `usuarios`, `organizacoes` e `comissoes`, total de documentos de cada coleção. Não exibido. |
|                              | `verificado_em`                     | `Date`             | Horário da consulta. Não exibido.                                                       |
| Relatórios                   | `usuarios.total`, `usuarios.ativos` | `number` (inteiro) | Todas as contas, inclusive administradores; contas com `ativo` verdadeiro.              |
|                              | `organizacoes.total`, `.pendentes`, `.aprovadas` | `number` (inteiro) | Todas; `status` `PENDENTE`; `status` `APROVADA`.                          |
|                              | `comissoes.total`, `comissoes.ativas` | `number` (inteiro) | Todas; `ativo` verdadeiro.                                                            |
| Dados de teste               | `status`                            | `string`           | Sempre `"dados_de_teste_criados"`.                                                      |
|                              | `usuarios`                          | lista de objetos   | Três itens `{ nome, email, cenario }`. Não traz senha nem `senha_padrao`.               |
| Reset                        | `status`                            | `string`           | Sempre `"sistema_resetado"`.                                                            |
|                              | `resetado_em`                       | `Date`             | Horário do fim das exclusões.                                                           |

## Classes de dados (campos usados)

### `Usuario`

| Atributo            | Tipo TypeScript/Mongoose     | Tipo BSON  | Obrigatório | Uso neste módulo                                                                                         |
| ------------------- | ---------------------------- | ---------- | ----------: | -------------------------------------------------------------------------------------------------------- |
| `_id`               | `Types.ObjectId`             | `objectId` |         Sim | Comparado com o `sub` do token no `JwtAuthGuard`.                                                        |
| `nome`              | `string`                     | `string`   |         Sim | Gravado nas contas de teste.                                                                             |
| `email`             | `string`                     | `string`   |         Sim | Único, em minúsculas. Gravado nas contas de teste; repetido, a criação falha.                            |
| `senha_hash`        | `string`                     | `string`   |         Não | Hash bcrypt de custo 10 da senha das contas de teste. Nunca é devolvido.                                 |
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

### `Reuniao`, `Contador` e `Notificacao`

Este módulo não lê nem grava campos dessas classes: só apaga todos os documentos no reset. `Reuniao` guarda dentro de si a ata, as votações, as presenças e os dados dos anexos, que somem junto. `Contador` tem `_id` (texto no formato `atas:<id da comissão>:<ano>`) e `valor` (último número de ata usado); depois do reset, a primeira ata aprovada de cada comissão volta a receber o número 1. `Notificacao` tem `usuario_id` e `chave`, que formam um par único; o reset apaga todas, inclusive as das contas de administrador do sistema.

## Enumerações

| Enumeração                | Valores                                       | Onde aparece                 |
| ------------------------- | --------------------------------------------- | ---------------------------- |
| `TipoUsuario`             | `USUARIO`, `ADMIN_SISTEMA`                    | `Usuario.tipo`               |
| `StatusOrganizacao`       | `PENDENTE`, `APROVADA`, `REVOGADA`            | `Organizacao.status`         |
| `PapelOrganizacao`        | `ADMIN`, `MEMBRO`                             | `MembroOrganizacao.papel`    |
| `StatusMembroOrganizacao` | `PENDENTE`, `APROVADO`, `REJEITADO`, `CONVIDADO` | `MembroOrganizacao.status` |
| `PapelComissao`           | `RESPONSAVEL`, `MEMBRO`                       | `MembroComissao.papel`       |

## Rastreabilidade

RF-SIS-01 a RF-SIS-04; [Diagrama de classes](3%20-%20diagrama-classes-informacao-sistemas.md) e [Dicionário relacional](4%20-%20dicionario-dados-relacional-informacao-sistemas.md).
