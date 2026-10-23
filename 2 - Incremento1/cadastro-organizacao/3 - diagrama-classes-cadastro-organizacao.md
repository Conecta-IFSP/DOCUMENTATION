# Diagrama de classes — Cadastro de organização

> Padrão do Incremento 1 · Classes confirmadas no código auditado.

## Resumo

`OrganizacoesController` delega regras ao `OrganizacoesService`. A organização contém seus vínculos como subdocumentos e referencia o usuário criador.

## Diagrama

```mermaid
classDiagram
    class OrganizacoesController {
        +criar(dto)
        +listar(busca)
        +atualizar(id, dto)
        +remover(id)
        +entrar(dto)
        +convidar(id, dto)
        +responderConvite(id, dto)
        +alterarPapelMembro(id, usuarioId, dto)
    }
    class OrganizacoesService {
        +criar(dto, usuarioId)
        +listar(usuario, busca)
        +atualizar(id, dto, usuario)
        +remover(id)
        +entrarComCodigo(codigo, usuarioId)
        +convidar(id, email, adminId)
        +responderConvite(id, aceitar, usuarioId)
        +alterarPapelMembro(id, alvoId, papel, adminId)
    }
    class Organizacao {
        +ObjectId _id
        +String nome
        +String cidade
        +String descricao
        +StatusOrganizacao status
        +ObjectId criada_por
        +String codigo_acesso
        +MembroOrganizacao[] membros
    }
    class MembroOrganizacao {
        +ObjectId usuario_id
        +PapelOrganizacao papel
        +StatusMembroOrganizacao status
        +Date solicitado_em
        +Date aprovado_em
    }
    class Usuario
    class Comissao

    OrganizacoesController --> OrganizacoesService : delega
    OrganizacoesService --> Organizacao : consulta e atualiza
    OrganizacoesService --> Usuario : valida e convida
    OrganizacoesService --> Comissao : remove em cascata
    Organizacao "1" *-- "1..*" MembroOrganizacao : contém
    Usuario "1" --> "0..*" Organizacao : cria
    Usuario "1" --> "0..*" MembroOrganizacao : possui vínculo
```

## Regras representadas

- o criador inicia como `ADMIN/APROVADO`;
- o código de dez caracteres é único e oculto por padrão;
- os papéis internos não alteram o tipo global do usuário;
- uma organização mantém ao menos um administrador aprovado.

## Rastreabilidade

RF-ORG-01 a RF-ORG-09 e dicionário de classes do módulo.
