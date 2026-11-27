# Catálogo de softwares e licenças

Este documento cataloga os softwares usados no desenvolvimento do **Conecta+** e suas licenças: bibliotecas do aplicativo e da API, componentes copiados para o projeto, serviços externos, ferramentas de desenvolvimento, de documentação e de apoio.

Levantamento feito em 27/11/2026 na branch `main` do FRONTEND (`4f53a25`) e do BACKEND (`6e0d3f5`) e refeito no mesmo dia para o FRONTEND no merge `def3ca2`, que traz o commit `299afc2` com a biblioteca `jspdf` 4.2.1 e 20 dependências dela, com Node.js 24.12.0 e `license-checker` 25.0.1. As licenças das bibliotecas foram lidas do `package.json` de cada pacote instalado em `node_modules`; antes do levantamento, as versões instaladas foram comparadas com o `package-lock.json` de cada repositório, sem nenhuma diferença.

## 1. Resumo

| Repositório | Pacotes (total) | Usados em produção | Usados só em desenvolvimento | Licença predominante |
| ----------- | --------------: | -----------------: | ---------------------------: | -------------------- |
| BACKEND     |             375 |                152 |                          223 | MIT (88%)            |
| FRONTEND    |             789 |                573 |                          216 | MIT (90%)            |

Os totais incluem as dependências diretas, declaradas no `package.json`, e as indiretas, que são as dependências delas. A lista completa, pacote por pacote, está em [licencas/backend.csv](licencas/backend.csv) e [licencas/frontend.csv](licencas/frontend.csv).

Todas as licenças encontradas são **permissivas** ou de **copyleft fraco**. Nenhuma impede o uso, a modificação ou a distribuição do Conecta+.

### Contagem por licença

| Licença                   | BACKEND | FRONTEND |
| ------------------------- | ------: | -------: |
| MIT                       |     329 |      709 |
| ISC                       |      16 |       27 |
| Apache-2.0                |      10 |       21 |
| BSD-3-Clause              |      10 |        6 |
| BSD-2-Clause              |       4 |        9 |
| BlueOak-1.0.0             |       1 |        2 |
| MPL-2.0                   |       2 |        2 |
| 0BSD                      |       1 |        1 |
| MIT-0                     |       1 |        — |
| Unlicense                 |       — |        2 |
| (MIT OR CC0-1.0)          |       1 |        2 |
| (MIT OR Apache-2.0)       |       — |        1 |
| (BSD-3-Clause OR GPL-2.0) |       — |        1 |
| MIT AND OFL-1.1           |       — |        1 |
| Python-2.0                |       — |        1 |
| CC-BY-4.0                 |       — |        1 |
| (MPL-2.0 OR Apache-2.0)   |       — |        1 |
| (MIT AND Zlib)            |       — |        1 |
| MIT*                      |       — |        1 |

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
| **Zlib** | Permissiva | Não apresentar o software como próprio, manter o aviso e indicar as versões alteradas. |
| **OFL-1.1** (SIL Open Font License) | Licença de fonte | A fonte pode ser usada e embutida no app; não pode ser vendida sozinha. |
| **CC-BY-4.0** | Atribuição | Dar crédito ao autor. |
| **MPL-2.0** | Copyleft fraco, por arquivo | Só os arquivos da própria biblioteca, se modificados, precisam continuar abertos. O Conecta+ não modifica nenhum. |
| **GPL-2.0** | Copyleft forte | Só aparece em licença dupla; o projeto adota a outra opção (ver seção 3). |

## 3. Pacotes com licenças fora do padrão MIT/ISC/Apache/BSD

