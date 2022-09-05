# Lesson 1: A Hello World for Node.js and Express

We're going to build a basic Node.js server that uses Express for the
API.

## Initialize a new node project

The first step with Node.js is to initialize a new project. You do
this with `npm init`.

```
mkdir lesson1
cd lesson1
npm init
```

This will ask you a number of questions. Here is how I answered them:

```
package name: (lesson1)
version: (1.0.0)
description: hello world
entry point: (index.js) server.js
test command:
git repository:
keywords:
author:
license: (ISC)
```

Now you need to install Express. This will automatically save express
as a dependency in `package.json`:

```
npm install express
```

**Tip**: when putting code into a repository that uses node.js, be
  sure to create a file called `.gitignore` in the top level of your
  repository that contains:

```
node_modules
```

This will prevent you from adding the node modules directory, which
often contains many libraries, into your git repository. Instead,
because you have a `package.json`, anyone can use `npm install` to
install all the dependencies for your project. To see how this works,
you can always try cloning your repository to a new directory and do
`npm install` there.

## Hello World

We're now going to build a basic node server with Express. Create a
file called `server.js` that contains:

```
const express = require('express');
const app = express();

app.get('/', (req, res) => {
  res.send('Hello World!');
});

app.listen(3000, () => console.log('Server listening on port 3000!'));
```

When you run `node server.js`, your server will run and listen on port
3000. You can visit it in a browser at `<YourAWShost>:3000`.  
You can also test to see if your API is working by opening another terminal and entering ```curl localhost:3000```


In this example, we are using `require` to include the Express
module. A good explanation of how require works is at this article
called [Requiring modules in Node.js: Everything you need to
know](https://medium.freecodecamp.org/requiring-modules-in-node-js-everything-you-need-to-know-e7fbd119be8).

`express()` is the top-level function exported by Express. There are a
number of functions defined on this in the [API
documentation](https://expressjs.com/en/4x/api.html).

The next section defines what the server will do when it receives a
GET request for `/`. It defines a function that takes request and
response objects, then just sends a string response. This is what the
browser displays when it visits the root of the server.

Finally, the last line starts a web server that listens on port 3000
for incoming requests.

You can run this code with Node.js:

```
node server.js
```

You can stop it by entering `control-c`.

## REST API

When you define an API, you can technically follow any rules you would
like. However, many people (loosely, or sometimes strictly) follow a
design known as REST. If you would like to learn more, this is a good
[REST API
Tutorial](https://www.vinaysahni.com/best-practices-for-a-pragmatic-restful-api).

For now, we'll just introduce the convention used for the HTTP
methods. Typically these methods are used in these situations:

Method   |   Purpose
---------|--------------------
POST     |   Create an object
GET      |   Read an object
PUT	 |   Update an object
DELETE	 |   Delete an object

Notice that if you read down, the letters spell CRUD. This is a common
acronym for APIs and databases, representing the four basic operations
you can perform.

Add these lines to `server.js`, just before the call to `listen`:

```
app.post('/', (req, res) => {
  res.send('Here is the response to your POST, man!\n');
});

app.put('/', (req, res) => {
  res.send('I am updated.\n');
});

app.delete('/', (req, res) => {
  res.send('All my memories have been deleted. Are you happy now?\n');
});
```

These just illustrate the different methods. Normally, the methods
would access a database to create, update, or delete particular items
in the database.

You can access these URLs using curl:

```
curl -X GET localhost:3000/
curl -X POST localhost:3000/
curl -X PUT localhost:3000/
curl -X DELETE localhost:3000/
```

## Naming your resources

We can add any kind of path for the API. For example, add these lines
to `server.js`, just before the call to `listen`:

```
app.get('/secret', (req, res) => {
  res.send('Psst. You are being watched.\n');
});

app.get('/api/user/1', (req, res) => {
  res.send({
    name: "Amy Caprietti",
    avatar: "/avatars/supergirl.jpg",
    role: "admin"
  });
});
```
Test with:

```
curl -X GET localhost:3000/secret
curl -X GET localhost:3000/api/user/1
```

Notice that we can return JSON responses.

When it comes to designing a RESTful API, there are a number of [rules
... er, guidelines ... that people
follow](https://www.vinaysahni.com/best-practices-for-a-pragmatic-restful-api).

![guidelines](images/guidelines.jpg)

For example, if you build a web site for an art museum, your server
may need an API that does some of the following:

Route | HTTP method | Description
------| ------------|---------------
/api/art | GET | get all the art pieces held by the museum
/api/art | POST | create a new art piece held by the museum
/api/art/:id | GET | get a single art piece
/api/art/:id | PUT | update the information about a single art piece
/api/art/:id | DELETE | delete the information about a single art piece

Here is an example of how to use these routes:

```
GET /art - Retrieves a list of all the art pieces
GET /art/12 - Retrieves art piece #12
POST /art - Creates a new art piece
PUT /art/12 - Updates art piece #12
DELETE /art/12 - Deletes art piece #12
```
[Next Lesson](https://github.com/BYU-CS-260/learning-node-express/blob/master/lesson2/README.md)
