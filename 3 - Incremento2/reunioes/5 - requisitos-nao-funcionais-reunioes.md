# Requisitos Não Funcionais - Reuniões

---

## Visão Geral

Os requisitos não funcionais abaixo definem critérios de segurança, confiabilidade, desempenho, portabilidade, usabilidade, acessibilidade e manutenibilidade das funcionalidades de reuniões. Cada requisito possui um critério de aceitação que pode ser verificado.

| Código | Categoria | Requisito | Prioridade |
|---|---|---|---|
| RNF-01 | Segurança | As rotas de reuniões devem aceitar somente requisições que apresentem a chave de acesso do aplicativo. | Crítica |
| RNF-02 | Segurança | O acesso às reuniões deve exigir um token de login com validade de 1 hora. | Crítica |
| RNF-03 | Segurança | A API deve atender navegadores somente a partir das origens autorizadas. | Média |
| RNF-04 | Confiabilidade | Gravações simultâneas sobre a mesma reunião não podem sobrescrever uma à outra. | Alta |
| RNF-05 | Confiabilidade | Datas e horários devem ser armazenados em UTC e exibidos no fuso horário do aparelho. | Alta |
| RNF-06 | Desempenho | As consultas de agenda devem ser atendidas por índice, sem percorrer toda a coleção. | Média |
| RNF-07 | Portabilidade | O aplicativo deve funcionar em Android, iOS e navegador a partir do mesmo código. | Alta |
| RNF-08 | Usabilidade | A tela da reunião deve se atualizar sem que o usuário precise trocar de tela. | Média |
| RNF-09 | Acessibilidade | Os elementos de interação devem ter tamanho mínimo de toque e nome para leitores de tela. | Média |
| RNF-10 | Manutenibilidade | As regras de negócio das reuniões devem ser cobertas por testes automatizados. | Alta |

---

# RNF-01 - Chave de Acesso da API

| Campo | Descrição |
|---|---|
| **Categoria** | Segurança |
| **Descrição** | Toda requisição às rotas de reuniões deve conter o cabeçalho `X-API-Key` com a chave definida na variável `API_ACCESS_TOKEN` do servidor. A comparação deve levar o mesmo tempo independentemente do valor recebido, para que a chave não possa ser descoberta pelo tempo de resposta. |
| **Critério de Aceitação** | Uma requisição sem o cabeçalho ou com um valor diferente recebe a resposta `401` sem executar nenhuma operação. Com o servidor sem a variável `API_ACCESS_TOKEN`, toda requisição recebe `503`. |
| **Forma de Verificação** | Enviar requisições às rotas de reuniões sem o cabeçalho e com um valor incorreto e conferir a resposta `401`. |
| **Aplicação** | RF-01 a RF-12 |
| **Prioridade** | Crítica |

---

# RNF-02 - Token de Login

| Campo | Descrição |
|---|---|
| **Categoria** | Segurança |
| **Descrição** | As rotas de reuniões devem exigir o cabeçalho `Authorization: Bearer <token>`, com um token JWT assinado pelo servidor que expira 1 hora depois de emitido. A cada requisição, o servidor deve conferir que a conta do token existe e está ativa. |
| **Critério de Aceitação** | Uma requisição sem token, com token adulterado ou emitido há mais de 1 hora recebe `401`. Uma requisição com token válido de uma conta desativada depois do login também recebe `401`. |
| **Forma de Verificação** | Testes automatizados de acesso com token expirado e com conta desativada, executados com `npm test` no servidor. |
| **Aplicação** | RF-01 a RF-12 |
| **Prioridade** | Crítica |

---

# RNF-03 - Origens Autorizadas

| Campo | Descrição |
|---|---|
| **Categoria** | Segurança |
| **Descrição** | A API deve permitir chamadas de navegador apenas das origens `localhost`, `127.0.0.1`, do esquema do aplicativo `conectamais://` e do endereço definido na variável `FRONTEND_URL`. Os únicos cabeçalhos aceitos são `Content-Type`, `Authorization` e `X-API-Key`. |
| **Critério de Aceitação** | Uma requisição de pré-verificação (`OPTIONS`) vinda de outra origem recebe a resposta sem o cabeçalho `Access-Control-Allow-Origin`, e o navegador bloqueia a chamada. Uma requisição de origem autorizada recebe esse cabeçalho com a própria origem. |
| **Forma de Verificação** | Enviar `OPTIONS /reunioes` com `Origin: https://exemplo.com` e com `Origin: http://localhost:8081` e comparar os cabeçalhos das respostas. |
| **Aplicação** | RF-01 a RF-12 |
| **Prioridade** | Média |

---

# RNF-04 - Gravações Simultâneas

| Campo | Descrição |
|---|---|
| **Categoria** | Confiabilidade |
| **Descrição** | Cada alteração de uma reunião deve ser gravada em uma única operação atômica, cuja condição repete as verificações feitas na leitura, como a situação da reunião, a existência da presença e a ausência de saída. Se outra requisição alterar a reunião entre a leitura e a gravação, nada deve ser gravado. |
| **Critério de Aceitação** | Duas requisições simultâneas que alteram o mesmo dado resultam em uma gravação bem-sucedida e uma resposta `409`, sem perda da primeira. Duas entradas simultâneas do mesmo participante geram uma única presença. |
| **Forma de Verificação** | Enviar ao mesmo tempo duas requisições de entrada do mesmo participante e conferir que uma recebe `409` e que a coleção `reunioes` registra uma única presença. |
| **Aplicação** | RF-04 a RF-08 |
| **Prioridade** | Alta |