| Pacote | Versão | Licença | Onde | Observação |
| ------ | ------ | ------- | ---- | ---------- |
| `@expo-google-fonts/poppins` | 0.4.1 | MIT AND OFL-1.1 | FRONTEND (direta) | Código em MIT e fonte Poppins em OFL-1.1. |
| `nodemailer` | 10.0.10 | MIT-0 | BACKEND (direta) | Envio de e-mails de confirmação, recuperação e do link de presença. |
| `node-forge` | 1.4.4 | (BSD-3-Clause OR GPL-2.0) | FRONTEND (indireta) | Licença dupla; **o projeto usa sob a BSD-3-Clause**. |
| `caniuse-lite` | 1.0.30001818 | CC-BY-4.0 | FRONTEND (indireta) | Base de dados de compatibilidade de navegadores; atribuição ao projeto Can I Use. |
| `argparse` | 2.0.1 | Python-2.0 | FRONTEND (indireta) | Porte da biblioteca do Python. |
| `lightningcss`, `lightningcss-win32-x64-msvc` | 1.33.0 | MPL-2.0 | Ambos (indireta) | Ferramenta de build de CSS; não é modificada. No FRONTEND, o `license-checker` a conta como produção porque vem pelas dependências de produção do Expo. |
| `minimatch` | 10.2.6 | BlueOak-1.0.0 | Ambos (indireta) | Comparação de caminhos de arquivos. |
| `sax` | 1.6.1 | BlueOak-1.0.0 | FRONTEND (indireta) | Leitura de XML. |
| `big-integer`, `stream-buffers` | 1.6.52, 2.2.0 | Unlicense | FRONTEND (indireta) | — |
| `tslib` | 2.8.1 | 0BSD | BACKEND (indireta) | — |
| `jsc-safe-url` | 0.2.4 | 0BSD | FRONTEND (indireta) | — |
| `type-fest` | 0.21.3 (ambos), 0.7.1 (FRONTEND) | (MIT OR CC0-1.0) | Ambos (indireta) | Licença dupla, ambas permissivas. |
| `fb-dotslash` | 0.5.8 | (MIT OR Apache-2.0) | FRONTEND (indireta) | Licença dupla, ambas permissivas. |
| `dompurify` | 3.5.0 | (MPL-2.0 OR Apache-2.0) | FRONTEND (indireta) | Dependência opcional do `jspdf`; licença dupla; **o projeto usa sob a Apache-2.0**. |
| `pako` | 2.2.0 | (MIT AND Zlib) | FRONTEND (indireta) | Compressão usada pelo `fast-png`, dependência do `jspdf`; as duas licenças são permissivas. |
| `rgbcolor` | 1.0.1 | MIT* | FRONTEND (indireta) | Dependência do `canvg`, opcional do `jspdf`. O asterisco é do `license-checker`: o `package.json` declara "MIT OR SEE LICENSE IN FEEL-FREE.md", e o `LICENSE.md` do pacote é o texto da MIT; o projeto usa sob a MIT. |

## 4. Dependências diretas

Versões exatas instaladas, iguais às do `package-lock.json` de cada repositório.

### BACKEND (API NestJS)

| Pacote | Versão | Licença | Uso |
| ------ | ------ | ------- | --- |
| `@nestjs/common` | 12.0.1 | MIT | Framework da API |
| `@nestjs/core` | 12.0.1 | MIT | Framework da API |
| `@nestjs/config` | 12.0.0 | MIT | Variáveis de ambiente |
| `@nestjs/jwt` | 12.0.1 | MIT | Tokens de autenticação |
| `@nestjs/mongoose` | 12.0.0 | MIT | Integração com MongoDB |
| `@nestjs/platform-express` | 12.0.1 | MIT | Servidor HTTP e upload de anexos |
| `mongoose` | 9.9.5 | MIT | Schemas e consultas no MongoDB (instala o driver `mongodb` 7.5.0, usado também no GridFS dos anexos) |
| `bcrypt` | 6.0.0 | MIT | Hash de senhas |
| `class-validator` | 0.15.1 | MIT | Validação dos DTOs |
| `class-transformer` | 0.5.1 | MIT | Conversão dos DTOs |
| `nodemailer` | 10.0.10 | MIT-0 | Envio de e-mails |
| `reflect-metadata` | 0.2.2 | Apache-2.0 | Metadados dos decorators |
| `rxjs` | 7.8.2 | Apache-2.0 | Programação reativa (NestJS) |

Desenvolvimento:

| Pacote | Versão | Licença | Uso |
| ------ | ------ | ------- | --- |
| `@nestjs/cli` | 12.0.0 | MIT | Compilação (`nest build`) e execução local |
| `@nestjs/schematics` | 12.0.0 | MIT | Geração de código do NestJS |
| `@nestjs/testing` | 12.0.1 | MIT | Módulos de teste do NestJS |
| `@nestjs/mau` | 0.2.6 | MIT | Ferramenta de linha de comando do NestJS |
| `vitest` | 4.1.11 | MIT | Testes automatizados |
| `@vitest/coverage-v8` | 4.1.11 | MIT | Cobertura dos testes |
| `supertest` | 7.2.2 | MIT | Testes das rotas HTTP |
| `oxlint` | 1.82.0 | MIT | Análise estática |
| `prettier` | 3.9.6 | MIT | Formatação do código |
| `source-map-support` | 0.5.21 | MIT | Linhas do TypeScript nos erros |
| `typescript` | 6.0.3 | Apache-2.0 | Tipagem e compilação |
| `@types/bcrypt` | 6.0.0 | MIT | Tipos do TypeScript |
| `@types/express` | 5.0.6 | MIT | Tipos do TypeScript |
| `@types/node` | 24.13.3 | MIT | Tipos do TypeScript |
| `@types/nodemailer` | 8.0.2 | MIT | Tipos do TypeScript |
| `@types/supertest` | 7.2.1 | MIT | Tipos do TypeScript |

