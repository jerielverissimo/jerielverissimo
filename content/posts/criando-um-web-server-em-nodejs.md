---
title: "Criando um web server em Nodejs"
date: 2021-08-29T03:03:41-03:00
draft: false
---

## Introdução

Quando você visualiza uma página web no seu browser, seu computador está fazendo uma requisição para um outro computador (🌈 Internet), a resposta dessa requisição normalmente é uma página HTML.
Na real as coisas não são tão simples assim, tem bastante coisa que rola no meio do caminho para isso funcionar, mas normalmente quem devolve essa resposta para seu browser é um web server.
Caso queira se aprofundar tem esse
**[artigo](https://developer.mozilla.org/pt-BR/docs/Learn/Getting_started_with_the_web/How_the_Web_works)**
como um bom ponto de partida que explica como a web funciona.

## Web server

E é sobre isso que vamos falar hoje neste post, vamos criar um servidor web bem simples usando o ***Nodejs***.

A comunicação entre seu browser e o *servidor* (quem responde suas requisições 😉), é feita por meio do protocolo ***HTTP***,
  e o nodejs já traz em sua biblioteca padrão um módulo chamado ***http*** que adivinha o que ele faz? Isso mesmo, abstrair pra gente as funcionalidades deste protocolo.

### Primeiro passo -- Criando um servidor HTTP básico

  Vamos começar criando uma pasta para nosso projeto e dentro dela crie um arquivo js que será usado para criar o servidor.

  ```bash
  $ mkdir basic-server
  $ cd basic-server
  $ touch server.js
  ```

  Abra o arquivo server.js que a gente acabou de criar com seu editor preferido e começe importando o módulo http.

  ```js
  const http = require('http');
  ```

  Para nosso web server funcionar precisamos falar para ele ficar escutando de algum lugar as requisições.
  Então passamos para ele um endereço e uma porta, que nesse caso podemos usar o endereço ***localhost***  e a porta ***8080***
  já que vamos rodar em nossa própria máquina.

  Usamos localhost porque é um apelido para nosso próprio computador, poderiamos usar o IP 127.0.0.1 que é a tradução
  de localhost, caso queira entender melhor como o ***DNS*** funciona tem esse
  ***[artigo](https://developer.mozilla.org/pt-BR/docs/Learn/Common_questions/What_is_a_domain_name)***
  que pode lhe ajudar.


  Assim toda requisição que chegar nesse endereço e porta, nosso Sistema Operacional magicamente irá redirecionar
  para nosso web server e irá devolver a resposta do nosso web server para quem fez a requisição.

  Bom voltando para o nosso arquivo server.js, cria logo abaixo da importação do módulo http o endereço e porta para usarmos depois.

  ```js
  ...
  const host = 'localhost';
  const port = 8080;
  ```
  Vamos adicionar uma função que será responsável por gerenciar as requisições que nosso servidor for recebendo.
  Essa função recebe como argumentos dois objetos, um de ***request*** e outro de ***response***, que podemos chamar elas de ***req*** e ***res***.
  Como no nosso caso queremos devolver uma resposta, vamos utilizar o objeto de *response*.

  ```js
  ...
  const listener = function (req, res) {
    res.writeHead(200);
    res.end("Hello, from my server :)");
  };
```

No protocolo HTTP quando fazemos uma requisição obtemos uma resposta com um status, que indica como essa requisição
foi tratada pelo servidor. Vamos focar por enquanto em uma requisição do tipo ***GET***.

Com o método ***writeHead*** passamos o status da resposta, que nesse caso é 200 que vai indicar que tudo ocorreu com sucesso.
Já o método ***end*** devolve de fato a resposta HTTP com tudo o que passamos pra ele.

Apenas com essa função não conseguimos fazer muita coisa, quem de fato vai fazer a mágica é a lib http, basta a gente
criar um servidor usando a lib passando nossa função que acabamos de criar como argumento, que ela será usada toda vez que esse servidor
receber uma requisição.

```js
...
const server = http.createServer(listener);
server.listen(port, host, () => {
  console.log(`Server is running on http://${host}:${port}`);
});
```

O que o método ***listen*** faz é atrelar o servidor que foi criado a um endereço e porta,
  ele recebe também uma *callback* que é executada quando tudo está pronto.

  Bom com isso basta abrir um terminal e rodar nosso script pra subir nosso servidorzinho.


  ```bash
  $ node server.js
  ```

  Você vai ver o log da callback de sucesso que passamos para o método listen indicando que nosso servidor
  está pronto para receber requisições.

  ```
  Server is running on http://localhost:8080
  ```

  Para testarmos basta fazer uma requisição **GET** usando o ***cURL***, caso prefira pode abrir o mesmo endereço no seu browser.

  ```bash
  $ curl http://localhost:8080
  ```
  E você vai ver a mensagem que nosso servidorzinho enviou 😎.

  ```
  Hello, from my server :)
  ```

### Segundo passo -- Servindo uma página HTML

  Só com isso não é suficiente para criarmos uma aplicação mais complexa, podemos por exemplo querer enviar uma página **HTML**
  para o browser.

  Para isso usamos os cabeçalhos, que sã́o atributos que passamos nas requests/responses no protocolo HTTP.
  Usando nosso exemplo, vamos alterar a função listener que criamos para adicionar na resposta um ***header***  que vai indicar
  que estamos enviando como resposta um arquivo HTML, e mudamos também o conteúdo para um html válido.

  ```js
  ...
  const listener = function (req, res) {
    res.setHeader('Content-Type', 'text/html');
    res.writeHead(200);
    res.end("<html><body><h1>This is an HTML file</h1></body></html>");
  };
