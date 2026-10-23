# Requisitos não funcionais — Configurações de usuário

> Padrão do Incremento 1 · Módulo CFG

## Resumo

Os requisitos abaixo usam o mesmo formato verificável dos demais módulos.

## Requisitos

| ID         | Qualidade          | Critério e medida                                                                                | Condição                     | Verificação/evidência                       | Situação                                  |
| ---------- | ------------------ | ------------------------------------------------------------------------------------------------ | ---------------------------- | ------------------------------------------- | ----------------------------------------- |
| RNF-CFG-01 | Segurança          | Rotas privadas exigem JWT; códigos/tokens ficam em hash e expiram em 30 min.                     | Perfil, e-mail e senha.      | Guards, schema e testes de auth.            | Atendido.                                 |
| RNF-CFG-02 | Compatibilidade    | Recuperação funciona no Expo Go sem URL, `localhost` ou IP do computador.                        | Android/iOS/web.             | Fluxo de código e token; CT-CFG-14.         | Implementado; teste em aparelho pendente. |
| RNF-CFG-03 | Consistência       | Tema só permanece após sucesso; falha da API restaura valor anterior.                            | Alteração de tema.           | Inspeção de `TemaContext` e roteiro manual. | Atendido por inspeção.                    |
| RNF-CFG-04 | Usabilidade        | Nome atualizado aparece sem sair da tela; erros são apresentados em português.                   | Edição do perfil.            | `sessao.atualizar()` e CT-CFG.              | Atendido por inspeção.                    |
| RNF-CFG-05 | Autenticação local | Biometria/PIN é solicitada apenas quando disponível e cadastrada; ausência não bloqueia o fluxo. | Telas sensíveis no aparelho. | Expo Local Authentication e teste manual.   | Verificação manual pendente.              |
| RNF-CFG-06 | Desempenho         | Atualização de perfil/tema deve responder em até 2 s em rede local normal.                       | Ambiente de desenvolvimento. | Medição via DevTools/Postman.               | Critério proposto; medição pendente.      |

## Rastreabilidade

Aplica-se a RF-CFG-01 a RF-CFG-06 e CT-CFG-01 a CT-CFG-14.
