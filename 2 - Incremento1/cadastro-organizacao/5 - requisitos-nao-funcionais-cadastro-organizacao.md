# Requisitos Não Funcionais - Cadastro de Organização

| Código | Descrição |
|---|---|
| RNF-01 | As rotas de organização exigem token válido, e o papel e a situação da conta são conferidos a cada requisição: conta desativada perde o acesso na requisição seguinte |
| RNF-02 | Autorizar, revogar, editar e excluir exigem `ADMIN_SISTEMA`, verificado no servidor e não apenas escondendo opções na tela (`AdminSistemaGuard`) |
| RNF-03 | O cadastro de organização é recusado para `ADMIN_SISTEMA`, que administra a plataforma e não participa de organizações (`UsuarioComumGuard`) |
| RNF-04 | Alterações simultâneas na mesma organização não corrompem os dados: em conflito, o sistema recusa a operação e pede para atualizar a tela |
| RNF-05 | Revogação e exclusão, por serem irreversíveis, exigem confirmação explícita antes de executar |
| RNF-06 | A listagem de organizações devolve apenas os campos usados na tela, mantendo a resposta rápida na tela de gestão |

**Válidos para toda a plataforma**, e por isso não repetidos aqui: mensagens em português junto do campo ou da ação, tema claro e escuro com alvos de toque de no mínimo 44 pontos, mesmo código-fonte em Android e iOS, campos em snake_case no banco e na API, e regras cobertas por testes automatizados.

**Fora do escopo:** privacidade e gestão dos dados de membros, tratadas no requisito de cadastro de usuário, e reinicialização de subsistemas, tratada no requisito de informações do sistema.