...
```

Agora se você subir de novo o servidor e abrir o endereço no browser irá visualizar a página HTML que enviamos, da mesma forma via terminal.

```bash
$ curl http://localhost:8080
<html><body><h1>This is an HTML file</h1></body></html>
```

### Terceiro passo -- Servindo um JSON

Um outro formato que é bem comum é o **JSON**, que é bastante utilizado quando se envolve ***APIs***.
Caso queira suportar esse formato basta mudar novamente o listener para:

```js
...
const listener = function (req, res) {
  res.setHeader('Content-Type', 'application/json');
  res.writeHead(200);
  res.end('{ "message": "valid JSON object" }');
};
...
```
E novamente...
```bash
$ curl http://localhost:8080
{ "message": "valid JSON object" }
```

### Quarto passo -- Recebendo um POST

Até agora lidamos apenas com requisições do tipo **GET**, outro tipo que é bastante comum são as do tipo **POST**.
Elas dão um pouco mais de trabalho para usarmos, mas não é nenhuma coisa de outro mundo.

O módulo http do nodejs usa as ***Streams*** para gerenciar as requests e responses, ou seja tudo é feito em um fluxo
contínuo de forma **assíncrona** para que, por exemplo, uma requisição que demore mais para ser respondida não bloqueia o servidor de receber
outras requisições que vão chegando ao mesmo tempo.

Por isso uma requisição por exemplo não chega pronta com tudo de uma vez, ela chega em pedaços, precisamos ir *"montando"* ela
primeiro como se fosse peças de um ***Lego***, antes de conseguir usar ela.

Já que uma requisição do tipo **POST** possui um ***body*** como conteúdo, vamos precisar primeiro montar esse body à medida que ele vai chegando.

```js
...
const listener = function (req, res) {
  let body = ''; // buffer para salvar os dados da requisição
  req.on('data', (chunk) => {
    body += chunk; // os dados vão chegando em pedaços e não tudo de uma vez
  });
  ...
};
...
```

Criamos uma variável chamada *body* para receber o conteúdo da requisição, e passamos ela na callback para ir incrementando seu conteúdo.
Essa callback a gente passa no evento **data**, que é etapa da requisição que recebe um pedaço do seu conteúdo.

Então toda vez que chegar um pedacinho dos dados, um evento do tipo ***data*** é emitido e essa callback que passamos é executada.

```js
...
const listener = function (req, res) {
  let body = ''; // buffer para salvar os dados da requisição
  req.on('data', (chunk) => {
    body += chunk; // os dados vão chegando em pedaços e não tudo de uma vez
  });

  // quando todos os pedaços da requisição chegaram :)
  req.on('end', () => {
    console.log(body); // mostra o conteúdo que recebemos da requisição
    res.writeHead(200, { 'Content-Type': 'application/json' });
    res.end('Request received'); // devolve resposta para o cliente
  });
};
...
```

Quando não tem mais dados para receber da requisição, ou seja já recebemos todos os "pedaços" dela e conseguimos montar o "lego",
um evento do tipo **end** é emitido e com isso passamos pra ele uma callback
que nesse caso vamos apenas printar o conteúdo que recebemos que é o body da requisição **POST**
e devolver uma resposta para o cliente.

Vamos testar mandando um json para o servidor.

```bash
$ curl --location --request POST 'localhost:8080' \
--header 'Content-Type: application/json' \
--data-raw '{ "message": "JSON from client." }'

