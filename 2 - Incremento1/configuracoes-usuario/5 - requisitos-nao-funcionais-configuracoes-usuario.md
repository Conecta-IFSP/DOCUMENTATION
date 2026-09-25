# Requisitos Não Funcionais - Configurações de Usuário

| Código | Categoria | Descrição |
|---|---|---|
| RNF-01 | Segurança | Perfil e troca de senha exigem JWT válido; o backend identifica a conta pelo `sub` do token, sem aceitar um ID de usuário enviado pela interface. |
| RNF-02 | Segurança | Senhas são armazenadas com bcrypt e fator de custo 10; a senha em texto é usada somente durante a comparação ou geração do hash. |
| RNF-03 | Segurança | O token de recuperação possui 32 bytes aleatórios, validade de 30 minutos, uso único e armazenamento somente como hash SHA-256. |
| RNF-04 | Privacidade | A solicitação de recuperação sempre devolve a mesma mensagem, exista ou não uma conta ativa, dificultando a enumeração de e-mails cadastrados. |
| RNF-05 | Segurança | A URL recebida do Expo Go só é aceita com protocolo `exp://` ou `exps://`, host definido, sem parâmetros prévios e com a rota exata de redefinição. |
| RNF-06 | Integridade | E-mails são normalizados e únicos; a nova senha autenticada não pode repetir a senha atual; campos não previstos nos DTOs são removidos pelo `ValidationPipe`. |
| RNF-07 | Usabilidade | A interface valida nome, e-mail, senha e confirmação antes da chamada e apresenta mensagens junto ao campo ou à ação. Botões ficam indisponíveis durante o envio. |
| RNF-08 | Consistência | O tema selecionado só é aplicado depois que o backend confirma o salvamento; falhas mantêm a aparência anterior. |
| RNF-09 | Portabilidade | O mesmo código React Native atende Android e iOS pelo Expo. No Expo Go, `Linking.createURL()` produz o endereço adequado à sessão atual. |
| RNF-10 | Persistência | A preferência de tema é gravada no MongoDB para acompanhar a conta e no SecureStore para estar disponível desde a abertura do aplicativo; na web, usa `localStorage`. |
| RNF-11 | Ambiente local | Na execução com Expo Go, celular e computador devem estar na mesma rede, e os processos do frontend e do backend devem permanecer ativos durante a recuperação. |
| RNF-12 | Manutenibilidade | Controllers apenas recebem as requisições; regras de negócio ficam em `UsuariosService` e `AuthService`, e validações de entrada ficam nos DTOs e utilitários do frontend. |

## Limites atuais conhecidos

- A troca ou redefinição de senha não invalida JWTs emitidos anteriormente.
- Não há limitação de frequência específica para pedidos de recuperação; uma evolução recomendada é adicionar rate limiting.
- A página intermediária da recuperação usa HTTP na rede local porque o projeto permanece no Expo Go; uma versão publicada deveria usar HTTPS e links universais.
- A interface exige nome com no mínimo 3 caracteres, enquanto o backend aceita 2; a regra deve ser padronizada em uma revisão futura.

