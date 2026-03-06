Title: ✧ Aprofundando em Node.js e Express
Date: 2026-03-06
Category: Tutoriais
Tags: node.js, back-end, express
Author: Amanda Meneses

**Entendendo o Repository Pattern**

Continuando o artigo anterior ([Primeiros passos com Node.js e Express]({filename}primeiros-passos-com-Node.md)), note que colocamos os dados dentro do Controller. Isso viola um pouco a separação de responsabilidades. O Controller deveria apenas gerenciar a regra, não guardar os dados. Em vez disso, ele deve pedir os dados.

O Repository Pattern é um padrão de projeto que cria uma camada intermediária chamada **Repository**, que é responsável **apenas por acessar os dados** e que atua como uma camada de abstração entre a lógica de negócios (controller) de uma aplicação e a fonte de dados (banco de dados, seja ela um array, SQL, NoSQL, .json, API). Ele serve de mediador entre as camadas de domínio e de mapeamento de dados, permitindo um código mais limpo ao separar a lógica de acesso a dados em uma classe centralizada e especializada. Isso promove a testabilidade, reduz a duplicação e isola os objetos de domínio.

Antes: controller ↔ data source

Repository Pattern: controller ↔ repository ↔ data source

Dessa forma, por exemplo, se começamos um projeto utilizados um mock de dados e futuramente trocamos para um banco SQL, não precisaremos mexer no controller onde estão as regras de negócio, apenas refatoramos o repository.

**Criando o Repository de Contatos**

Vamos começar trabalhando com um mock, para não ter que ir para SQL agora e para vermos a real utilidade do repository pattern, pois posteriormente vamos trabalhar com Postgres.

Para o mock de dados, em vez de números sequenciais para os IDs, usaremos hash por meio de um padrão chamado Universal Unique ID, instalando ele com o comando:

```bash
yarn add uuidv4
```

Dentro de `app`, criaremos a pasta `repositories` com o arquivo `ContactsRepository.js`. Assim como criamos um controller por entidade, criaremos um repositorie com entidade.

```jsx
// mycontacts/src/app/repositories/ContactsRepository.js
const { uuid } = require("uuidv4");

const contacts = [
    {
        id: uuid(),
        name: "Amanda",
        email: "amanda@email.com",
        phone: "123123123",
        category_id: uuid()
    }
]

class ContactsRepository {
    findAll() {
        return contacts;
    }
}

module.exports = new ContactsRepository();
```

- `contacts` → simula um banco de dados
- `findAll()` → retorna todos os contatos
- o controller não acessa o array diretamente

No `ContactController.js`, precisamos importar o `ContactsRepository.js`:

```jsx
// mycontacts/src/app/controllers/ContactController.js
const ContactsRepository = require("../repositories/ContactsRepository");

class ContactController {
    index(request, response) {
        const contacts = ContactsRepository.findAll();
        response.json(contacts);
    }

    show() {
    }

    store() {
    }

    update() {
    }

    delete() {
    }
}

module.exports = new ContactController();
```

**Assincronismo**

Atualmente, nosso mock é um Array na memória, então o acesso é instantâneo. Mas, quando conectarmos no Postgres, o acesso ao banco de dados demora alguns milissegundos. No JavaScript, tudo que demora (I/O) é **Assíncrono** (`Promise`).

*Síncrono* ou *assíncrono* diz respeito ao fluxo de execução de um programa. Quando uma operação executa **completamente** antes de passar o controle à seguinte, a execução é *síncrona*. Esse é o método padrão de execução de código.

Quando uma ou mais operações são demoradas, pode ser interessante executá-las de maneira *assíncrona*, para que o restante do código possa ser executado sem precisar esperar que elas terminem.

<aside>

I/O Assíncrono (Entrada/Saída Assíncrona ou Asynchronous Input/Output) é uma técnica de computação que permite que um programa inicie uma operação de leitura ou escrita (como ler um arquivo, fazer uma requisição de rede ou acessar um banco de dados) e continue executando outras tarefas sem esperar que essa operação termine.

