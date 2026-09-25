# Requisitos não funcionais

## RNF-COM-01 — Segurança

Todas as rotas de comissão devem exigir usuário autenticado.

Além do login, o backend deve conferir se o usuário é administrador aprovado da organização antes de permitir o gerenciamento.

## RNF-COM-02 — Validação dos dados

Os dados enviados devem ser validados antes de serem gravados.

Exemplos:

- nome da comissão deve possuir pelo menos 2 caracteres;
- IDs devem possuir formato válido do MongoDB;
- papel deve ser `MEMBRO` ou `RESPONSAVEL`;
- o campo `ativo` deve ser booleano.

A validação é feita no backend pelos DTOs e pelo `ValidationPipe`.

## RNF-COM-03 — Integridade dos dados

O mesmo usuário não deve aparecer duas vezes na mesma comissão.

Ao remover um integrante, somente o vínculo com a comissão deve ser removido. O usuário e seu vínculo com a organização devem continuar existentes.

## RNF-COM-04 — Preservação de dados

A desativação da comissão deve manter seus dados e integrantes no banco para permitir uma futura reativação.

## RNF-COM-05 — Usabilidade

A interface deve informar ao usuário quando os dados estão carregando, quando ocorreu um erro e quando uma ação foi concluída.

Ações importantes, como desativar comissão e remover integrante, devem pedir confirmação.

## RNF-COM-06 — Compatibilidade

O aplicativo deve funcionar em **Android e iOS**, utilizando React Native com Expo.

## RNF-COM-07 — Organização do código

O código deve manter as responsabilidades separadas:

- **Tela:** interface com o usuário;
- **api.ts:** comunicação HTTP;
- **Controller:** recebe a requisição;
- **DTO:** valida os dados;
- **Service:** executa as regras do módulo;
- **Schema/Mongoose:** estrutura e acesso ao MongoDB.

---
