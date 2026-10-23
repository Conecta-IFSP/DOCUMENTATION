# Tratamento das observações da avaliação

Este documento separa correções implementadas, documentação atualizada e pendências. Não transforma recomendação em funcionalidade concluída.

| Observação | Evidência atual na `homolog` | Situação documental |
|---|---|---|
| Ícone transborda no Android | novos `adaptive-icon.png` e `adaptive-icon-monocromatico.png` em `app.json` | tecnologia/configuração registrada; teste visual ainda necessário |
| API sem chave | `ApiKeyGuard` global e cabeçalho `X-API-Key` em `api.ts` | registrado com limite: chave pública no bundle não substitui JWT |
| Ações destrutivas sem confirmação | componente `Confirmacao` em organização, comissão, sistema e reunião | casos de uso/testes devem incluir confirmação/cancelamento |
| Teclado bloqueia campos no Android | `KeyboardProvider` e `react-native-keyboard-controller` 1.21.9 | biblioteca e critério manual documentados |
| Biblioteca a ser descontinuada | dependência removida não pode ser identificada somente pela imagem; lista atual vem do lockfile | versões atuais inventariadas; decisão histórica precisa da identificação da biblioteca |
| Usar biblioteca de formulário | `react-hook-form` 7.90.0 nas telas de formulário | registrado na arquitetura e módulos |
| Precisa trocar de tela para atualizar | `useFocusEffect` nas telas dinâmicas e atualizações locais após operações | documentado; testes manuais ainda necessários |
| Pesquisa de mercado ausente | existe documento inicial, mas fontes/metodologia precisam ser revisadas separadamente | fora do escopo do código; pendência explícita |
| Organização: buscar em vez de listar todas | `GET /organizacoes?busca=` com regex segura e limite de 100 caracteres | catálogo atualizado |
| Organização: convite | `POST /:id/convites` e resposta a convite | catálogo/requisitos atuais |
| Organização: troca de administrador | promoção/rebaixamento via `/papel`; impede zero administradores | catálogo atualizado |
| Organização: revogação irreversível | `PATCH` de status aceita reativação para `APROVADA` | comportamento atual documentado |
| Comissão sem responsável / vários responsáveis | schema e service exigem exatamente um; rota atômica substitui responsável | documentação antiga foi corrigida para refletir a regra implementada |
| Confirmação de e-mail | código de 7 caracteres e validade de 30 min | documentado |
| OAuth | login Google web/nativo implementado | arquitetura e catálogo atualizados |
| Recuperação por código, sem redirecionamento | fluxo de código + token, sem link `localhost`/`exp://` | módulo de configurações refeito |
| Endereço da API exposto | mensagem de erro não exibe endereço; URL pública continua necessária ao cliente | limite arquitetural documentado |
| Confirmação de senha por e-mail assíncrono | troca de e-mail/cadastro usam e-mail; troca de senha autenticada não envia confirmação posterior | pendência de produto, não documentada como pronta |
| Biometria após primeiro acesso | `expo-local-authentication` protege acesso às ações e recuperação quando disponível | comportamento e fallback documentados |
| Tema pré-visualizado | aplicação otimista imediata; erro restaura tema anterior | documentado conforme código atual |
| Troca de senha desloga | não implementado | pendência explícita |
| Mensagem em inglês para senha curta | DTOs atuais possuem mensagens em português | documentado e coberto por inspeção/testes |
| Perfil exige autenticação adicional | autenticação local antes das telas sensíveis; alteração de nome/tema usa somente JWT | limite descrito sem exagero |
| Reset mais seguro | modal exige texto de confirmação na interface; backend exige admin/JWT/API key | documentação precisa distinguir confirmação de UI de autorização do servidor |
| Casos de uso sem padrão | padrão define ator, pré/pós, fluxos e exceções | módulos revisados devem seguir `UC-<MÓDULO>-NN` |
| Dicionários e diagramas sem padrão | padrão separa classe e modelo lógico MongoDB | módulos revisados devem explicar atributos/restrições |
| RNFs vagos ou funcionais | padrão exige medida, condição, verificação e evidência | metas não medidas são marcadas como tal |
| RFs simples demais | catálogo registra rota, DTO, service, coleção, campos e retorno | referência transversal criada |
| Sequências com classes/métodos inválidos | padrão exige participantes e métodos reais | módulo de configurações já refeito |
| Testes não documentados | backend: 182 aprovados; frontend: 9 aprovados; typecheck pendente | resultado real registrado, sem afirmar o que não foi executado |
