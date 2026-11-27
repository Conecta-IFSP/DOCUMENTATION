# Dicionário relacional — Configurações de usuário

> Padrão da disciplina · Módulo CFG · Campos do diagrama relacional · Revisado em 26/11/2026

## Resumo

Descreve cada campo das entidades do [diagrama relacional](4%20-%20diagrama-relacional-configuracoes-usuario.md), com tipo, obrigatoriedade, chaves e forma de geração, e os dois valores que o aplicativo guarda no aparelho. As colunas seguem o dicionário de dados do documento de exemplo da disciplina: campo, tipo de dado, se pode ser nulo, chave primária, chave estrangeira, outros e descrição. No MongoDB, "pode ser nulo" significa que o campo pode estar ausente do documento.

## Como os identificadores são criados

| Identificador                          | Quem gera                                                     | Formato e valor inicial                                                                                                                                                                                                                                                      | Incremento                                                                                                                                       |
| -------------------------------------- | ------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------ |
| `USUARIO._id`                          | Driver do MongoDB, no servidor da API, no momento da inserção da conta (cadastro, primeiro login com Google ou criação do administrador inicial). Este módulo não cria contas. | `ObjectId` de 12 bytes, mostrado como 24 caracteres hexadecimais: 4 bytes com os segundos desde 01/01/1970, 5 bytes sorteados uma vez por processo e 3 bytes de um contador. O contador começa num valor sorteado; não há valor inicial fixo. | O contador soma 1 a cada novo identificador gerado pelo mesmo processo. Como os outros bytes mudam com o tempo e com o processo, os valores não são contínuos. |
| `TROCA_EMAIL` (chave lógica)           | Não é gerada                                                  | É o próprio `_id` da conta: a troca pendente é identificada pela conta em que está.                                                                                                                                                                                          | Não se aplica. Um novo pedido substitui o anterior.                                                                                              |
| `RECUPERACAO_SENHA` (chave lógica)     | Não é gerada                                                  | É o próprio `_id` da conta.                                                                                                                                                                                                                                                  | Não se aplica. Um novo pedido substitui o anterior.                                                                                              |
| Código de confirmação e de recuperação | `AuthService.gerarCodigoVerificacao`, no servidor             | 7 caracteres; cada um é sorteado entre as 36 letras de A a Z e algarismos de 0 a 9, com um byte de `randomBytes` (resto da divisão por 36). É enviado por e-mail como `XXX-XXXX` e guardado só como SHA-256.                                                                | Não há sequência: cada pedido sorteia um código novo.                                                                                            |
| Token de redefinição                   | `AuthService.verificarRecuperacao`, no servidor               | 32 bytes de `randomBytes`, devolvidos ao aplicativo como 64 caracteres hexadecimais e guardados só como SHA-256.                                                                                                                                                            | Não há sequência: cada verificação sorteia um token novo.                                                                                       |
| `iat` do token da sessão               | `JwtService.signAsync`, no login (módulo de cadastro)         | Inteiro com os segundos desde 01/01/1970 no momento em que o token foi emitido. Não é gravado no banco; vai dentro do token.                                                                                                                                                 | Não se aplica. É comparado com `senha_alterada_em` em toda requisição autenticada.                                                              |

## USUARIO

Representa a conta de uma pessoa. É a coleção `usuarios`. A tabela traz os campos usados por este módulo.