Diferente do I/O síncrono (bloqueante), onde o programa "congela" e aguarda a conclusão da tarefa, o I/O assíncrono (não bloqueante) libera a CPU para realizar trabalhos paralelos, retornando o resultado da operação apenas quando ela for finalizada.

</aside>

O Assincronismo não bloqueante utiliza mecanismos como `async/await`, `Promises`, `futures` ou *callbacks* para tratar a resposta no futuro sem travar a thread.

A consulta no banco de dados é uma operação assíncrona. Se deixarmos o código do jeito que está agora (síncrono), quando colocar o banco de dados, o código vai quebrar. Por isso, usaremos `async/await` desde agora.

**1. Ajustando o Repository (`ContactsRepository.js`)**

```jsx
// mycontacts/src/app/repositories/ContactsRepository.js
const { uuid } = require("uuidv4");

const contacts = [
    {
        id: uuid(),
        name: "Amanda",
        email: "amanda@email.com",
        phone: "123123123",
        category_id: uuid()
    }
]

class ContactsRepository {
    async findAll() {
        return new Promise((resolve) => resolve(contacts));
    }
}

module.exports = new ContactsRepository();
```

Mesmo retornando um array simples instantaneamente, vamos retornar uma Promise. Isso cria um "contrato" de que essa operação pode demorar. Dessa forma, definimos que qualquer chamada a findAll() deve ser tratada como algo assíncrono.

**2. Ajustando o Controller (`ContactController.js`)**

Agora o controller precisa saber esperar (`await`) a Promise resolver antes de continuar.

```jsx
// mycontacts/src/app/controllers/ContactController.js
const ContactsRepository = require("../repositories/ContactsRepository");

class ContactController {
    async index(request, response) {
        const contacts = await ContactsRepository.findAll();
        response.json(contacts);
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

| **Promise** | Um valor que virá no futuro (caixa) |
| --- | --- |
| **async** | Marca uma função como assíncrona (permite usar a caixa) |
| **await** | Espera a Promise resolver (abre a caixa quando ela estiver pronta) |

**Próximo passo: Criando as rotas de show e delete**

O método `show` é responsável por buscar **um único registro** baseado no ID passado na URL.

**1. Rota (`routes.js`)**

Primeiro, precisamos ensinar o Express a receber um parâmetro dinâmico na URL (o `:id`).

```jsx
// mycontacts/src/routes.js
const { Router } = require("express");
const ContactController = require("./app/controllers/ContactController");

const router = Router();

router.get("/contacts", ContactController.index);
router.get("/contacts/:id", ContactController.show); // nova rota com método GET para buscar um contato pelo ID

module.exports = router;
```

**2. O Controller (`ContactController.js`)**

O Controller vai pegar o ID da URL (`request.params`), pedir para o Repository buscar e tratar o erro caso o contato não exista (retornando status 404).

```jsx
// mycontacts/src/app/controllers/ContactController.js
const ContactsRepository = require("../repositories/ContactsRepository");

class ContactController {
    async index(request, response) {
        const contacts = await ContactsRepository.findAll();
        response.json(contacts);
    }

    async show(request, response) {
        const { id } = request.params; // desestrutura (extrai) o ID da URL
        const contact = await ContactsRepository.findById(id);

        if (!contact) {
            return response.status(404).json({ error: "Contato não encontrado" }); // se não encontrou, retorna um erro e encerra a requisição
        }

        response.json(contact); // se encontrou, retorna o contato
    }

    store() {}

    update() {}

    delete() {}
}

module.exports = new ContactController();
```

**3. O Repository (`ContactsRepository.js`)**

Precisamos criar o método `findById(id)`, que vai vasculhar nosso *mock* e encontrar o contato com o ID específico. Ele também retornará uma `Promise`.

```jsx
// mycontacts/src/app/repositories/ContactsRepository.js
const { v4 } = require("uuid");

