## Simple To-Do application on MERN Web Stack

### **`I learnt the following:`**

>Better understanding of the components of MERN Web stack

>I learnt about different types of databases - RDBMS and NoSQL.

>I browsed through the Web Application Frameworks concept, Javascript syntax, CSS 
syntax and RESTful APIs.

>I gained knowledge on how to set up frontend and backend configuration for an App.

### **`Below are some of the logs and screenshots of the results generated:`** 

<br/>


## Simple To-Do application on MERN Web Stack
## 1. Backend configuration

<br/>

> *ubuntu@ip-172-31-11-117:~$ sudo apt update*

> To upgrade Ubuntu:   
*ubuntu@ip-172-31-11-117:~$ sudo apt upgrade*

> To find the location of Nodejs:  
*ubuntu@ip-172-31-11-117:~$ curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -*  
*## Installing the NodeSource Node.js 12.x repo...*  
*## Populating apt-get cache...*  
...

<br/>

## `Install Node.js on the server`

> *ubuntu@ip-172-31-11-117:~$ sudo apt-get install -y nodejs*

> Confirm installation:
*ubuntu@ip-172-31-11-117:~$ node -v*  
v12.22.1  
*ubuntu@ip-172-31-11-117:~$ npm -v*  
6.14.12

<br/>

## `Application Code Setup`

<br/>

> Created a directory for my project.  
*ubuntu@ip-172-31-11-117:~$ mkdir Todo*

> *ubuntu@ip-172-31-11-117:~$ ls -l  
total 4  
drwxrwxr-x 2 ubuntu ubuntu 4096 Apr 17 07:36 Todo*

> Changed my directory to the newly created one:  
*$ cd Todo*

> ubuntu@ip-172-31-11-117:~$ cd Todo/  
*ubuntu@ip-172-31-11-117:~/Todo$ ls -l*   
total 0

> To initialize my project (This is to create a new file called package.json)  
*ubuntu@ip-172-31-11-117:~/Todo$ npm init*
```
...
Press ^C at any time to quit.
package name: (todo)
version: (1.0.0)
description: A todo app
git repository:
license: (ISC)
About to write to /home/ubuntu/Todo/package.json:

{
  "name": "todo",
  "version": "1.0.0",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "Romoke Ajayi",
  "license": "ISC",
  "keywords": [
    "todo",
    "application"
  ],
  "description": "A todo app"
}


Is this OK? (yes) yes
```
> *ubuntu@ip-172-31-11-117:~/Todo$ ls  
package.json*

<br/>


## `Install ExpressJS`

`Express helps to define routes of our application based on HTTP methods and URLs.`
> *ubuntu@ip-172-31-11-117:~/Todo$ npm install express  
npm notice created a lockfile as package-lock.json. You should commit this file.  
npm WARN todo@1.0.0 No repository field. <br/><br/>+ express@4.17.1
added 50 packages from 37 contributors and audited 50 packages in 2.004s
found 0 vulnerabilities*

> *ubuntu@ip-172-31-11-117:~/Todo$ ls -l  
total 24  
drwxrwxr-x 52 ubuntu ubuntu  4096 Apr 17 11:03 node_modules  
-rw-rw-r--  1 ubuntu ubuntu 14280 Apr 17 11:03 package-lock.json  
-rw-rw-r--  1 ubuntu ubuntu   323 Apr 17 11:03 package.json*  

> *ubuntu@ip-172-31-11-117:~/Todo$ touch index.js*

`The dotenv is a zero-dependency module that loads environment variables from a .env file into process.env, simply helps inject env variables in our application. dotenv library allows us to load environment variables from a file.`

> *ubuntu@ip-172-31-11-117:~/Todo$ npm install dotenv*  
npm WARN todo@1.0.0 No repository field.
```
+ dotenv@8.2.0
added 1 package and audited 51 packages in 0.765s
found 0 vulnerabilities
```

> *ubuntu@ip-172-31-11-117:~/Todo$vim index.js*

> I put the below code in the index.js file and saved:<br/>  
*ubuntu@ip-172-31-11-117:~/Todo$ cat index.js  
const express = require('express');  
require('dotenv').config();  <br/><br/>const app = express();<br/><br/>
const port = process.env.PORT || 5000;    
app.use((req, res, next) => {  
res.header("Access-Control-Allow-Origin", "\*");  
res.header("Access-Control-Allow-Headers", "Origin, X-Requested-With,   Content-Type, Accept");  
next();  
});  <br/>
app.use((req, res, next) => {  
res.send('Welcome to Express');  
});
<br/><br/>app.listen(port, () => {  
console.log(`Server running on port ${port}`)  
});*