| Campo                   | Tipo de dado                        | Pode ser nulo | Chave primária  | Chave estrangeira | Outros                                                                     | Descrição                                                                                                  |
| ----------------------- | ----------------------------------- | ------------- | --------------- | ----------------- | -------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------- |
| `_id`                   | `ObjectId`                          | Não           | Sim (exclusiva) | Não               | Índice padrão do MongoDB.                                                  | Identifica a conta. É o valor `sub` do token da sessão.                                                    |
| `nome`                  | Texto (`string`)                    | Não           | Não             | Não               | Gravado sem espaços nas pontas; 2 caracteres ou mais pela API.             | Nome mostrado no perfil, no avatar e nas listas de membros.                                                |
| `email`                 | Texto (`string`)                    | Não           | Não             | Não               | Único entre as contas (índice único); gravado em minúsculas.               | Endereço usado no login e na recuperação.                                                                  |
| `senha_hash`            | Texto (`string`, 60 caracteres)     | Sim           | Não             | Não               | Oculto nas consultas comuns; bcrypt de custo 10.                           | Senha transformada por bcrypt; a senha digitada nunca é gravada. Ausente na conta criada só pelo Google.   |
| `senha_alterada_em`     | Data e hora (`date`, UTC)           | Sim           | Não             | Não               | Gravado na troca (RF-CFG-03) e na redefinição (RF-CFG-04).                 | Momento da última troca de senha. Token emitido antes dele é recusado. Ausente se a senha nunca mudou.     |
| `tipo`                  | Texto com valores fixos             | Não           | Não             | Não               | `USUARIO` ou `ADMIN_SISTEMA`; padrão `USUARIO`.                            | Papel da conta no sistema; o `JwtAuthGuard` lê o valor atual a cada requisição.                            |
| `tema`                  | Texto com valores fixos             | Não           | Não             | Não               | `sistema`, `claro` ou `escuro`; padrão `sistema`.                          | Preferência de tema da conta, aplicada ao abrir o perfil em qualquer aparelho.                             |
| `email_verificado`      | Booleano (`bool`)                   | Não           | Não             | Não               | Padrão verdadeiro; o cadastro local grava falso até a confirmação.         | Indica se o e-mail atual foi confirmado por código. A troca de e-mail confirmada grava verdadeiro.         |
| `ativo`                 | Booleano (`bool`)                   | Não           | Não             | Não               | Padrão verdadeiro.                                                         | Conta desativada não passa pelo `JwtAuthGuard` e não recebe código de recuperação.                         |
| `provedor_autenticacao` | Texto com valores fixos             | Não           | Não             | Não               | `LOCAL`, `GOOGLE` ou `AMBOS`; padrão `LOCAL`.                              | Forma de entrada da conta. Este módulo não altera o valor.                                                 |
| `criado_em`             | Data e hora (`date`, UTC)           | Não           | Não             | Não               | Preenchido pelo Mongoose.                                                  | Momento da criação da conta.                                                                               |
| `atualizado_em`         | Data e hora (`date`, UTC)           | Não           | Não             | Não               | Atualizado pelo Mongoose a cada gravação.                                  | Momento da última alteração do documento.                                                                  |

## TROCA_EMAIL

Representa um pedido de troca de e-mail ainda não confirmado. Fica nos campos `email_pendente*` do documento em `usuarios`. Todos são ocultos nas consultas comuns e só são lidos pela confirmação.

| Campo                        | Tipo de dado                      | Pode ser nulo | Chave primária  | Chave estrangeira   | Outros                                                    | Descrição                                                                   |
| ---------------------------- | --------------------------------- | ------------- | --------------- | ------------------- | --------------------------------------------------------- | --------------------------------------------------------------------------- |
| `usuario_id`                 | `ObjectId`                        | Não           | Sim (exclusiva) | Sim (`USUARIO._id`) | Não é gravado: é o documento que contém os campos.        | Conta que pediu a troca.                                                    |
| `email_pendente`             | Texto (`string`)                  | Sim           | Não             | Não                 | Em minúsculas; diferente do e-mail atual e de outras contas. | Novo endereço que espera a confirmação.                                   |
| `email_pendente_codigo_hash` | Texto (`string`, 64 caracteres)   | Sim           | Não             | Não                 | SHA-256 em hexadecimal.                                   | Hash do código enviado ao novo endereço.                                    |
| `email_pendente_expira_em`   | Data e hora (`date`, UTC)         | Sim           | Não             | Não                 | Horário do pedido mais 30 minutos.                        | Fim da validade do código.                                                  |

Os três campos são gravados juntos no pedido e apagados juntos na confirmação; um novo pedido sobrescreve os três. Pedidos vencidos e não confirmados ficam no documento até o próximo pedido, mas não são mais aceitos.