### FRONTEND (aplicativo Expo / React Native)

| Pacote | Versão | Licença | Uso |
| ------ | ------ | ------- | --- |
| `react`, `react-dom` | 19.3.0 | MIT | Biblioteca de interface |
| `react-native` | 0.88.1 | MIT | Aplicativo nativo Android e iOS |
| `expo` | 58.1.0 | MIT | SDK Expo |
| `expo-asset` | 58.0.21 | MIT | Leitura do manual em PDF e do PDF.js embutidos |
| `expo-auth-session` | 58.0.20 | MIT | Login com Google na web |
| `expo-build-properties` | 58.0.12 | MIT | Ajustes da compilação nativa Android |
| `expo-clipboard` | 58.0.4 | MIT | Cópia do código de acesso e do link de presença |
| `expo-constants` | 58.0.16 | MIT | Configuração do Expo e endereço local da API |
| `expo-dev-client` | 58.0.22 | MIT | Versões de desenvolvimento |
| `expo-document-picker` | 58.0.5 | MIT | Escolha dos anexos da pauta |
| `expo-file-system` | 58.0.15 | MIT | Anexos, CSV do relatório e leitura do manual |
| `expo-font` | 58.0.12 | MIT | Carregamento das fontes |
| `expo-linear-gradient` | 58.0.4 | MIT | Fundos visuais |
| `expo-linking` | 58.0.17 | MIT | Links e retornos do login |
| `expo-local-authentication` | 58.0.3 | MIT | Biometria, Face ID ou PIN do aparelho |
| `expo-print` | 58.0.9 | MIT | PDF da ata, da lista de presença e do relatório no celular |
| `expo-router` | 58.0.33 | MIT | Navegação entre telas |
| `expo-secure-store` | 58.0.3 | MIT | Token e tema no aparelho |
| `expo-sharing` | 58.0.24 | MIT | Menu de compartilhamento do PDF e do CSV |
| `expo-splash-screen` | 58.0.15 | MIT | Tela de abertura |
| `expo-status-bar` | 58.0.5 | MIT | Cor da barra de status |
| `expo-system-ui` | 58.0.7 | MIT | Tema da janela nativa do Android |
| `@react-native-google-signin/google-signin` | 16.1.5 | MIT | Login com Google no aplicativo instalado |
| `@react-native-community/datetimepicker` | 9.2.1 | MIT | Seletor de data e hora |
| `@react-native-vector-icons/ionicons` | 14.0.0 | MIT | Ícones |
| `@expo-google-fonts/poppins` | 0.4.1 | MIT AND OFL-1.1 | Fonte Poppins |
| `react-hook-form` | 7.90.0 | MIT | Formulários |
| `react-native-webview` | 14.0.1 | MIT | Exibição do manual offline |
| `react-native-reanimated` | 4.7.1 | MIT | Animações |
| `react-native-worklets` | 0.13.0 | MIT | Animações |
| `react-native-gesture-handler` | 3.3.0 | MIT | Gestos |
| `react-native-keyboard-controller` | 1.22.7 | MIT | Controle do teclado |
| `react-native-safe-area-context` | 5.11.1 | MIT | Áreas seguras da tela |
| `react-native-screens` | 4.30.1 | MIT | Telas nativas |
| `react-native-web` | 0.21.4 | MIT | Execução na web |
| `jspdf` | 4.2.1 | MIT | PDF do relatório individual baixado direto na web |

Desenvolvimento:

| Pacote | Versão | Licença | Uso |
| ------ | ------ | ------- | --- |
| `eslint` | 9.39.5 | MIT | Análise estática |
| `eslint-config-expo` | 58.0.4 | MIT | Regras do ESLint para Expo |
| `typescript` | 6.0.3 | Apache-2.0 | Tipagem e verificação estática |
| `@types/react` | 19.3.0 | MIT | Tipos do TypeScript |

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
| MongoDB Community Server 8.3.8, local | Server Side Public License (SSPL) | Banco de dados, incluindo os anexos no GridFS |
| Google Identity (OAuth 2.0 / OpenID Connect) | Termos de Serviço das APIs do Google | Login com Google |
| Google Cloud Console | Termos do Google Cloud | Configuração do projeto e das credenciais OAuth |
| Servidor SMTP | Depende do provedor configurado em `MAIL_*` | Envio de e-mails |
| GitHub | Termos de Serviço do GitHub | Hospedagem dos repositórios |

