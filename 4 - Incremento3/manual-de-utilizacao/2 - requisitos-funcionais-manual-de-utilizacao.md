# Especificação de Requisitos Funcionais - Manual de Utilização (R4)

---

## RF-MAN-01 - Consultar Ajuda

| Campo | Descrição |
|---|---|
| **Nome do Caso de Uso** | Consultar Ajuda |
| **ID do Requisito (RF)** | RF-MAN-01 |
| **Ator Principal** | Usuário (com ou sem login) |
| **Objetivo** | Permitir que o usuário tire dúvidas sobre o uso do aplicativo lendo perguntas frequentes organizadas por assunto, com a opção de buscar por palavra. |
| **Prioridade** | Alta |
| **Pré-Condições** | O aplicativo deve estar instalado e aberto no aparelho.<br>Não é preciso estar autenticado: a tela Ajuda fica fora das rotas protegidas (`src/app/_layout.tsx`).<br>Não é preciso ter conexão com a internet: as perguntas e respostas fazem parte do próprio aplicativo (`src/constants/ajuda.ts`), e a tela não faz nenhuma requisição à API. |
| **Pós-Condições** | As perguntas frequentes são apresentadas ao usuário.<br>Nenhum dado é registrado ou alterado no aparelho ou no servidor.<br>Ao sair da tela, as perguntas abertas e o texto da busca são descartados. |
| **Fluxo Principal (Caminho de Sucesso)** | 1. O usuário abre a tela Ajuda (ver FA-01 e FA-02).<br>2. O sistema apresenta o título "Ajuda", um texto de orientação, o campo Buscar e os tópicos na ordem de `TOPICOS_AJUDA`: Conta e acesso, Organizações e comissões, Reuniões, Presença, Ata, Votação e Relatório individual. Cada tópico mostra seu ícone e suas perguntas fechadas.<br>3. O usuário toca em uma pergunta.<br>4. O sistema abre a resposta com animação. A resposta aparece como texto corrido ou como lista numerada de passos. Perguntas com perfil `responsavel` exibem o selo "Responsável", que apenas informa e não esconde a pergunta de nenhum usuário.<br>5. O usuário toca de novo na pergunta, e o sistema fecha a resposta. Várias perguntas podem ficar abertas ao mesmo tempo.<br>6. O usuário toca em voltar. O sistema retorna à tela anterior; se não houver tela anterior, abre o Início (com login) ou a tela Entrar (sem login). |
| **Fluxos Alternativos** | **FA-01: Acesso sem login pela tela Entrar**<br>1. Na tela Entrar, o usuário toca no link "Precisa de ajuda?".<br>2. O aplicativo abre a tela Ajuda com o parâmetro `topico=conta`.<br>3. O sistema abre a primeira pergunta do tópico Conta e acesso ("Esqueci minha senha. O que faço?") e rola a tela até esse tópico.<br>4. O fluxo continua no passo 3. No passo 6, o usuário volta à tela Entrar.<br><br>**FA-02: Acesso pelo Perfil**<br>1. Com login feito, em qualquer aba (Início, Agenda, Organizações ou Relatório), o usuário toca no avatar, no canto superior direito.<br>2. O sistema apresenta a tela Perfil.<br>3. Na seção Suporte, o usuário toca em "Ajuda" ("Perguntas frequentes").<br>4. O sistema apresenta a tela Ajuda com todas as perguntas fechadas, e o fluxo continua no passo 2. No passo 6, o usuário volta à tela Perfil.<br><br>**FA-03: Buscar uma pergunta**<br>1. No passo 2, o usuário digita uma ou mais palavras no campo Buscar.<br>2. O sistema ignora acentos e diferenças entre maiúsculas e minúsculas e mantém apenas as perguntas cujo título de tópico, pergunta ou resposta contém todas as palavras digitadas. Tópicos sem nenhuma pergunta encontrada deixam de ser exibidos.<br>3. O sistema informa a quantidade, por exemplo "3 perguntas encontradas".<br>4. O fluxo continua no passo 3.<br>5. Para desfazer a busca, o usuário toca em "Limpar busca", o ícone no fim do campo, e o sistema volta a apresentar todos os tópicos.<br><br>**FA-04: Abrir o manual completo**<br>1. Em qualquer momento após o passo 2, o usuário toca em "Ver manual completo (PDF)", no fim da tela.<br>2. Executa-se o RF-MAN-02 - Abrir manual em PDF. |
| **Fluxos de Exceção** | **FE-01: Busca sem resultado**<br>1. Nenhuma pergunta contém todas as palavras digitadas.<br>2. O sistema informa "Nenhuma pergunta encontrada." e "Tente outra palavra, como o nome da tela ou do botão. Se não encontrar, consulte o manual completo.".<br>3. O usuário altera ou limpa a busca, e o fluxo retorna ao FA-03, passo 2. |

