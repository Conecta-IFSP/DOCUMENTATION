# Catálogo de softwares e licenças

Este documento cataloga os softwares usados no desenvolvimento do **Conecta+** e suas licenças: bibliotecas do aplicativo e da API, componentes copiados para o projeto, serviços externos e ferramentas de desenvolvimento.

Levantamento feito em 26/11/2026, na branch `homolog` do FRONTEND e do BACKEND, com Node.js 24.12.0 e `license-checker` 25.0.1. As licenças das bibliotecas foram lidas do `package.json` de cada pacote instalado em `node_modules`.

## 1. Resumo

| Repositório | Pacotes (total) | Usados em produção | Usados só em desenvolvimento | Licença predominante |
| ----------- | --------------: | -----------------: | ---------------------------: | -------------------- |
| BACKEND     |             375 |                152 |                          223 | MIT (87%)            |
| FRONTEND    |             780 |                562 |                          218 | MIT (89%)            |

Os totais incluem as dependências diretas, declaradas no `package.json`, e as indiretas, que são as dependências delas. A lista completa, pacote por pacote, está em [licencas/backend.csv](licencas/backend.csv) e [licencas/frontend.csv](licencas/frontend.csv).

Todas as licenças encontradas são **permissivas** ou de **copyleft fraco**. Nenhuma impede o uso, a modificação ou a distribuição do Conecta+.

### Contagem por licença

| Licença               | BACKEND | FRONTEND |
| --------------------- | ------: | -------: |
| MIT                   |     328 |      692 |
| ISC                   |      16 |       27 |
| Apache-2.0            |      10 |       22 |
| BSD-3-Clause          |      10 |        8 |
| BSD-2-Clause          |       4 |        9 |
| BlueOak-1.0.0         |       1 |        6 |
| MPL-2.0               |       2 |        2 |
| 0BSD                  |       1 |        2 |
| MIT-0                 |       1 |        — |
| Unlicense             |       — |        2 |
| (MIT OR CC0-1.0)      |       1 |        2 |
| (MIT OR Apache-2.0)   |       — |        1 |
| (BSD-3-Clause OR GPL-2.0) |   — |        1 |
| MIT AND OFL-1.1       |       — |        1 |
| Python-2.0            |       — |        1 |
| CC-BY-4.0             |       — |        1 |

A contagem não inclui o próprio Conecta+ (ver seção 6).

## 2. Licenças encontradas

| Licença | Tipo | O que exige |
| ------- | ---- | ----------- |
| **MIT**, **ISC**, **BSD-2-Clause** | Permissiva | Manter o aviso de copyright e o texto da licença. |
| **BSD-3-Clause** | Permissiva | Igual à BSD-2, e não usar o nome dos autores para promover o produto. |
| **Apache-2.0** | Permissiva | Manter avisos e o arquivo NOTICE, quando existir; inclui concessão de patentes. |
| **0BSD**, **MIT-0**, **Unlicense**, **CC0-1.0** | Permissiva sem condições | Nada; equivalem a domínio público. |
| **BlueOak-1.0.0** | Permissiva | Manter a referência à licença. |
| **Python-2.0** | Permissiva | Manter o aviso de copyright. |
| **OFL-1.1** (SIL Open Font License) | Licença de fonte | A fonte pode ser usada e embutida no app; não pode ser vendida sozinha. |
| **CC-BY-4.0** | Atribuição | Dar crédito ao autor. |
| **MPL-2.0** | Copyleft fraco, por arquivo | Só os arquivos da própria biblioteca, se modificados, precisam continuar abertos. O Conecta+ não modifica nenhum. |
| **GPL-2.0** | Copyleft forte | Só aparece em licença dupla; o projeto adota a outra opção (ver seção 3). |

## 3. Pacotes com licenças fora do padrão MIT/ISC/Apache/BSD

