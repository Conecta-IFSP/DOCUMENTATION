# Testes — Manual de utilização

> Padrão da disciplina · Módulo MAN · Revisado em 13/11/2026

## Resumo

Este documento registra os casos de teste do manual de utilização: a tela Ajuda do aplicativo e a abertura do manual completo em PDF. Os resultados só são marcados como aprovados depois de executados; os que ainda não foram executados aparecem como **Pendente**.

---

## 1. Ambiente e Execução

| Projeto | Comando | Ferramenta |
|---|---|---|
| FRONTEND | `node --test tests/*.test.cjs` | Executor de testes do Node.js 24.12.0, no Windows (inclui os 5 testes da busca da Ajuda) |
| DOCUMENTATION | `pdfinfo Manual-do-Usuario-Conecta.pdf` | Poppler `pdfinfo` |
| Aparelho | Roteiro manual da seção 3 | iPhone 14, iOS 27.2, com o aplicativo aberto pelo Expo Go (`npx expo start --go`) e o backend em execução na mesma rede. Android não executado: a equipe não tinha aparelho Android disponível em 13/11/2026. |

Fonte conferida: FRONTEND `homolog` (`66806e4`) e DOCUMENTATION `main` em 13/11/2026.

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
| CT-MAN-01 | RF-MAN-01 (Ação, itens 1 e 3), RNF-MAN-01 | Sem sessão ativa, na tela Entrar. | Tocar em "Precisa de ajuda?". | A tela Ajuda abre sem pedir login, rola até Conta e acesso e mostra aberta a pergunta "Esqueci minha senha. O que faço?". | Aprovado em 13/11/2026, no iPhone, via Expo Go. |
| CT-MAN-02 | RF-MAN-01 (Ação, itens 1 e 2) | Com sessão ativa. | Tocar no avatar, depois em Ajuda, na seção Suporte do Perfil. | A tela Ajuda abre com os 7 tópicos e todas as perguntas fechadas. | Aprovado em 13/11/2026, no iPhone, via Expo Go. |
| CT-MAN-03 | RF-MAN-01 (Ação, item 5) | Tela Ajuda aberta. | Tocar em uma pergunta; tocar nela de novo. | A resposta abre (texto ou passos numerados) e depois fecha. | Aprovado em 13/11/2026, no iPhone, via Expo Go. |
| CT-MAN-04 | RF-MAN-01 (Ação, itens 6 e 7) | Tela Ajuda aberta. | Digitar "VOTACAO" no campo Buscar. | A tela mostra "7 perguntas encontradas" e só o tópico Votação, com as suas 7 perguntas: a busca ignora acentos e maiúsculas e também procura no título do tópico. | Aprovado em 13/11/2026, no iPhone, via Expo Go. |
| CT-MAN-05 | RF-MAN-01 (Ação, item 7) | Tela Ajuda aberta. | Digitar uma palavra inexistente, por exemplo "xyzabc". | Aparecem "Nenhuma pergunta encontrada." e a orientação para consultar o manual completo. | Aprovado em 13/11/2026, no iPhone, via Expo Go. |
| CT-MAN-06 | RF-MAN-01 (Ação, item 7) | Busca com texto digitado. | Tocar em "Limpar busca". | Os 7 tópicos voltam a ser exibidos. | Aprovado em 13/11/2026, no iPhone, via Expo Go. |
| CT-MAN-07 | RNF-MAN-02 | Aparelho em modo avião. | Abrir a Ajuda, abrir uma resposta e buscar "senha". | A tela exibe os 7 tópicos, a resposta abre, e a busca mostra "4 perguntas encontradas", todas do tópico Conta e acesso, sem nenhuma mensagem de erro de conexão. | Aprovado em 13/11/2026, no iPhone, via Expo Go. |
| CT-MAN-08 | RNF-MAN-06 | Tela Ajuda aberta, com uma conta que não é responsável por nenhuma comissão. | Rolar até o tópico Reuniões e localizar a pergunta "Como agendo uma reunião?". | A pergunta exibe o selo "Responsável" e continua visível e pode ser aberta. Em toda a tela, 9 perguntas exibem o selo. | Aprovado em 13/11/2026, no iPhone, via Expo Go. |
| CT-MAN-09 | RF-MAN-02 (Ação, itens 1 a 3 e 5), RNF-MAN-03 | Tela Ajuda aberta, aparelho com internet. | Rolar até o fim e tocar em "Ver manual completo (PDF)". | O navegador abre o manual no GitHub, e o aplicativo continua na tela Ajuda ao voltar. | Aprovado em 13/11/2026, no iPhone, via Expo Go. |
| CT-MAN-10 | RF-MAN-02 (Ação, item 4) | Aparelho sem internet. | Tocar em "Ver manual completo (PDF)". | O navegador mostra a própria mensagem de falta de conexão; ao voltar, o aplicativo continua na tela Ajuda. | Aprovado em 13/11/2026, no iPhone, via Expo Go. |
| CT-MAN-11 | RNF-MAN-04 | Arquivo publicado na `main`. | Executar `pdfinfo Manual-do-Usuario-Conecta.pdf`. | Todas as páginas em A4. | Aprovado em 13/11/2026: 27 páginas de 594.96 × 841.92 pt (A4). |
| CT-MAN-12 | RNF-MAN-07 | Aplicativo instalado em um Android e em um iPhone. | Executar CT-MAN-01 a CT-MAN-04 e CT-MAN-09 em cada aparelho. | Mesmo comportamento nos dois sistemas. | Parcial em 13/11/2026: CT-MAN-01 a CT-MAN-04 e CT-MAN-09 aprovados no iPhone, via Expo Go. Android não executado. |
| CT-MAN-13 | RNF-MAN-08, RNF-MAN-09 | FRONTEND na branch `homolog`. | Executar `node --test tests/*.test.cjs`. | Todos os testes aprovados, incluindo os 5 da busca da Ajuda. | Aprovado em 13/11/2026: 18 testes, 0 falhas, incluindo os 5 da busca da Ajuda. |

---

## 4. Situação dos casos

| Situação | Casos |
|---|---|
| Aprovados | CT-MAN-01 a CT-MAN-11 e CT-MAN-13 |
| Parcial | CT-MAN-12 (aprovado no iPhone; Android não executado) |
| Pendentes | Nenhum |

Os roteiros manuais foram executados no aparelho descrito na seção 1. Quando o CT-MAN-12 for executado em Android, registre na coluna "Resultado obtido" a data, o aparelho e o resultado.
