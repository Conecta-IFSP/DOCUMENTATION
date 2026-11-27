# Requisitos não funcionais — Cadastro de usuário

> Padrão do Incremento 1 · Módulo USR

## Resumo

Os requisitos abaixo usam o mesmo formato verificável dos demais módulos.

## Requisitos

| ID         | Qualidade      | Critério e medida                                                                                 | Condição                       | Verificação/evidência                                  | Situação                             |
| ---------- | -------------- | ------------------------------------------------------------------------------------------------- | ------------------------------ | ------------------------------------------------------ | ------------------------------------ |
| RNF-USR-01 | Segurança      | Senhas são armazenadas somente como bcrypt custo 10; códigos/tokens somente como SHA-256.         | Cadastro, troca e recuperação. | Inspeção de `UsuariosService` e `AuthService`; testes. | Atendido.                            |
| RNF-USR-02 | Autenticação   | JWT expira em 1 h e conta ativa é revalidada em cada rota privada.                                | Requisição autenticada.        | Configuração JWT e `JwtAuthGuard`.                     | Atendido.                            |
| RNF-USR-03 | Privacidade    | Hashes, Google ID, tokens e códigos não aparecem nas respostas comuns.                            | Toda resposta de usuário.      | Campos `select:false` e inspeção das respostas.        | Atendido.                            |
| RNF-USR-04 | Validação      | DTOs rejeitam e-mail, senha, código e enum inválidos antes do service.                            | Entradas HTTP.                 | `class-validator` e testes 400.                        | Atendido.                            |
| RNF-USR-05 | Confiabilidade | Confirmação por e-mail expira em 30 min; falha SMTP não deixa cadastro provisório utilizável.     | Cadastro local.                | Testes de `AuthService` e inspeção do rollback.        | Atendido.                            |
| RNF-USR-06 | Usabilidade    | Formulários usam `react-hook-form`, mensagens em português e teclado não cobre campos no Android. | Cadastro e login no Expo Go.   | Roteiro manual em Android.                             | Verificação manual pendente.         |
| RNF-USR-07 | Desempenho     | Login deve responder em até 2 s em rede local normal, excluído o tempo do Google/SMTP.            | Ambiente de desenvolvimento.   | Medição via DevTools/Postman.                          | Critério proposto; medição pendente. |

## Rastreabilidade

Aplica-se a RF-USR-01 a RF-USR-07 e CT-USR-01 a CT-USR-09.
