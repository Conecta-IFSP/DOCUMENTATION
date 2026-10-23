# Requisitos não funcionais — Reuniões

> Padrão do Incremento 2 · Módulo REU

## Resumo

Os requisitos abaixo possuem critério, condição e forma de verificação. “Atendido” indica evidência por inspeção ou teste; itens dependentes de aparelho ou medição são identificados como validação necessária.

## Requisitos

| ID         | Qualidade      | Critério e medida                                                                                              | Condição                                           | Verificação/evidência                                          | Situação                               |
| ---------- | -------------- | -------------------------------------------------------------------------------------------------------------- | -------------------------------------------------- | -------------------------------------------------------------- | -------------------------------------- |
| RNF-REU-01 | Segurança      | Toda rota exige `X-API-Key`, JWT válido e conta comum ativa.                                                   | Qualquer rota `/reunioes`.                         | `ApiKeyGuard`, `JwtAuthGuard`, `UsuarioComumGuard`; CT-REU-13. | Atendido.                              |
| RNF-REU-02 | Autorização    | Usuário acessa somente reunião da qual participa; mutações especiais exigem organizador/responsável/aprovador. | Consulta, cancelamento, presença e ata.            | Testes `reunioes.regras.spec.ts` e `atas-presencas.spec.ts`.   | Atendido.                              |
| RNF-REU-03 | CORS           | Navegador recebe `Access-Control-Allow-Origin` apenas para origem configurada.                                 | Requisição com cabeçalho `Origin`.                 | Teste HTTP com origem permitida e bloqueada.                   | Atendido.                              |
| RNF-REU-04 | Concorrência   | Escrita com estado ou versão desatualizada retorna conflito sem sobrescrever dado novo.                        | Convite, presença, cancelamento e ata simultâneos. | Filtros de `findOneAndUpdate` e testes de conflito.            | Atendido.                              |
| RNF-REU-05 | Data/hora      | API armazena UTC/ISO 8601 e aplicativo apresenta no fuso do aparelho.                                          | Agenda, presença e ata.                            | Inspeção de DTOs/utilitários; teste em dois fusos.             | Código atendido; aparelho a validar.   |
| RNF-REU-06 | Desempenho     | Listagem deve responder em até 2 s em condição normal de desenvolvimento.                                      | Até 100 reuniões no período consultado.            | Medição com Postman/DevTools.                                  | Critério definido; medição necessária. |
| RNF-REU-07 | Portabilidade  | Fluxos devem funcionar em Android e iOS pelo Expo Go.                                                          | Execução do roteiro móvel.                         | Registrar versão do Expo Go, aparelho e resultado.             | Validação necessária.                  |
| RNF-REU-08 | Atualização    | Alteração observável em outra tela aparece em até 30 s, sem navegação.                                         | Tela da reunião aberta.                            | Consulta periódica e roteiro em dois aparelhos.                | Implementado; aparelho a validar.      |
| RNF-REU-09 | Acessibilidade | Ações principais têm rótulo acessível, contraste e área de toque utilizável.                                   | Agenda, reunião e registros.                       | Leitor de tela e inspeção visual Android/iOS.                  | Validação necessária.                  |
| RNF-REU-10 | Testabilidade  | `npm test` do backend e testes Node do frontend terminam sem falha.                                            | Commits auditados.                                 | 13 arquivos/182 testes no backend; 9 testes no frontend.       | Atendido em 23/10/2026.                |

## Rastreabilidade

Aplica-se a RF-REU-01 a RF-REU-12 e à matriz CT-REU.