const contacts = [
    {
        id: v4(),
        name: "Amanda",
        email: "amanda@email.com",
        phone: "123123123",
        category_id: v4()
    },
    {
        id: v4(),
        name: "Ranilo",
        email: "ranilo@email.com",
        phone: "123123123",
        category_id: v4()
    }
]

class ContactsRepository {
    async findAll() {
        return new Promise((resolve) => resolve(contacts));
    }

    async findById(id) {
        return new Promise((resolve) => resolve(
            contacts.find((contact) => contact.id === id)
        ));
    }
}

module.exports = new ContactsRepository();
```

**Como testar (navegador ou thunder client):**

1. Rode a aplicação (`yarn dev`).
2. Acesse `http://localhost:3000/contacts` no navegador para ver a lista e **copie o ID** que foi gerado pelo `uuid()`.
3. Acesse `http://localhost:3000/contacts/COLE_O_ID_AQUI` para ver o método `show` funcionando.
4. Tente acessar `http://localhost:3000/contacts/123` para ver a mensagem de erro.

<aside>

**Nota**

Quando colocamos a palavra `async` na frente de uma função, o JavaScript **automaticamente embrulha o seu retorno dentro de uma Promise resolvida**. Como já declaramos o método como `async`, **não precisamos criar manualmente uma Promise.** Uma função `async` já retorna automaticamente uma Promise.

```jsx
// mycontacts/src/app/repositories/ContactsRepository.js
class ContactsRepository {
    async findAll() {
        return contacts;
    }

    async findById(id) {
        return contacts.find((contact) => contact.id === id);
    }
}
```

</aside>

**Criando a rota delete**

O método `delete` é responsável por deletar **um único registro** baseado no ID passado na URL.

**1. Rota (`routes.js`)**

Primeiramente, ensinamos o roteador a ouvir o verbo HTTP correto (`DELETE`) no mesmo endereço.

```jsx
// mycontacts/src/routes.js
const { Router } = require("express");
const ContactController = require("./app/controllers/ContactController");

const router = Router();

router.get("/contacts", ContactController.index);
router.get("/contacts/:id", ContactController.show);
router.delete("/contacts/:id", ContactController.delete);

module.exports = router;
```

**2. O Controller (`ContactController.js`)**

```jsx
// mycontacts/src/app/controllers/ContactController.js
const ContactsRepository = require("../repositories/ContactsRepository");

class ContactController {
    (...)

    async delete(request, response) {
        const { id } = request.params;
        const contact = await ContactsRepository.findById(id);

        if (!contact) {
            return response.status(404).json({ error: "Contato não encontrado" });
        }

        await ContactsRepository.delete(id);
        response.sendStatus(204); // retorna status 204 (No Content: deu certo e não tem body na resposta)
    }
}

module.exports = new ContactController();
```

**3. O Repository (`ContactsRepository.js`)**

Obs.: No `ContactsRepository.js`, a lista de contatos está declarada como `const contacts = [...]`. O problema é que, para deletar um contato de um Array, a forma mais comum é usar o `.filter()` para criar uma nova lista sem aquele ID. Mas **não podemos reatribuir** um novo Array a uma variável `const`. Então, no nosso *mock* mudaremos de `const contacts` para `let contacts`.

Agora, vamos adicionar o método `delete(id)` usando o `.filter()`. Ele vai varrer a lista e manter apenas os contatos que têm o ID **diferente** do ID que queremos apagar.

```jsx
// mycontacts/src/app/repositories/ContactsRepository.js
const { v4 } = require("uuid");

let contacts = (...)

class ContactsRepository {
    async findAll() {
        return contacts;
    }

    async findById(id) {
        return contacts.find((contact) => contact.id === id);
    }

    async delete(id) {
        contacts = contacts.filter((contact) => contact.id !== id);
    }
}

module.exports = new ContactsRepository();
```

**Como testar:**

O navegador comum só faz requisições do tipo `GET`. Para testar o `DELETE`, você vai precisar do Thunder Client:

