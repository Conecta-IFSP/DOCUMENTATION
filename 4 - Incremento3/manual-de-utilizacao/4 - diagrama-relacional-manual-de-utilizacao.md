# Diagrama relacional — Manual de utilização

> Padrão da disciplina · Módulo MAN · Revisado em 25/11/2026

## Resumo

**Não se aplica.** O R4 não lê nem grava dados em banco. Por isso não há tabelas, chaves nem relacionamentos para representar: este documento não contém diagrama relacional e, pelo mesmo motivo, também não há dicionário de dados relacional.

---

## Justificativa

**Conteúdo estático no aplicativo.** As perguntas e respostas da tela Ajuda são declaradas no próprio código do aplicativo, na constante `TOPICOS_AJUDA` do arquivo `src/constants/ajuda.ts`, e fazem parte do pacote instalado no aparelho. A tela Ajuda não envia nenhuma requisição à API e não consulta nenhuma coleção do MongoDB. Também não grava nada no aparelho: as perguntas abertas e o texto da busca existem apenas enquanto a tela está aberta. Para alterar o conteúdo, é preciso editar o arquivo e publicar uma nova versão do aplicativo.

**PDF e leitor empacotados no aplicativo.** O manual completo é o arquivo `assets/manual/manual.pdf` do FRONTEND, que vai junto com o aplicativo instalado. Ele é exibido na tela "Manual de utilização" pelo leitor PDF.js, também empacotado no aplicativo (`assets/pdfjs/`). A mesma versão do arquivo fica neste repositório, em `4 - Incremento3/manual-de-utilizacao/manual-usuario/manual.pdf`, como documentação; o aplicativo não acessa esse endereço. Para exibir o manual, o aplicativo lê os arquivos, monta uma página HTML e a guarda em memória (`cacheConteudo`), enquanto o aplicativo estiver aberto. A cópia que a biblioteca `expo-asset` deixa na pasta de cache do aparelho é um arquivo temporário do sistema, não um registro. Nada disso passa pelo servidor do Conecta+ nem é registrado no banco.

| Dado | Onde fica | Persistência em banco |
|---|---|---|
| Tópicos, perguntas e respostas | `src/constants/ajuda.ts`, no aplicativo | Nenhuma |
| Texto do selo "Responsável" | `SELO_PERFIL`, em `src/constants/ajuda.ts` | Nenhuma |
| Perguntas abertas e texto da busca | Estado da tela `src/app/ajuda.tsx`, descartado ao sair | Nenhuma |
| Manual completo em PDF | `assets/manual/manual.pdf`, no aplicativo; cópia de documentação em `4 - Incremento3/manual-de-utilizacao/manual-usuario/manual.pdf` do repositório `Conecta-IFSP/DOCUMENTATION` | Nenhuma |
| Leitor PDF.js | `assets/pdfjs/pdf.min.js.txt` e `assets/pdfjs/pdf.worker.min.js.txt`, no aplicativo | Nenhuma |
| Página HTML montada do manual | `cacheConteudo`, em memória, em `src/components/ManualPdf.tsx`; descartada ao fechar o aplicativo | Nenhuma |
| Cópia dos arquivos lidos | Pasta de cache do aplicativo no aparelho, gravada por `downloadAsync()` da `expo-asset` | Nenhuma |
| Fase da tela do manual (preparando, renderizando, pronto ou erro) | Estado do componente `Visualizador`, descartado ao sair | Nenhuma |

A estrutura do conteúdo, com os tipos `TopicoAjuda`, `PerguntaAjuda` e `PerfilAjuda`, está descrita no diagrama e no dicionário de classes do R4. As coleções do sistema (`usuarios`, `organizacoes`, `comissoes`, `reunioes`, `notificacoes` e `contadores`) pertencem aos outros módulos e não são usadas pelo R4.