---

# RNF-05 - Datas em UTC

| Campo | Descrição |
|---|---|
| **Categoria** | Confiabilidade |
| **Descrição** | Início, término, entradas e saídas devem ser armazenados em UTC e trocados entre aplicativo e servidor no formato ISO 8601. O aplicativo deve converter as datas para o fuso horário do aparelho apenas na exibição, em português do Brasil. |
| **Critério de Aceitação** | Uma reunião agendada às 19:00 em um aparelho no fuso UTC−3 é armazenada como 22:00 UTC e exibida às 19:00 em qualquer aparelho no mesmo fuso. |
| **Forma de Verificação** | Agendar uma reunião pelo aplicativo, consultar o documento na coleção `reunioes` e comparar o horário armazenado com o exibido. |
| **Aplicação** | RF-01 a RF-03, RF-06, RF-07, RF-09 |
| **Prioridade** | Alta |

---

# RNF-06 - Consultas por Índice

| Campo | Descrição |
|---|---|
| **Categoria** | Desempenho |
| **Descrição** | A coleção `reunioes` deve ter os índices compostos (`participantes.usuario_id`, `inicio`) e (`comissao_id`, `inicio`), usados pelas consultas de agenda do usuário e de reuniões de uma comissão. |
| **Critério de Aceitação** | O plano de execução da consulta de agenda de um usuário mostra a etapa `IXSCAN` sobre o índice `participantes.usuario_id_1_inicio_1`, e o da consulta por comissão mostra `IXSCAN` sobre `comissao_id_1_inicio_1`. Nenhuma das duas mostra `COLLSCAN`. |
| **Forma de Verificação** | No `mongosh`, executar `db.reunioes.find({ "participantes.usuario_id": <id> }).sort({ inicio: 1 }).explain()` e a consulta equivalente por `comissao_id` e conferir o índice usado. |
| **Aplicação** | RF-02 |
| **Prioridade** | Média |

---

# RNF-07 - Portabilidade

| Campo | Descrição |
|---|---|
| **Categoria** | Portabilidade |
| **Descrição** | O aplicativo deve ser desenvolvido uma única vez em React Native com Expo SDK 57 e gerado para Android, iOS e navegador. |
| **Critério de Aceitação** | O mesmo código gera o aplicativo Android, para a versão 7.0 (API 24) ou superior, o aplicativo iOS e a versão web, e os fluxos de agendamento, convite e presença funcionam nos três. |
| **Forma de Verificação** | Executar os fluxos de reunião em um aparelho Android, em um aparelho iOS e no navegador. |
| **Aplicação** | RF-01 a RF-12 |
| **Prioridade** | Alta |

---

# RNF-08 - Atualização Sem Trocar de Tela

| Campo | Descrição |
|---|---|
| **Categoria** | Usabilidade |
| **Descrição** | Com a tela da reunião aberta, as presenças e as respostas de convite dos outros participantes devem aparecer sem que o usuário saia e volte para a tela. Todas as telas de reunião devem permitir recarregar os dados puxando a tela para baixo. |
| **Critério de Aceitação** | Uma entrada registrada por um participante aparece na tela do organizador em até 30 segundos, sem nenhuma ação do organizador. Puxar a tela para baixo recarrega os dados imediatamente. |
| **Forma de Verificação** | Abrir a mesma reunião em dois aparelhos, registrar a entrada em um deles e cronometrar o tempo até ela aparecer no outro. |
| **Aplicação** | RF-02, RF-03, RF-05 a RF-09 |
| **Prioridade** | Média |

---

# RNF-09 - Acessibilidade

| Campo | Descrição |
|---|---|
| **Categoria** | Acessibilidade |
| **Descrição** | Os botões e itens tocáveis das telas de reunião devem ter altura mínima de 40 pontos e um nome acessível, pelo texto visível ou por um rótulo próprio, para os leitores de tela TalkBack e VoiceOver. |
| **Critério de Aceitação** | Com o TalkBack ativo, todos os botões das telas de reunião são anunciados pelo nome da ação, como "Confirmar presença de Ana Souza". Nenhum alvo de toque tem menos de 40 pontos de altura. |
| **Forma de Verificação** | Percorrer as telas de reunião com o TalkBack ativo e medir os alvos de toque com o inspetor de layout. |
| **Aplicação** | RF-01 a RF-12 |
| **Prioridade** | Média |

---

# RNF-10 - Testes Automatizados

| Campo | Descrição |
|---|---|
| **Categoria** | Manutenibilidade |
| **Descrição** | Cada requisito funcional das reuniões deve ter ao menos um caso de teste automatizado que verifique suas regras de negócio. |
| **Critério de Aceitação** | O comando `npm test` no servidor termina sem falhas; os fluxos de ata possuem cobertura em `atas-presencas.spec.ts`, e os demais requisitos permanecem associados aos casos de teste registrados. |
| **Forma de Verificação** | Executar `npm test` no servidor e `node --test tests/*.cjs` no aplicativo. |
| **Aplicação** | RF-01 a RF-12 |
| **Prioridade** | Alta |
