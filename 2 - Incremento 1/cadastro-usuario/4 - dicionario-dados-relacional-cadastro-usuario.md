# Dicionário relacional — Cadastro de usuário

> Padrão da disciplina · Módulo USR · Campos do diagrama relacional · Revisado em 27/11/2026

## Resumo

Descreve cada campo da entidade `USUARIO`, a única do [diagrama relacional](4%20-%20diagrama-relacional-cadastro-usuario.md), com tipo, obrigatoriedade, chaves e forma de geração. As colunas seguem o dicionário de dados do documento de exemplo da disciplina: campo, tipo de dado, se pode ser nulo, chave primária, chave estrangeira, outros e descrição. "Pode ser nulo = Sim" quer dizer que o campo pode faltar no documento.

## Como os identificadores e valores gerados são criados

| Identificador ou valor                | Quem gera                                                  | Formato e valor inicial                                                                                                                                                                                                                                     | Incremento                                                                                                         |
| ------------------------------------- | ---------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------ |
| `USUARIO._id`                         | Driver do MongoDB, no servidor da API, no momento da inserção | `ObjectId` de 12 bytes, mostrado como 24 caracteres hexadecimais: 4 bytes com os segundos passados desde 1970, 5 bytes sorteados uma vez por processo e 3 bytes de um contador que começa num valor sorteado. Não há valor inicial fixo. | O contador soma 1 a cada novo id gerado pelo mesmo processo; como os outros bytes mudam, os valores não são contínuos nem previsíveis. |
| `USUARIO.google_id`                   | Google                                                     | Texto com o identificador da conta Google (`sub` do token), recebido no primeiro login pelo Google.                                                                                                                                                       | Em regra não muda; é substituído se outra conta Google com o mesmo e-mail entrar (limitação do RF-USR-03).          |
| Código de confirmação do cadastro     | `AuthService.gerarCodigoVerificacao`                       | 7 caracteres sorteados com `randomBytes` entre A–Z e 0–9. Só o SHA-256 dele é gravado, em `verificacao_email_codigo_hash`.                                                                                                                                  | Um código por cadastro; não há reenvio.                                                                            |
| Token de acesso (JWT)                 | `AuthService`, com a chave `JWT_SECRET`                    | Texto assinado com `sub`, `email`, `tipo`, `iat` e `exp`, válido por 1 hora.                                                                                                                                                                                      | Não é gravado no banco: fica só no aparelho, com a chave `conecta_mais_token`.                                     |

## USUARIO

Guarda uma conta de pessoa: os dados de identificação, a forma de entrar e as datas. É a coleção `usuarios`.

