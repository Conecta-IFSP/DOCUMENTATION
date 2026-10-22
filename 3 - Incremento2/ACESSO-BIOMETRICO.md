# Acesso protegido no celular

Implementação local de 21/10/2026, sem publicação no GitHub.

## Comportamento

- Após confirmar o cadastro por e-mail, oferece salvar o acesso se o cadastro ocorreu na mesma execução do app (credenciais temporárias em memória, consumidas uma vez, válidas por 15 minutos). Se o app foi encerrado, a oferta ocorre após o primeiro login com senha.
- A oferta é opcional e aparece apenas em celulares com biometria forte disponível. Recusar mantém o login convencional.
- Cada conta fica separada; na tela de login é possível selecionar a conta, desbloqueá-la com biometria ou esquecer seu acesso neste celular.
- O app realiza um novo POST /auth/login com a credencial desbloqueada. O servidor continua verificando senha, conta ativa e demais regras existentes. Não existe um token permanente que ignore essas verificações.
- No celular, o token de sessão fica apenas em memória. Ao reiniciar o processo, é necessário entrar novamente com senha ou biometria. Tokens persistidos pela implementação anterior são removidos.
- Ao voltar do segundo plano, a sessão aberta fica atrás da tela de desbloqueio. É possível desbloquear ou sair para escolher outra conta.
- Criar conta, salvar o perfil, alterar senha e redefinir senha pedem a autenticação nativa antes do envio. Cancelar impede o envio. O sistema pode oferecer o código do aparelho como alternativa. Aparelhos sem bloqueio configurado recebem orientação para configurá-lo.
- Alterar a senha remove o acesso salvo da conta naquele celular e encerra a sessão local. Uma recuperação feita em outro aparelho torna a senha salva antiga inválida no servidor; será preciso entrar com a nova senha e salvar novamente.
- O e-mail permanece visível e não editável no perfil.

## Proteção e limites

`src/services/acesso-dispositivo.ts` usa `expo-secure-store` com `requireAuthentication: true` e `WHEN_UNLOCKED_THIS_DEVICE_ONLY`. A senha é armazenada cifrada pelo armazenamento nativo, não em localStorage, arquivos ou parâmetros de navegação. A leitura é protegida pelo sistema operacional. A lista de e-mails é armazenada separadamente no SecureStore para permitir escolher a conta antes do desbloqueio. Nenhum dado biométrico é coletado pelo aplicativo.

Qualquer pessoa com biometria cadastrada no aparelho pode desbloquear as contas salvas: a oferta informa isso. Mudanças nas biometrias podem invalidar o segredo e exigir entrada por senha. Para o login salvo, a biometria é necessária; o PIN alternativo se aplica às confirmações e ao desbloqueio de uma sessão já aberta.

A autenticação nativa das ações é uma proteção local de interface, não uma prova biométrica enviada à API. Não substitui autorização do servidor, senha atual na troca de senha ou código de confirmação por e-mail. Não foi implementado protocolo de passkeys, refresh token ou revogação de todos os JWTs em outros aparelhos. As demais regras da API permanecem vigentes.

O preenchimento automático tem marcações de usuário/senha nos formulários. A oferta do gerenciador de senhas do Android/iOS depende do sistema e das preferências do usuário; o app não consegue obrigar sua exibição. O botão de salvar acesso implementado pertence ao Conecta+ e usa o armazenamento protegido, não gerencia diretamente Google Password Manager ou Senhas da Apple.

O navegador mantém o login e as validações existentes; não simula uma autenticação biométrica bem-sucedida nem exibe contas biométricas. É preciso gerar um novo build nativo após adicionar a dependência e as permissões do app.json. Face ID não pode ser validado no Expo Go.

## Validação

Testes automatizados com módulos nativos simulados cobrem: contas separadas, exigência de autenticação ao ler, cancelamento, recusa da oferta, ausência de persistência de senha na web, consumo único das credenciais temporárias e token nativo somente em memória. Eles não validam hardware, permissões ou diálogos reais.

Checklist pendente em aparelho real (Android e iOS): ativar/recusar; cancelar a biometria; fechar e reabrir; voltar do segundo plano; alternar entre duas contas; esquecer uma; alterar senha; adicionar nova digital/Face ID; tentar com aparelho sem bloqueio; verificar preenchimento automático. Testar os quatro formulários sensíveis e confirmar que cancelar não envia a alteração.

Referências oficiais: [Expo LocalAuthentication](https://docs.expo.dev/versions/latest/sdk/local-authentication/) e [Expo SecureStore](https://docs.expo.dev/versions/latest/sdk/securestore/).