## RECUPERACAO_SENHA

Representa uma recuperação de conta em andamento. Fica nos campos `reset_senha_*` do documento em `usuarios`, todos ocultos nas consultas comuns.

| Campo                     | Tipo de dado                    | Pode ser nulo | Chave primária  | Chave estrangeira   | Outros                                                        | Descrição                                                                  |
| ------------------------- | ------------------------------- | ------------- | --------------- | ------------------- | ------------------------------------------------------------- | -------------------------------------------------------------------------- |
| `usuario_id`              | `ObjectId`                      | Não           | Sim (exclusiva) | Sim (`USUARIO._id`) | Não é gravado: é o documento que contém os campos.            | Conta em recuperação.                                                      |
| `reset_senha_codigo_hash` | Texto (`string`, 64 caracteres) | Sim           | Não             | Não                 | SHA-256 em hexadecimal; apagado na verificação.               | Hash do código enviado por e-mail.                                        |
| `reset_senha_token_hash`  | Texto (`string`, 64 caracteres) | Sim           | Não             | Não                 | SHA-256 em hexadecimal; apagado na redefinição e num novo pedido. | Hash do token entregue ao aplicativo depois do código certo.          |
| `reset_senha_expira_em`   | Data e hora (`date`, UTC)       | Sim           | Não             | Não                 | Horário do pedido mais 30 minutos; vale para o código e o token. | Fim da validade da recuperação.                                         |

Ciclo dos campos: o pedido grava o hash do código e a validade e apaga o token anterior; a verificação grava o hash do token e apaga o código; a redefinição grava a senha e `senha_alterada_em` em `USUARIO` e apaga o token e a validade.

## Dados guardados no aparelho

Fora do MongoDB, o aplicativo guarda dois valores no armazenamento seguro do aparelho (`expo-secure-store`, no Android e no iOS) ou no `localStorage` do navegador (na web).

| Campo (chave)        | Tipo de dado                        | Pode ser nulo | Chave primária  | Chave estrangeira | Outros                                                    | Descrição                                                                                                     |
| -------------------- | ----------------------------------- | ------------- | --------------- | ----------------- | --------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------- |
| `conecta_mais_token` | Texto (token JWT)                   | Sim           | Sim (exclusiva) | Não               | Gravado no login; apagado ao sair e em todo 401 de rota fora de `/auth/`. | Token da sessão, com `sub` (identificador da conta), `email`, `tipo`, `iat` e `exp` (1 hora depois de `iat`). |
| `conecta_mais_tema`  | Texto com valores fixos             | Sim           | Sim (exclusiva) | Não               | `sistema`, `claro` ou `escuro`; valor inválido é ignorado. | Preferência de tema lida ao abrir o aplicativo, antes do login.                                               |

A chave do armazenamento é o próprio nome do campo, por isso cada uma aparece como chave primária. O token apagado do aparelho continua aceito pelo servidor até o `exp`, a menos que a senha seja trocada.

## Índices e restrições

- `_id` tem o índice padrão do MongoDB;
- `email` tem índice único, criado pelo Mongoose a partir de `unique: true` no schema; a API também procura outra conta com o mesmo e-mail antes de gravar, para devolver uma mensagem clara em vez do erro do índice;
- os campos ocultos (`senha_hash`, `reset_senha_*` e `email_pendente*`) usam `select: false`: só aparecem quando o serviço pede com `select('+campo')`, e nunca nas respostas da API;
- a busca da redefinição usa `reset_senha_token_hash` e `reset_senha_expira_em` sem índice próprio; com poucas contas isso não pesa, e um índice nesses campos ainda não foi criado;
- as gravações deste módulo usam `save()`, que grava só os campos alterados, sem filtro de versão: duas alterações do mesmo campo ao mesmo tempo ficam com a última que chegar.

## Rastreabilidade

RF-CFG-01 a RF-CFG-06; [Diagrama relacional](4%20-%20diagrama-relacional-configuracoes-usuario.md) e [Testes](7%20-%20testes-configuracoes-usuario.md).
