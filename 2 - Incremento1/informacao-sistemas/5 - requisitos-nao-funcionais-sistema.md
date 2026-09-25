# Requisitos Não Funcionais

| Código | Descrição |
|--------|-----------|
| RNF-01 | Status e relatórios devem responder em menos de 1s |
| RNF-02 | Todas as rotas exigem `ADMIN_SISTEMA` (JwtAuthGuard + AdminSistemaGuard) |
| RNF-03 | Reset e popular-teste exigem confirmação antes de executar |
| RNF-04 | Reset nunca apaga o administrador do sistema |
| RNF-05 | Status do servidor usa só o módulo nativo `os` do Node, sem dependência externa |

**Fora do escopo:** monitoramento de hardware do celular do usuário (inviável em Expo sem módulo nativo) e reinicialização real de subsistemas (arquitetura monolítica não permite).
