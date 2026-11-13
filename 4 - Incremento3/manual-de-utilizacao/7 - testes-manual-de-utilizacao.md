# Testes - Manual de Utilização (R4)

Este documento registra os casos de teste do manual de utilização: a tela Ajuda do aplicativo e a abertura do manual completo em PDF. Os resultados só são marcados como aprovados depois de executados; os que ainda não foram executados aparecem como **Pendente**.

---

## 1. Ambiente e Execução

| Projeto | Comando | Ferramenta |
|---|---|---|
| FRONTEND | `node --test tests/*.test.cjs` | Executor de testes do Node.js (inclui os 5 testes da busca da Ajuda) |
| DOCUMENTATION | `pdfinfo Manual-do-Usuario-Conecta.pdf` | Poppler `pdfinfo` |
| Aparelhos | Roteiro manual da seção 3 | Um aparelho Android e um iPhone com o aplicativo instalado |

Fonte conferida: FRONTEND `homolog` e DOCUMENTATION `main` em 13/11/2026.

---

## 2. Massa de Dados

| Item | Valor |
|---|---|
| Usuário | Conta de teste com e-mail e senha, membro de uma comissão. |
| Conteúdo da Ajuda | `TOPICOS_AJUDA` atual: 7 tópicos e 42 perguntas, 9 com perfil `responsavel`. |
| Manual | `Manual-do-Usuario-Conecta.pdf` na pasta `4 - Incremento3/manual-de-utilizacao/manual-usuario/` da branch `main`. |

---

## 3. Casos de Teste

| ID | Requisito | Pré-condições | Passos | Resultado esperado | Resultado obtido |
|---|---|---|---|---|---|
| CT-MAN-01 | RF-MAN-01 (FA-01), RNF-MAN-01 | Sem sessão ativa, na tela Entrar. | Tocar em "Precisa de ajuda?". | A tela Ajuda abre sem pedir login, rola até Conta e acesso e mostra aberta a pergunta "Esqueci minha senha. O que faço?". | Pendente |
| CT-MAN-02 | RF-MAN-01 (FA-02) | Com sessão ativa. | Tocar no avatar, depois em Ajuda, na seção Suporte do Perfil. | A tela Ajuda abre com os 7 tópicos e todas as perguntas fechadas. | Pendente |
| CT-MAN-03 | RF-MAN-01 (passos 3 a 5) | Tela Ajuda aberta. | Tocar em uma pergunta; tocar nela de novo. | A resposta abre (texto ou passos numerados) e depois fecha. | Pendente |
| CT-MAN-04 | RF-MAN-01 (FA-03) | Tela Ajuda aberta. | Digitar "VOTACAO" no campo Buscar. | Aparecem só as perguntas que contêm "votação", ignorando acentos e maiúsculas, com a quantidade de perguntas encontradas. | Pendente |
| CT-MAN-05 | RF-MAN-01 (FE-01) | Tela Ajuda aberta. | Digitar uma palavra inexistente, por exemplo "xyzabc". | Aparecem "Nenhuma pergunta encontrada." e a orientação para consultar o manual completo. | Pendente |
| CT-MAN-06 | RF-MAN-01 (FA-03, passo 5) | Busca com texto digitado. | Tocar em "Limpar busca". | Os 7 tópicos voltam a ser exibidos. | Pendente |
| CT-MAN-07 | RNF-MAN-02 | Aparelho em modo avião. | Abrir a Ajuda, abrir uma resposta e buscar "ata". | A tela funciona sem nenhuma mensagem de erro de conexão. | Pendente |
| CT-MAN-08 | RNF-MAN-06 | Tela Ajuda aberta. | Abrir o tópico Reuniões e localizar uma pergunta exclusiva do responsável. | A pergunta exibe o selo "Responsável" e continua visível para qualquer usuário. | Pendente |
| CT-MAN-09 | RF-MAN-02, RNF-MAN-03 | Tela Ajuda aberta, aparelho com internet. | Rolar até o fim e tocar em "Ver manual completo (PDF)". | O navegador abre o manual no GitHub, e o aplicativo continua na tela Ajuda ao voltar. | Pendente |
| CT-MAN-10 | RF-MAN-02 (FE-01) | Aparelho sem internet. | Tocar em "Ver manual completo (PDF)". | O navegador mostra a própria mensagem de falta de conexão; ao voltar, o aplicativo continua na tela Ajuda. | Pendente |
| CT-MAN-11 | RNF-MAN-04 | Arquivo publicado na `main`. | Executar `pdfinfo Manual-do-Usuario-Conecta.pdf`. | Todas as páginas em A4. | Aprovado em 13/11/2026: 27 páginas de 594.96 × 841.92 pt (A4). |
| CT-MAN-12 | RNF-MAN-07 | Aplicativo instalado em um Android e em um iPhone. | Executar CT-MAN-01 a CT-MAN-04 e CT-MAN-09 em cada aparelho. | Mesmo comportamento nos dois sistemas. | Pendente |
| CT-MAN-13 | RNF-MAN-08, RNF-MAN-09 | FRONTEND na branch `homolog`. | Executar `node --test tests/*.test.cjs`. | Todos os testes aprovados, incluindo os 5 da busca da Ajuda. | Pendente |

---

## 4. Resumo

| Situação | Casos |
|---|---|
| Aprovados | CT-MAN-11 |
| Pendentes | CT-MAN-01 a CT-MAN-10, CT-MAN-12 e CT-MAN-13 |

Ao executar os pendentes, registre na coluna "Resultado obtido" a data, o aparelho (modelo e versão do sistema) e o resultado.
