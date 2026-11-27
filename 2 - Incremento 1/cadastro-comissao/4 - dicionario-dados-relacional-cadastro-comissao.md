# Dicionário relacional — Cadastro de comissão

> Padrão da disciplina · Módulo COM · Campos do diagrama relacional · Revisado em 27/11/2026

## Resumo

Descreve cada campo das entidades do [diagrama relacional](4%20-%20diagrama-relacional-cadastro-comissao.md), com tipo, obrigatoriedade, chaves e forma de geração. As colunas seguem o dicionário de dados do documento de exemplo da disciplina: campo, tipo de dado, se pode ser nulo, chave primária, chave estrangeira, outros e descrição.

## Como os identificadores são criados

| Identificador                     | Quem gera                                                     | Formato e valor inicial                                                                                                                                                                                                                                       | Incremento                                                                                                                                                       |
| --------------------------------- | ------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `COMISSAO._id`                    | Driver do MongoDB, no servidor da API, no momento da inserção feita por `ComissaoModel.create` (RF-COM-03) | `ObjectId` de 12 bytes, mostrado como 24 caracteres hexadecimais: 4 bytes com os segundos desde 1º/1/1970 no momento da geração, 5 bytes aleatórios sorteados uma vez por processo e 3 bytes de um contador que começa num valor aleatório. Não há valor inicial fixo. | O contador soma 1 a cada novo identificador gerado pelo mesmo processo; como os demais bytes mudam com o tempo e de um processo para outro, os valores não são contínuos nem previsíveis. |
| `COMISSAO_MEMBRO` (chave lógica)  | Não é gerado                                                  | O integrante não tem identificador próprio (`_id: false`): é identificado pela comissão que o contém mais o `usuario_id`.                                                                                                                                     | Não há.                                                                                                                                                          |
| `ORGANIZACAO._id` e `USUARIO._id` | Driver do MongoDB, na inserção feita pelos módulos de organização e de usuário | Mesmo formato de `COMISSAO._id`.                                                                                                                                                                                                                       | Mesmo comportamento de `COMISSAO._id`.                                                                                                                          |

As datas `criado_em` e `atualizado_em` são gravadas pelo Mongoose (opção `timestamps` do schema): `criado_em` recebe o horário do servidor na inserção e não muda mais; `atualizado_em` recebe o mesmo valor na inserção e é trocado pelo horário de cada atualização. `adicionado_em` é preenchido pelo `ComissoesService` com o horário do servidor quando a pessoa entra na equipe.

## COMISSAO

Representa uma comissão, isto é, um grupo de trabalho de uma organização. É a coleção `comissoes`; cada documento é uma comissão.

| Campo            | Tipo de dado                     | Pode ser nulo | Chave primária  | Chave estrangeira          | Outros                                                                 | Descrição                                                                               |
| ---------------- | -------------------------------- | ------------- | --------------- | -------------------------- | ---------------------------------------------------------------------- | --------------------------------------------------------------------------------------- |
| `_id`            | `ObjectId`                       | Não           | Sim (exclusiva) | Não                        | Gerado na inserção; ver a tabela acima.                                | Identifica a comissão.                                                                  |
| `nome`           | Texto (`string`, 2 a 100)        | Não           | Não             | Não                        | Sem espaços nas pontas. Pode repetir o nome de outra comissão.         | Nome mostrado na lista, no cabeçalho da tela e nas reuniões.                            |
| `descricao`      | Texto (`string`, até 1.000)      | Sim           | Não             | Não                        | Ausente ou vazio quando não informado; o aplicativo aceita até 200.    | Explicação livre sobre a finalidade da comissão.                                        |
| `organizacao_id` | `ObjectId`                       | Não           | Não             | Sim (`ORGANIZACAO._id`)    | Definido no cadastro e nunca alterado.                                 | Organização a que a comissão pertence.                                                  |
| `ativo`          | Booleano (`bool`)                | Não           | Não             | Não                        | Padrão verdadeiro.                                                     | Situação da comissão: verdadeiro para ativa, falso para desativada (exclusão lógica).   |
| `criado_em`      | Data e hora (`date`, UTC)        | Não           | Não             | Não                        | Gravado pelo Mongoose; usado para ordenar a lista da API.              | Momento do cadastro.                                                                    |
| `atualizado_em`  | Data e hora (`date`, UTC)        | Não           | Não             | Não                        | Gravado pelo Mongoose a cada alteração.                                | Momento da última alteração de dados, situação ou equipe.                               |

