# Lesson 2: A help ticket server

Let's build a help ticket server. We have provided a React front end for you, but you should understand what is happening in the front end so you can write your own in following assignments.

## Back end

1. The first step is to initialize a new project. You do
this with `npm init`.

```sh
mkdir lesson2
cd lesson2
mkdir back-end
cd back-end
npm init
```

Like the first lesson, we'll answer these questions:

```sh
package name: (lesson2)
version: (1.0.0)
description: help ticket server
entry point: (index.js)
test command:
git repository:
keywords:
author:
license: (ISC)
```

2. Now we need to install Express and an npm library for parsing the body of POST requests.

```sh
npm install express body-parser
```

3. Next, create a file called `ticket.js` with the following code:

```js
const express = require('express');
const bodyParser = require("body-parser");

const app = express();
app.use(bodyParser.json());
app.use(bodyParser.urlencoded({ extended: false }));
app.use(function(req, res, next) {
  res.header("Access-Control-Allow-Origin", "*");
  res.header("Access-Control-Allow-Methods", "POST, GET, DELETE");
  res.header("Access-Control-Allow-Headers", "Origin, X-Requested-With, Content-Type, Accept");
  next();
});
```

This includes the modules we're using and initializes them.  It also tells node to set the Access-Control headers so that you wont get CORS errors.

```js
app.use(express.static('public'));
```

This tells Express that it should serve any files in the `public` directory as if they were just
static files on a web server. 

4. Since we don't have a database setup yet, we're just going to store our tickets in a global variable.

```js
let tickets = [];
let id = 0;
```

5. Now create the GET route 

```js
app.get('/api/tickets', (req, res) => {
  console.log("In get");
  res.send(tickets);
});
```

This is the REST endpoint for getting all the tickets in the system. We just send our list,
which by default comes with a 200 OK response.

6. Now create a POST route to create tickets

```js
app.post('/api/tickets', (req, res) => {
  console.log("In post");
  id = id + 1;
  let ticket = {
    id: id,
    name: req.body.name,
    problem: req.body.problem
  };
  tickets.push(ticket);
  res.send(ticket);
});
```

We get the parameters from the request body,
create a new ticket, then send back the same ticket we created in a 200 OK response. We've left out
some error checking---we should check whether the request body includes the desired information.  
Lets add a console.log so we can see what it happening.

7. Now create a DELETE route to remove tickets when they have been completed

```js
app.delete('/api/tickets/:id', (req, res) => {
  console.log("In delete");
  let id = parseInt(req.params.id);
  let removeIndex = tickets.map(ticket => {
      return ticket.id;
    })
    .indexOf(id);
  if (removeIndex === -1) {
    res.status(404)
      .send("Sorry, that ticket doesn't exist");
    return;
  }
  tickets.splice(removeIndex, 1);
  res.sendStatus(200);
});
```

The ID is passed in the URL so we use a different
method to parse it. We check whether this ID is present and return a 404 error if it doesn't.
Otherwise, we remove it and return 200 OK.

8. Finally, start the server on port 3000
```js
app.listen(3000, () => console.log('Server listening on port 3000!'));
```

## Testing with curl

9. Run the server:

```sh
node tickets.js
```

Then you can test it with curl from another terminal window.  First create a entry by calling your POST API.

```sh
$ curl -d '{"name":"Daniel","problem":"Nothing works! This software is junk!"}' -H "Content-Type: application/json" -X POST localhost:3000/api/tickets
```
You should see the following response
```sh
{"id":1,"name":"Daniel","problem":"Nothing works! This software is junk!"}
```
Next call the GET API to see if it worked
```
$ curl localhost:3000/api/tickets
```
You should see the following response
```sh
[{"id":1,"name":"Daniel","problem":"Nothing works! This software is junk!"}]
```
Add another ticket with POST
```sh
$ curl -d '{"name":"Daniel","problem":"Never mind, this system is cool. It was a feature, not a bug!"}' -H "Content-Type: application/json" -X POST localhost:3000/api/tickets
```
You should see the following response
```sh
{"id":2,"name":"Daniel","problem":"Never mind, this system is cool. It was a feature, not a bug!"}
```
Now use GET to see both tickets
```sh
$ curl localhost:3000/api/tickets
```
You should see the following response
```sh
[{"id":1,"name":"Daniel","problem":"Nothing works! This software is junk!"},{"id":2,"name":"Daniel","problem":"Never mind, this system is cool. It was a feature, not a bug!"}]
```
Congratulations!  Your back end is working!  Now press ^C to stop it while we build the front end.  If you dont, some students have seen a message indicating that they have run out of memory on their Cloud9 instance.

