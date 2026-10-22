# Requisitos não funcionais

IDs exclusivos deste módulo: RNF-COM-xx. Regras como exigir responsável, validar nome e aprovar ata pertencem aos RF. Os critérios abaixo são verificáveis; um requisito proposto não deve ser apresentado como teste executado.

|ID|Qualidade e critério de aceitação|Método/condição de verificação|Estado desta revisão|
|---|---|---|---|
|RNF-COM-01|Integridade concorrente: duas gravações de equipe originadas da mesma fotografia, ambas alterando o array, não podem sobrescrever silenciosamente uma à outra; uma deve falhar com conflito|Duas chamadas concorrentes a gravarEquipe, mesma comissão em MongoDB local, arrays finais diferentes; contar sucessos/erros e responsável final|Verificado pelo teste local com MongoDB|
|RNF-COM-02|Controle de acesso: nenhuma rota de comissão ou registro novo deve aceitar requisição sem JWT válido; nenhuma mutação de equipe deve aceitar integrante sem ADMIN/APROVADO|Testes de guard/serviço + roteiro HTTP com sem token, token inválido, usuário comum, admin de outra organização|Guard/serviços automatizados e amostra HTTP real verificados; matriz HTTP completa pendente|
|RNF-COM-03|Usabilidade móvel: em telas 360x800 e 390x844, escala de fonte 1 e 1,3, campos de comissão e ações de salvar devem permanecer alcançáveis por rolagem com teclado aberto; nomes de 100 caracteres não podem impedir ações|Android e iOS físicos/emuladores, selecionar organização/pessoa, buscar, editar e confirmar; registrar modelo/OS/capturas|Critério de aceite proposto, ensaio nativo pendente; build web não comprova|
|RNF-COM-04|Manutenibilidade: TypeScript sem erros e lint sem erros nos módulos alterados; rotinas de autorização e gravação compartilhadas, sem repetição da regra de responsável nas telas como única proteção|npm run build no backend; npm run typecheck e npm run lint no app; revisão de serviço|Verificado; dois avisos de lint preexistentes no app|
|RNF-COM-05|Recuperação de conflito: diante de 409, app mantém documento recebido por último, informa erro e permite atualizar antes de repetir, sem apresentar sucesso falso|Forçar versão antiga de ata e fotografia antiga de equipe; observar erro retornado e executar Atualizar|Backend verificado; interação visual nativa pendente|

Não foi especificado prazo de resposta genérico “em condições normais”. Para adotar meta de desempenho, o grupo deve primeiro fixar hardware, rede, quantidade de comissões/integrantes, concorrência e percentil (por exemplo p95), medir e acordar limiar. Paginação visual em oito itens é decisão de interface; não comprova redução de tráfego nem paginação do servidor.

A API já usa JWT individual. Não foi embutida chave secreta estática no aplicativo para simular segurança. Caso o professor exija também identificação da aplicação em gateway, o grupo precisa definir esse contrato e infraestrutura separadamente; isso permanece pendente e não é afirmado como resolvido. JWT autentica usuário; chave de aplicação é outra finalidade.
