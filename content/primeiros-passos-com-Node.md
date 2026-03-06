Title: ✧ Primeiros passos com Node.js e Express
Date: 2026-03-06
Category: Tutoriais
Tags: node.js, back-end, express
Author: Amanda Meneses

Node.js revolucionou a forma como o JavaScript é utilizado no desenvolvimento web. Antes restrita ao navegador, a linguagem passou a ser utilizada também no lado do servidor, permitindo que desenvolvedores criem aplicações completas utilizando apenas JavaScript.

Entre as ferramentas mais populares dentro do ecossistema do Node.js está o Express.js, um framework minimalista que facilita a criação de servidores e APIs. Com poucas linhas de código, é possível configurar rotas, lidar com requisições HTTP e estruturar uma aplicação de forma organizada e escalável.

Neste artigo, vamos dar os primeiros passos com Node.js e Express, entendendo como preparar o ambiente, instalar dependências e criar um servidor básico. A partir desses conceitos fundamentais, você terá a base necessária para começar a desenvolver APIs e aplicações back-end utilizando JavaScript.

### Express.js

Express é um **framework para Node.js** que facilita a criação de servidores e APIs. Ele atua como uma camada de abstração sobre o módulo nativo `http` do Node, facilitando a construção de APIs RESTful e aplicações renderizadas no servidor (SSR).

- **Função principal:** Gerenciar o roteamento de URLs, verbos HTTP (GET, POST, etc.) e a cadeia de *middlewares* (funções interceptadoras que processam a requisição antes da resposta final).

### NPM (Node Package Manager)

NPM é o **gerenciador de pacotes padrão do Node.js**. Ele consiste em dois componentes principais: uma ferramenta de linha de comando (CLI) e um registro remoto (Registry) que hospeda pacotes de código aberto.

- **Função principal:** Gerenciar a árvore de dependências do projeto através do arquivo `package.json` e do diretório `node_modules`. Ele utiliza o arquivo de bloqueio `package-lock.json` para garantir que as versões das bibliotecas instaladas sejam consistentes entre diferentes ambientes.

### Yarn

Yarn é **uma alternativa ao npm**, também usada para gerenciar pacotes, com foco em determinismo, segurança e performance.

- **Diferenciais:** Utiliza um cache global agressivo (evitando downloads repetidos) e executa instalações de pacotes em paralelo, ao contrário da execução serial tradicional do NPM antigo.

Para instalar o Yarn **globalmente** (uma única vez no computador):

```bash
npm install -g yarn
yarn -v (apenas verifica a versão)
```

Depois disso, podemos usar o comando `yarn` em qualquer projeto.


### **Inicializando um projeto Node.js**

O comando `yarn init` é o primeiro passo para transformar uma pasta comum em um projeto Node.js. Ele inicia um questionário interativo no terminal para configurar os metadados do projeto.

```bash
yarn init
```

O terminal fará perguntas (Nome do projeto? Versão? Autor?). As respostas geram o arquivo `package.json`. Exemplo de `package.json` criado com `yarn init`:

```json
{
  "name": "mycontacts",
  "version": "1.0.0",
  "description": "API para guardar contatos",
  "main": "src/index.js",
  "author": "Amanda",
  "license": "MIT",
}
```

Para pular o questionário, utiliza-se a *flag* `-y` (que significa *yes* para tudo). O Yarn assume os valores padrão baseados no nome da pasta e configurações do sistema.  O arquivo `package.json` é criado com o mínimo necessário.

```json
{
  "name": "mycontacts",
  "version": "1.0.0",
  "main": "index.js",
  "license": "MIT",
}
```

**Instalação de dependências**

Para adicionar bibliotecas externas (como o framework Express) ao projeto, utilizamos o comando `add`.

```bash
yarn add express
```

Esse comando:

- Baixa o Express (e todas as dependências que ele precisa) para a pasta `node_modules`.
- **Registro:** Adiciona o Express à lista de `"dependencies"` no `package.json`.
- **Travamento de versão:** Cria ou atualiza o arquivo `yarn.lock`. Esse arquivo garante que qualquer pessoa da sua equipe instale exatamente a mesma versão do Express que você, evitando erros de compatibilidade.

