# Publicação das alterações — 21/10/2026

Branch de entrega nos três repositórios: `codex/comissoes-incremento-2`.

## Conteúdo

- Backend: responsável único e transferência de responsabilidade; atas e lista de presença; seleção dos convidados; bloqueio de alteração do e-mail; testes automatizados e integração MongoDB.
- Frontend: melhorias de comissões, reuniões, convidados e horários; atualização por gesto; atas e presença; e-mail visível e não editável; acesso biométrico opcional no aplicativo nativo e confirmação local de ações sensíveis.
- Documentação: pacote `comissoes` com requisitos, dicionários, diagramas e testes; [acesso biométrico](ACESSO-BIOMETRICO.md).

## Bases e integração

Backend baseado em `7b8876e`, atualizado de `origin/homolog`. Frontend baseado em `18bde43`. Na preparação da publicação, `origin/homolog` do frontend havia avançado para `f437286`; essas alterações posteriores do grupo não foram incorporadas nesta branch. É necessário conciliar as alterações antes de mesclar na homolog, especialmente login, perfil, configuração e dependências. Nenhuma branch homolog/main foi sobrescrita.

## Verificação

- Backend: 184 testes aprovados em 14 arquivos e compilação aprovada nesta publicação. A integração HTTP/JWT/MongoDB foi validada anteriormente durante a implementação.
- Frontend: 16 testes aprovados, verificação de tipos, lint dos arquivos de autenticação/perfil alterados e exportação web aprovados.
- Pendente: build nativo e validação em aparelhos Android/iOS. Os testes de biometria usam simulações e não comprovam o funcionamento de sensores ou permissões reais.
- Não foram incluídos arquivos .env, credenciais locais, logs, bancos, node_modules ou builds de demonstração.

Os registros anteriores que dizem “sem commit/push” descrevem o estado daquela revisão; esta nota registra a entrega autorizada posteriormente pela responsável.
