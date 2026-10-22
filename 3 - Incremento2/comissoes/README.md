# Comissões — incremento 2

Revisão técnica: 20/10/2026. Responsável pela apresentação: Rafaela. Escopo: comissões e complementos de atas/lista de presença vinculados às reuniões. Os cadastros de usuários e organizações são dependências, não entregas desta parte.

## Documentos

1. [Casos de uso e atores](01-casos-de-uso.md)
2. [Requisitos funcionais no formulário adotado pelo grupo](02-requisitos-funcionais.md)
3. [Classes e dicionário](03-classes.md)
4. [Modelo relacional e dicionário, com correspondência MongoDB](04-dados.md)
5. [Requisitos não funcionais verificáveis](05-nao-funcionais.md)
6. [Sequências de todos os casos de uso deste escopo](06-sequencias.md)
7. [Pesquisa de mercado e decisões](07-pesquisa-e-decisoes.md)
8. [Testes, rastreabilidade e pendências](08-testes.md)
9. [Execução e roteiro de apresentação](09-execucao.md)
10. [Galeria das 21 imagens](10-imagens.md)

## Base e autoria das alterações

Código consultado na organização Conecta-IFSP: BACKEND/homolog `6614dab`, FRONTEND/homolog `18bde43`; DOCUMENTATION/main `549f6cf85f69d26a0b094478ba087e84ce223fe8`. Usou-se homolog porque contém as alterações recentes de reuniões e interface que não estavam em main. Alterações preparadas na branch local `codex/comissoes-incremento-2`, sem publicação.

Já existiam: agendamento/edição/cancelamento de reuniões, convite dentro do app, aceite/recusa, entrada/saída, validação de presença, calendário, react-hook-form, controle de teclado e substituição da biblioteca de ícones. Foram preservados. Acrescentados nesta revisão: responsável obrigatório e único, transferência atômica, regularização de dados antigos, listas curtas com busca, ata versionada com aprovação e relatório textual de presença.

Não houve entrevista ou homologação com a comunidade nesta revisão. As regras de aprovação de ata e os limites de campos são decisões implementadas que precisam dessa validação. Não são apresentados como exigências confirmadas pelo professor. O arquivo de slides fornecido exige continuidade coerente entre código/documentação e validação dos campos com a comunidade.

Os diagramas de classes, relacionamento e sequência usam Mermaid, visualizável no GitHub. O diagrama de casos de uso é uma imagem SVG com atores, limite do sistema e elipses UML. Não é um projeto exportado pelo Astah.

11. [Complemento: horários, convidados e atualização por gesto](11-ajustes-agendamento.md)