---

## RF-MAN-02 - Abrir manual em PDF

| Campo | Descrição |
|---|---|
| **Nome do Caso de Uso** | Abrir manual em PDF |
| **ID do Requisito (RF)** | RF-MAN-02 |
| **Ator Principal** | Usuário (com ou sem login) |
| **Objetivo** | Permitir que o usuário consulte o manual completo do aplicativo, em PDF, quando as perguntas frequentes não forem suficientes. |
| **Prioridade** | Média |
| **Pré-Condições** | O usuário deve estar na tela Ajuda (RF-MAN-01).<br>A constante `URL_MANUAL_PDF` (`src/constants/ajuda.ts`) deve estar preenchida; caso contrário, o botão "Ver manual completo (PDF)" não é exibido. |
| **Pós-Condições** | O manual é apresentado pelo navegador ou pelo leitor de PDF do aparelho, fora do aplicativo.<br>O aplicativo permanece na tela Ajuda, e nenhum dado é registrado ou alterado. |
| **Fluxo Principal (Caminho de Sucesso)** | 1. Na tela Ajuda, o usuário toca no botão "Ver manual completo (PDF)", no fim da lista de tópicos.<br>2. O aplicativo entrega o endereço `https://github.com/Conecta-IFSP/DOCUMENTATION/blob/main/4%20-%20Incremento3/manual-de-utilizacao/manual-usuario/Manual-do-Usuario-Conecta.pdf` ao sistema operacional por meio de `Linking.openURL`.<br>3. O sistema operacional abre o endereço no navegador ou no aplicativo padrão para PDF.<br>4. O GitHub devolve a página do arquivo, e o manual é exibido no visualizador de PDF do site, com a opção de baixá-lo.<br>5. O usuário retorna ao aplicativo, que continua na tela Ajuda no mesmo ponto. |
| **Fluxos Alternativos** | Não há. |
| **Fluxos de Exceção** | **FE-01: Sem internet ao abrir o PDF**<br>1. No passo 3, o aparelho está sem conexão com a internet.<br>2. O aplicativo não verifica a conexão antes de entregar o endereço: o navegador ou o leitor de PDF é aberto e exibe a sua própria mensagem de falta de conexão.<br>3. O aplicativo não exibe mensagem e continua na tela Ajuda; o usuário pode retornar e tocar no botão de novo quando estiver conectado.<br><br>**FE-02: Nenhum aplicativo abre o endereço**<br>1. No passo 2, o sistema operacional não encontra navegador ou leitor de PDF para abrir o endereço.<br>2. A falha de `Linking.openURL` é ignorada pelo aplicativo, que não exibe mensagem e permanece na tela Ajuda.<br><br>**FE-03: Manual não publicado**<br>1. No passo 4, o arquivo não está publicado no caminho esperado do repositório DOCUMENTATION (foi removido, renomeado ou movido).<br>2. O GitHub responde "404 - Not Found", exibido no navegador.<br>3. O aplicativo permanece na tela Ajuda. |
