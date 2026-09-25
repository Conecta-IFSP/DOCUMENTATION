# Requisitos Funcionais - Cadastro de Organização

| Código | Caso de Uso | Ator |
|---|---|---|
| RF-01 | Cadastrar organização | Usuário Comum |
| RF-02 | Consultar organizações cadastradas | Administrador do Sistema |
| RF-03 | Autorizar cadastro de organização | Administrador do Sistema |
| RF-04 | Revogar organização | Administrador do Sistema |
| RF-05 | Editar dados da organização | Administrador do Sistema |
| RF-06 | Excluir organização revogada | Administrador do Sistema |

---

## RF-01 - Cadastrar organização

| Campo | Descrição |
|---|---|
| **Descrição** | Permitir que o usuário cadastre uma nova organização, que fica aguardando a autorização do administrador do sistema. |
| **Entradas** | • Nome da organização (de 2 a 100 caracteres)<br>• Descrição (opcional, até 200 caracteres) |
| **Origem** | Usuário autenticado, pela tela de cadastro de organização. |
| **Saída** | • Organização criada com situação “pendente”<br>• Mensagem de erro quando o nome já existe |
| **Destino** | Lista “Minhas organizações” do usuário e fila de autorização do administrador do sistema. |
| **Ação** | O sistema valida o nome e a descrição, verifica que não existe outra organização com o mesmo nome — sem distinguir maiúsculas —, grava a organização com situação pendente e registra quem cadastrou como administrador aprovado dela. |
| **Pré-condição** | Usuário autenticado, com conta ativa, e que não seja administrador do sistema. |
| **Pós-condição** | Organização gravada como pendente, com o criador vinculado a ela como administrador aprovado. |
| **Efeitos colaterais** | A organização entra na fila de autorização do administrador do sistema. Nome repetido não grava nada. |

---

## RF-02 - Consultar organizações cadastradas

| Campo | Descrição |
|---|---|
| **Descrição** | Permitir que o administrador do sistema consulte todas as organizações cadastradas e a situação de cada uma. |
| **Entradas** | • Nenhuma; a conta é identificada pelo token de acesso |
| **Origem** | Administrador do sistema, pela tela de gerenciar organizações. |
| **Saída** | • Lista das organizações com nome, descrição, situação e responsável pelo cadastro |
| **Destino** | Tela de gerenciar organizações, com as pendentes no topo. |
| **Ação** | O sistema identifica o papel de administrador do sistema, consulta todas as organizações cadastradas e devolve cada uma com a situação e os dados de quem a cadastrou. |
| **Pré-condição** | Solicitante é administrador do sistema, com conta ativa. |
| **Pós-condição** | Nenhuma alteração nos dados; a operação é de consulta. |
| **Efeitos colaterais** | Contas que não são administradoras do sistema recebem apenas as organizações a que têm direito. |

---

## RF-03 - Autorizar cadastro de organização

| Campo | Descrição |
|---|---|
| **Descrição** | Permitir que o administrador do sistema autorize o cadastro de uma organização pendente. |
| **Entradas** | • Organização<br>• Situação “aprovada” |
| **Origem** | Administrador do sistema, pela tela de gerenciar organizações. |
| **Saída** | • Organização com situação “aprovada”<br>• Mensagem de erro quando a organização não é encontrada |
| **Destino** | Lista de organizações disponíveis para solicitação de acesso. |
| **Ação** | O sistema confere que quem pede é administrador do sistema e grava a nova situação da organização. |
| **Pré-condição** | Solicitante é administrador do sistema, com conta ativa; a organização existe e não está aprovada. |
| **Pós-condição** | Organização com situação aprovada. |
| **Efeitos colaterais** | A organização passa a aceitar solicitações de acesso e a permitir o cadastro de comissões. Uma organização revogada pode ser reativada por esta mesma operação. |

---

## RF-04 - Revogar organização

| Campo | Descrição |
|---|---|
| **Descrição** | Permitir que o administrador do sistema revogue uma organização, recusando um cadastro pendente ou suspendendo uma organização aprovada. |
| **Entradas** | • Organização<br>• Situação “revogada” |
| **Origem** | Administrador do sistema, pela tela de gerenciar organizações. |
| **Saída** | • Organização com situação “revogada” |
| **Destino** | Lista de organizações, marcada como revogada. |
| **Ação** | O sistema confere a permissão do solicitante e grava a situação revogada. |
| **Pré-condição** | Solicitante é administrador do sistema, com conta ativa; a organização existe e não está revogada. |
| **Pós-condição** | Organização com situação revogada. |
| **Efeitos colaterais** | A organização deixa de aparecer para novos pedidos de acesso, a gestão de comissões dela fica bloqueada e a exclusão definitiva passa a ser permitida. |

---

## RF-05 - Editar dados da organização

| Campo | Descrição |
|---|---|
| **Descrição** | Permitir que o administrador do sistema corrija o nome e a descrição de uma organização. |
| **Entradas** | • Nome (de 2 a 100 caracteres)<br>• Descrição (até 200 caracteres) |
| **Origem** | Administrador do sistema, pela tela de gerenciar organizações. |
| **Saída** | • Organização com os dados atualizados<br>• Mensagem de erro quando o nome já pertence a outra organização |
| **Destino** | Lista de organizações e telas que mostram o nome da organização. |
| **Ação** | O sistema valida os campos enviados, verifica que o nome não pertence a outra organização e grava apenas o que foi informado. |
| **Pré-condição** | Solicitante é administrador do sistema, com conta ativa; a organização existe. |
| **Pós-condição** | Dados da organização atualizados. |
| **Efeitos colaterais** | Nome repetido não grava nada; o novo nome aparece para todos os membros. |

---

## RF-06 - Excluir organização revogada

| Campo | Descrição |
|---|---|
| **Descrição** | Permitir que o administrador do sistema exclua em definitivo uma organização já revogada. |
| **Entradas** | • Organização<br>• Confirmação da exclusão |
| **Origem** | Administrador do sistema, pela tela de gerenciar organizações. |
| **Saída** | • Confirmação da exclusão<br>• Mensagem pedindo que a organização seja revogada antes |
| **Destino** | Lista de organizações, de onde o registro desaparece. |
| **Ação** | O sistema exclui a organização somente se a situação for revogada e, em seguida, apaga as comissões que pertenciam a ela. |
| **Pré-condição** | Solicitante é administrador do sistema, com conta ativa; a organização está revogada. |
| **Pós-condição** | Organização e comissões dela removidas do banco. |
| **Efeitos colaterais** | As comissões da organização são apagadas junto, com suas equipes. Organização não revogada não é excluída. |

---
