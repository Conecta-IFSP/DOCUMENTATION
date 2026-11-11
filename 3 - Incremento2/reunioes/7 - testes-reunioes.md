# Testes — Reuniões

> Padrão da disciplina · Módulo REU · Revisado em 10/11/2026

## Resumo

Registra evidência executada e cenários manuais no formato comum do projeto. Os testes automatizados cobrem regras de reunião, presença, lista, atas, autorização e concorrência.

## Evidência automatizada

| Data       | Projeto/commit      | Comando/teste                    | Resultado                          | Arquivos relacionados                                               |
| ---------- | ------------------- | -------------------------------- | ---------------------------------- | ------------------------------------------------------------------- |
| 09/11/2026 | BACKEND `homolog`   | `npm test`                       | 14 arquivos e 204 testes aprovados | `reunioes.regras.spec.ts`, `anexos.spec.ts`, `atas-presencas.spec.ts` |
| 09/11/2026 | BACKEND `homolog`   | `npm run build` e `npm run lint` | Aprovado, sem diagnóstico          | NestJS/TypeScript, Oxlint                                           |
| 09/11/2026 | FRONTEND `homolog`  | `npx tsc --noEmit` e `npx eslint src` | Sem erro e sem diagnóstico    | Projeto do aplicativo                                               |
| 23/10/2026 | BACKEND `6f2ffc4`   | `npm test`                       | 13 arquivos e 182 testes aprovados | Entrega do Incremento 2                                             |
| 23/10/2026 | FRONTEND `792fab4`  | testes Node do diretório `tests` | 9 testes aprovados                 | `horarios.test.cjs`, `sessao-api.test.cjs`                          |

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
| CT-REU-10 a CT-REU-12 | RF-ATA    | Testes da ata.                                                                         | Substituídos por CT-ATA-01 a CT-ATA-13, no módulo ATA.                                    | `atas-presencas.spec.ts`  |
| CT-REU-13 | RNF-REU-01/04 | Executar rotas sem chave/JWT e simular escrita concorrente.                            | 401/403 para credencial ausente; 409 para estado ou versão alterada.                      | Suíte backend             |
| CT-REU-14 | RF-REU-01     | Calcular a hora em que o seletor abre, no mesmo dia e em dia futuro.                   | Intervalos de 30 minutos e nenhum horário 24:00.                                          | `horarios.test.cjs`       |
| CT-REU-21 | RF-REU-13     | Editar título e pauta; mudar o horário antes e depois do início; trocar convidados; editar como membro comum; editar reunião encerrada. | Campos gravados; mudança de horário só antes do início, zerando os convites menos o do organizador; convidado externo recusado; membro comum recebe 403; reunião encerrada recebe 400. | `reunioes.regras.spec.ts` |
| CT-REU-22 | RF-REU-14 a RF-REU-16 | Enviar anexo válido, tipo não aceito, décimo primeiro anexo e arquivo em reunião encerrada; baixar como participante e como pessoa de fora; remover anexo existente e inexistente. | Anexo aceito só dentro das regras; download apenas para participantes; remoção apaga o arquivo no GridFS. | `anexos.spec.ts`          |
| CT-REU-23 | RF-REU-01     | Criar reunião de dia inteiro e sem local; criar com local de uma letra.                | Início às 00:00 e fim às 23:59:59.999; local vazio aceito; local de uma letra recusado.   | `reunioes.regras.spec.ts` |

## Matriz de testes manuais

| ID        | Requisito           | Cenário/entrada                                                     | Resultado esperado                                                         | Situação              |
| --------- | ------------------- | ------------------------------------------------------------------- | -------------------------------------------------------------------------- | --------------------- |
| CT-REU-15 | RF-REU-01/02/03     | Agendar no Expo Go, abrir agenda e visualizar em Android/iOS.       | Dados aparecem sem corte, teclado não bloqueia campos e filtros respondem. | Aprovado pela equipe. |
| CT-REU-16 | RF-REU-04/05        | Cancelar e responder convite, incluindo desistência da confirmação. | Confirmação protege ação destrutiva; estados são atualizados.              | Aprovado pela equipe. |
| CT-REU-17 | RF-REU-06/07/08     | Registrar entrada/saída e validar presença em dois aparelhos.       | Segundo aparelho atualiza em até 30 segundos.                              | Aprovado pela equipe. |
| CT-REU-18 | RF-REU-09           | Abrir e compartilhar lista de presença.                             | Dados e menu nativo de compartilhamento são exibidos.                      | Aprovado pela equipe. |
| CT-REU-19 | RF-REU-10/11/12     | Salvar, publicar e aprovar ata com dois usuários.                   | Versão, bloqueio do texto e contagem de aprovações permanecem coerentes.   | Substituído pelos roteiros CT-ATA-14 a CT-ATA-19. |
| CT-REU-20 | RNF-REU-03/05/07/09 | Testar origem web, fusos, Expo Go e leitor de tela.                 | CORS, horários, portabilidade e acessibilidade atendem aos critérios.      | Aprovado pela equipe. |
| CT-REU-24 | RF-REU-13           | Editar reunião pelo aplicativo, inclusive sair sem salvar.          | Formulário abre preenchido, avisa que o horário mudou e pede confirmação para descartar. | Aprovado em 09/11/2026, emuladores Android 16 e 7.0. |
| CT-REU-26 | RF-REU-01/RNF-REU-13 | Escolher início e término pelo seletor de hora do aparelho; ligar e desligar "Dia inteiro"; marcar um término anterior ao início; agendar a reunião. | O relógio do sistema abre na hora sugerida (início: próxima meia hora; término: uma hora depois do início), grava a escolha, "Dia inteiro" esconde e limpa os dois campos, o aviso "O término deve ser depois do início." aparece assim que a hora inválida é escolhida e a reunião é criada com o horário escolhido. | Aprovado em 10/11/2026, emuladores Android 16 e 7.0. No Android 7 o diálogo do sistema não oferece o modo de digitação, só o relógio. |
| CT-REU-27 | RF-REU-05/06/07, RNF-REU-14 | Aceitar o convite e depois tocar em "Não vou mais"; marcar entrada e saída com o aparelho bloqueado por PIN e cancelando a confirmação. | A desistência só é enviada depois do "Confirmar"; entrada e saída pedem a biometria ou o PIN, e o cancelamento mostra "Não foi possível confirmar sua identidade." sem gravar nada. | Aprovado em 10/11/2026, emuladores Android 16 e 7.0, ambos com PIN de tela. |
| CT-REU-25 | RF-REU-14/15/16     | Anexar, abrir e remover arquivo com nome acentuado.                 | Envio, download e remoção funcionam; o nome mantém os acentos.             | Aprovado em 09/11/2026, emulador Android 16. |

## Conclusão da validação manual

A equipe informou a execução dos roteiros CT-REU-15 a CT-REU-18 e CT-REU-20 em Android e iOS, na entrega do Incremento 2, com todos os cenários aprovados.

Os roteiros CT-REU-24 e CT-REU-25 foram executados em 09/11/2026, nos emuladores Android, durante as correções pedidas na avaliação do Incremento 2. Não houve execução em aparelho iOS nessa data, por falta de equipamento.