To start my server:

> *ubuntu@ip-172-31-11-117:~/Todo$ node index.js  
Server running on port 5000*

> *ubuntu@ip-172-31-11-117:~$ curl -I http://34.220.254.59:5000  
HTTP/1.1 200 OK  
X-Powered-By: Express  
Access-Control-Allow-Origin: *  
Access-Control-Allow-Headers: Origin, X-Requested-With, Content-Type, Accept   
Content-Type: text/html; charset=utf-8  
Content-Length: 18  
ETag: W/"12-0I3iBDcXjk+Wjy4+CC0sjbHCV48"  
Date: Sat, 17 Apr 2021 23:33:36 GMT  
Connection: keep-alive  
Keep-Alive: timeout=5*

After this, I went to my security group inbound rules and adedd port 5000 to the list of allowed traffic

http://34.220.254.59:5000

![nodejs express page 1](https://user-images.githubusercontent.com/70076627/115373682-04bba380-a1c4-11eb-9a93-e55fea525c31.PNG)

<br/>

## `Routes`


To use my To-do application to carry out different actions/tasks related to HTTP request methods like POST, GET and DELETE, these steps are required:

> *ubuntu@ip-172-31-11-117:~/Todo$ mkdir routes  
ubuntu@ip-172-31-11-117:~/Todo$ cd routes/  
ubuntu@ip-172-31-11-117:~/Todo/routes$ touch api.js  
ubuntu@ip-172-31-11-117:~/Todo/routes$ vi api.js  
ubuntu@ip-172-31-11-117:~/Todo/routes$ cat api.js*    
```
const express = require ('express');
const router = express.Router();

router.get('/todos', (req, res, next) => {

});

router.post('/todos', (req, res, next) => {

});

router.delete('/todos/:id', (req, res, next) => {

})

module.exports = router;
```

## `Models`

To create a model and a Schema, I followed steps below. This is required to connect our app to the database (Mongodb -NoSQL).

> *ubuntu@ip-172-31-11-117:~/Todo$ npm install mongoose  
ubuntu@ip-172-31-11-117:~/Todo$ mkdir models  
ubuntu@ip-172-31-11-117:~/Todo$ cd models/  
ubuntu@ip-172-31-11-117:~/Todo/models$ touch todo.js  
ubuntu@ip-172-31-11-117:~/Todo/models*  

> *ubuntu@ip-172-31-11-117:~$ cd Todo/models/  
ubuntu@ip-172-31-11-117:~/Todo/models$ ls -l  
total 0  
-rw-rw-r-- 1 ubuntu ubuntu 0 Apr 18 12:40 todo.js  
ubuntu@ip-172-31-11-117:~/Todo/models$ vi todo.js*  

> *ubuntu@ip-172-31-11-117:~/Todo/models$ cat todo.js*
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

Our api.js file needs to be updated:

> *ubuntu@ip-172-31-11-117:~/Todo/routes$ ls -l  
total 4  
-rw-rw-r-- 1 ubuntu ubuntu 250 Apr 18 12:25 api.js  
ubuntu@ip-172-31-11-117:~/Todo/routes$ vi api.js*  

> *ubuntu@ip-172-31-11-117:~/Todo/routes$ cat api.js*  
```
const express = require ('express');
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

## `MongoDB Database`

**To set up our database:**  
> I signed up on mLab (DBaaS) to achieve this.

> After adding AWS as my Cloud Provider, I clicked on Create Cluster.<br/>  
On `network Access tab`, I created IP Access list and allowed access from anywhere.<br/>  
On `Database Access tab`, I clicked on "Add New Database User", where I added the username and password details for "Password Authentication".<br/>  
From the `Cluster tab`, I navigated to Collections AND clicked on "Add My Own Data", where I entered the database name and collection name.

<br/>

![mLab Network Access](https://user-images.githubusercontent.com/70076627/115408652-f469ef80-a1e8-11eb-8717-b711a11009cc.PNG)


![mLab DB access](https://user-images.githubusercontent.com/70076627/115409358-9558aa80-a1e9-11eb-96cc-f7682b219a56.PNG)

![mLab Cluster Collections](https://user-images.githubusercontent.com/70076627/115409456-aef9f200-a1e9-11eb-9039-e88b45fd07be.PNG)

<br/>

In the index.js created in step "Install ExpressJS", process.env was specified and now need to be created.

> *ubuntu@ip-172-31-11-117:~/Todo$ touch .env  
ubuntu@ip-172-31-11-117:~/Todo$ vi .env*


>I retrieved the below information from my cluster to be injected in the .env file:
<br/><br/>
*mongodb+srv://romola:password@cluster0.p4zlh.mongodb.net/myFirstDatabase?retryWrites=true&w=majority*

> I filled with my details as below: 
<br/>   
*ubuntu@ip-172-31-11-117:~/Todo$ cat .env*  <br/>     
`DB = 'mongodb+srv://romola:fxxmittly@cluster0.p4zlh.mongodb.net/romolaDB?retryWrites=true&w=majority'`

<br/>

Now I need to update the index.js to reflect the use of .env so that Node.js can connect to the database.
>*ubuntu@ip-172-31-11-117:~$ cd Todo/  
ubuntu@ip-172-31-11-117:~/Todo$ ls -l  
total 40  
-rw-r--r--  1 root   root     457 Apr 17 23:15 index.js  
drwxrwxr-x  2 ubuntu ubuntu  4096 Apr 18 13:09 models  
drwxrwxr-x 77 ubuntu ubuntu  4096 Apr 18 12:36 node_modules  
-rw-rw-r--  1 ubuntu ubuntu 23216 Apr 18 12:36 package-lock.json  
-rw-rw-r--  1 ubuntu ubuntu   399 Apr 18 12:36 package.json*  

Update index.js file:
> *ubuntu@ip-172-31-11-117:~/Todo$ vi index.js*

> *ubuntu@ip-172-31-11-117:~/Todo$ cat index.js* 
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
mongoose.connect(process.env.DB, { useNewUrlParser: true })
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

> *ubuntu@ip-172-31-11-117:~/Todo$ node index.js  
`Server running on port 5000`  
(node:29465) [MONGODB DRIVER] Warning: Current Server Discovery and Monitoring engine is deprecated, and will be removed in a future version. To use the new Server Discover and Monitoring engine, pass option { useUnifiedTopology: true } to the MongoClient constructor.  
`Database connected successfully`*

Result shows backend is configured.


<br/>

## `Testing Backend Code without Frontend using RESTful API`

* `To test backend part of our To-Do application:`  

> I installed the Postman App.   
> I opened Postman, then clicked New ---> Headers ---> KEY (Selection was Content-Type) ----> VALUE (application/json)

> On the Body" tab I selected "raw" and added the below:
```
{
    "action":"Finish Project 8 and 9"
}

{
    "action":"Working on Project 3 and encountered some errors at first due to my server and DB not running, I ran 'node index.js' again to resolve it"
}
```

> I then sent a `GET` request to my API on http://34.220.254.59:5000/api/todos and   
`POST` request to my API on http://34.220.254.59:5000/api/todos


> *`I got an error in the first try:`  
POST http://34.220.254.59:5000/api/todos  
Error: connect ETIMEDOUT 34.220.254.59:5000  
Request Headers  
Content-Type: application/json  
User-Agent: PostmanRuntime/7.26.10  
Accept: */*  
Postman-Token: 2450c3e9-479c-4f0f-9042-3e80edd937cd  
Host: 34.220.254.59:5000  
Accept-Encoding: gzip, deflate, br  
Connection: keep-alive  
Request Body*  

<br/>

> Then I went to run the "node index.js" to trigger running the server and DB.
The results are displayed below:

![Backend code REST API  POST request](https://user-images.githubusercontent.com/70076627/115462221-4bd88180-a222-11eb-8722-b85ec62ea8c3.PNG)

![Backend code REST API  GET request](https://user-images.githubusercontent.com/70076627/115462570-b2f63600-a222-11eb-9a15-41d125dc2e28.PNG)

<br/>

> `I also tried the DELETE request`, the first one failed. Then I added the ID to the end of the URL and it worked. (Not sure at this time if it is the right way or there are other ways to achieve it).
I sent a DELETE request to http://34.220.254.59:5000/api/todos/607ef1afbe5bb4926aeeba50


![Backend code REST API DELETE request](https://user-images.githubusercontent.com/70076627/115462615-c30e1580-a222-11eb-9a85-39cdd467cf4e.PNG)

<br/>
<br/>

# 2. Frontend creation

> To create a user interface for a Web client (browser) to interact with the application via API. `create-react-app` command will be used to scaffold the app. <br/><br/> 
**From the Todo folder, I did:**<br/>   
*ubuntu@ip-172-31-11-117:~/Todo$ npx create-react-app client  
Need to install the following packages:
  create-react-app
Ok to proceed? (y) y <br/><br/>
Creating a new React app in /home/ubuntu/Todo/client.<br/>  
Installing packages. This might take a couple of minutes.  
Installing react, react-dom, and react-scripts with cra-template...  
...  
Success! Created client at /home/ubuntu/Todo/client  
Inside that directory, you can run several commands:  
....*

<br/> 

## `Running a React App`

This step is to install dependencies (concurrently and nodemon) prior to testing the react app:

> ubuntu@ip-172-31-11-117:~$ cd Todo/  
> *ubuntu@ip-172-31-11-117:~/Todo$ npm install concurrently --save-dev*

> *ubuntu@ip-172-31-11-117:~/Todo$ npm install nodemon --save-dev*

To edit package.json (change the "script" part of the file)
> *ubuntu@ip-172-31-11-117:~/Todo$ vi package.json*  
> *ubuntu@ip-172-31-11-117:~/Todo$ cat package.json*
```
{
  "name": "todo",
  "version": "1.0.0",
  "main": "index.js",
  "scripts": {
   "start": "node index.js",
   "start-watch": "nodemon index.js",
   "dev": "concurrently \"npm run start-watch\" \"cd client && npm start\""
  },
  "author": "Romoke Ajayi",
  "license": "ISC",
  "keywords": [
    "todo",
    "application"
  ],
  "description": "A todo app",
  "dependencies": {
    "dotenv": "^8.2.0",
    "express": "^4.17.1",
    "mongoose": "^5.12.4"
  },
  "devDependencies": {
    "concurrently": "^6.0.2",
    "nodemon": "^2.0.7"
  }
}
```
**To configure proxy in package.json:**

> I added "proxy": "http://localhost:5000" to my package.json

> *ubuntu@ip-172-31-11-117:~/Todo/client$ cat package.json* 
```
{
  "name": "client",
  "version": "0.1.0",
  "private": true,
  "proxy": "http://localhost:5000",
  "dependencies": {
    "@testing-library/jest-dom": "^5.11.10",
    "@testing-library/react": "^11.2.6",
    "@testing-library/user-event": "^12.8.3",
    "react": "^17.0.2",
    "react-dom": "^17.0.2",
    "react-scripts": "4.0.3",
    "web-vitals": "^1.1.1"
  },
 ...
```

**To open and run the App:**

> *ubuntu@ip-172-31-11-117:~/Todo$ npm run dev*
```
todo@1.0.0 dev 
concurrently "npm run start-watch" "cd client && npm start"  

[0] 
[0] > todo@1.0.0 start-watch
[0] > nodemon index.js
[0] 
[1] 
[1] > client@0.1.0 start
[1] > react-scripts start
[1] 
[0] [nodemon] 2.0.7
[0] [nodemon] to restart at any time, enter `rs`
[0] [nodemon] watching path(s): *.*
[0] [nodemon] watching extensions: js,mjs,json
[0] [nodemon] starting `node index.js`
[0] Server running on port 5000
[0] (node:32921) [MONGODB DRIVER] Warning: Current Server Discovery and Monitoring engine is deprecated, and will be removed in a future version. To use the new Server Discover and Monitoring engine, pass option { useUnifiedTopology: true } to the MongoClient constructor.
[0] Database connected successfully
[1] ℹ ｢wds｣: Project is running at http://172.31.11.117/
[1] ℹ ｢wds｣: webpack output is served from 
[1] ℹ ｢wds｣: Content not from webpack is served from /home/ubuntu/Todo/client/public
[1] ℹ ｢wds｣: 404s will fallback to /
[1] Starting the development server...
[1] 
[1] Compiled successfully!
[1] 
[1] You can now view client in the browser.
[1] 
[1]   Local:            http://localhost:3000
[1]   On Your Network:  http://172.31.11.117:3000
[1] 
[1] Note that the development build is not optimized.
[1] To create a production build, use npm run build.
[1] 
```
**ubuntu@ip-172-31-11-117:~$ curl http://localhost:3000**
```
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <link rel="icon" href="/favicon.ico" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <meta name="theme-color" content="#000000" />
    <meta
      name="description"
      content="Web site created using create-react-app"
    />
    <link rel="apple-touch-icon" href="/logo192.png" />
    <!--
      manifest.json provides metadata used when your web app is installed on a
      user's mobile device or desktop. See https://developers.google.com/web/fundamentals/web-app-manifest/
    -->
    <link rel="manifest" href="/manifest.json" />
    <!--
      Notice the use of  in the tags above.
      It will be replaced with the URL of the `public` folder during the build.
      Only files inside the `public` folder can be referenced from the HTML.

      Unlike "/favicon.ico" or "favicon.ico", "/favicon.ico" will
      work correctly both with client-side routing and a non-root public URL.
      Learn how to configure a non-root public URL by running `npm run build`.
    -->
    <title>React App</title>
  </head>
  ----
```
<br/>
I added the port 3000 on the allowed ports under Inbound rules on the EC2 AWS instance.
<br/>
<br/>

![React app port 3000_2](https://user-images.githubusercontent.com/70076627/115447270-33f80200-a210-11eb-9fce-77c150edd27a.PNG)


 ## `Creating your React Components`

 **To create the components:**  
Note: client folder previously generated from npx command.  
> *ubuntu@ip-172-31-11-117:~$ cd Todo/  
ubuntu@ip-172-31-11-117:~/Todo$ cd client/ <br/><br/> 
ubuntu@ip-172-31-11-117:~/Todo/client$ ls -l  
total 1552  
-rw-rw-r--    1 ubuntu ubuntu    3369 Apr 19 09:12 README.md  
drwxrwxr-x 1066 ubuntu ubuntu   36864 Apr 19 13:03 node_modules  
-rw-rw-r--    1 ubuntu ubuntu 1530150 Apr 19 09:12 package-lock.json  
-rw-rw-r--    1 ubuntu ubuntu     846 Apr 19 13:02 package.json  
drwxrwxr-x    2 ubuntu ubuntu    4096 Apr 19 09:12 public  
drwxrwxr-x    2 ubuntu ubuntu    4096 Apr 19 09:12 src <br/><br/>
ubuntu@ip-172-31-11-117:~/Todo/client$ cd src/<br/>  
ubuntu@ip-172-31-11-117:~/Todo/client/src$   
ubuntu@ip-172-31-11-117:~/Todo/client/src$ mkdir components  
ubuntu@ip-172-31-11-117:~/Todo/client/src$ cd components/<br/>  
ubuntu@ip-172-31-11-117:~/Todo/client/src/components$ touch Input.js   ListTodo.js Todo.js*

> *ubuntu@ip-172-31-11-117:~/Todo/client/src/components$ vi Input.js  
ubuntu@ip-172-31-11-117:~/Todo/client/src/components$* 

> *ubuntu@ip-172-31-11-117:~/Todo/client/src/components$ cat Input.js* 
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

**To make use of Axios (A promise based HTTP client for the browser and node.js):**
> *ubuntu@ip-172-31-11-117:~/Todo/client/src/components$ cd ..  
ubuntu@ip-172-31-11-117:~/Todo/client/src$ cd ..  
ubuntu@ip-172-31-11-117:~/Todo/client$ npm install axios*
```
added 1 package, and audited 1984 packages in 5s

135 packages are looking for funding
  run `npm fund` for details

found 0 vulnerabilities
```
<br/>

> *ubuntu@ip-172-31-11-117:~/Todo/client$ cd src/components  
ubuntu@ip-172-31-11-117:~/Todo/client/src/components$ vi ListTodo.js  
ubuntu@ip-172-31-11-117:~/Todo/client/src/components$ cat ListTodo.js*
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
```

<br/>

**Adding code to Todo.js:**

> *ubuntu@ip-172-31-11-117:~/Todo/client/src/components$ cat Todo.js* 
```
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
<br/>

**To delete the Logo in the existing App.js ,I made adjustment to the react code:**

Moving to src folder:
> *ubuntu@ip-172-31-11-117:~/Todo/client/src/components$ cd ..  
ubuntu@ip-172-31-11-117:~/Todo/client/src$ ls -l  
total 36  
-rw-rw-r-- 1 ubuntu ubuntu  564 Apr 19 09:12 App.css  
-rw-rw-r-- 1 ubuntu ubuntu  528 Apr 19 09:12 App.js  
-rw-rw-r-- 1 ubuntu ubuntu  246 Apr 19 09:12 App.test.js  
drwxrwxr-x 2 ubuntu ubuntu 4096 Apr 19 16:10 components  
-rw-rw-r-- 1 ubuntu ubuntu  366 Apr 19 09:12 index.css  
-rw-rw-r-- 1 ubuntu ubuntu  500 Apr 19 09:12 index.js  
-rw-rw-r-- 1 ubuntu ubuntu 2632 Apr 19 09:12 logo.svg  
-rw-rw-r-- 1 ubuntu ubuntu  362 Apr 19 09:12 reportWebVitals.js  
-rw-rw-r-- 1 ubuntu ubuntu  241 Apr 19 09:12 setupTests.js*  

> *ubuntu@ip-172-31-11-117:~/Todo/client/src$ vi App.js*


> *ubuntu@ip-172-31-11-117:~/Todo/client/src$ cat App.js*
```
import React from 'react';

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

<br/>

**Edit the App.css also:**
> *ubuntu@ip-172-31-11-117:~/Todo/client/src$ vi App.css*  
> *ubuntu@ip-172-31-11-117:~/Todo/client/src$ cat App.css*

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

**Also edit index.css**  
> *ubuntu@ip-172-31-11-117:~/Todo/client/src$ vi index.css*

> *ubuntu@ip-172-31-11-117:~/Todo/client/src$ cat index.css*
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

**Going back to Todo directory and launching the app.**

> *ubuntu@ip-172-31-11-117:~/Todo/client/src$ cd ../..  
ubuntu@ip-172-31-11-117:~/Todo$ npm run dev*

```
> todo@1.0.0 dev
> concurrently "npm run start-watch" "cd client && npm start"

[0] 
[0] > todo@1.0.0 start-watch
[0] > nodemon index.js
[0] 
[1] 
[1] > client@0.1.0 start
[1] > react-scripts start
[1] 
[0] [nodemon] 2.0.7
[0] [nodemon] to restart at any time, enter `rs`
[0] [nodemon] watching path(s): *.*
[0] [nodemon] watching extensions: js,mjs,json
[0] [nodemon] starting `node index.js`
[0] events.js:291
[0]       throw er; // Unhandled 'error' event
[0]       ^
[0] 
[0] Error: listen EADDRINUSE: address already in use :::5000
[0]     at Server.setupListenHandle [as _listen2] (net.js:1316:16)
[0]     at listenInCluster (net.js:1364:12)
[0]     at Server.listen (net.js:1450:7)
[0]     at Function.listen (/home/ubuntu/Todo/node_modules/express/lib/application.js:618:24)
[0]     at Object.<anonymous> (/home/ubuntu/Todo/index.js:35:5)
[0]     at Module._compile (internal/modules/cjs/loader.js:999:30)
[0]     at Object.Module._extensions..js (internal/modules/cjs/loader.js:1027:10)
[0]     at Module.load (internal/modules/cjs/loader.js:863:32)
[0]     at Function.Module._load (internal/modules/cjs/loader.js:708:14)
[0]     at Function.executeUserEntryPoint [as runMain] (internal/modules/run_main.js:60:12)
[0]     at internal/main/run_main_module.js:17:47
[0] Emitted 'error' event on Server instance at:
[0]     at emitErrorNT (net.js:1343:8)
[0]     at processTicksAndRejections (internal/process/task_queues.js:84:21) {
[0]   code: 'EADDRINUSE',
[0]   errno: 'EADDRINUSE',
[0]   syscall: 'listen',
[0]   address: '::',
[0]   port: 5000
[0] }
[0] [nodemon] app crashed - waiting for file changes before starting...
[1] ℹ ｢wds｣: Project is running at http://172.31.11.117/
[1] ℹ ｢wds｣: webpack output is served from 
[1] ℹ ｢wds｣: Content not from webpack is served from /home/ubuntu/Todo/client/public
[1] ℹ ｢wds｣: 404s will fallback to /
[1] Starting the development server...
[1] 
[1] Compiled successfully!
[1] 
[1] You can now view client in the browser.
[1] 
[1]   Local:            http://localhost:3000
[1]   On Your Network:  http://172.31.11.117:3000
[1] 
[1] Note that the development build is not optimized.
[1] To create a production build, use npm run build.
[1] 
```

I refreshed my http://34.220.254.59:3000/ page, the result is captured in screenshot below.

![Frontend GUI interface](https://user-images.githubusercontent.com/70076627/115452703-ac61c180-a216-11eb-9f53-eecec5d3c942.PNG)


<br/> 

**`I had to go through this Project thrice. First time, a few things were mixed up and server not starting (ran node command again).`** <br/> 

**`Second time, I encountered errors at different points including the last section (issues with codes that I pasted in the various files).`** <br/> 

**`Third time was to go through the whole project again in order to link all I have done together. At the end, it made a lot of sense to me, more practise will be required for all to stick.`**