Request received
```

E o servidor vai printar o conteúdo da requisição.

```bash
$ node server.js
{ "message": "JSON from client." }
```


### Quinto passo -- Lidando com rotas

Por último, quando lidamos com um servidor web é bem comum fazermos requisições via **rotas** assim podemos dividir nosso
servidor em partes menores, e cada rota ser responsável por uma funcionalide.

Por exemplo podemos ter uma rota chamada **/add** para adicionar um usuário e outra rota **/users** para listar os usuários adicionados.
A rota /add vai ser uma rota do tipo **POST**  e rota /users vai ser uma rota do tipo **GET**.

Vamos mover todo conteúdo da nossa função listener para uma outra chamada de post.

```js
...
const post = function (req, res) {
  let body = ''; // buffer para salvar os dados da requisição
  req.on('data', (chunk) => {
    body += chunk; // os dados vão chegando em pedaços e não tudo de uma vez
  });

  // quando todos os pedaços da requisição chegaram :)
  req.on('end', () => {
    console.log(body); // mostra o conteúdo que recebemos da requisição
    res.writeHead(200, { 'Content-Type': 'application/json' });
    res.end('Request received'); // devolve resposta para o cliente
  });
};
...
```
```js
...
const listener = function (req, res) {};
...
```

Agora no nosso listener, vamos olhar para a url da requisição para decidirmos as rotas.
Caso a url seja diferente da que estamos esperando, devolvemos um ***404***.

Na rota /add chamamos a função post que criamos para tratar a requisição desta rota.

```js
...
const listener = function (req, res) {
  switch (req.url) {
    case '/add':
      post(req, res);
      break;
    case '/users':
      break;
    default:
      res.setHeader('Content-Type', 'application/json');
      res.writeHead(404);
      res.end(JSON.stringify({ error: 'Resource not found' }));
  }
};
...
```

Para adicionar um usuário vamos simplesmente adicionar o conteúdo da request em array, então nossa função *post vai* fica assim:

```js
...
const users = [];

const post = function (req, res) {
  let body = ''; // buffer para salvar os dados da requisição
  req.on('data', (chunk) => {
    body += chunk; // os dados vão chegando em pedaços e não tudo de uma vez
  });

  // quando todos os pedaços da requisição chegaram :)
  req.on('end', () => {
    const user = JSON.parse(body);
    users.push(user);

    // também podemos adicionar ao mesmo tempo o status e os headers
    res.writeHead(200, { 'Content-Type': 'application/json' });
    res.end('User added'); // devolve resposta para o cliente
  });
};
...
```

Agora para listar os usuários adicionados simplesmente criamos uma função chamada get que devolve o array de usuários.

```js
...
const get = function (req, res) {
  res.setHeader('Content-Type', 'application/json');
  res.writeHead(200);
  res.end(JSON.stringify(users));
};
...
```
E por fim adicionamos a chamada desse get na rota de /users no nosso listener.

```js
...
const listener = function (req, res) {
  switch (req.url) {
    case '/add':
      post(req, res);
      break;
    case '/users':
      get(req, res);
      break;
    default:
      res.setHeader('Content-Type', 'application/json');
      res.writeHead(404);
      res.end(JSON.stringify({ error: 'Resource not found' }));
  }
};
...
```

Assim conseguimos adicionar usuários.

```bash
$ curl --location --request POST 'localhost:8080/add' \
--header 'Content-Type: application/json' \
--data-raw '{ "name": "João" }'

User added
$ curl --location --request POST 'localhost:8080/add' \
--header 'Content-Type: application/json' \
--data-raw '{ "name": "Maria" }'

User added
```

E visualizar os usuários adicionados.

```bash
$ curl --location --request GET 'localhost:8080/users'

[{"name":"João"},{"name":"Maria"}]
```
E assim terminamos esse longo artigo, espero que tenha aprendido algo com ele, caso queira ver o arquivo final criei este
[gist](https://gist.github.com/jerielverissimo/c84d24fb66bf813d056d8cef0aff270e).

Claro que com isso que vimos hoje, talvez você não sai criando uma aplicação grande,
já que é um pouco mais "baixo" nível, e normalmente usamos um framework para fazer esse tipo tarefas,
porém com esse conteúdo podemos tentar criar nosso mini-framework ao estilo express. que adivinha será nosso próximo desafio hehe.



#### Referências

[How To Create a Web Server in Node.js with the HTTP Module](https://www.digitalocean.com/community/tutorials/how-to-create-a-web-server-in-node-js-with-the-http-module)

[NodeJS server receiving POST request](https://stackoverflow.com/questions/51071321/nodejs-server-receiving-post-request)

[Node.js - Streams](https://www.tutorialspoint.com/nodejs/nodejs_streams.htm)





