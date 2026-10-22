# Testes e rastreabilidade — execução iniciada em 19/10/2026, revisão em 20/10/2026

## Ambiente e evidências

Windows, Node.js 24.14.0; dependências instaladas por `npm ci` a partir dos lockfiles de homolog. Houve aviso de engine de ferramenta Angular/Nest esperando versão mais recente de Node; os comandos abaixo concluíram. Nenhuma dependência de produção foi adicionada. Banco de integração: MongoDB em 127.0.0.1:27017, banco com prefixo `conecta_teste_comissoes_` e UUID exclusivo por execução; removido pelo próprio teste ao final. Não usou dados de produção.

|Verificação|Comando (na pasta correspondente)|Resultado|
|---|---|---|
|Backend, suíte completa|npm test|157 testes aprovados, 13 arquivos|
|Backend, recorte comissões/reuniões|npm test -- src/comissoes src/reunioes|65 testes aprovados, 5 arquivos (subconjunto dos 157, não somar)|
|Backend, compilação|npm run build|Aprovado|
|Backend, análise estática|npm run lint|Aprovado|
|MongoDB real, serviço + persistência|node test/comissoes-mongo.mjs, após build|Aprovado: fluxo de equipe, disputa de escrita, ata e lista; amostra HTTP com guards e DTO reais|
|Frontend, tipos|npm run typecheck|Aprovado|
|Frontend, lint|npm run lint|Sem erros; dois avisos preexistentes: nome MinhasComissoes redeclarado e import CampoTexto não utilizado em verificar-cadastro|
|Frontend, sessão HTTP simulada|node --test tests/*.test.cjs|5 testes aprovados; são testes preexistentes do helper, não testes visuais dos novos formulários|
|Frontend, empacotamento web|npx expo export --platform web|Aprovado; rota /reunioes/registros incluída|

Vitest usa models simulados na suíte unitária. O script `test/comissoes-mongo.mjs` usa schemas e serviços compilados com MongoDB real e também inicia aplicação Nest de teste com ComissoesController/ReunioesController, guards reais e ValidationPipe. A amostra HTTP verifica 401 sem token/token inválido, 400 para DTO inválido, 403 para não administrador e ADMIN_SISTEMA, transferência autenticada e leitura da lista. Não cobre todas as combinações HTTP das treze operações. O teste real de concorrência dispara duas escritas diferentes com a mesma fotografia e exige exatamente um sucesso e um erro.

## Casos cobertos e correspondência

|Grupo|Cenário/resultado esperado|Evidência|
|---|---|---|
|COM-01/02|Leitura autorizada; bloquear usuário alheio, vínculo não aprovado e organização não aprovada|comissoes.regras.spec.ts|
|COM-03|Responsável obrigatório/elegível; exatamente um vínculo inicial; limites de nome e ID|comissoes.regras.spec.ts + MongoDB|
|COM-04/05|Atualização permitida somente ao admin; exclusão lógica preserva comissão|comissoes.regras.spec.ts|
|COM-06/08|Legado com zero/dois responsáveis não pode ser reativado sem regularizar; transferência conserva situação inativa|comissoes.regras.spec.ts|
|COM-07|Inclusão única de MEMBRO, recusa papel RESPONSAVEL e duplicação|comissoes.regras.spec.ts + MongoDB|
|COM-08/09|Substituição deixa um responsável; antigo vira membro; remoção/rebaixamento do responsável bloqueados|comissoes.regras.spec.ts + MongoDB|
|RNF-COM-01|Snapshot antigo não sobrescreve equipe concorrente|comissoes.regras.spec.ts + test/comissoes-mongo.mjs|
|ATA-01|Bloquear antes do fim/cancelada/não responsável/versão antiga; salvar incremento de versão|reunioes/atas-presencas.spec.ts + MongoDB|
|ATA-02|Exigir presença confirmada e congelar aprovadores; impedir edição após envio|atas-presencas.spec.ts + MongoDB|
|ATA-03|Unanimidade, voto único/idempotente, negar fora do conjunto, conflito 409|atas-presencas.spec.ts + MongoDB|
|PRE-01|Nomes e estados corretos; minutos só com saída; não inventar presença ausente|atas-presencas.spec.ts + MongoDB|

## Roteiro manual ainda a executar com o grupo

Registrar aparelho, versão Android/iOS, dimensões, escala de fonte, data, executor, resultado e captura. Não preencher “aprovado” sem executar.

1. Em Android e iOS, cadastrar comissão escolhendo responsável; tentar enviar sem pessoa. Esperado: erro de campo/servidor, sem comissão vazia.
2. Cadastrar 17 candidatos fictícios; buscar pelo nome/e-mail; navegar páginas; abrir teclado; confirmar que botão de seleção/salvar permanece alcançável.
3. Alterar responsável; cancelar confirmação e conferir ausência de mudança; confirmar e verificar que antigo permanece MEMBRO.
4. Tentar excluir responsável, inclusive via cliente HTTP independente. Esperado: 400 e manutenção da equipe.
5. Abrir mesma comissão em duas sessões; realizar alterações concorrentes. Esperado: conflito, atualização da tela e repetição consciente.
6. Com reunião encerrada, confirmar presenças, salvar ata, enviar, votar com todas as pessoas. Conferir contador, bloqueio de edição e estado APROVADA.
7. Repetir voto, tentar votar sem permissão e usar versão antiga. Esperado: sem duplicidade / 403 / 409.
8. Gerar lista com convidado sem presença, entrada sem saída, presença recusada e confirmada. Conferir SEM_REGISTRO, null e duração. Abrir compartilhamento e cancelar.
9. Simular rede indisponível: formulário deve informar falha, conservar texto e permitir tentativa; não exibir confirmação de sucesso falsa.

## Limites e pendências da entrega

Não foram feitos teste de carga, auditoria de segurança, homologação comunitária, teste de loja/build nativo ou ensaio com teclado físico Android/iOS. Lista de reuniões e organizações ainda segue a interface herdada. A paginação de comissões é visual, não no banco. Dados antigos não são alterados automaticamente: o admin deve regularizar comissão por comissão. A aprovação da ata é unanimidade do conjunto congelado; desativação de conta pode impedir completar essa aprovação e requer decisão de produto sobre reabertura/substituição.

O feedback global sobre chave de API depende de definição de infraestrutura do grupo; os problemas de cadastro de usuário, sistema e organização não foram reescritos nesta parte. O texto deste documento não certifica atendimento integral do incremento 2 nem substitui o aceite do professor.
