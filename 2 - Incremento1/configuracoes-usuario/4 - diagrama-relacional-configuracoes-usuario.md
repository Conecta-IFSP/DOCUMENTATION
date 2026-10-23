# Diagrama relacional — Configurações de usuário

O MongoDB armazena todo o módulo em um único documento da coleção `usuarios`. O diagrama abaixo separa grupos lógicos de campos apenas para leitura; não existem tabelas físicas de recuperação ou troca de e-mail.

```mermaid
erDiagram
    USUARIO {
        ObjectId _id PK
        String nome
        String email UK
        String senha_hash
        String tipo
        String tema
        Boolean email_verificado
        Boolean ativo
        String provedor_autenticacao
        Date criado_em
        Date atualizado_em
    }
    RECUPERACAO_TEMPORARIA {
        String reset_senha_codigo_hash
        String reset_senha_token_hash
        Date reset_senha_expira_em
    }
    TROCA_EMAIL_TEMPORARIA {
        String email_pendente
        String email_pendente_codigo_hash
        Date email_pendente_expira_em
    }
    USUARIO ||--o| RECUPERACAO_TEMPORARIA : "campos opcionais embutidos"
    USUARIO ||--o| TROCA_EMAIL_TEMPORARIA : "campos opcionais embutidos"
```

## Decisões

- O e-mail tem índice único criado pelo schema Mongoose.
- Códigos e tokens em texto não são persistidos; somente SHA-256.
- A senha é persistida apenas como bcrypt.
- Os grupos temporários são apagados após uso; não representam histórico.

