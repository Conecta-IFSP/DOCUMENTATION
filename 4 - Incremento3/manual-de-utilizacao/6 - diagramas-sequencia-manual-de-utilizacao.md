# Diagramas de Sequência - Manual de Utilização (R4)

Os participantes de cada diagrama são as telas, os componentes e os módulos do aplicativo, e as mensagens são as chamadas de funções entre eles. A tela Ajuda não envia requisições à API do Conecta+. Por isso, nenhum diagrama tem guards, controllers, services ou models: o conteúdo é lido de `TOPICOS_AJUDA`, no próprio aplicativo, e o manual em PDF é obtido pelo navegador diretamente do GitHub. As notas descrevem o processamento feito dentro de uma função.

| Participante | Origem |
|---|---|
| `TelaAjuda` | `src/app/ajuda.tsx` |
| `ajuda` | `src/constants/ajuda.ts` (`TOPICOS_AJUDA`, `SELO_PERFIL`, `URL_MANUAL_PDF`) |
| `buscaAjuda` | `src/utils/busca-ajuda.ts` (`filtrarTopicosAjuda`, `normalizarBusca`) |
| `CampoTexto` | `src/components/CampoTexto.tsx` |
| `Sanfona` | `src/components/Sanfona.tsx` |
| `Botao` | `src/components/Botao.tsx` |
| `Linking` | biblioteca `react-native` |
| Navegador | navegador ou leitor de PDF padrão do aparelho |
| GitHub | Pasta `4 - Incremento3/manual-de-utilizacao/manual-usuario/` do repositório `Conecta-IFSP/DOCUMENTATION`, branch `main` |

---

## SD-MAN-01 - Consultar Ajuda (RF-MAN-01)

```mermaid
sequenceDiagram
    autonumber
    actor U as Usuário
    participant T as TelaAjuda
    participant D as ajuda
    participant B as buscaAjuda
    participant C as CampoTexto
    participant S as Sanfona

    U->>T: abre a tela Ajuda, pela tela Entrar ou pelo Perfil
    T->>D: TOPICOS_AJUDA
    D-->>T: 7 tópicos com 42 perguntas
    T->>B: filtrarTopicosAjuda(TOPICOS_AJUDA, '')
    B-->>T: TOPICOS_AJUDA, sem alteração, porque a busca está vazia
    loop para cada pergunta de cada tópico visível
        T->>S: Sanfona({pergunta, resposta, selo: SELO_PERFIL[perfil], aberta, aoAlternar})
    end
    T-->>U: exibe os tópicos e as perguntas (pela tela Entrar, a primeira de Conta e acesso já vem aberta)

    opt busca por palavra
        U->>C: digita a palavra no campo Buscar
        C->>T: onChangeText(texto)
        T->>T: definirBusca(texto)
        T->>B: filtrarTopicosAjuda(TOPICOS_AJUDA, busca)
        B->>B: normalizarBusca(busca)
        Note over B: mantém as perguntas que contêm todas as palavras no título do tópico, na pergunta ou na resposta
        B-->>T: tópicos filtrados
        alt nenhuma pergunta encontrada
            T-->>U: exibe "Nenhuma pergunta encontrada." e o EstadoVazio
        else perguntas encontradas
            T-->>U: exibe "N perguntas encontradas" e apenas os tópicos com resultado
        end
    end

    U->>S: toca na pergunta
    S->>S: alternar()
    S->>T: aoAlternar(true)
    T->>T: alternar(pergunta.id, true)
    Note over T: definirAbertas acrescenta pergunta.id ao conjunto abertas
    T->>S: Sanfona({..., aberta: true})
    S-->>U: exibe a resposta em texto ou em passos numerados
```

Para fechar a resposta, o usuário toca de novo na pergunta. A sequência a partir do passo "toca na pergunta" se repete com `aoAlternar(false)` e `alternar(pergunta.id, false)`, que retira `pergunta.id` do conjunto `abertas`.

---

## SD-MAN-02 - Abrir manual em PDF (RF-MAN-02)

```mermaid
sequenceDiagram
    autonumber
    actor U as Usuário
    participant T as TelaAjuda
    participant D as ajuda
    participant Bt as Botao
    participant K as Linking
    participant N as Navegador
    participant G as GitHub

    T->>D: URL_MANUAL_PDF
    D-->>T: endereço do manual
    alt URL_MANUAL_PDF vazia
        Note over T: o Botao não é exibido e o caso de uso não pode ser iniciado
    else URL_MANUAL_PDF preenchida
        T->>Bt: Botao({titulo: 'Ver manual completo (PDF)', icone: 'document-outline', variante: 'secundario', aoTocar})
        U->>Bt: toca em "Ver manual completo (PDF)"
        Bt->>T: aoTocar()
        T->>K: Linking.openURL(URL_MANUAL_PDF)
        alt nenhum aplicativo abre o endereço
            K-->>T: Promise rejeitada
            T->>T: .catch(() => {})
            Note over T: o erro é ignorado e nenhuma mensagem é exibida
        else endereço entregue ao sistema
            K->>N: abre o endereço no aplicativo padrão do sistema
            K-->>T: Promise resolvida
            Note over T: o aplicativo continua na tela Ajuda
            N->>G: GET /Conecta-IFSP/DOCUMENTATION/blob/main/4 - Incremento3/manual-de-utilizacao/manual-usuario/Manual-do-Usuario-Conecta.pdf
            alt aparelho sem internet
                N-->>U: exibe a mensagem de falta de conexão do próprio navegador
            else arquivo ausente no caminho
                G-->>N: 404 Not Found
                N-->>U: exibe a página de erro do GitHub
            else arquivo publicado
                G-->>N: 200 OK, página do arquivo com o visualizador de PDF
                N-->>U: exibe o manual, com a opção de baixá-lo
            end
            U->>T: retorna ao aplicativo
            T-->>U: exibe a tela Ajuda no mesmo ponto
        end
    end
```

O aplicativo não verifica a conexão nem o resultado da abertura do arquivo: depois de `Linking.openURL`, o carregamento do PDF fica a cargo do navegador. No Android, o endereço é entregue ao sistema como uma intenção de visualização; no iOS, é aberto pelo sistema no Safari ou no aplicativo associado a PDFs.
