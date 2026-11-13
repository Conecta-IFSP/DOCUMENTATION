# Diagramas de sequência — Manual de utilização

> Padrão da disciplina · Módulo MAN · Revisado em 13/11/2026

## Resumo

Há um diagrama para cada requisito de [Requisitos funcionais](2%20-%20requisitos-funcionais-manual-de-utilizacao.md), com o mesmo número. As linhas de vida são telas, componentes e módulos que existem no código do aplicativo. Toda mensagem é a chamada de uma função existente, com seus parâmetros, e as mensagens são numeradas na ordem em que acontecem. O que não é uma função, como a leitura de uma constante, uma regra de comparação ou o que o navegador faz fora do aplicativo, aparece em notas. Todas as funções chamadas estão no [Diagrama de classes](3%20-%20diagrama-classes-manual-de-utilizacao.md), com os tipos dos parâmetros e do retorno.

A tela Ajuda não envia requisições à API do Conecta+. Por isso nenhum diagrama tem `api`, guards, controller, service ou model.

| Linha de vida | Origem no código |
| --- | --- |
| `TelaAjuda` | `src/app/ajuda.tsx` (`TelaAjuda`, `primeiraPerguntaDo`, `definirBusca`, `rolarParaTopico`, `alternar` e `voltar`) |
| `buscaAjuda` | `src/utils/busca-ajuda.ts` (`filtrarTopicosAjuda` e `normalizarBusca`) |
| `Sanfona` | `src/components/Sanfona.tsx` (`alternar` e a propriedade `aoAlternar`) |
| `Botao` | `src/components/Botao.tsx` (propriedade `aoTocar`) |
| `Linking` | biblioteca `react-native` (`openURL`) |

## SD-MAN-01 — Consultar Ajuda

```mermaid
sequenceDiagram
    autonumber
    actor U as Usuário
    participant T as TelaAjuda
    participant B as buscaAjuda
    participant S as Sanfona
    Note over U,T: O usuário toca em "Precisa de ajuda?" na tela Entrar (topico igual a conta)<br/>ou em Ajuda, na seção Suporte do Perfil (sem topico)
    U->>T: TelaAjuda()
    T->>T: primeiraPerguntaDo(topico)
    Note over T: Compara topico com o id de cada tópico de TOPICOS_AJUDA, por igualdade.<br/>Se encontra, o id da primeira pergunta entra no conjunto abertas
    T->>B: filtrarTopicosAjuda(TOPICOS_AJUDA, busca)
    B->>B: normalizarBusca(busca)
    Note over B: Busca vazia: devolve a lista inteira, com 7 tópicos e 42 perguntas
    B-->>T: topicosVisiveis
    loop para cada pergunta de cada tópico visível
        T->>S: Sanfona(pergunta, resposta, selo, aberta, aoAlternar, primeira)
    end
    Note over T,S: selo vem de SELO_PERFIL: responsavel mostra "Responsável", todos não mostra selo
    opt topico recebido no endereço
        T->>T: rolarParaTopico()
    end
    opt busca por palavra
        Note over U,T: O usuário digita no campo Buscar, e o onChangeText do CampoTexto chama definirBusca
        U->>T: definirBusca(texto)
        T->>B: filtrarTopicosAjuda(TOPICOS_AJUDA, busca)
        B->>B: normalizarBusca(busca)
        Note over B: Mantém a pergunta quando o título do tópico, a pergunta e a resposta,<br/>juntos e sem acentos, contêm todas as palavras digitadas
        B-->>T: topicosVisiveis
        Note over U,T: A tela mostra "N perguntas encontradas" ou, sem resultado,<br/>"Nenhuma pergunta encontrada." e o EstadoVazio
    end
    U->>S: alternar()
    S->>T: aoAlternar(aberta)
    T->>T: alternar(perguntaId, aberta)
    Note over T: aberta true acrescenta perguntaId ao conjunto abertas, false retira
    T->>S: Sanfona(pergunta, resposta, selo, aberta, aoAlternar, primeira)
    Note over U,S: A resposta aparece em parágrafo ou em passos numerados
    U->>T: voltar()
    Note over T: router.back() se há tela anterior, senão router.replace para /inicio (com login) ou /login (sem login)
```

Para fechar a resposta, o usuário toca de novo na pergunta: as mensagens a partir de `alternar()` se repetem com `aberta` igual a `false`.

## SD-MAN-02 — Abrir manual em PDF

```mermaid
sequenceDiagram
    autonumber
    actor U as Usuário
    participant T as TelaAjuda
    participant Bt as Botao
    participant K as Linking
    Note over T: O Botao só é montado quando URL_MANUAL_PDF, de src/constants/ajuda.ts, não está vazia
    T->>Bt: Botao(titulo, icone, variante, pilula, aoTocar)
    U->>Bt: aoTocar()
    Note over T,Bt: aoTocar é a função que a TelaAjuda entregou ao Botao
    Bt->>K: openURL(URL_MANUAL_PDF)
    alt nenhum aplicativo do aparelho abre o endereço
        K-->>Bt: Promise rejeitada
        Note over T,Bt: O catch vazio de aoTocar ignora o erro e nenhuma mensagem é exibida
    else endereço entregue ao sistema operacional
        K-->>Bt: Promise resolvida
        Note over U,K: Fora do aplicativo: o navegador padrão abre a página do arquivo no GitHub.<br/>Arquivo publicado: 200 e o manual é exibido. Arquivo ausente: 404 do GitHub.<br/>Sem internet: mensagem de falta de conexão do próprio navegador
        Note over U,T: Ao voltar, o usuário encontra a tela Ajuda no mesmo ponto
    end
```

O aplicativo não verifica a conexão nem o resultado do carregamento: depois de `openURL`, tudo fica a cargo do navegador, que busca o endereço `https://github.com/Conecta-IFSP/DOCUMENTATION/blob/main/4%20-%20Incremento3/manual-de-utilizacao/manual-usuario/Manual-do-Usuario-Conecta.pdf`.

## Rastreabilidade

SD-MAN-01 e SD-MAN-02 correspondem, na mesma ordem, a UC-MAN-01 e UC-MAN-02 e a RF-MAN-01 e RF-MAN-02.
