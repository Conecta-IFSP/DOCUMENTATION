# Testes — Atas de reunião

> Padrão da disciplina · Módulo ATA · Revisado em 10/11/2026

## Resumo

Registra o que foi executado, quando, em qual ambiente e qual foi o resultado. Os testes automatizados cobrem as regras do servidor; os roteiros manuais cobrem as telas, em dois emuladores Android. Nada aqui está marcado como aprovado sem ter sido executado.

## Ambiente

| Item                | Valor                                                                                     |
| ------------------- | ------------------------------------------------------------------------------------------- |
| Data da execução    | 09/11/2026                                                                                  |
| Node.js / npm       | 24.19.0 / 11.17.0                                                                           |
| Backend             | branch `homolog` com as alterações do Incremento 3, banco `conecta-teste-claude` local     |
| Aplicativo          | branch `homolog`, APK de release gerado para `x86_64`, apontando para a API local          |
| Aparelho 1          | Emulador Android 16 (API 36), 1080 × 2400, com PIN cadastrado                               |
| Aparelho 2          | Emulador Android 7.0 (API 24, versão mínima do aplicativo), 720 × 1280, sem bloqueio de tela |
| iOS                 | Não executado: a equipe não dispõe de computador Apple nem de aparelho iOS                 |

## Evidência automatizada

| Data       | Projeto  | Comando                   | Resultado                                                  |
| ---------- | -------- | ------------------------- | ---------------------------------------------------------- |
| 09/11/2026 | BACKEND  | `npm test`                | 14 arquivos e 204 testes aprovados, nenhuma falha          |
| 09/11/2026 | BACKEND  | `npm run build`           | Aprovado                                                   |
| 09/11/2026 | BACKEND  | `npm run lint`            | Sem diagnóstico                                            |
| 09/11/2026 | FRONTEND | `npx tsc --noEmit`        | Sem erro                                                   |
| 09/11/2026 | FRONTEND | `npx eslint src`          | Sem diagnóstico                                            |

Dos 204 testes, 12 são do arquivo `BACKEND/src/reunioes/atas-presencas.spec.ts`, descritos abaixo.

## Matriz de testes automatizados

| ID        | Requisito            | Cenário e entrada                                                                                             | Resultado esperado                                                                                        | Teste                                                        |
| --------- | -------------------- | --------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------ |
| CT-ATA-01 | RF-ATA-01            | Salvar a primeira ata de uma reunião encerrada, com versão 0.                                                  | Grava com `ata: { $exists: false }` no filtro, versão 1, situação `RASCUNHO` e `avaliacoes` vazia.         | "salva ata versionada apenas pelo responsável…"              |
| CT-ATA-02 | RF-ATA-01, RNF-ATA-01 | Salvar como participante comum; consultar a lista de presença como pessoa de fora.                            | 403 nos dois casos.                                                                                        | "recusa participante comum como autor…"                      |
| CT-ATA-03 | RF-ATA-01            | Salvar antes do fim da reunião, com a reunião cancelada e com versão antiga.                                  | 400, 400 e 409, nessa ordem.                                                                               | "recusa edição antes do fim, cancelamento e versão obsoleta" |
| CT-ATA-04 | RF-ATA-02            | Enviar com prazo de 72 horas; depois tentar enviar sem nenhuma presença confirmada.                           | Situação `EM_APROVACAO`, aprovadores com os dois presentes, aprovação de quem enviou, prazo 72 h; depois 400. | "envia com prazo, congela os presentes…"                   |
| CT-ATA-05 | RF-ATA-03, RNF-ATA-05 | Aprovar com um dos dois presentes e depois com o segundo.                                                     | Primeiro segue `EM_APROVACAO` sem tocar no contador; o segundo gera `APROVADA`, número 7, ano de São Paulo e `$inc` com `upsert`. | "só aprova com todos e então numera a ata…"       |
| CT-ATA-06 | RF-ATA-04            | Reprovar apontando um trecho existente, sem justificativa; depois apontar um trecho que não está na ata.      | Situação `REPROVADA`, avaliação com trecho e sem justificativa; depois 400 "O trecho não faz parte da ata". | "reprova com justificativa opcional…"                        |
| CT-ATA-07 | RF-ATA-03, RF-ATA-04, RNF-ATA-02 | Repetir a mesma decisão; trocar a decisão; avaliar sem estar entre os aprovadores.                  | Repetição não grava nada; troca recebe 400; quem não é aprovador recebe 403.                              | "não deixa trocar a decisão nem avaliar…"                    |
| CT-ATA-08 | RF-ATA-05            | Editar a ata reprovada; refazer como participante comum; refazer como presidente; refazer uma ata em aprovação. | 400; 403; a versão vai para `atas_canceladas` com `CANCELADA` e nasce o rascunho na versão seguinte; 400.  | "cancela a ata reprovada e abre uma nova…"                   |
| CT-ATA-09 | RF-ATA-07            | Consultar as atas pendentes de um participante.                                                               | A consulta usa `$or` com aprovador sem avaliação e autor de ata reprovada.                                 | "lista como pendente a ata a avaliar…"                       |
| CT-ATA-10 | RF-ATA-02, RF-ATA-04 | Validar `EnviarAtaDto` com prazo de 5 horas; `ReprovarAtaDto` com justificativa em branco e com 501 caracteres. | Prazo inválido gera erro; justificativa em branco vira ausente e é aceita; 501 caracteres gera erro.       | "valida o prazo de envio e deixa a justificativa opcional"   |
| CT-ATA-11 | RNF-ATA-04           | Simular `findOneAndUpdate` devolvendo `null` ao salvar.                                                        | 409 "A ata mudou. Atualize antes de continuar."                                                            | "informa conflito quando outra escrita alterou a ata"        |
| CT-ATA-12 | RF-ATA-06            | Gerar a lista de presença com um presente e um sem registro.                                                  | Nome, situação e minutos corretos; a ausência não cria presença.                                           | "gera lista com nomes, situações, duração…"                  |
| CT-ATA-13 | RF-ATA-01 a RF-ATA-05 | Percorrer o fluxo inteiro contra a API em execução, com MongoDB real: salvar, enviar, reprovar, refazer, corrigir, reenviar e aprovar. | Cada passo devolve a situação esperada; a ata termina `APROVADA`, numerada, com uma versão cancelada no histórico. | Roteiro executado pela API de teste (porta 3001).   |

