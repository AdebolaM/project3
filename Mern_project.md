This project is to build a simple Todo app using MERn web stack implementation

M-MongoDB (database)

E- express (frontend)

R= Reactjs (backend)

N=Node.js (backend)


I will also be trying out Mongodb to test some request on my app  before configuring the frontend of the application

lets go!


1. backend configuration 

AS usual, the first time is to update and upgade "apt" and look for the location node.js in Ubuntu's repos and then install node.js 

``` 
curl -fssl https://deb.nodesource.com/ setup_ 18.x | sudo -E bash -
```
```
sudo apt-get install -y nodejs
```
to confirm the installation of nodejs and npm by checking their versions


![project3](https://github.com/AdebolaM/project3/blob/main/images/node%20version.png?raw=true)


now I made a folder that would contain the appication backend configuration 


```
mkdir Todo && cd Todo
```
then in the Todo directory, I used the command 

``` 
npm init
```
 to create a package.json file which contains the imformation and dependencies of todo app. I place in information and followed the prompt  until I could conclude the page and th file was formed 


 Still on the backend of the app, I install express, 
 created a file for index.js that was mentioned as the extry point  in the package.json file 
 I then installed "dotenv"

 * side note, after install Node.js everything that has to do with installation about the app will be in the code 'npm'

```
npm install express
npm install dotenv
touch index.js
vi index.js <vim and vi are the same thing>
```

```
const express = require('express');
require('dotenv').config();

const app = express();

const port = process.env.PORT || 5000;

app.use((req, res, next) => {
res.header("Access-Control-Allow-Origin", "\*");
res.header("Access-Control-Allow-Headers", "Origin, X-Requested-With, Content-Type, Accept");
next();
});

app.use((req, res, next) => {
res.send('Welcome to Express');
});

app.listen(port, () => {
console.log(`Server running on port ${port}`)
});
```





And just like the backend of the App is done !

```
node index.js
```
the code above gets the server running on port 5000. 


* I configured the security group of the Ec2 instance to allow such inbound traffic 


I confirmed this by opening the app on the web with http://<PublicIP-or-PublicDNS>:5000 

![project3](https://github.com/AdebolaM/project3/blob/main/images/welcome%20to%20empress.png?raw=true)



Next is to set up routes for the app.
there are 3 things that this app must be able to do which are

create- post 
display -get 
delete - delete

I created a folder for routes in the Todo folder and create the file api.js 

```
mkdir routes && cd routes && touch api.js 
```
```
 vim api.js 
 ```
 ```const express = require ('express');
const router = express.Router();

router.get('/todos', (req, res, next) => {

});

router.post('/todos', (req, res, next) => {

});

router.delete('/todos/:id', (req, res, next) => {

})

module.exports = router;
```

In irder ti create models and schema which the components that make the app interactive since I will be using a Nosql database like mongodb, I have to install mongoose, and the create a folder for models and file named todo.js inside the model folder.

``` 
npm install mongoose
```
```
mkdir models && cd models && touch todo.js 
```
```
const mongoose = require('mongoose');
const Schema = mongoose.Schema;

//create schema for todo
const TodoSchema = new Schema({
action: {
type: String,
required: [true, 'The todo text field is required']
}
})

//create model for todo
const Todo = mongoose.model('todo', TodoSchema);

module.exports = Todo;
```
I then updated api.js to make use of the new models

* api = application programme interface 

```
#const express = require ('express');
const router = express.Router();
const Todo = require('../models/todo');

router.get('/todos', (req, res, next) => {

//this will return all the data, exposing only the id and action field to the client
Todo.find({}, 'action')
.then(data => res.json(data))
.catch(next)
});

router.post('/todos', (req, res, next) => {
if(req.body.action){
Todo.create(req.body)
.then(data => res.json(data))
.catch(next)
}else {
res.json({
error: "The input field is empty"
})
}
});

router.delete('/todos/:id', (req, res, next) => {
Todo.findOneAndDelete({"_id": req.params.id})
.then(data => res.json(data))
.catch(next)
})

module.exports = router;
```

2. database config

to configure the database, I had to register on the mlab to access mongodb.

I chose AWS as my cloud provider and made sure that traffic from all IP address is allowed on the cluster. I completed the set up for the password and security. I also changed the time for deleting enrty to 1 week and I launched the cluster by adding my own data.

I clicked on connect and i was able to copy the connection string which I placed in a .env file that I created in the Todo directory 

```
touch .env && vi .env
```
```
DB = 'mongodb+srv://<username>:<password>@<network-address>/<dbname>?retryWrites=true&w=majority'
```


I update the index.js file to recorgnise the .env file that I just created so that Node.js can connect to the database

```
const express = require('express');
const bodyParser = require('body-parser');
const mongoose = require('mongoose');
const routes = require('./routes/api');
const path = require('path');
require('dotenv').config();

const app = express();

const port = process.env.PORT || 5000;

//connect to the database
mongoose.connect(process.env.DB, { useNewUrlParser: true, useUnifiedTopology: true })
.then(() => console.log(`Database connected successfully`))
.catch(err => console.log(err));

//since mongoose promise is depreciated, we overide it with node's promise
mongoose.Promise = global.Promise;

app.use((req, res, next) => {
res.header("Access-Control-Allow-Origin", "\*");
res.header("Access-Control-Allow-Headers", "Origin, X-Requested-With, Content-Type, Accept");
next();
});

app.use(bodyParser.json());

app.use('/api', routes);

app.use((err, req, res, next) => {
console.log(err);
next();
});

app.listen(port, () => {
console.log(`Server running on port ${port}`)
});

```

running node index.js in the Todo directory get the app running 


![project](https://github.com/AdebolaM/project3/blob/main/images/database%20connect.png?raw=true)



3. Frontend configuration 

I need to configure the api to allow my users to interact with the backend configuration that I have.

``` npx create-react-app client ``` 
creates the fold client in the Todo directory this is where is will configure all the react codes ie frontend 


I installed concurrently 
```
 npm install concurrently --save-dev
 ```
  this allows to run more than ine command in the same terminal windows 

  and I also installed nodemon 
  ```
  npm install nodemon --save-dev
  ```
  this runs and monitor the serve so that he can automatically restarts itself 

  so now I have to include the two new softwares in the index.js code so that Node.js can recorgise them

   in the index.js file I replaced theh scripts with 

   ```
   "scripts": {
"start": "node index.js",
"start-watch": "nodemon index.js",
"dev": "concurrently \"npm run start-watch\" \"cd client && npm start\""
},
```

the next thing is to configure proxy in the package.json file so that I can call the app directly from the internet without having to iclude the entire path with api like http://localhost:5000/api/todos so i can do it with http://localhost:5000

inside the client directory, I add   ``` "proxy": "http://localhost:5000"```


```
npm run dev 
```
 gets the app runing on port 3000. but to be able to view this on the internet, the security settings for the ec2 instance was adjusted to allow for inbound traffic from the port 

![project3](https://github.com/AdebolaM/project3/blob/main/images/port%20running%20on%203000.png?raw=true)




 
in order to create the that who show onthe frontend i.e react components 

One of the advantages of react is that it makes use of components, which are reusable and also makes code modular. For our Todo app, there will be two stateful components and one stateless component.

in the src directory in the client directory, I made a folder for components, 
and create  the following files Input.js ListTodo.js Todo.js and put in the following information respectively


Input.js

```
import React, { Component } from 'react';
import axios from 'axios';

class Input extends Component {

state = {
action: ""
}

addTodo = () => {
const task = {action: this.state.action}

    if(task.action && task.action.length > 0){
      axios.post('/api/todos', task)
        .then(res => {
          if(res.data){
            this.props.getTodos();
            this.setState({action: ""})
          }
        })
        .catch(err => console.log(err))
    }else {
      console.log('input field required')
    }

}

handleChange = (e) => {
this.setState({
action: e.target.value
})
}

render() {
let { action } = this.state;
return (
<div>
<input type="text" onChange={this.handleChange} value={action} />
<button onClick={this.addTodo}>add todo</button>
</div>
)
}
}

export default Input
```

 
 ListTodo.js

 ```
 import React from 'react';

const ListTodo = ({ todos, deleteTodo }) => {

return (
<ul>
{
todos &&
todos.length > 0 ?
(
todos.map(todo => {
return (
<li key={todo._id} onClick={() => deleteTodo(todo._id)}>{todo.action}</li>
)
})
)
:
(
<li>No todo(s) left</li>
)
}
</ul>
)
}

export default ListTodo
Then in your Todo.js file you write the following code

import React, {Component} from 'react';
import axios from 'axios';

import Input from './Input';
import ListTodo from './ListTodo';

class Todo extends Component {

state = {
todos: []
}

componentDidMount(){
this.getTodos();
}

getTodos = () => {
axios.get('/api/todos')
.then(res => {
if(res.data){
this.setState({
todos: res.data
})
}
})
.catch(err => console.log(err))
}

deleteTodo = (id) => {

    axios.delete(`/api/todos/${id}`)
      .then(res => {
        if(res.data){
          this.getTodos()
        }
      })
      .catch(err => console.log(err))

}

render() {
let { todos } = this.state;

    return(
      <div>
        <h1>My Todo(s)</h1>
        <Input getTodos={this.getTodos}/>
        <ListTodo todos={todos} deleteTodo={this.deleteTodo}/>
      </div>
    )

}
}

export default Todo;
```

 
 Todo.js

In the client folder. I installed axios ```npm install axios```


in the src folder. I created a file for Api.js 

```import React from 'react';

import Todo from './components/Todo';
import './App.css';

const App = () => {
return (
<div className="App">
<Todo />
</div>
);
}

export default App;
```

and a file for app.css

```
.App {
text-align: center;
font-size: calc(10px + 2vmin);
width: 60%;
margin-left: auto;
margin-right: auto;
}

input {
height: 40px;
width: 50%;
border: none;
border-bottom: 2px #101113 solid;
background: none;
font-size: 1.5rem;
color: #787a80;
}

input:focus {
outline: none;
}

button {
width: 25%;
height: 45px;
border: none;
margin-left: 10px;
font-size: 25px;
background: #101113;
border-radius: 5px;
color: #787a80;
cursor: pointer;
}

button:focus {
outline: none;
}

ul {
list-style: none;
text-align: left;
padding: 15px;
background: #171a1f;
border-radius: 5px;
}

li {
padding: 15px;
font-size: 1.5rem;
margin-bottom: 15px;
background: #282c34;
border-radius: 5px;
overflow-wrap: break-word;
cursor: pointer;
}

@media only screen and (min-width: 300px) {
.App {
width: 80%;
}

input {
width: 100%
}

button {
width: 100%;
margin-top: 15px;
margin-left: 0;
}
}

@media only screen and (min-width: 640px) {
.App {
width: 60%;
}

input {
width: 50%;
}

button {
width: 30%;
margin-left: 10px;
margin-top: 0;
}
}
```

and then a file for index.css

```
body {
margin: 0;
padding: 0;
font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", "Roboto", "Oxygen",
"Ubuntu", "Cantarell", "Fira Sans", "Droid Sans", "Helvetica Neue",
sans-serif;
-webkit-font-smoothing: antialiased;
-moz-osx-font-smoothing: grayscale;
box-sizing: border-box;
background-color: #282c34;
color: #787a80;
}

code {
font-family: source-code-pro, Menlo, Monaco, Consolas, "Courier New",
monospace;
}
```

in the Todo folder. 

```
npm run dev
```


![project3](https://github.com/AdebolaM/project3/blob/main/images/my%20tod%20app.png?raw=true)






