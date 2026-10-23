# Testes — Reuniões

> Padrão do Incremento 2 · Módulo REU

## Resumo

Registra evidência executada e cenários manuais no formato comum do projeto. Os testes automatizados cobrem regras de reunião, presença, lista, atas, autorização e concorrência.

## Evidência automatizada

| Data       | Projeto/commit     | Comando/teste                    | Resultado                          | Arquivos relacionados                               |
| ---------- | ------------------ | -------------------------------- | ---------------------------------- | --------------------------------------------------- |
| 23/10/2026 | BACKEND `6f2ffc4`  | `npm test`                       | 13 arquivos e 182 testes aprovados | `reunioes.regras.spec.ts`, `atas-presencas.spec.ts` |
| 23/10/2026 | BACKEND `6f2ffc4`  | `npm run build`                  | Aprovado                           | NestJS/TypeScript                                   |
| 23/10/2026 | FRONTEND `792fab4` | testes Node do diretório `tests` | 9 testes aprovados                 | `horarios.test.cjs`, `sessao-api.test.cjs`          |

## Matriz de testes automatizados

| ID        | Requisito     | Cenário/entrada                                                                        | Resultado esperado                                                                        | Cobertura                 |
| --------- | ------------- | -------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------- | ------------------------- |
| CT-REU-01 | RF-REU-01     | Criar reunião com todos, lista vazia, convidados escolhidos e convidado externo.       | Participantes corretos; dados, horário, papel e comissão inválidos recusados.             | `reunioes.regras.spec.ts` |
| CT-REU-02 | RF-REU-02     | Listar por participante, período e comissão.                                           | Apenas reuniões permitidas, ordenadas por início.                                         | `reunioes.regras.spec.ts` |
| CT-REU-03 | RF-REU-03     | Consultar como participante e usuário externo.                                         | Participante recebe dados; externo recebe 403.                                            | `reunioes.regras.spec.ts` |
| CT-REU-04 | RF-REU-04     | Cancelar como membro, organizador e após o término.                                    | Apenas organizador cancela antes do fim; alteração aceita somente `CANCELADA`.            | `reunioes.regras.spec.ts` |
| CT-REU-05 | RF-REU-05     | Aceitar/recusar como convidado, organizador, após entrada e em reunião cancelada.      | Somente transições permitidas são persistidas.                                            | `reunioes.regras.spec.ts` |
| CT-REU-06 | RF-REU-06     | Marcar entrada antes, durante, depois, repetida e com convite recusado.                | Entrada única somente durante reunião válida.                                             | `reunioes.regras.spec.ts` |
| CT-REU-07 | RF-REU-07     | Marcar saída sem entrada e depois do término.                                          | Sem entrada é recusado; saída é limitada ao fim.                                          | `reunioes.regras.spec.ts` |
| CT-REU-08 | RF-REU-08     | Confirmar presença como membro/organizador, antes e depois do início.                  | Apenas organizador valida depois do início.                                               | `reunioes.regras.spec.ts` |
| CT-REU-09 | RF-REU-09     | Gerar lista com presente e usuário sem registro; consultar como externo.               | Nome, situação e minutos corretos; ausência não cria presença; externo recebe 403.        | `atas-presencas.spec.ts`  |
| CT-REU-10 | RF-REU-10     | Salvar ata inicial/seguinte, como membro, antes do fim, cancelada e com versão antiga. | Versão incrementada somente pelo responsável após encerramento; conflito não sobrescreve. | `atas-presencas.spec.ts`  |
| CT-REU-11 | RF-REU-11     | Publicar com presentes confirmados e sem nenhum confirmado.                            | Aprovadores são congelados; ausência de aprovador recusa publicação.                      | `atas-presencas.spec.ts`  |
| CT-REU-12 | RF-REU-12     | Aprovar em sequência, repetir voto e votar fora do conjunto.                           | Unanimidade conclui; duplicidade é idempotente; usuário externo recebe 403.               | `atas-presencas.spec.ts`  |
| CT-REU-13 | RNF-REU-01/04 | Executar rotas sem chave/JWT e simular escrita concorrente.                            | 401/403 para credencial ausente; 409 para estado ou versão alterada.                      | Suíte backend             |
| CT-REU-14 | RF-REU-01     | Gerar sugestões de início e término no mesmo dia.                                      | Intervalos de 30 minutos e nenhum horário 24:00.                                          | `horarios.test.cjs`       |

## Matriz de testes manuais

| ID        | Requisito           | Cenário/entrada                                                     | Resultado esperado                                                         | Situação              |
| --------- | ------------------- | ------------------------------------------------------------------- | -------------------------------------------------------------------------- | --------------------- |
| CT-REU-15 | RF-REU-01/02/03     | Agendar no Expo Go, abrir agenda e visualizar em Android/iOS.       | Dados aparecem sem corte, teclado não bloqueia campos e filtros respondem. | Aprovado pela equipe. |
| CT-REU-16 | RF-REU-04/05        | Cancelar e responder convite, incluindo desistência da confirmação. | Confirmação protege ação destrutiva; estados são atualizados.              | Aprovado pela equipe. |
| CT-REU-17 | RF-REU-06/07/08     | Registrar entrada/saída e validar presença em dois aparelhos.       | Segundo aparelho atualiza em até 30 segundos.                              | Aprovado pela equipe. |
| CT-REU-18 | RF-REU-09           | Abrir e compartilhar lista de presença.                             | Dados e menu nativo de compartilhamento são exibidos.                      | Aprovado pela equipe. |
| CT-REU-19 | RF-REU-10/11/12     | Salvar, publicar e aprovar ata com dois usuários.                   | Versão, bloqueio do texto e contagem de aprovações permanecem coerentes.   | Aprovado pela equipe. |
| CT-REU-20 | RNF-REU-03/05/07/09 | Testar origem web, fusos, Expo Go e leitor de tela.                 | CORS, horários, portabilidade e acessibilidade atendem aos critérios.      | Aprovado pela equipe. |

## Conclusão da validação manual

A equipe informou a execução dos roteiros CT-REU-15 a CT-REU-20 em Android e iOS, com todos os cenários aprovados.
