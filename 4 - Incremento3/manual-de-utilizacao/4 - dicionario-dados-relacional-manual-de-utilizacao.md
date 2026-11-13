# Dicionário relacional — Manual de utilização

> Padrão da disciplina · Módulo MAN · Revisado em 13/11/2026

## Resumo

**Não se aplica.** O R4 não lê nem grava dados em banco. Como não há tabelas, coleções, chaves nem relacionamentos (ver [Diagrama Relacional](4%20-%20diagrama-relacional-manual-de-utilizacao.md)), também não há campos para descrever neste dicionário. Os únicos identificadores do R4 são os `id` de tópico e de pergunta, que não ficam em banco; a forma como são criados está no [Dicionário de classes](3%20-%20dicionario-dados-classes-manual-de-utilizacao.md), seção "Como os identificadores são criados".

---

## Justificativa

A tela Ajuda não envia requisições à API e não consulta nenhuma coleção do MongoDB. O conteúdo das perguntas e respostas é declarado no próprio aplicativo, em `src/constants/ajuda.ts`, e o manual completo é um arquivo PDF versionado no repositório `Conecta-IFSP/DOCUMENTATION`. Nenhum desses dados é persistido em banco.

| Dado | Onde fica | Tabela ou coleção |
|---|---|---|
| Tópicos, perguntas e respostas da Ajuda | `src/constants/ajuda.ts`, no aplicativo | Nenhuma |
| Perguntas abertas e texto da busca | Estado da tela `src/app/ajuda.tsx`, descartado ao sair | Nenhuma |
| Manual completo em PDF | `4 - Incremento3/manual-de-utilizacao/manual-usuario/` do repositório `Conecta-IFSP/DOCUMENTATION` | Nenhuma |

A estrutura do conteúdo da Ajuda (`TopicoAjuda`, `PerguntaAjuda` e `PerfilAjuda`) está descrita no [Dicionário de Dados - Classes](3%20-%20dicionario-dados-classes-manual-de-utilizacao.md). As coleções do sistema (`usuarios`, `organizacoes`, `comissoes`, `reunioes`, `notificacoes` e `contadores`) pertencem aos outros módulos e não são usadas pelo R4.
