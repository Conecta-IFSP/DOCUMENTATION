# Requisitos não funcionais

## RNF-COM-01 — Autorização verificável

Todas as rotas de comissão exigem `X-API-Key`, JWT válido e tipo `USUARIO`.

Além do login, o service confere vínculo `ADMIN/APROVADO` em organização `APROVADA` para escrita; leitura exige vínculo aprovado e restringe membros às comissões das quais participam. A verificação ocorre no servidor em todas as requisições.

## RNF-COM-02 — Validação dos dados

Os dados enviados devem ser validados antes de serem gravados.

Exemplos:

- nome da comissão deve possuir pelo menos 2 caracteres;
- IDs devem possuir formato válido do MongoDB;
- papel deve ser `MEMBRO` ou `RESPONSAVEL`;
- o campo `ativo` deve ser booleano.

A validação é feita no backend pelos DTOs e pelo `ValidationPipe`.

## RNF-COM-03 — Integridade dos dados

O schema recusa gravação se houver usuário repetido ou quantidade de responsáveis diferente de um.

Alterações da equipe usam compare-and-swap: se outra gravação mudar a equipe entre leitura e escrita, a operação retorna `409` em vez de sobrescrever o estado mais novo.

Ao remover um integrante, somente o vínculo com a comissão deve ser removido. O usuário e seu vínculo com a organização devem continuar existentes.

## RNF-COM-04 — Preservação de dados

A desativação da comissão deve manter seus dados e integrantes no banco para permitir uma futura reativação.

## RNF-COM-05 — Usabilidade

A interface deve informar ao usuário quando os dados estão carregando, quando ocorreu um erro e quando uma ação foi concluída.

Ações importantes, como desativar comissão e remover integrante, devem pedir confirmação.

## RNF-COM-06 — Compatibilidade

O aplicativo deve funcionar em **Android e iOS**, utilizando React Native com Expo.

Critério de aceitação: cadastrar, consultar e gerenciar uma comissão em ao menos um aparelho Android e um iOS, sem rolagem horizontal e com todas as ações alcançáveis. Teste iOS não foi registrado nesta revisão.

## RNF-COM-08 — Desempenho

Em rede local estável e com até 1.000 comissões, `GET /comissoes` deve responder em até 2 segundos no percentil 95. A medição deve usar 30 execuções após aquecimento. Trata-se de meta; não há medição registrada.

## RNF-COM-07 — Organização do código

O código deve manter as responsabilidades separadas:

- **Tela:** interface com o usuário;
- **api.ts:** comunicação HTTP;
- **Controller:** recebe a requisição;
- **DTO:** valida os dados;
- **Service:** executa as regras do módulo;
- **Schema/Mongoose:** estrutura e acesso ao MongoDB.

---