## React front end
1. Change directory to lesson2 in a new terminal window
```sh
cd ~/environment/public_html/node/lesson2
```

2. Create a new React project
```
npx create-react-app front-end
```

3. Now go into this directory and run:

```sh
cd front-end
npm install
npm install axios
```
This will install all of the dependencies this code needs. 

4. Insert a proxy line in the "package.json" file so that requests to port 8080 to the "/api/tickets" route will be forwarded to port 3000.
```
{
  "name": "front-end",
  "version": "0.1.0",
  "private": true,
  "proxy": "http://localhost:3000",
```

5. You will also need to create a file ".env.development.local" with the following content
```
DANGEROUSLY_DISABLE_HOST_CHECK=true
```
The development web server will normally not serve files to a browser from another host, but we want it to so you can develop on Cloud9.
This ".env" file will make things work.  If you dont have this file, you will get the error "Invalid Host header" when you access your React development server.

6. Now insert the code to call the back end into src/App.js
```
import { useState, useEffect } from 'react';
import axios from 'axios';
import './App.css';

function App() {
  // setup state
  const [tickets, setTickets] = useState([]);
  const [error, setError] = useState("");
  const [name, setName] = useState("");
  const [problem, setProblem] = useState("");

  const fetchTickets = async() => {
    try {      
      const response = await axios.get("/api/tickets");
      setTickets(response.data);
    } catch(error) {
      setError("error retrieving tickets: " + error);
    }
  }
  const createTicket = async() => {
    try {
      await axios.post("/api/tickets", {name: name, problem: problem});
    } catch(error) {
      setError("error adding a ticket: " + error);
    }
  }
  const deleteOneTicket = async(ticket) => {
    try {
      await axios.delete("/api/tickets/" + ticket.id);
    } catch(error) {
      setError("error deleting a ticket" + error);
    }
  }

  // fetch ticket data
  useEffect(() => {
    fetchTickets();
  },[]);

  const addTicket = async(e) => {
    e.preventDefault();
    await createTicket();
    fetchTickets();
    setName("");
    setProblem("");
  }

  const deleteTicket = async(ticket) => {
    await deleteOneTicket(ticket);
    fetchTickets();
  }

  // render results
  return (
    <div className="App">
      {error}
      <h1>Create a Ticket</h1>
      <form onSubmit={addTicket}>
        <div>
          <label>
            Name:
            <input type="text" value={name} onChange={e => setName(e.target.value)} />
          </label>
        </div>
        <div>
          <label>
            Problem:
            <textarea value={problem} onChange={e=>setProblem(e.target.value)}></textarea>
          </label>
        </div>
        <input type="submit" value="Submit" />
      </form>
      <h1>Tickets</h1>
      {tickets.map( ticket => (
        <div key={ticket.id} className="ticket">
          <div className="problem">
            <p>{ticket.problem}</p>
            <p><i>-- {ticket.name}</i></p>
          </div>
          <button onClick={e => deleteTicket(ticket)}>Delete</button>
        </div>
      ))}     
    </div>
  );
}

export default App;

```

7. Now start the front end server
```sh
npm start
```

You now have a React front end for a ticket service. 

### State hooks

