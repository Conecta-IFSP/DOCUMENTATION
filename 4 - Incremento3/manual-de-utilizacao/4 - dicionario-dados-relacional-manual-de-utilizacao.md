# Dicionário de Dados - Modelo Relacional do Manual de Utilização (R4)

**Não se aplica.** O R4 não lê nem grava dados em banco. Como não há tabelas, coleções, chaves nem relacionamentos (ver [Diagrama Relacional](4%20-%20diagrama-relacional-manual-de-utilizacao.md)), também não há campos para descrever neste dicionário.

---

## Justificativa

A tela Ajuda não envia requisições à API e não consulta nenhuma coleção do MongoDB. O conteúdo das perguntas e respostas é declarado no próprio aplicativo, em `src/constants/ajuda.ts`, e o manual completo é um arquivo PDF versionado no repositório `Conecta-IFSP/DOCUMENTATION`. Nenhum desses dados é persistido em banco.

| Dado | Onde fica | Tabela ou coleção |
|---|---|---|
| Tópicos, perguntas e respostas da Ajuda | `src/constants/ajuda.ts`, no aplicativo | Nenhuma |
| Perguntas abertas e texto da busca | Estado da tela `src/app/ajuda.tsx`, descartado ao sair | Nenhuma |
| Manual completo em PDF | `4 - Incremento3/manual-de-utilizacao/manual-usuario/` do repositório `Conecta-IFSP/DOCUMENTATION` | Nenhuma |

A estrutura do conteúdo da Ajuda (`TopicoAjuda`, `PerguntaAjuda` e `PerfilAjuda`) está descrita no [Dicionário de Dados - Classes](3%20-%20dicionario-dados-classes-manual-de-utilizacao.md). As coleções do sistema (`usuarios`, `organizacoes`, `comissoes` e `reunioes`) pertencem aos Incrementos 1 e 2 e não são usadas pelo R4.