| Pacote | Versão | Licença | Onde | Observação |
| ------ | ------ | ------- | ---- | ---------- |
| `@expo-google-fonts/poppins` | 0.4.1 | MIT AND OFL-1.1 | FRONTEND (direta) | Código em MIT e fonte Poppins em OFL-1.1. |
| `nodemailer` | 10.0.10 | MIT-0 | BACKEND (direta) | Envio de e-mails de confirmação e recuperação. |
| `node-forge` | 1.4.3 | (BSD-3-Clause OR GPL-2.0) | FRONTEND (indireta) | Licença dupla; **o projeto usa sob a BSD-3-Clause**. |
| `caniuse-lite` | 1.0.30001818 | CC-BY-4.0 | FRONTEND (indireta) | Base de dados de compatibilidade de navegadores; atribuição ao projeto Can I Use. |
| `argparse` | 2.0.1 | Python-2.0 | FRONTEND (indireta) | Porte da biblioteca do Python. |
| `lightningcss` | 1.33.0 | MPL-2.0 | Ambos (indireta) | Usada nas ferramentas de build; não é modificada. |
| `glob`, `lru-cache`, `minimatch`, `minipass`, `path-scurry`, `sax` | várias | BlueOak-1.0.0 | Ambos (indireta) | Utilitários de arquivos e cache. |
| `big-integer`, `stream-buffers` | 1.6.52, 2.2.0 | Unlicense | FRONTEND (indireta) | — |
| `tslib`, `jsc-safe-url` | 2.8.1, 0.2.4 | 0BSD | Ambos (indireta) | — |
| `type-fest` | 0.21.3, 0.7.1 | (MIT OR CC0-1.0) | Ambos (indireta) | Licença dupla, ambas permissivas. |
| `fb-dotslash` | 0.5.8 | (MIT OR Apache-2.0) | FRONTEND (indireta) | Licença dupla, ambas permissivas. |

## 4. Dependências diretas

### BACKEND (API NestJS)

| Pacote | Versão | Licença | Uso |
| ------ | ------ | ------- | --- |
| `@nestjs/common`, `@nestjs/core` | 12.0.1 | MIT | Framework da API |
| `@nestjs/config` | 12.0.0 | MIT | Variáveis de ambiente |
| `@nestjs/jwt` | 12.0.1 | MIT | Tokens de autenticação |
| `@nestjs/mongoose` | 12.0.0 | MIT | Integração com MongoDB |
| `@nestjs/platform-express` | 12.0.1 | MIT | Servidor HTTP e upload de anexos |
| `mongoose` | 9.9.5 | MIT | Schemas e consultas no MongoDB |
| `bcrypt` | 6.0.0 | MIT | Hash de senhas |
| `class-validator` | 0.15.1 | MIT | Validação dos DTOs |
| `class-transformer` | 0.5.1 | MIT | Conversão dos DTOs |
| `nodemailer` | 10.0.10 | MIT-0 | Envio de e-mails |
| `reflect-metadata` | 0.2.2 | Apache-2.0 | Metadados dos decorators |
| `rxjs` | 7.8.2 | Apache-2.0 | Programação reativa (NestJS) |

Desenvolvimento: `@nestjs/cli`, `@nestjs/schematics`, `@nestjs/testing` e `@nestjs/mau` (MIT), `vitest` e `@vitest/coverage-v8` (MIT), `supertest` (MIT), `oxlint` (MIT), `prettier` (MIT), `source-map-support` (MIT), `typescript` (Apache-2.0) e os pacotes `@types/*` (MIT).

### FRONTEND (aplicativo Expo / React Native)