In `App.js`, are using a [functional component](https://reactjs.org/docs/components-and-props.html) and the [state hook](https://reactjs.org/docs/hooks-state.html). Thus we have four lines that create state variables:

```js
  const [tickets, setTickets] = useState([]);
  const [error, setError] = useState("");
  const [name, setName] = useState("");
  const [problem, setProblem] = useState("");
```

Each variable (such as `tickets`) comes with a setter function (such as `setTickets`). You also provide a default value in `useState`, such as `[]` or an empty string. You can [read more about the useState hook](https://reactjs.org/docs/hooks-reference.html#usestate).

### Calling the API

You will see three methods for calling the API. We use this function to GET all of the current tickets:

```js
const fetchTickets = async() => {
    try {      
      const response = await axios.get(myhostname+"/api/tickets");
      setTickets(response.data);
    } catch(error) {
      setError("error retrieving tickets: " + error);
    }
  }
```

Notice how we use `await` to wait for the API response. This is because axios returns a Promise. Using `await` means your code is free to handle other events (such as a button click), but this method will not run the next line of code until the Promise is finished.

We also wrap this API call in a `try/catch` block so that we can capture any errors that occur and display them in the UI.

### Fetching tickets when the component is rendered

You will see this line in the function, calling [useEffect](https://reactjs.org/docs/hooks-reference.html#useeffect):

```js
  useEffect(() => {
    fetchTickets();
  },[]);
```

This fetches the tickets once, when the application starts. Let's break down what this is doing. The first argument to `useEffect` is a function:

```js
() => { fetchTickets(); }
```

This function takes no arguments and simply calls `fetchTickets()`. We wrap `fetchTickets` in a function because it is an `async` function, and `useEffect` can't handle Promises.

The second argument to `useEffect` is the empty array `[]`. This is a list of dependencies, indicating when the hook should run (whenever its dependencies change). However, since we have given it an empty list, the hook won't run again. Be careful with this -- if you leave off this argument, the function will run after *every* render. This would result in an infinite loop in our case -- `fetchTickets` will cause the component to render (so it can show the tickets), which will trigger another call to `fetchTickets` and so on.

### Handling form events

Each form input needs to handle the `onChange` event. For example:

```js
<input type="text" value={name} onChange={e => setName(e.target.value)} />
```

Every time the input value changes, we call a function, which takes `e`, an event. This function calls `setName` with the value of what was typed into the input field, `e.target.value`. Notice we also set the value to the `name` state variable, so that if we change it in our code, this will be shown on the page.

Likewise, we need a function to handle the event that is triggered when the form is submitted:

```js
<form onSubmit={addTicket}>
```

This will call the `addTicket` function:

```js
  const addTicket = async(e) => {
    e.preventDefault();
    await createTicket();
    fetchTickets();
    setName("");
    setProblem("");
  }
```

We first use `e.preventDefault()` so that the page is not reloaded (the standard browser behavior when submitting a form). We then create the ticket, fetch all tickets (which should include the new one), and reset the `name` and `problem` state variables. Calling `fetchTickets` will result in a change to the `tickets` state variable. Changing all three state variables will cause the page to be rendered again, showing the changes.

### The rest of the code

You will see the rest of the code uses one of these concepts.

### Connecting the back end to the front end

By default, the React app runs on port `8080`. Your back end server runs on port `3000`. You will be tempted to put the full URL into your API requests, like this:

```js
const response = await axios.get("http://yourserverurl:3000/api/tickets");
```

You don't want to do this! This hard-codes a particular hostname (localhost) and port (3030) into your app. It will break when you deploy it.

Instead, notice how our code does this:

```js443
const response = await axios.get("/api/tickets");
```

We leave off the hostname and the port number. By default, this means it goes to the same host and port where the front end is running. While developing the code, this is `yourserverurl:8080`. When running the code, it might be `yourserverulr` at port 443 (because we are using a secure server).

For this to work during development, we have the following line in `package.json`:

```js
  "proxy": "http://localhost:3000",
```

This tells the front end to act as a `proxy` for the back end, sending any request that it doesn't handle (such as for `/api/tickets`) to the listed hostname and port: `localhost:3000`.

When we deploy a React + Node app on a server, we will likewise setup your web server (nginx or Caddy) so that it can reverse proxy API requests to the Node server.