1. Faça um `GET /contacts` e copie um ID.
2. Mude o verbo HTTP na ferramenta para **DELETE**.
3. Acesse `/contacts/COLE_O_ID_AQUI` e clique em Send. Você deve receber um status **204 No Content**.
4. Faça um `GET /contacts` novamente e comprove que o contato sumiu da lista.

**O que são middlewares?**

O próximo passo é dar vida ao método `store` no Controller, que será responsável por tratar a rota `POST /contacts`, recebendo os dados enviados pelo cliente para criar um novo contato no nosso *mock*.

Para que isso funcione, precisamos acessar as informações enviadas no corpo da requisição (`request.body`). Porém, por padrão, o Express **não interpreta automaticamente o corpo das requisições no formato JSON**. Isso significa que, se tentarmos fazer um `console.log(request.body)` nesse momento, o resultado será `undefined`.

Ou seja, antes de conseguirmos criar ou editar contatos, precisamos ensinar o Express a entender os dados que estão sendo enviados pelo cliente. Para isso, utilizaremos um **middleware**, que irá interceptar a requisição e converter o JSON recebido em um objeto JavaScript acessível dentro do Controller.

Por meio de middlewares é possível manipular os objetos `request` e `response`, controlar o fluxo entre as rotas, além de aplicar recursos como autenticação, validações e logs.

Nesse momento, utilizaremos middleware para fazer gerenciamento de requisições, controlando rotas antes de chegarem ao controller. Podemos pensar nos Middlewares como uma interceptação pelo qual todas as requisições passam antes de chegar na sua rota. Dentro de um middleware podemos dizer se a request deve continuar (ir para o controller) ou se ela deve parar.

<aside>

**Entendendo o ciclo de vida de uma requisição**

Sempre que uma request chega, precisamos resolver essa requisição e após isso dar uma resposta:

request → controller → response

Os middlewares mudam um pouco esse comportamento. Quando há um middleware, antes de a request chegar no controller, ela é interceptada pelo middleware:

request → midldlewares → controller → response

</aside>

Tecnicamente, um middleware no Express é apenas uma função que tem acesso a três coisas:

- `request` (os dados que chegaram)
- `response` (os métodos para devolver a resposta)
- **`next`** (função que manda continuar)

Todo middleware tem esse formato:

```jsx
function middleware(request, response, next) {
    // faz alguma coisa

    next(); // libera para continuar
}
```

Dentro de um middleware, temos apenas **duas escolhas**:

- **encerrar a requisição:** enviando um `response.send()` ou `response.json()` (ex: um middleware de segurança que barra um usuário sem senha).
- **passar para frente:** chamando a função `next()`.

No nosso caso, por padrão, quando um cliente envia um pacote de dados (como os dados de um novo contato), os dados são incompreensíveis para o JavaScript. Quando usamos o `app.use(express.json())`, estamos colocando um interceptador no início do processo. Para utilizar esse middleware, temos que ir no arquivo principal (`index.js`) e adicionar esta linha:

```jsx
// mycontacts/src/index.js
const express = require("express");
const routes = require("./routes");

const app = express();
app.use(express.json()); // middleware global que transforma o body da requisição em json
app.use(routes);

app.listen(3000, () => {
  console.log("Server rodando na porta 3000, http://localhost:3000");
});
```

**Atenção:** Ela deve vir *antes* de `app.use(routes)`, pois a requisição precisa ser transformada em JSON antes de chegar no Controller.

**Criando a rota de cadastro**

O método `store` é responsável por criar um novo contato a partir dos dados enviados no corpo da requisição (`request.body`).

**1. Rota (`routes.js`)**

Primeiramente, adicionamos uma nova rota utilizando o verbo HTTP `POST`, que é o verbo padrão para criação de recursos:

