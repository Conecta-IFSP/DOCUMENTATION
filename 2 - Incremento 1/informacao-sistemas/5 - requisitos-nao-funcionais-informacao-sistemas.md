# Requisitos não funcionais — Informações do sistema

> Padrão do Incremento 1 · Módulo SIS

## Resumo

Os requisitos abaixo usam o mesmo formato verificável dos demais módulos.

## Requisitos

| ID         | Qualidade             | Critério e medida                                                                     | Condição                        | Verificação/evidência                      | Situação                             |
| ---------- | --------------------- | ------------------------------------------------------------------------------------- | ------------------------------- | ------------------------------------------ | ------------------------------------ |
| RNF-SIS-01 | Segurança             | Toda rota exige chave, JWT, conta ativa e `ADMIN_SISTEMA`.                            | Qualquer endpoint `/sistema`.   | Três guards e testes 401/403.              | Atendido.                            |
| RNF-SIS-02 | Segurança operacional | Popular e resetar exigem confirmação explícita na interface.                          | Ações mutáveis/destrutivas.     | Roteiro CT-SIS-04/05/06.                   | Verificação manual pendente.         |
| RNF-SIS-03 | Integridade           | Reset nunca remove contas `ADMIN_SISTEMA`.                                            | `POST /sistema/reset`.          | Filtro de exclusão e teste de permanência. | Atendido por inspeção.               |
| RNF-SIS-04 | Transparência         | Métricas são identificadas como dados do backend, não do celular.                     | Tela de status.                 | Texto da interface e documentação.         | Atendido.                            |
| RNF-SIS-05 | Desempenho            | Status e relatórios devem responder em até 1 s em condição normal de desenvolvimento. | Banco disponível e carga comum. | Medição via Postman.                       | Critério proposto; medição pendente. |
| RNF-SIS-06 | Manutenibilidade      | Métricas usam módulo nativo `os`; o módulo não adiciona coleção própria.              | Build e evolução.               | Inspeção de imports/schema.                | Atendido.                            |

## Rastreabilidade

Aplica-se a RF-SIS-01 a RF-SIS-04 e CT-SIS-01 a CT-SIS-06.
