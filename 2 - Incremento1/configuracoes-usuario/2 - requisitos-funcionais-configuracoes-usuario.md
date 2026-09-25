# Requisitos Funcionais - Configurações de Usuário

| Código | Caso de Uso | Ator |
|---|---|---|
| RF-01 | Trocar senha | Usuário autenticado |
| RF-02 | Recuperar conta | Usuário não autenticado |
| RF-03 | Alterar dados pessoais | Usuário autenticado |
| RF-04 | Definir tema preferido | Usuário autenticado |

---

## RF-01 - Trocar senha

| Campo | Descrição |
|---|---|
| **Descrição** | Permitir que o usuário autenticado substitua sua senha, confirmando primeiro a senha atual. |
| **Entradas** | • Senha atual<br>• Nova senha, com no mínimo 8 caracteres<br>• Confirmação da nova senha na interface |
| **Origem** | Usuário autenticado, pela tela “Alterar senha”, acessada a partir do perfil. |
| **Saída** | • Mensagem de sucesso<br>• Erro quando a senha atual está incorreta<br>• Erro quando a nova senha repete a senha atual ou não atende à validação |
| **Destino** | O novo hash é gravado no documento do usuário na coleção `usuarios`. |
| **Ação** | A interface valida os campos e envia `PATCH /usuarios/me/senha`. O backend identifica o usuário pelo JWT, compara a senha atual com o hash armazenado usando bcrypt, impede a repetição e grava o hash bcrypt da nova senha. |
| **Pré-condição** | Usuário autenticado, com conta ativa e senha cadastrada. |
| **Pós-condição** | A senha anterior deixa de autenticar e a nova senha passa a valer nos próximos acessos. |
| **Efeitos colaterais** | Os campos da tela são limpos após o sucesso. A sessão atual permanece aberta, pois o sistema não mantém uma versão de sessão para invalidar JWTs já emitidos. |

---

## RF-02 - Recuperar conta

| Campo | Descrição |
|---|---|
| **Descrição** | Permitir que um usuário que não lembra a senha recupere o acesso por meio de um link temporário enviado ao e-mail cadastrado. |
| **Entradas** | • E-mail da conta<br>• Endereço dinâmico da tela no Expo Go, criado por `Linking.createURL()`<br>• Token recebido no link<br>• Nova senha, com no mínimo 8 caracteres<br>• Confirmação da nova senha na interface |
| **Origem** | Tela “Recuperar conta”, acessível sem autenticação, e mensagem enviada ao e-mail do usuário. |
| **Saída** | • Mensagem genérica após a solicitação, exista ou não uma conta<br>• E-mail com link válido por 30 minutos quando existe uma conta ativa<br>• Confirmação da redefinição ou aviso de link inválido/expirado |
| **Destino** | O pedido usa `POST /auth/recuperar-conta`; a página intermediária usa `GET /auth/abrir-redefinicao`; a nova senha é enviada para `POST /auth/redefinir-senha`. |
| **Ação** | O backend procura uma conta ativa pelo e-mail. Se encontrar, gera 32 bytes aleatórios, envia o token original no link e grava somente seu hash SHA-256 com a expiração. O e-mail abre uma página HTTP local, que direciona ao Expo Go. Na conclusão, o backend compara o hash do token, verifica o prazo, grava o hash bcrypt da nova senha e apaga os campos de recuperação. |
| **Pré-condição** | Para receber o e-mail, deve existir uma conta ativa com o endereço informado. O backend, o Expo e o celular devem estar acessíveis na mesma rede durante a execução local. |
| **Pós-condição** | A nova senha passa a autenticar; o token utilizado não pode ser reutilizado. |
| **Efeitos colaterais** | Uma nova solicitação substitui o token anterior. A resposta da solicitação não informa se o e-mail existe, reduzindo a descoberta de contas. Sem SMTP configurado, o backend registra o link no terminal apenas para desenvolvimento. |

---

## RF-03 - Alterar dados pessoais

| Campo | Descrição |
|---|---|
| **Descrição** | Permitir que o usuário autenticado consulte e altere seu nome e seu e-mail. |
| **Entradas** | • Nome<br>• E-mail válido |
| **Origem** | Usuário autenticado, pela tela “Perfil”. |
| **Saída** | • Perfil atualizado<br>• Erros de validação junto aos campos<br>• Erro quando o e-mail já pertence a outra conta |
| **Destino** | Documento do próprio usuário na coleção `usuarios`. |
| **Ação** | A tela carrega os dados com `GET /usuarios/me` e envia `PATCH /usuarios/me`. O backend identifica a conta pelo JWT, normaliza o e-mail para minúsculas, remove espaços das extremidades, verifica duplicidade e salva os campos informados. |
| **Pré-condição** | Usuário autenticado e com conta ativa. |
| **Pós-condição** | Nome e e-mail atualizados para consultas posteriores e novos logins. |
| **Efeitos colaterais** | A tentativa de usar o e-mail de outra conta não altera o perfil. O frontend exige nome com pelo menos 3 caracteres; o DTO do backend aceita a partir de 2 caracteres. |

---

## RF-04 - Definir tema preferido

| Campo | Descrição |
|---|---|
| **Descrição** | Permitir que o usuário escolha a aparência clara, escura ou baseada no tema do aparelho. |
| **Entradas** | • Preferência `claro`, `escuro` ou `sistema` |
| **Origem** | Usuário autenticado, pela seção “Tema preferido” da tela “Perfil”. |
| **Saída** | • Tema aplicado após o salvamento ser confirmado<br>• Mensagem de sucesso ou erro |
| **Destino** | Campo `tema` do usuário no MongoDB e armazenamento local do aplicativo. |
| **Ação** | A escolha fica como rascunho na tela. Ao tocar em “Salvar alterações”, o frontend envia `PATCH /usuarios/me`. Somente após a resposta de sucesso o `TemaContext` aplica a preferência e a grava no SecureStore; na web, usa `localStorage`. A opção `sistema` resolve o tema com `useColorScheme()`. |
| **Pré-condição** | Usuário autenticado e perfil carregado. |
| **Pós-condição** | A aparência é atualizada no aparelho e a preferência fica associada à conta para outros acessos. |
| **Efeitos colaterais** | Se o salvamento falhar, a seleção não altera o tema em uso. Ao carregar o perfil ou entrar novamente, a preferência devolvida pelo backend é reaplicada. |