O banco usado é o **MongoDB Community Server instalado no computador da equipe**, e não um serviço em nuvem. Em 27/11/2026, o servidor local (`mongodb://127.0.0.1:27017`, o mesmo endereço do `MONGODB_URI` usado no desenvolvimento) respondeu ao comando `buildInfo`, consulta que só lê informações, com a versão **8.3.8**, sem módulos Enterprise. O `BACKEND/.env.example` sugere, num comentário, o instalador do MongoDB 6.0.29 para Windows: é só o link de instalação deixado no exemplo, e não a versão em que o sistema foi executado e testado, que é a 8.3. A 6.0.29 não foi testada nesta revisão.

A SSPL não afeta o Conecta+: ela só impõe obrigações a quem oferece o próprio MongoDB como serviço para terceiros.

**Previsto na proposta, não integrado:** Google Meet. A [proposta inicial](1%20-%20Proposta%20Inicial/objetivos.md) previa um link do Google Meet gerado para cada reunião, mas o código do FRONTEND e do BACKEND não usa o Google Meet nem suas APIs (conferido em 27/11/2026).

### Ferramentas de desenvolvimento

| Ferramenta | Licença |
| ---------- | ------- |
| Node.js | MIT |
| npm | Artistic-2.0 |
| TypeScript | Apache-2.0 |
| Expo CLI | MIT |
| Android Studio, Android SDK e emulador | Android Software Development Kit License Agreement (componentes da AOSP em Apache-2.0) |
| JDK Eclipse Temurin (build Android) | GPL-2.0 com Classpath Exception |
| CMake (build Android) | BSD-3-Clause |
| Git | GPL-2.0 |
| Visual Studio Code | Licença Microsoft (binário); código-fonte em MIT |

### Ferramentas de documentação e apoio

| Ferramenta | Uso | Licença ou termos |
| ---------- | --- | ----------------- |
| Figma | Identidade visual (nome, ícone, logotipo, fontes e cores), citada na proposta inicial | Serviço proprietário; termos de serviço da Figma |
| Mermaid | Diagramas de classes, relacional e de sequência da documentação | MIT |
| draw.io (diagrams.net) | Diagramas UML de casos de uso (SVG) | Apache-2.0 (código aberto, gratuito) |
| `license-checker` 25.0.1 | Levantamento das licenças deste catálogo | BSD-3-Clause |
| GitHub Wiki | Publicação da documentação | Termos de Serviço do GitHub |
| Expo Go | Testes do aplicativo no iPhone | MIT |
| MongoDB Compass | Consulta ao banco nos testes | Gratuito; termos da MongoDB |

## 6. Licença do próprio Conecta+

O Conecta+ é distribuído sob a **licença MIT**, nos três repositórios: FRONTEND, BACKEND e DOCUMENTATION, este incluindo a documentação. Cada repositório tem um arquivo `LICENSE` com o texto oficial da MIT e o aviso `Copyright (c) 2026 Equipe Conecta+`, e os `package.json` do FRONTEND e do BACKEND declaram `"license": "MIT"`.

A MIT **permite** usar, copiar, modificar, juntar a outros trabalhos, publicar, distribuir, sublicenciar e vender cópias do software, sem custo. Ela **exige** apenas que o aviso de copyright e o texto da licença acompanhem todas as cópias ou partes relevantes do software. O software é fornecido "no estado em que se encontra", sem garantia, e os autores não respondem por danos decorrentes do uso.

A MIT é compatível com todas as dependências do projeto. Todas as licenças encontradas são permissivas (MIT, ISC, Apache-2.0, BSD, BlueOak, 0BSD, MIT-0, Unlicense, CC0, Python-2.0, Zlib) ou de copyleft fraco por arquivo (MPL-2.0, que só alcança arquivos da própria biblioteca, que o projeto não modifica). Nenhuma exige que o Conecta+ adote a mesma licença. O `node-forge`, o único pacote com GPL-2.0, tem licença dupla e é usado sob a BSD-3-Clause. A fonte Poppins (OFL-1.1) pode ser embutida no aplicativo, e o `caniuse-lite` (CC-BY-4.0) só pede atribuição, registrada neste catálogo.

## 7. Como atualizar este catálogo

Em cada repositório, depois de conferir que o `node_modules` está igual ao `package-lock.json`:

```bash
npx license-checker@25.0.1 --summary
npx license-checker@25.0.1 --json --out todas.json
npx license-checker@25.0.1 --production --json --out producao.json
```

Os arquivos `licencas/*.csv` são gerados a partir desses dois JSON, fora do repositório, com as colunas `pacote`, `versao`, `licenca`, `uso` (`producao`, quando o pacote aparece no JSON de produção, ou `desenvolvimento`) e `repositorio`, separadas por `;`, em UTF-8 com BOM, em ordem alfabética e sem o próprio pacote do aplicativo.