## COMISSAO_MEMBRO

Representa a participação de uma pessoa na equipe de uma comissão, com o seu papel. Fica em `comissoes.membros[]`.

| Campo           | Tipo de dado              | Pode ser nulo | Chave primária  | Chave estrangeira     | Outros                                                      | Descrição                                                                 |
| --------------- | ------------------------- | ------------- | --------------- | --------------------- | ----------------------------------------------------------- | ------------------------------------------------------------------------- |
| `comissao_id`   | `ObjectId`                | Não           | Sim (inclusiva) | Sim (`COMISSAO._id`)  | Não é gravado: é o documento que contém a lista.            | Comissão da equipe.                                                       |
| `usuario_id`    | `ObjectId`                | Não           | Sim (inclusiva) | Sim (`USUARIO._id`)   | Não se repete dentro da mesma comissão.                     | Pessoa que integra a equipe.                                              |
| `papel`         | Texto com valores fixos   | Não           | Não             | Não                   | `RESPONSAVEL` ou `MEMBRO`; exatamente um `RESPONSAVEL` por comissão. | Papel da pessoa na comissão.                                     |
| `adicionado_em` | Data e hora (`date`, UTC) | Não           | Não             | Não                   | Mantido quando a equipe é regravada.                        | Momento em que a pessoa entrou na equipe.                                 |

## ORGANIZACAO

Representa a organização dona das comissões. É a coleção `organizacoes`, mantida pelo [módulo de organização](../cadastro-organizacao/README.md); abaixo estão só os campos que este módulo lê.

| Campo    | Tipo de dado            | Pode ser nulo | Chave primária  | Chave estrangeira | Outros                                      | Descrição                                                     |
| -------- | ----------------------- | ------------- | --------------- | ----------------- | ------------------------------------------- | ------------------------------------------------------------- |
| `_id`    | `ObjectId`              | Não           | Sim (exclusiva) | Não               | Gerado na inserção.                         | Identifica a organização.                                     |
| `nome`   | Texto                   | Não           | Não             | Não               | -                                           | Nome mostrado na tela da comissão e no cadastro.              |
| `status` | Texto com valores fixos | Não           | Não             | Não               | `PENDENTE`, `APROVADA` ou `REVOGADA`.       | Só a organização `APROVADA` permite consultar e gerenciar comissões. |

## ORGANIZACAO_MEMBRO

Representa o vínculo de uma pessoa com a organização. Fica em `organizacoes.membros[]`; este módulo só o lê.

| Campo            | Tipo de dado            | Pode ser nulo | Chave primária  | Chave estrangeira       | Outros                                                   | Descrição                                                        |
| ---------------- | ----------------------- | ------------- | --------------- | ----------------------- | -------------------------------------------------------- | ---------------------------------------------------------------- |
| `organizacao_id` | `ObjectId`              | Não           | Sim (inclusiva) | Sim (`ORGANIZACAO._id`) | Não é gravado: é o documento que contém a lista.         | Organização do vínculo.                                          |
| `usuario_id`     | `ObjectId`              | Não           | Sim (inclusiva) | Sim (`USUARIO._id`)     | -                                                        | Pessoa vinculada.                                                |
| `papel`          | Texto com valores fixos | Não           | Não             | Não                     | `ADMIN` ou `MEMBRO`.                                     | `ADMIN` gerencia as comissões da organização.                    |
| `status`         | Texto com valores fixos | Não           | Não             | Não                     | `PENDENTE`, `APROVADO`, `REJEITADO` ou `CONVIDADO`.      | Só `APROVADO` dá acesso às comissões e permite entrar numa equipe. |

## USUARIO

Representa a conta de uma pessoa. É a coleção `usuarios`, mantida pelo [módulo de usuário](../cadastro-usuario/README.md); abaixo estão só os campos que este módulo lê.

