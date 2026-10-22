# Execução e roteiro de apresentação

## Pastas locais

As cópias editadas estão em `C:\Users\Rafaela\Documents\ChatGPT\Conecta+\incremento-2`:

- BACKEND: API NestJS, TypeScript e MongoDB.
- FRONTEND: aplicativo React Native/Expo, TypeScript/TSX.
- DOCUMENTATION: documentação; este pacote está em `3 - Incremento2/comissoes`.

Cada pasta é seu próprio repositório. Não copiar `.git` entre elas. As mudanças não foram commitadas nem enviadas ao GitHub nesta revisão.

## Preparar ambiente

Backend precisa de `.env` local com `MONGODB_URI` apontando para banco de desenvolvimento e `JWT_SECRET` exclusivo do ambiente. Não usar banco de produção para demonstração. Não publicar o segredo. Verificar configurações de e-mail do módulo de autenticação com o responsável dessa parte antes de testar cadastro/recuperação; não são requisitos da comissão.

Na pasta BACKEND:

```powershell
npm.cmd ci
npm.cmd run start:dev
```

No FRONTEND, criar `.env` local com `EXPO_PUBLIC_API_URL=http://localhost:3000` para web. No aparelho, usar IP acessível do computador e verificar rede/firewall; o helper existente tenta substituir localhost pelo host do Expo em ambiente nativo. URL pública não é segredo. Em outro terminal:

```powershell
npm.cmd ci
npm.cmd start
```

Usar usuário comum ativo, membro ADMIN/APROVADO de organização APROVADA e ao menos outro integrante ativo/APROVADO. ADMIN_SISTEMA não deve ser usado como administrador da organização nessa demonstração. Nesta revisão não foram geradas credenciais reais nem alteradas contas existentes.

## Explicar ao professor

- **Qual linguagem?** TypeScript. TSX combina TypeScript e marcação JSX nas telas React Native. NestJS organiza a API; Mongoose define documentos e consultas MongoDB.
- **Qual API externa?** Nenhuma para comissões/atas. O app chama a API do próprio grupo via fetch. Bibliotecas não são serviços externos de dados. A pesquisa de mercado não significa integração com APIs dos concorrentes.
- **Por que validar também no servidor?** Uma pessoa pode chamar a rota sem usar a tela. DTO valida formato; serviço valida autorização e regra de negócio; schema reforça estrutura do documento.
- **Por que não usar dois booleans de responsável?** A equipe tem papel por integrante, mas a regra exige contagem igual a um. Transferência troca todos os papéis em uma única escrita; não desmarca primeiro e marca depois.
- **O que impede concorrência?** O filtro da atualização inclui o array que foi lido. Se alguém mudar antes, o filtro não encontra o mesmo estado e o serviço responde 409. Não existe transação entre todas as coleções.
- **Por que DELETE não apaga?** A comissão vira ativo=false para preservar referências e histórico. O nome da operação HTTP não exige exclusão física.
- **O responsável pode administrar toda a organização?** Não. ADMIN/APROVADO é vínculo de organização; RESPONSAVEL é papel na comissão.
- **O que é ObjectId?** Identificador BSON do MongoDB; no JSON aparece como string hexadecimal de 24 caracteres. Não é senha nem prova de autorização.
- **Para que serve populate?** Substitui referências pelos campos selecionados, como nome/e-mail. Não é uma tabela SQL nem cria foreign key.
- **A versão da ata guarda todas as versões?** Não. Apenas o texto atual é persistido, com contador para detectar edição desatualizada.
- **Aceitar convite é comprovar presença?** Não. São campos diferentes. A lista pode mostrar ACEITO e SEM_REGISTRO.
- **A ata é assinada digitalmente?** Não. Há aprovação autenticada por ID de usuário; não certificado criptográfico e nem timestamp individual de cada voto.
- **O que ainda falta validar?** Regras de aprovação/reabertura com a comunidade, fluxo após troca de responsável e testes nativos Android/iOS.

## Publicação posterior

Revisar `git diff` em cada repositório, decidir a branch de destino com o grupo e executar os testes antes de commit/push. Este pacote não executa publicação automaticamente. Não incluir `.env`, node_modules, dist ou dados locais no commit.
