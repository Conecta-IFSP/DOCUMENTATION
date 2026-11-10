# Requisitos não funcionais — Reuniões

> Padrão da disciplina · Módulo REU · Revisado em 10/11/2026

## Resumo

Os requisitos não funcionais aparecem em dois níveis. Os **de usuário** dizem, em uma frase, a qualidade esperada. Os **de sistema** completam cada um com critério e medida, condição, forma de verificação e situação. "Atendido" indica evidência por inspeção ou teste; o que depende de aparelho ou de medição está marcado como pendente.

A classificação segue a árvore de Sommerville usada na disciplina: requisitos de **produto**, **organizacionais** e **externos**.

## Requisitos não funcionais de usuário

| ID         | Classificação                     | Requisito de usuário |
| ---------- | --------------------------------- | -------------------- |
| RNF-REU-01 | Externo › Legal › Segurança | Só o aplicativo, com um usuário identificado, consegue usar as funções de reunião. |
| RNF-REU-02 | Externo › Legal › Privacidade | Cada pessoa só vê as reuniões para as quais foi convidada. |
| RNF-REU-03 | Externo › Interoperabilidade | A versão web só funciona a partir do endereço autorizado. |
| RNF-REU-04 | Produto › Confiabilidade | Duas pessoas agindo ao mesmo tempo não podem apagar o registro uma da outra. |
| RNF-REU-05 | Organizacional › Padrões | Datas e horários devem aparecer no horário do aparelho de quem consulta. |
| RNF-REU-06 | Produto › Eficiência › Desempenho | A agenda deve abrir em até 2 segundos. |
| RNF-REU-07 | Produto › Portabilidade | As reuniões devem funcionar em celulares Android a partir da versão 7.0 e em iOS. |
| RNF-REU-08 | Produto › Facilidade de uso | O que outra pessoa altera deve aparecer na tela aberta em até 30 segundos, sem sair dela. |
| RNF-REU-09 | Produto › Facilidade de uso | Os botões principais devem funcionar com leitor de tela. |
| RNF-REU-10 | Organizacional › Entrega | Os testes automatizados devem passar antes de cada entrega. |
| RNF-REU-11 | Produto › Eficiência › Espaço | Cada reunião aceita até 10 anexos de até 10 MB. |
| RNF-REU-12 | Produto › Confiabilidade | Editar a reunião não pode apagar respostas de convite nem presenças. |
| RNF-REU-13 | Produto › Facilidade de uso | A hora deve ser escolhida no mesmo seletor que o aparelho usa nos outros aplicativos. |

## Requisitos não funcionais de sistema

| ID         | Qualidade      | Critério e medida                                                                                              | Condição                                           | Verificação/evidência                                          | Situação                               |
| ---------- | -------------- | -------------------------------------------------------------------------------------------------------------- | -------------------------------------------------- | -------------------------------------------------------------- | -------------------------------------- |
| RNF-REU-01 | Segurança      | Toda rota exige `X-API-Key`, JWT válido e conta comum ativa.                                                   | Qualquer rota `/reunioes`.                         | `ApiKeyGuard`, `JwtAuthGuard`, `UsuarioComumGuard`; CT-REU-13. | Atendido.                              |
| RNF-REU-02 | Autorização    | Usuário acessa somente reunião da qual participa; mutações especiais exigem organizador/responsável/aprovador. | Consulta, cancelamento, presença e ata.            | Testes `reunioes.regras.spec.ts` e `atas-presencas.spec.ts`.   | Atendido.                              |
| RNF-REU-03 | CORS           | Navegador recebe `Access-Control-Allow-Origin` apenas para origem configurada.                                 | Requisição com cabeçalho `Origin`.                 | Teste HTTP com origem permitida e bloqueada.                   | Atendido.                              |
| RNF-REU-04 | Concorrência   | Escrita com estado ou versão desatualizada retorna conflito sem sobrescrever dado novo.                        | Convite, presença, cancelamento e ata simultâneos. | Filtros de `findOneAndUpdate` e testes de conflito.            | Atendido.                              |
| RNF-REU-05 | Data/hora      | API armazena UTC/ISO 8601 e aplicativo apresenta no fuso do aparelho.                                          | Agenda, presença e ata.                            | Inspeção de DTOs/utilitários; teste em dois fusos.             | Código atendido; aparelho a validar.   |
| RNF-REU-06 | Desempenho     | Listagem deve responder em até 2 s em condição normal de desenvolvimento.                                      | Até 100 reuniões no período consultado.            | Medição com Postman/DevTools.                                  | Critério definido; medição necessária. |
| RNF-REU-07 | Portabilidade  | Fluxos devem funcionar do Android 7.0 (API 24) ao Android 16, e no iOS.                                        | Execução do roteiro móvel.                         | Emuladores Android 16 e Android 7.0, com APK de release.       | Atendido no Android em 09/11/2026; iOS pendente, sem equipamento. |
| RNF-REU-08 | Atualização    | Alteração observável em outra tela aparece em até 30 s, sem navegação.                                         | Tela da reunião aberta.                            | Consulta periódica e roteiro em dois aparelhos.                | Implementado; aparelho a validar.      |
| RNF-REU-09 | Acessibilidade | Ações principais têm rótulo acessível, contraste e área de toque utilizável.                                   | Agenda, reunião e registros.                       | Leitor de tela e inspeção visual Android/iOS.                  | Validação necessária.                  |
| RNF-REU-10 | Testabilidade  | `npm test` do backend termina sem falha e o frontend passa em `tsc` e `eslint`.                                | Branch `homolog` auditada.                         | 14 arquivos/204 testes no backend; frontend sem erro de tipo nem diagnóstico de lint. | Atendido em 09/11/2026.                |
| RNF-REU-11 | Anexos         | Cada anexo tem até 10 MB, no máximo 10 por reunião, e só nos tipos PDF, imagem, texto, CSV, Office e LibreOffice. | Envio pela aba Pauta.                              | `anexos.ts`, `anexos.spec.ts` (7 testes) e CT-REU-22.          | Atendido em 09/11/2026.                |
| RNF-REU-13 | Facilidade de uso | Os campos "Início" e "Término" abrem o seletor nativo do sistema, em formato de 24 horas, já na hora sugerida; enquanto nada foi escolhido o campo mostra "Escolher". | Formulário de nova reunião e de edição. | `SeletorHora.tsx`; emuladores Android 16 e Android 7.0; CT-REU-26. | Atendido no Android em 10/11/2026. Limite conhecido: o diálogo é desenhado pelo sistema, então segue a aparência e o idioma do aparelho, e não o tema do aplicativo; no Android 7 ele aceita apenas o relógio, sem o modo de digitação que existe a partir do Android 8. iOS pendente, sem equipamento. |
| RNF-REU-12 | Edição segura  | Editar a reunião nunca apaga uma resposta de convite ou uma presença registrada em paralelo.                   | Edição simultânea a uma resposta de convite.       | Filtro por `atualizado_em` em `editar` e testes do bloco "edição". | Atendido em 09/11/2026.             |

## Rastreabilidade

Aplica-se a RF-REU-01 a RF-REU-09 e RF-REU-13 a RF-REU-16, e à matriz CT-REU. Os requisitos não funcionais da ata estão em [RNF-ATA](../../4%20-%20Incremento3/atas/5%20-%20requisitos-nao-funcionais-atas.md).
