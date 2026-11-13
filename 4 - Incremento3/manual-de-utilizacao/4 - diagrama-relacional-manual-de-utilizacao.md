# Diagrama relacional — Manual de utilização

> Padrão da disciplina · Módulo MAN · Revisado em 13/11/2026

## Resumo

**Não se aplica.** O R4 não lê nem grava dados em banco. Por isso não há tabelas, chaves nem relacionamentos para representar: este documento não contém diagrama relacional e, pelo mesmo motivo, também não há dicionário de dados relacional.

---

## Justificativa

**Conteúdo estático no aplicativo.** As perguntas e respostas da tela Ajuda são declaradas no próprio código do aplicativo, na constante `TOPICOS_AJUDA` do arquivo `src/constants/ajuda.ts`, e fazem parte do pacote instalado no aparelho. A tela Ajuda não envia nenhuma requisição à API e não consulta nenhuma coleção do MongoDB. Também não grava nada no aparelho: as perguntas abertas e o texto da busca existem apenas enquanto a tela está aberta. Para alterar o conteúdo, é preciso editar o arquivo e publicar uma nova versão do aplicativo.

**PDF hospedado no GitHub.** O manual completo é o arquivo `Manual-do-Usuario-Conecta.pdf`, versionado na pasta `4 - Incremento3/manual-de-utilizacao/manual-usuario/` do repositório `Conecta-IFSP/DOCUMENTATION`, branch `main`. O aplicativo guarda apenas o endereço desse arquivo, na constante `URL_MANUAL_PDF`, e o entrega ao sistema operacional para ser aberto no navegador ou no leitor de PDF. O arquivo não passa pelo servidor do Conecta+ e não é registrado no banco.

| Dado | Onde fica | Persistência em banco |
|---|---|---|
| Tópicos, perguntas e respostas | `src/constants/ajuda.ts`, no aplicativo | Nenhuma |
| Texto do selo "Responsável" | `SELO_PERFIL`, em `src/constants/ajuda.ts` | Nenhuma |
| Perguntas abertas e texto da busca | Estado da tela `src/app/ajuda.tsx`, descartado ao sair | Nenhuma |
| Manual completo em PDF | GitHub, pasta `4 - Incremento3/manual-de-utilizacao/manual-usuario/` do repositório `Conecta-IFSP/DOCUMENTATION` (branch `main`) | Nenhuma |
| Endereço do PDF | `URL_MANUAL_PDF`, em `src/constants/ajuda.ts` | Nenhuma |

A estrutura do conteúdo, com os tipos `TopicoAjuda`, `PerguntaAjuda` e `PerfilAjuda`, está descrita no diagrama e no dicionário de classes do R4. As coleções do sistema (`usuarios`, `organizacoes`, `comissoes`, `reunioes`, `notificacoes` e `contadores`) pertencem aos outros módulos e não são usadas pelo R4.
