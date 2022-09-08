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
to confirm the installation of nodejs and npm bbyb checking their version 


now let's make folder that would contain our appication backend configuration 


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


 