| Campo                           | Tipo de dado                   | Pode ser nulo | Chave primária  | Chave estrangeira | Outros                                                     | Descrição                                                                                     |
| ------------------------------- | ------------------------------ | ------------- | --------------- | ----------------- | ---------------------------------------------------------- | --------------------------------------------------------------------------------------------- |
| `_id`                           | `ObjectId`                     | Não           | Sim (exclusiva) | Não               | Gerado na inserção.                                        | Identifica a conta; é o `sub` do token de acesso.                                             |
| `nome`                          | Texto                          | Não           | Não             | Não               | Sem espaços nas pontas; 2 caracteres ou mais no cadastro.  | Nome mostrado nas telas e nas listas de membros.                                              |
| `email`                         | Texto                          | Não           | Não             | Não               | Único; minúsculas; sem espaços nas pontas.                 | E-mail usado para entrar, para convites e para receber o código.                              |
| `senha_hash`                    | Texto (60 caracteres)          | Sim           | Não             | Não               | Oculto; bcrypt com custo 10.                               | Resumo da senha; falta na conta criada pelo Google.                                           |
| `tipo`                          | Texto com valores fixos        | Não           | Não             | Não               | `USUARIO` (padrão) ou `ADMIN_SISTEMA`.                     | Diferencia a conta comum do administrador do sistema.                                         |
| `tema`                          | Texto com valores fixos        | Não           | Não             | Não               | `claro`, `escuro` ou `sistema` (padrão).                   | Aparência preferida.                                                                          |
| `verificacao_email_codigo_hash` | Texto (64 caracteres)          | Sim           | Não             | Não               | Oculto; SHA-256 em hexadecimal.                            | Resumo do código de confirmação; existe do cadastro até a confirmação pelo código (o login Google não o apaga). |
| `verificacao_email_expira_em`   | Data e hora (`date`, UTC)      | Sim           | Não             | Não               | Oculto; gravação do código + 30 minutos.                   | Fim da validade do código.                                                                    |
| `email_verificado`              | Booleano                       | Não           | Não             | Não               | Padrão verdadeiro; o cadastro local grava falso.           | Diz se o e-mail foi confirmado; sem ele não há login com senha.                               |
| `ativo`                         | Booleano                       | Não           | Não             | Não               | Padrão verdadeiro.                                         | Conta desativada não entra e perde a sessão aberta.                                           |
| `google_id`                     | Texto                          | Sim           | Não             | Não               | Oculto; único entre as contas que o têm.                   | Liga a conta à conta Google.                                                                  |
| `provedor_autenticacao`         | Texto com valores fixos        | Não           | Não             | Não               | `LOCAL` (padrão), `GOOGLE` ou `AMBOS`.                     | Forma de entrar.                                                                              |
| `avatar_url`                    | Texto                          | Sim           | Não             | Não               | Oculto.                                                    | Endereço da foto da conta Google.                                                             |
| `senha_alterada_em`             | Data e hora (`date`, UTC)      | Sim           | Não             | Não               | Gravado pela troca ou redefinição de senha (módulo CFG).   | Tokens emitidos antes deste instante são recusados.                                           |
| `criado_em`                     | Data e hora (`date`, UTC)      | Não           | Não             | Não               | Preenchido pelo Mongoose.                                  | Momento do cadastro.                                                                          |
| `atualizado_em`                 | Data e hora (`date`, UTC)      | Não           | Não             | Não               | Preenchido pelo Mongoose.                                  | Momento da última gravação.                                                                   |

O documento também guarda os campos da recuperação de conta e da troca de e-mail (`reset_senha_*` e `email_pendente*`), descritos no módulo de configurações, e o campo `__v`, controle interno de versão do Mongoose, que o sistema não usa.

As entidades dos vínculos do usuário com organizações (`ORGANIZACAO` e `MEMBRO_ORGANIZACAO`) e as restrições deles estão documentadas no módulo [Cadastro de organização](../cadastro-organizacao/README.md).

## Restrições e como são garantidas

- **E-mail único.** O banco tem um índice único em `usuarios.email`. Como o e-mail é gravado em minúsculas e sem espaços nas pontas, "Ana@Exemplo.com" e "ana@exemplo.com" contam como o mesmo. O cadastro confere antes de gravar e responde 409 se já existir. Se dois cadastros com o mesmo e-mail chegarem ao mesmo tempo, os dois passam pela conferência e o índice recusa o segundo, que recebe 500 em vez de 409 (limitação).
- **Conta Google única.** O índice de `usuarios.google_id` é único e esparso: contas sem `google_id` ficam fora dele, e duas contas não podem ter o mesmo. O login pelo Google tem uma conferência para recusar ligar um e-mail que já está ligado a outra conta Google, mas ela não funciona: a busca pelo e-mail não lê o campo oculto `google_id`, e o valor gravado é substituído pelo novo (limitação do RF-USR-03).
- **Dados sensíveis.** `senha_hash`, `verificacao_email_codigo_hash`, `verificacao_email_expira_em`, `google_id` e `avatar_url` são ocultos: não vêm nas consultas, nem nas respostas da API, a menos que a consulta peça cada um deles pelo nome.

## Rastreabilidade

RF-USR-01 a RF-USR-03; [Diagrama relacional](4%20-%20diagrama-relacional-cadastro-usuario.md) e [Testes](7%20-%20testes-cadastro-usuario.md).
