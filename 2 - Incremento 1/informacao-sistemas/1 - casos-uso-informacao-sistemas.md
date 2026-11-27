# Casos de uso — Informações do sistema

> Padrão da disciplina · Módulo SIS · Revisado em 27/11/2026

## Resumo

Este documento mostra quem usa as informações do sistema e o que essa pessoa pode fazer. Cada caso de uso é descrito em detalhe, como requisito funcional no formulário padrão, em [Requisitos funcionais](2%20-%20requisitos-funcionais-informacao-sistemas.md).

## Diagrama de casos de uso

![Diagrama de casos de uso das informações do sistema](diagrama-casos-uso-informacao-sistemas.svg)

Figura 1 — Diagrama de casos de uso das informações do sistema. Fonte: equipe Conecta+.

Cada linha entre o ator e um caso de uso leva o código do requisito funcional que o descreve.

## Atores

**Administrador do sistema.** É a conta ativa cujo campo `tipo` vale `ADMIN_SISTEMA`. A primeira conta desse tipo é criada pela própria API ao iniciar, a partir da configuração do servidor, quando ainda não existe nenhum administrador do sistema nem outra conta com o e-mail configurado; outras contas podem receber esse tipo pela edição de usuários, descrita em [Cadastro de usuário](../cadastro-usuario/README.md). Só ele vê o item "Sistema" na tela inicial e abre a tela "Sistema". Pode consultar as informações sobre o sistema, o status do servidor e os relatórios, criar os dados de teste e resetar o sistema; para resetar, precisa digitar "RESET" e a própria senha, que a API confere. Não pode escolher o que o reset apaga nem desfazer o reset. A conta de administrador do sistema que não tem senha cadastrada não consegue resetar. O administrador do sistema também não cria nem participa de organizações, comissões e reuniões: essas rotas o recusam.

O usuário comum não é ator deste módulo: o aplicativo não mostra a tela para ele, e a API recusa as rotas com a mensagem "Apenas administradores do sistema podem realizar esta operação".

## Relacionamentos entre casos de uso

Não há `«include»`, `«extend»` nem generalização neste módulo. Os cinco casos de uso são independentes e têm um único ator. A tela pede ao mesmo tempo as informações sobre o sistema, o status e os relatórios, mas cada pedido é atendido sozinho. Depois de popular os dados de teste ou de resetar o sistema, a tela consulta tudo de novo para mostrar os números atualizados; isso é uma atualização da tela, não um caso de uso incluído.

## Requisitos funcionais de usuário

| Caso de uso | Requisito | Nome                                  | Ator principal           |
| ----------- | --------- | ------------------------------------- | ------------------------ |
| UC-SIS-01   | RF-SIS-01 | Consultar status do servidor          | Administrador do sistema |
| UC-SIS-02   | RF-SIS-02 | Consultar relatórios                  | Administrador do sistema |
| UC-SIS-03   | RF-SIS-03 | Popular dados de teste                | Administrador do sistema |
| UC-SIS-04   | RF-SIS-04 | Resetar sistema                       | Administrador do sistema |
| UC-SIS-05   | RF-SIS-05 | Consultar informações sobre o sistema | Administrador do sistema |

**Requisito RF-SIS-01 — Consultar status do servidor.** O sistema deve permitir ao administrador do sistema ver quanto da CPU e da memória do computador onde a API roda está em uso, há quanto tempo o computador e a API estão ligados, e se o banco de dados respondeu, em quanto tempo e quanto espaço ocupa.

**Requisito RF-SIS-02 — Consultar relatórios.** O sistema deve permitir ao administrador do sistema ver quantos usuários existem e quantos estão ativos, quantas organizações existem e quantas estão aprovadas ou pendentes, quantas comissões existem e quantas estão ativas, quantas reuniões existem e quantas estão agendadas ou canceladas, e quantas atas existem e quantas estão aprovadas ou em aprovação.

**Requisito RF-SIS-03 — Popular dados de teste.** O sistema deve permitir ao administrador do sistema, depois de confirmar, criar três contas de teste, uma organização aprovada e uma comissão, para demonstrar o aplicativo.

**Requisito RF-SIS-04 — Resetar sistema.** O sistema deve permitir ao administrador do sistema, depois de digitar a palavra "RESET" e a própria senha, apagar os usuários comuns, as organizações, as comissões, as reuniões com seus anexos, a numeração das atas e as notificações, mantendo as contas de administrador do sistema.

**Requisito RF-SIS-05 — Consultar informações sobre o sistema.** O sistema deve permitir ao administrador do sistema ver o nome do sistema, as versões do aplicativo e da API, o ambiente em que a API roda, quando ela foi iniciada, o fuso horário, as tecnologias usadas no servidor e no aplicativo, com as versões, e o sistema operacional e o processador do computador onde a API roda.
