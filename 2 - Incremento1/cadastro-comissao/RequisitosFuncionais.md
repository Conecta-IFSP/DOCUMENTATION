# Requisitos funcionais

## RF-COM-01 — Listar e filtrar comissões

| Campo | Descrição |
|---|---|
| **Descrição** | O sistema deve mostrar as comissões das organizações que o usuário pode administrar. |
| **Entradas** | Texto de busca e filtro: Todas, Ativas ou Inativas. |
| **Origem** | Administrador da organização. |
| **Saída** | Lista de comissões. |
| **Destino** | Tela de Comissões. |
| **Ação** | A API busca as comissões permitidas e a tela aplica a busca e o filtro. |
| **Pré-condição** | Usuário autenticado, ativo e administrador aprovado de uma organização aprovada. |
| **Pós-condição** | A lista é exibida sem alterar nenhum dado. |
| **Efeitos colaterais** | Nenhum. |

## RF-COM-02 — Consultar comissão

| Campo | Descrição |
|---|---|
| **Descrição** | O sistema deve mostrar os dados de uma comissão e seus integrantes. |
| **Entradas** | ID da comissão. |
| **Origem** | Administrador seleciona uma comissão na lista. |
| **Saída** | Nome, descrição, organização, situação e equipe. |
| **Destino** | Tela de gerenciamento da comissão. |
| **Ação** | A API busca a comissão pelo ID e verifica se o usuário pode administrá-la. |
| **Pré-condição** | Usuário autenticado e autorizado para a organização da comissão. |
| **Pós-condição** | Os dados são exibidos. |
| **Efeitos colaterais** | Nenhum. |

## RF-COM-03 — Cadastrar comissão

| Campo | Descrição |
|---|---|
| **Descrição** | O sistema deve permitir o cadastro de uma nova comissão. |
| **Entradas** | Organização, nome e descrição opcional. |
| **Origem** | Tela Nova comissão. |
| **Saída** | Comissão criada. |
| **Destino** | MongoDB e tela de gerenciamento da comissão. |
| **Ação** | O frontend envia um `POST /comissoes`. O backend valida os dados e grava a comissão no MongoDB. |
| **Pré-condição** | Usuário autenticado, ativo e administrador aprovado da organização escolhida. |
| **Pós-condição** | A comissão é criada com `ativo = true` e `membros = []`. |
| **Efeitos colaterais** | O MongoDB gera o ID e o sistema registra as datas de criação e atualização. |

### Dados do cadastro

- **Nome:** obrigatório, entre 2 e 100 caracteres.
- **Descrição:** opcional.
- **Organização:** obrigatória e deve ser uma organização que o usuário administra.

## RF-COM-04 — Editar comissão

| Campo | Descrição |
|---|---|
| **Descrição** | O sistema deve permitir alterar o nome e a descrição da comissão. |
| **Entradas** | ID da comissão, nome e descrição. |
| **Origem** | Tela de gerenciamento da comissão. |
| **Saída** | Comissão atualizada. |
| **Destino** | MongoDB e tela de gerenciamento. |
| **Ação** | O frontend envia `PATCH /comissoes/:id` e o backend atualiza os campos enviados. |
| **Pré-condição** | Usuário autorizado para administrar a comissão. |
| **Pós-condição** | Nome e/ou descrição ficam atualizados. |
| **Efeitos colaterais** | A data de atualização da comissão é alterada. |

## RF-COM-05 — Desativar comissão

| Campo | Descrição |
|---|---|
| **Descrição** | O sistema deve permitir desativar uma comissão sem apagá-la do banco. |
| **Entradas** | ID da comissão e confirmação do usuário. |
| **Origem** | Botão Desativar comissão. |
| **Saída** | Comissão com `ativo = false`. |
| **Destino** | MongoDB e tela de gerenciamento. |
| **Ação** | O backend altera `ativo` para `false`. |
| **Pré-condição** | Usuário autorizado. |
| **Pós-condição** | Comissão fica inativa e sua equipe é preservada. |
| **Efeitos colaterais** | Não é possível alterar a equipe enquanto a comissão estiver inativa. |

> Esta é uma **exclusão lógica (soft delete)**. O registro não é apagado fisicamente.

## RF-COM-06 — Reativar comissão

| Campo | Descrição |
|---|---|
| **Descrição** | O sistema deve permitir reativar uma comissão desativada. |
| **Entradas** | ID da comissão. |
| **Origem** | Botão Reativar comissão. |
| **Saída** | Comissão com `ativo = true`. |
| **Destino** | MongoDB e tela de gerenciamento. |
| **Ação** | O frontend envia `PATCH /comissoes/:id` com `{ "ativo": true }`. |
| **Pré-condição** | Comissão existente e usuário autorizado. |
| **Pós-condição** | Comissão fica ativa novamente. |
| **Efeitos colaterais** | O gerenciamento da equipe volta a ser permitido. |

## RF-COM-07 — Adicionar integrante

| Campo | Descrição |
|---|---|
| **Descrição** | O sistema deve permitir adicionar um usuário à equipe da comissão. |
| **Entradas** | ID da comissão, ID do usuário e papel (`MEMBRO` ou `RESPONSAVEL`). |
| **Origem** | Seção Adicionar integrante. |
| **Saída** | Comissão com o novo integrante. |
| **Destino** | Array `membros` da comissão no MongoDB. |
| **Ação** | O frontend envia `POST /comissoes/:id/membros`. O backend valida o usuário e usa `$push` para adicioná-lo. |
| **Pré-condição** | Comissão ativa. O usuário deve estar ativo e aprovado na mesma organização. |
| **Pós-condição** | O usuário passa a fazer parte da comissão. |
| **Efeitos colaterais** | O sistema impede que o mesmo usuário seja adicionado duas vezes. |

## RF-COM-08 — Alterar papel do integrante

| Campo | Descrição |
|---|---|
| **Descrição** | O sistema deve permitir trocar o papel de um integrante. |
| **Entradas** | ID da comissão, ID do usuário e novo papel. |
| **Origem** | Botão Tornar membro ou Tornar responsável. |
| **Saída** | Integrante com o novo papel. |
| **Destino** | Array `membros` da comissão. |
| **Ação** | O frontend envia `PATCH /comissoes/:id/membros/:usuarioId`. O backend altera somente o campo `papel`. |
| **Pré-condição** | Comissão ativa e usuário já pertencente à comissão. |
| **Pós-condição** | O papel fica como `MEMBRO` ou `RESPONSAVEL`. |
| **Efeitos colaterais** | Ser `RESPONSAVEL` não dá permissão de administrador da organização. |

## RF-COM-09 — Remover integrante

| Campo | Descrição |
|---|---|
| **Descrição** | O sistema deve permitir retirar um integrante da comissão. |
| **Entradas** | ID da comissão, ID do usuário e confirmação. |
| **Origem** | Botão Remover. |
| **Saída** | Comissão sem aquele integrante. |
| **Destino** | Array `membros` da comissão. |
| **Ação** | O frontend envia `DELETE /comissoes/:id/membros/:usuarioId`. O backend usa `$pull` para retirar o integrante. |
| **Pré-condição** | Comissão ativa e usuário pertencente à comissão. |
| **Pós-condição** | O usuário deixa de fazer parte da comissão. |
| **Efeitos colaterais** | O usuário continua cadastrado no sistema e continua pertencendo à organização. |

---