## Matriz de testes manuais

| ID        | Requisito                | Aparelho            | Cenário                                                                                                | Resultado obtido                                                                                            |
| --------- | ------------------------ | ------------------- | -------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------- |
| CT-ATA-14 | RF-ATA-01, RF-ATA-02     | Android 16          | Abrir a ata de uma reunião encerrada, escrever o texto, salvar e enviar com prazo de 3 dias.           | Aprovado. A folha mostrou os presentes, o prazo calculado e o aviso de que enviar registra a própria aprovação. |
| CT-ATA-15 | RF-ATA-03, RF-ATA-04, RF-ATA-07 | Android 16   | Entrar com o segundo participante, ver a ata na tela inicial, apontar um parágrafo e reprovar com justificativa. | Aprovado. A ata ficou "Reprovada" e a faixa mostrou o nome, a data, a justificativa e o trecho.              |
| CT-ATA-16 | RF-ATA-05, RF-ATA-06     | Android 16          | Como presidente, criar a nova versão, corrigir o texto e reenviar; conferir, como aprovador, o quadro de mudanças. | Aprovado. A versão 1 ficou no histórico como cancelada e o quadro mostrou só o trecho alterado.             |
| CT-ATA-17 | RF-ATA-03, RF-ATA-08, RNF-ATA-03, RNF-ATA-05 | Android 16 | Aprovar com o PIN do aparelho e gerar o PDF da ata aprovada.                                | Aprovado. A aprovação pediu "Confirme sua identidade"; a ata virou "Ata nº 4/2026" e o PDF saiu como `ata-4-2026.pdf`, conferido na pré-visualização de impressão. |
| CT-ATA-18 | RF-ATA-04, RNF-ATA-08    | Android 7.0, tela pequena | Abrir a ata, apontar um parágrafo, abrir a folha de reprovação com o teclado aberto e reprovar sem justificativa. | Aprovado. Nada ficou cortado e a ata registrou "Sem justificativa.".                                   |
| CT-ATA-19 | RNF-ATA-09               | Android 7.0         | Repetir consulta, comparação de versões, reprovação e aprovação com o aplicativo no tema escuro.       | Aprovado. Textos, faixas e o destaque verde e vermelho da comparação permaneceram legíveis.                 |
| CT-ATA-20 | RNF-ATA-03, RNF-ATA-08   | iOS                 | Repetir CT-ATA-14 a CT-ATA-17 em aparelho iOS.                                                         | Não executado: sem equipamento Apple disponível.                                                            |

## Defeitos encontrados e corrigidos durante os testes

| Data       | Defeito                                                                                                   | Correção                                                                               |
| ---------- | ----------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------- |
| 09/11/2026 | Na tela inicial, o título da ata pendente aparecia cortado, porque vinha depois do rótulo "Ata para aprovar". | O nome da reunião passou para a primeira linha e o prazo para a segunda.                |
| 09/11/2026 | Ao puxar a tela para atualizar, a faixa verde da ação anterior continuava visível mesmo com a ata já alterada por outra pessoa. | `atualizar` passou a limpar a mensagem de sucesso antes de recarregar.                  |
| 09/11/2026 | Quem não é presidente recebia a mensagem "Apenas o responsável da comissão pode agendar reuniões" ao tentar mexer na ata. | A mensagem passou a dizer "escrever a ata".                                             |

## Pendências

- executar CT-ATA-20 em aparelho iOS;
- medir o tempo de abertura da ata com texto próximo de 20.000 caracteres (RNF-ATA-07);
- testar a virada de ano para confirmar o ano do número da ata (RNF-ATA-06).

## Rastreabilidade

CT-ATA-01 a CT-ATA-13 cobrem RF-ATA-01 a RF-ATA-07; CT-ATA-14 a CT-ATA-19 cobrem as telas e os RNFs; CT-ATA-20 está pendente.