`package.json` após a instalação:

```json
{
  "name": "mycontacts",
  "version": "1.0.0",
  "main": "index.js",
  "license": "MIT",
  "dependencies": {
    "express": "^5.2.1"
  }
}
```

### Criando um servidor http com Express

Agora, podemos criar o arquivo `index.js` dentro da pasta `src`, que será o ponto de entrada da aplicação.

```jsx
const express = require('express'); // importa o express

const app = express(); // instancia o express e cria uma aplicação (servidor)

app.get('/', (request, response) => { // cria uma rota GET
    response.send('Hello world!'); // envia uma resposta
});

app.listen(3000, () => {
    console.log('Server rodando na porta 3000, http://localhost:3000');
}); // escuta as requisições
```

- Cria um **servidor HTTP**
- Define uma rota:
    - **GET /** → retorna `"Hello world!"`
- Coloca o servidor para rodar na **porta 3000**

Ao acessar `http://localhost:3000`, o navegador faz uma requisição e recebe a resposta.

Para executar o servidor, utilizamos o comando no terminal:

```bash
node src/index.js
```

Sempre que ocorrer uma alteração no código, o servidor deve ser reiniciado com `ctrl+c` e o comando acima deve ser executado novamente. Para evitar esse processo, instalaremos o pacote **nodemon**.

```bash
yarn add nodemon -D
```

(O `-D` [`--dev`] significa dependência apenas de desenvolvimento**,** não é usada em produção.)

Para executar o servidor utilizando o nodemon, utilizamos o comando no terminal:

```bash
npx nodemon src/index.js
```

O nodemon detecta mudanças no código e reinicia o servidor assim que o arquivo é salvo.

**Dica: npm scripts no `package.json`**

Para não ter que ficar digitando `npx nodemon src/index.js` ou outros comandos longos, é possível criar atalhos (scripts).

Abra o arquivo `package.json` e adicione a seção `"scripts"`:

```json
{
  "name": "mycontacts",
  "version": "1.0.0",
  "main": "index.js",
  "license": "MIT",
  "scripts": {
    "dev": "nodemon src/index.js"
  },
  "dependencies": {
    "express": "^4.18.2"
  },
  "devDependencies": {
    "nodemon": "^3.0.1"
  }
}
```

Agora, no terminal, basta rodar:

```bash
npm run dev
```

**Configurando o ESLint e o editorconfig**

Uma coisa muito importante para manter o código limpo é manter a consistência. o **ESLint** pode ajudar seu código a ser mais consistente e robusto.

Como o ESLint é usado apenas durante o desenvolvimento, ele deve ser instalado como dependência de desenvolvimento:

```bash
yarn add eslint -D
```

Apenas instalar não faz ele funcionar. Diferente do Express, o ESLint precisa ser configurado para saber quais regras seguir. Geralmente, logo após instalar, você roda este comando para criar o arquivo de configuração (`.eslintrc` ou `eslint.config.js`):

```bash
yarn eslint --init
```

Ao pressionar `Ctrl + S`, o VS Code vai rodar o ESLint e aplicar todas as correções automáticas possíveis.

O **EditorConfig** serve para padronizar coisas como tamanho da indentação, uso de espaços ou tabs, quebra de linha. Ele garante que o código fique igual em qualquer editor (VS Code, WebStorm, etc.). Normalmente é um arquivo simples chamado `.editorconfig`.

Ao instalar a extensão “EditorConfig” no VS Code, um arquivo `.editorconfig` pode ser criado através do menu da barra lateral do Explorador de Arquivos, clicando com o botão direito do mouse no espaço em branco da pasta onde você deseja criá-lo e selecionando `Generate .editorconfig`.

```
# EditorConfig is awesome: https://EditorConfig.org

# top-most EditorConfig file
root = true

[*]
indent_style = space
indent_size = 4
end_of_line = crlf
charset = utf-8
trim_trailing_whitespace = true // tira os espaços em branco no final da linha
insert_final_newline = true // adiciona uma linha ao final do arquivo
```

**Criando controller de contatos e aprimorando rotas**

Um controller serve para centralizar toda a regra de negócio que está relacionada a uma entidade da aplicação. Criamos o arquivo `ContactController.js`, que fica no caminho:

mycontacts/src/app/controllers/ContactController.js

```jsx
mycontacts/
└─ src/
├─ app/
│  └─ controllers/
│     └─ ContactController.js
├─ routes.js
└─ index.js
```

```jsx
// mycontacts/src/app/controllers/ContactController.js
class ContactController {
    index() {
        // listar todos os registros (GET /contacts)
    }
    show() {
        // obter um registro específico (GET /contacts/:id)
    }
    store() {
        // criar um novo registro (POST /contacts)
    }
    update() {
        // atualizar (editar) um registro (PUT /contacts/:id)
    }
    delete() {
        // deletar um registro (DELETE /contacts/:id)
    }
}

module.exports = new ContactController();
// design pattern singleton = cada classe deve ter uma única instância. Ao exportar uma instância de ContactController, sempre teremos a mesma instância. Isso garante que o Node.js mantenha apenas uma cópia dessa classe na memória.
```

Agora, precisamos atualizar a rota. Para isso, dentro de `src`, criamos o arquivo `routes.js`. As rotas do `index.js` serão movidas para esse arquivo.

```jsx
// mycontacts/src/index.js
const express = require("express"); // importa o express

const app = express(); // instancia o express e cria uma aplicação

app.get("/", (request, response) => {
  // cria uma rota GET
  response.send("Hello world!"); // envia uma resposta
});

app.listen(3000, () => {
  console.log("Server rodando na porta 3000, http://localhost:3000");
}); // escuta as requisições
```

```jsx
// mycontacts/src/routes.js
const { Router } = require("express");

const router = Router();

router.get("/", (request, response) => {
  // cria uma rota GET
  response.send("Hello world!"); // envia uma resposta
});

module.exports = router;
```

Agora, no index.js, precisamos importar o arquivo de rotas:

```jsx
// mycontacts/src/index.js
const express = require("express"); // importa o express

const routes = require("./routes"); // importa as rotas

const app = express(); // instancia o express e cria uma aplicação
app.use(routes); // usa as rotas

app.listen(3000, () => {
  console.log("Server rodando na porta 3000, http://localhost:3000");
}); // escuta as requisições
```

Dessa forma, em ver de executar diretamente a rota (endpoint) GET /, que exibia “Hello world”, podemos executar qualquer método do controller. Para isso, no arquivo `routes.js`, devemos mudar o nome do endpoint e importar `ContactController.js`. No lugar da função que responde “Hello world”, podemos apenas apagá-la e, em vez disso, chamar a função que queremos (`ContactController.index`).

```jsx
// mycontacts/src/routes.js
const { Router } = require("express");

const ContactController = require("./app/controllers/ContactController");

const router = Router();

router.get("/contacts", ContactController.index);

module.exports = router;
```

- Teste de endpoints: extensão Thunder Client no VS Code.

Implementando o método `ContactController.index`:

Como ainda não temos banco de dados, usamos uma resposta simples:

```jsx
// mycontacts/src/app/controllers/ContactController.js
class ContactController {
    index(request, response) {
        // listar todos os registros (GET /contacts)
        response.send("Listagem de contatos: contato 1, contato 2.");
    }
    show() {
        // obter um registro específico (GET /contacts/:id)
    }
    store() {
        // criar um novo registro (POST /contacts)
    }
    update() {
        // atualizar (editar) um registro (PUT /contacts/:id)
    }
    delete() {
        // deletar um registro (DELETE /contacts/:id)
    }
}

module.exports = new ContactController();
```

**Fluxo da Requisição**

1. Chegada (`index.js`): O servidor recebe a requisição.
2. Roteamento (`routes.js`): O servidor olha para a URL (`/contacts`) e descobre quem deve tratar aquilo.
3. Processamento (`ContactController.js`): O método `index` é executado.

Este artigo continua [aqui]({filename}aprofundando-em-node.md).