| Pacote | Versão | Licença | Uso |
| ------ | ------ | ------- | --- |
| `react`, `react-dom` | 19.3.0 | MIT | Biblioteca de interface |
| `react-native` | 0.88.1 | MIT | Aplicativo nativo Android e iOS |
| `expo` e módulos `expo-*` (20 pacotes) | 58.0.x | MIT | SDK Expo: rotas, fontes, arquivos, impressão do PDF da ata, compartilhamento, biometria, armazenamento seguro, autenticação etc. |
| `expo-router` | 58.0.30 | MIT | Navegação entre telas |
| `@react-native-google-signin/google-signin` | 16.1.5 | MIT | Login com Google |
| `@react-native-community/datetimepicker` | 9.2.1 | MIT | Seletor de data e hora |
| `@react-native-vector-icons/ionicons` | 14.0.0 | MIT | Ícones |
| `@expo-google-fonts/poppins` | 0.4.1 | MIT AND OFL-1.1 | Fonte Poppins |
| `react-hook-form` | 7.90.0 | MIT | Formulários |
| `react-native-webview` | 14.0.1 | MIT | Exibição do manual offline |
| `react-native-reanimated`, `react-native-worklets` | 4.7.1, 0.13.0 | MIT | Animações |
| `react-native-gesture-handler` | 3.3.0 | MIT | Gestos |
| `react-native-keyboard-controller` | 1.22.7 | MIT | Controle do teclado |
| `react-native-safe-area-context` | 5.11.1 | MIT | Áreas seguras da tela |
| `react-native-screens` | 4.30.1 | MIT | Telas nativas |
| `react-native-web` | 0.21.4 | MIT | Execução na web |

Desenvolvimento: `eslint` e `eslint-config-expo` (MIT), `typescript` (Apache-2.0) e `@types/react` (MIT).

## 5. Componentes, serviços e ferramentas fora do npm

O `license-checker` só enxerga o que está em `node_modules`. Os itens abaixo foram catalogados manualmente.

### Componentes embutidos no código

| Componente | Versão | Licença | Uso |
| ---------- | ------ | ------- | --- |
| PDF.js (`pdfjs-dist`, arquivos `pdf.min.js` e `pdf.worker.min.js`) | 3.11.174 | Apache-2.0 | Renderização do manual offline; copiado para `FRONTEND/assets/pdfjs/` |
| Fonte Poppins | — | OFL-1.1 | Tipografia do aplicativo |

### Serviços externos

| Serviço | Licença / termos | Uso |
| ------- | ---------------- | --- |
| MongoDB | Server Side Public License (SSPL) para o MongoDB Community Server; termos de serviço no MongoDB Atlas | Banco de dados, incluindo os anexos no GridFS |
| Google Identity (OAuth 2.0 / OpenID Connect) | Termos de Serviço das APIs do Google | Login com Google |
| Google Meet | Termos de Serviço do Google | Link das reuniões |
| Google Cloud Console | Termos do Google Cloud | Configuração do projeto e das credenciais OAuth |
| Servidor SMTP | Depende do provedor configurado em `MAIL_*` | Envio de e-mails |
| GitHub | Termos de Serviço do GitHub | Hospedagem dos repositórios |

A SSPL não afeta o Conecta+: ela só impõe obrigações a quem oferece o próprio MongoDB como serviço para terceiros.

### Ferramentas de desenvolvimento

| Ferramenta | Licença |
| ---------- | ------- |
| Node.js | MIT |
| npm | Artistic-2.0 |
| TypeScript | Apache-2.0 |
| Expo CLI e Expo Go | MIT |
| Android Studio, Android SDK e emulador | Android Software Development Kit License Agreement (componentes da AOSP em Apache-2.0) |
| JDK Eclipse Temurin (build Android) | GPL-2.0 com Classpath Exception |
| CMake (build Android) | BSD-3-Clause |
| Git | GPL-2.0 |
| Visual Studio Code | Licença Microsoft (binário); código-fonte em MIT |

## 6. Licença do próprio Conecta+

O Conecta+ ainda não tem licença definida: o `package.json` do BACKEND declara `"license": "UNLICENSED"`, o do FRONTEND não declara licença, e nenhum dos repositórios tem um arquivo `LICENSE`. Sem licença, o código fica com todos os direitos reservados aos autores.

Como todas as dependências são compatíveis com licenças permissivas, a equipe pode adotar, por exemplo, a **MIT**. Para isso, basta incluir um arquivo `LICENSE` em cada repositório e atualizar o campo `license` dos `package.json`.

## 7. Como atualizar este catálogo

Em cada repositório, depois de `npm install`:

```bash
npx license-checker --summary
npx license-checker --json --out todas.json
npx license-checker --production --json --out producao.json
```

Os arquivos `licencas/*.csv` foram gerados a partir desses dois JSON, com as colunas `pacote`, `versao`, `licenca`, `uso` (produção ou desenvolvimento) e `repositorio`.
