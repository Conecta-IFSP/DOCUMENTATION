# Casos de uso

Um caso de uso descreve o objetivo do ator, não nomes de telas, tabelas, endpoints ou validações. Autenticação é pré-condição, não um include artificial em todos os casos.

## Atores

- **Administrador da organização:** usuário comum com vínculo ADMIN/APROVADO em organização APROVADA. Gerencia comissões.
- **Integrante autorizado:** usuário comum ativo, APROVADO na organização e integrante da comissão. Consulta suas comissões. Administrador consulta também as que não integra.
- **Responsável participante:** único RESPONSAVEL atual da comissão, com acesso à organização, também participante da reunião; prepara e publica ata.
- **Participante da reunião:** usuário ativo cujo ID pertence ao snapshot de participantes; consulta registros. Para aprovar, também deve pertencer ao conjunto ata.aprovadores.

O mesmo usuário pode desempenhar vários atores. ADMIN_SISTEMA não é administrador de organização e é barrado por UsuarioComumGuard. A atribuição RESPONSAVEL não concede automaticamente administração da organização.

## Diagramas

![Casos de uso de comissões](casos-comissoes.svg)

![Casos de uso de atas e presença](casos-registros.svg)

## Rastreabilidade

|Caso de uso|Objetivo|Ator|Requisito|
|---|---|---|---|
|UC-COM-01|Consultar comissões|Integrante aprovado da organização ou seu administrador|RF-COM-01|
|UC-COM-02|Consultar detalhes da comissão|Integrante da comissão ou administrador da organização|RF-COM-02|
|UC-COM-03|Cadastrar comissão com responsável|Administrador da organização|RF-COM-03|
|UC-COM-04|Editar dados da comissão|Administrador da organização|RF-COM-04|
|UC-COM-05|Desativar comissão|Administrador da organização|RF-COM-05|
|UC-COM-06|Reativar comissão|Administrador da organização|RF-COM-06|
|UC-COM-07|Adicionar integrante|Administrador da organização|RF-COM-07|
|UC-COM-08|Definir ou substituir responsável|Administrador da organização|RF-COM-08|
|UC-COM-09|Remover integrante|Administrador da organização|RF-COM-09|
|UC-ATA-01|Salvar rascunho de ata|Responsável atual da comissão que participa da reunião|RF-ATA-01|
|UC-ATA-02|Enviar ata para aprovação|Responsável atual da comissão que participa da reunião|RF-ATA-02|
|UC-ATA-03|Aprovar ata|Participante incluído em ata.aprovadores|RF-ATA-03|
|UC-PRE-01|Gerar e compartilhar lista de presença|Participante da reunião|RF-PRE-01|

As pré/pós-condições e alternativas de cada UC estão no RF correspondente, sem duplicação divergente de regras. Não há relação include/extend obrigatória entre os objetivos: por exemplo, transferir responsável pode ser executado sem remover o anterior.
