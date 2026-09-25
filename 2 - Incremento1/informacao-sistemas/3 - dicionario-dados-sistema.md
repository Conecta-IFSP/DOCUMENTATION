# Dicionário de Dados

O módulo não cria coleções novas. Só lê/escreve em `usuarios`, `organizacoes` e `comissoes` (já documentadas no incremento de cadastro). Abaixo, o formato de resposta de cada endpoint.

## GET /sistema/status

| Campo | Tipo | Descrição |
|-------|------|-----------|
| `servidor.memoria.uso_percentual` | Number | % de memória em uso |
| `servidor.cpu.carga_media_1min` | Number | Carga média de CPU |
| `servidor.tempo_ativo_segundos` | Number | Uptime do servidor |
| `banco_de_dados.conectado` | Boolean | Se o banco respondeu |

## GET /sistema/relatorios

| Campo | Tipo | Descrição |
|-------|------|-----------|
| `usuarios.total` / `usuarios.ativos` | Number | Contagem de usuários |
| `organizacoes.total` / `.pendentes` / `.aprovadas` | Number | Contagem de organizações |
| `comissoes.total` / `.ativas` | Number | Contagem de comissões |

## POST /sistema/popular-teste

| Campo | Tipo | Descrição |
|-------|------|-----------|
| `senha_padrao` | String | Senha dos usuários criados (`Teste@123`) |
| `usuarios[]` | Array | Nome, email e cenário de cada usuário criado |

## POST /sistema/reset

| Campo | Tipo | Descrição |
|-------|------|-----------|
| `status` | String | `"sistema_resetado"` |
| `resetado_em` | Date | Data/hora da execução |