```jsx
// mycontacts/src/routes.js
const { Router } = require("express");
const ContactController = require("./app/controllers/ContactController");

const router = Router();

router.get("/contacts", ContactController.index);
router.get("/contacts/:id", ContactController.show);
router.delete("/contacts/:id", ContactController.delete);
router.post("/contacts", ContactController.store);

module.exports = router;
```

**2. O Controller (`ContactController.js`)**

Aqui nós pegamos o corpo da requisição (`request.body`) e aplicamos as nossas **regras de negócio**.

Definimos duas regras principais:

1. O campo `name` é obrigatório (Validação).
2. O `email` não pode se repetir (Regra de negócio).

```jsx
// mycontacts/src/app/controllers/ContactController.js
const ContactsRepository = require("../repositories/ContactsRepository");

class ContactController {
		(...)

    async store(request, response) {
        const { name, email, phone, category_id } = request.body; // desestrutura (extrai) os dados do corpo da requisição
        
        // Validação
        if (!name) {
            return response.status(400).json({ error: "O nome é obrigatório" }); // retorna status 400 (Bad Request: deu erro e nao tem body na resposta)
        }

        // Regra de negócio: O e-mail já existe?
        const contactExists = await ContactsRepository.findByEmail(email); // busca um contato pelo e-mail

        if (contactExists) {
            return response.status(400).json({ error: "Esse e-mail já está cadastrado" }); // retorna status 400 (Bad Request: deu erro e nao tem body na resposta)
        }

        const contact = await ContactsRepository.create({
            name, email, phone, category_id,
        }) // se passou pela validação, chama o método create do ContactsRepository e passa os dados

        response.json(contact); // retorna o contato criado
    } 
}

module.exports = new ContactController();
```

**3. O Repository (`ContactsRepository.js`)**

Nosso Controller chamou dois métodos novos: `findByEmail` e `create`. Precisamos criá-los no Repository para interagir com o nosso *mock*.

```jsx
// mycontacts/src/app/repositories/ContactsRepository.js
const { v4 } = require("uuid");

let contacts = (...)

class ContactsRepository {
    async findAll() {
        return contacts;
    }

    async findById(id) {
        return contacts.find((contact) => contact.id === id);
    }

    async findByEmail(email) {
        return contacts.find((contact) => contact.email === email);
    } // busca um contato pelo e-mail

    async create({ name, email, phone, category_id }) {
        const newContact = { id: v4(), name, email, phone, category_id };
        contacts.push(newContact); // salva no "banco de dados" (mock)
        return newContact; // retorna o objeto criado para o Controller
    }

    async delete(id) {
        contacts = contacts.filter((contact) => contact.id !== id);
    }
}

module.exports = new ContactsRepository();
```

**Como testar (Thunder Client):**

- Método: **POST**
- URL: `http://localhost:3000/contacts`
- Vá na aba **Body** e selecione a opção **JSON**.
- Cole o objeto JSON com os dados e clique em **Send**.

```json
{
  "name": "Menelau",
  "email": "menelau@email.com",
  "phone": "999999999",
  "category_id": "categoria"
}
```

- Para verificar o contato criado, mude o método para GET (ainda em /contacts) e clique em Send.
- Ao tentar cadastrar novamente o mesmo contato, deve aparecer a seguinte resposta:

```json
{
  "error": "Esse e-mail já está cadastrado"
}
```

**Criando a rota de edição**

O método `update` é responsável por atualizar um contato existente com base no ID informado na URL. Para isso, utilizamos o verbo HTTP `PUT`, que representa a substituição completa de um recurso.

- `PUT` = substituição completa do recurso
- `PATCH` = atualização parcial

**1. Rota (`routes.js`)**

```jsx
// mycontacts/src/routes.js
const { Router } = require("express");
const ContactController = require("./app/controllers/ContactController");

const router = Router();

router.get("/contacts", ContactController.index);
router.get("/contacts/:id", ContactController.show);
router.delete("/contacts/:id", ContactController.delete);
router.post("/contacts", ContactController.store);
router.put("/contacts/:id", ContactController.update);

module.exports = router;
```