| Campo   | Tipo de dado            | Pode ser nulo | Chave primária  | Chave estrangeira | Outros                                | Descrição                                                         |
| ------- | ----------------------- | ------------- | --------------- | ----------------- | ------------------------------------- | ----------------------------------------------------------------- |
| `_id`   | `ObjectId`              | Não           | Sim (exclusiva) | Não               | Gerado na inserção.                   | Identifica a pessoa.                                              |
| `nome`  | Texto                   | Não           | Não             | Não               | -                                     | Nome mostrado na equipe e na escolha de pessoas.                  |
| `email` | Texto                   | Não           | Não             | Não               | Único entre os usuários.              | E-mail mostrado abaixo do nome.                                   |
| `tipo`  | Texto com valores fixos | Não           | Não             | Não               | `USUARIO` ou `ADMIN_SISTEMA`.         | O administrador do sistema não usa o módulo.                      |
| `ativo` | Booleano                | Não           | Não             | Não               | Padrão verdadeiro.                    | Conta inativa não usa o módulo nem pode entrar numa equipe.       |

## Restrições explicadas

- **Responsável único.** Toda comissão precisa ter exatamente uma pessoa com papel de responsável, porque é ela quem agenda as reuniões e escreve as atas. A regra é conferida pelo serviço antes de cada mudança de equipe e, de novo, pelo validador do campo `membros` no momento de gravar.
- **Pessoa sem repetição.** Uma pessoa não aparece duas vezes na mesma equipe; a inclusão de quem já está recebe 409 "Usuário já pertence à comissão", e o validador do campo `membros` também recusa a lista repetida.
- **Só pessoas da organização.** Só entra numa equipe quem tem conta ativa e vínculo aprovado na organização da comissão. A conferência é feita no momento da inclusão; se, depois, a conta for desativada, a pessoa continua na lista até ser removida.
- **Desativação sem perda.** Desativar não apaga a comissão: só muda `ativo` para falso, e a equipe fica como estava. Enquanto `ativo` é falso, a equipe não pode ser alterada pelo aplicativo.
- **Gravação condicionada.** Toda mudança de equipe só é gravada se a lista `membros` e o campo `ativo` ainda são os mesmos que foram lidos; se outra pessoa alterou a comissão nesse intervalo, nada é gravado e a API responde 409.
- **Referências sem conferência do banco.** O MongoDB não impede apagar uma organização ou um usuário que ainda é referenciado. Quem mantém as referências é o código, conforme a próxima seção.

## Alterações feitas por outros módulos

- Quando o administrador da organização remove uma pessoa da organização, o módulo de organização retira essa pessoa de `membros` de todas as comissões da organização com `updateMany` e `$pull`, sem rodar o validador de `membros`. Se a pessoa era o responsável, a comissão fica sem responsável. A comissão continua visível, mas adicionar, remover e reativar passam a ser recusados com "Defina exatamente um responsável para regularizar esta comissão" até que o administrador escolha outro responsável (RF-COM-08). Com a comissão inativa, o aplicativo não oferece essa troca (ver RF-COM-06).
- Quando o administrador do sistema exclui uma organização revogada, o módulo de organização apaga todas as comissões dela com `deleteMany`.
- O reset do sistema (`POST /sistema/reset`), feito pelo administrador do sistema com a própria senha e a palavra `RESET`, apaga todas as comissões com `deleteMany`, junto com as organizações, as reuniões e as contas comuns.
- A criação de dados de teste (`POST /sistema/popular-teste`) grava a "Comissão de Teste", ativa, na "Organização de Teste", com um único integrante, o responsável.

## Índices

- `comissoes._id` é a chave primária e já é indexada pelo MongoDB; o schema não declara outros índices;
- as consultas deste módulo filtram por `organizacao_id` e por `membros.usuario_id`; convém criar o índice `{ organizacao_id: 1, "membros.usuario_id": 1 }` quando o volume de comissões crescer (ainda não criado).

## Rastreabilidade

RF-COM-01 a RF-COM-09; [Diagrama relacional](4%20-%20diagrama-relacional-cadastro-comissao.md) e [Testes](7%20-%20testes-cadastro-comissao.md).
