# Informações do Sistema

Documentação conferida contra a branch `homolog` do BACKEND, commit `6f2ffc4`, em 23/10/2026.

## Escopo implementado

- `GET /sistema/status`: memória, CPU, tempo ativo e disponibilidade do banco;
- `GET /sistema/relatorios`: contagens de usuários, organizações e comissões;
- `POST /sistema/popular-teste`: cria um conjunto controlado de dados de demonstração;
- `POST /sistema/reset`: remove dados operacionais, preservando administradores do sistema.

Todas as rotas exigem `X-API-Key`, JWT válido e `tipo = ADMIN_SISTEMA`. O aplicativo também solicita confirmação antes das operações destrutivas, mas a autorização efetiva é aplicada pelo backend por `JwtAuthGuard` e `AdminSistemaGuard`.

## Limite conhecido

Não existe reinicialização independente de subsistemas: o backend NestJS é executado como um único processo. A interface não deve afirmar que reinicia componentes que não existem separadamente.

## Documentos

Os arquivos numerados de 1 a 6 registram casos de uso, requisitos, classes, dados, requisitos não funcionais e sequências. O arquivo de testes registra a evidência executada e o roteiro manual.