**2. O Controller (`ContactController.js`)**

No Controller, precisamos ter muito cuidado com a regra de negócio do e-mail. Quando atualizamos um contato, três cenários podem acontecer com o e-mail:

1. O usuário mudou para um e-mail que não existe no sistema (Permitido).
2. O usuário não alterou o e-mail, ou seja, enviou o próprio e-mail atual (Permitido).
3. O usuário tentou mudar para um e-mail que **já pertence a outro contato** (Bloqueado).

Para garantir essa lógica, buscamos se o e-mail já existe no banco (`findByEmail`). Se ele existir, verificamos se o dono desse e-mail tem um ID diferente do contato que estamos editando (`contactByEmail.id !== id`).

```jsx
// mycontacts/src/app/controllers/ContactController.js
const ContactsRepository = require("../repositories/ContactsRepository");

class ContactController {
    (...)
    
    async update(request, response) {
        const { id } = request.params; // pega o id da rota URL
        const { name, email, phone, category_id } = request.body; // pega os dados do corpo da requisição

        const contactExists = await ContactsRepository.findById(id); // busca o contato pelo id

        if (!contactExists) {
            return response.status(404).json({ error: "Contato não encontrado" });
        } // se o contato não existir, retorna um erro

        if (!name) {
            return response.status(400).json({ error: "O nome é obrigatório" });
        } // se o contato existir, mas o nome estiver vazio, retorna um erro

        const contactByEmail = await ContactsRepository.findByEmail(email); // busca o contato pelo e-mail

        if (contactByEmail && contactByEmail.id !== id) {
            return response.status(400).json({ error: "Esse e-mail já está cadastrado" });
        } // se o contato existir, mas o e-mail já estiver cadastrado e for diferente do contato atual, retorna um erro

        const contact = await ContactsRepository.update(id, {
            name, email, phone, category_id,
        }) // atualiza o contato

        response.json(contact); // retorna o contato atualizado
    }

    (...)
}

module.exports = new ContactController();
```

**3. O Repository (`ContactsRepository.js`)**

No Repository, recebemos o `id` e os novos dados. Como estamos usando um array na memória (`mock`), a forma mais eficiente de atualizar um único item é utilizando a função `.map()`. Ela percorre todos os contatos; se o ID do contato atual for igual ao ID que queremos editar, substituímos pelo objeto novo. Caso contrário, mantemos o contato intacto.

```jsx
// mycontacts/src/app/repositories/ContactsRepository.js
const { v4 } = require("uuid");

let contacts = (...)

class ContactsRepository {
    async findAll() {
        return contacts;
    }

    async findById(id) {
        return contacts.find((contact) => contact.id === id);
    }

    async findByEmail(email) {
        return contacts.find((contact) => contact.email === email);
    }

    async create({ name, email, phone, category_id }) {
        const newContact = { id: v4(), name, email, phone, category_id };
        contacts.push(newContact);
        return newContact;
    }

    async update(id, { name, email, phone, category_id }) {
        const updatedContact = { id, name, email, phone, category_id }; // cria um objeto com os dados atualizados
        contacts = contacts.map((contact) => contact.id === id ? updatedContact : contact); // substitui o contato atualizado no array
        return updatedContact; // retorna o objeto atualizado
    }

    async delete(id) {
        contacts = contacts.filter((contact) => contact.id !== id);
    }
}

module.exports = new ContactsRepository();
```

**Como testar (Thunder Client):**

- Método: **PUT**
- URL: `http://localhost:3000/contacts/[colar id de contato existente]`
- Body (JSON):

```json
{
  "name": "Menelau Novo",
  "email": "menelau.novo@email.com",
  "phone": "000000000",
  "category_id": "categoria"
}
```

- Se tentar colocar um ID que não existe na URL, deve retornar **404**.
- Se tentar enviar sem o nome no JSON, deve retornar **400**.
- Se enviar tudo certo, ele retorna o objeto atualizado com status **200 (OK)**.