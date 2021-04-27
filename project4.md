## MEAN Stack Deployment to Ubuntu in AWS

### **`I learnt the following:`**

>Good understanding of the components of MEAN Web stack.

>I practiced some simple HTML, CSS, JS editing.

>I was able to fully understand the functions of setting up routes, controller, DB, front-end and how they interconnect.

### **`Below are some of the logs and screenshots of the results generated:`** 

<br/>

## Simple Book Register web form using MEAN stack
## 1. Install NodeJs

<br/>

`Nodejs will be used to set up the Express routes and AngularJS controllers`

> *ubuntu@ip-172-31-18-30:~$ sudo apt update  
> ubuntu@ip-172-31-18-30:~$ sudo apt upgrade  
> ubuntu@ip-172-31-18-30:~$ sudo apt install -y nodejs*

<br/>

## 2. Install MongoDB

Download the trusted key:
> *ubuntu@ip-172-31-18-30:~$ **sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 0C49F3730359A14518585931BC711F9BA15703C6**  
Executing: /tmp/apt-key-gpghome.LtPcV6fxx7/gpg.1.sh --keyserver hkp://keyserver.ubuntu.com:80 --recv 0C49F3730359A14518585931BC711F9BA15703C6  
gpg: key BC711F9BA15703C6: public key "MongoDB 3.4 Release   Signing Key <packaging@mongodb.com>" imported  
gpg: Total number processed: 1  
gpg:               imported: 1*

<br/>

> *ubuntu@ip-172-31-18-30:~$ **echo "deb [ arch=amd64 ] https://repo.mongodb.org/apt/ubuntu trusty/mongodb-org/3.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.4.list**  
deb [ arch=amd64 ] https://repo.mongodb.org/apt/ubuntu trusty/mongodb-org/3.4 multiverse*


Installed Mongodb and started:

> *ubuntu@ip-172-31-18-30:~$ **sudo apt install -y mongodb**  
Reading package lists... Done  
Building dependency tree         
Reading state information... Done  
...*

> *ubuntu@ip-172-31-18-30:~$ **sudo service mongodb start**  

> *ubuntu@ip-172-31-18-30:~$ **sudo systemctl status mongodb**  
● mongodb.service - An object/document-oriented database  
     &nbsp; &nbsp;Loaded: loaded (/lib/systemd/system/&nbsp; &nbsp;mongodb.service;   enabled; vendor preset: enabled)  
     &nbsp; &nbsp;Active: active (running) since Sun 2021-04-25 19:51:54   UTC; 39s ago  
     &nbsp; &nbsp;  Docs: man:mongod(1)  
   Main PID: 21867 (mongod)  
      &nbsp; &nbsp;Tasks: 23 (limit: 1160)  
     Memory: 42.5M  
     CGroup: /system.slice/mongodb.service  
       &nbsp; &nbsp;&nbsp; &nbsp;      └─21867 /usr/bin/mongod --unixSocketPrefix=/run/mongodb --config /etc/mongodb.conf*  

I installed Node package manager:

> *ubuntu@ip-172-31-18-30:~$ **sudo apt install -y npm**  
Reading package lists... Done
Building dependency tree       
Reading state information... Done  
The following additional packages will be installed:   
...*

I also Installed body-parser package to help process JSON files passed in requests to the server.

> *ubuntu@ip-172-31-18-30:~$ sudo npm install body-parser*

<br/>

I created a directory for the project where npm will be initialized.

>*ubuntu@ip-172-31-18-30:~$ mkdir Books && cd Books*

>*ubuntu@ip-172-31-18-30:~/Books$ npm init*

```
This utility will walk you through creating a package.json file.
It only covers the most common items, and tries to guess sensible defaults.*

See `npm help json` for definitive documentation on these fields
and exactly what they do.

Use `npm install <pkg>` afterwards to install a package and
save it as a dependency in the package.json file.

Press ^C at any time to quit.
package name: (books) 
version: (1.0.0) 
description: 
entry point: (index.js) 
test command: 
git repository: 
keywords: 
author: romoke
license: (ISC) 
About to write to /home/ubuntu/Books/package.json:

{
  "name": "books",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "romoke",
  "license": "ISC"
}


Is this OK? (yes) yes
```

> *ubuntu@ip-172-31-18-30:~/Books$ ls -l
total 4
-rw-rw-r-- 1 ubuntu ubuntu 207 Apr 25 19:59 package.json*

> *ubuntu@ip-172-31-18-30:~/Books$ vi server.js*
```
var express = require('express');
var bodyParser = require('body-parser');
var app = express();
app.use(express.static(__dirname + '/public'));
app.use(bodyParser.json());
require('./apps/routes')(app);
app.set('port', 3300);
app.listen(app.get('port'), function() {
    console.log('Server up: http://localhost:' + app.get('port'));
});    
```

<br/>

## 3. Install Express and set up routes to the server

Express (Back-end application) will be used to pass book information to and from our MongoDB database.
I installed Mongoose to establish a schema for the database to store data of our book register.

> *ubuntu@ip-172-31-18-30:~/Books$ sudo npm install express mongoose
npm notice created a lockfile as package-lock.json. You should commit this file.*

In Books directory:
> *ubuntu@ip-172-31-18-30:~/Books$ mkdir apps && cd apps*

> *ubuntu@ip-172-31-18-30:~/Books/apps$ vi routes.js*

```
var Book = require('./models/book');
module.exports = function(app) {
  app.get('/book', function(req, res) {
    Book.find({}, function(err, result) {
      if ( err ) throw err;
      res.json(result);
    });
  }); 
  app.post('/book', function(req, res) {
    var book = new Book( {
      name:req.body.name,
      isbn:req.body.isbn,
      author:req.body.author,
      pages:req.body.pages
    });
    book.save(function(err, result) {
      if ( err ) throw err;
      res.json( {
        message:"Successfully added book",
        book:result
      });
    });
  });
  app.delete("/book/:isbn", function(req, res) {
    Book.findOneAndRemove(req.query, function(err, result) {
      if ( err ) throw err;
      res.json( {
        message: "Successfully deleted the book",
        book: result
      });
    });
  });
  var path = require('path');
  app.get('*', function(req, res) {
    res.sendfile(path.join(__dirname + '/public', 'index.html'));
  });
};
```

Created the models direstory:

> *ubuntu@ip-172-31-18-30:~/Books/apps$ mkdir models && cd models*

> *ubuntu@ip-172-31-18-30:~/Books/apps/models$ vi book.js*
```
var mongoose = require('mongoose');
var dbHost = 'mongodb://localhost:27017/test';
mongoose.connect(dbHost);
mongoose.connection;
mongoose.set('debug', true);
var bookSchema = mongoose.Schema( {
  name: String,
  isbn: {type: String, index: true},
  author: String,
  pages: Number
});
var Book = mongoose.model('Book', bookSchema);
module.exports = mongoose.model('Book', bookSchema);
```

## 4. Access the routes with AngularJS

AngularJS (Front-end application) will connect the web page with Express and perform actions on our book register.

`In Books directory:`
> *ubuntu@ip-172-31-18-30:~/Books$ mkdir public && cd public*

> *ubuntu@ip-172-31-18-30:~/Books/public$ vi script.js*

```
var app = angular.module('myApp', []);
app.controller('myCtrl', function($scope, $http) {
  $http( {
    method: 'GET',
    url: '/book'
  }).then(function successCallback(response) {
    $scope.books = response.data;
  }, function errorCallback(response) {
    console.log('Error: ' + response);
  });
  $scope.del_book = function(book) {
    $http( {
      method: 'DELETE',
      url: '/book/:isbn',
      params: {'isbn': book.isbn}
    }).then(function successCallback(response) {
      console.log(response);
    }, function errorCallback(response) {
      console.log('Error: ' + response);
    });
  };
  $scope.add_book = function() {
    var body = '{ "name": "' + $scope.Name + 
    '", "isbn": "' + $scope.Isbn +
    '", "author": "' + $scope.Author + 
    '", "pages": "' + $scope.Pages + '" }';
    $http({
      method: 'POST',
      url: '/book',
      data: body
    }).then(function successCallback(response) {
      console.log(response);
    }, function errorCallback(response) {
      console.log('Error: ' + response);
    });
  };
});
```
> *ubuntu@ip-172-31-18-30:~/Books/public$ vi index.js*

```
<!doctype html>
<html ng-app="myApp" ng-controller="myCtrl">
  <head>
    <script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.6.4/angular.min.js"></script>
    <script src="script.js"></script>
  </head>
  <body>
    <div>
      <table>
        <tr>
          <td>Name:</td>
          <td><input type="text" ng-model="Name"></td>
        </tr>
        <tr>
          <td>Isbn:</td>
          <td><input type="text" ng-model="Isbn"></td>
        </tr>
        <tr>
          <td>Author:</td>
          <td><input type="text" ng-model="Author"></td>
        </tr>
        <tr>
          <td>Pages:</td>
          <td><input type="number" ng-model="Pages"></td>
        </tr>
      </table>
      <button ng-click="add_book()">Add</button>
    </div>
    <hr>
    <div>
      <table>
        <tr>
          <th>Name</th>
          <th>Isbn</th>
          <th>Author</th>
          <th>Pages</th>

        </tr>
        <tr ng-repeat="book in books">
          <td>{{book.name}}</td>
          <td>{{book.isbn}}</td>
          <td>{{book.author}}</td>
          <td>{{book.pages}}</td>

          <td><input type="button" value="Delete" data-ng-click="del_book(book)"></td>
        </tr>
      </table>
    </div>
  </body>
</html>
```

> *ubuntu@ip-172-31-18-30:~/Books/public$ cd ..*

> *ubuntu@ip-172-31-18-30:~/Books$ **node server.js**  
(node:36170) DeprecationWarning: current URL string parser is deprecated, and will be removed in a future version. To use the new parser, pass option { useNewUrlParser: true } to MongoClient.connect.  
(node:36170) [MONGODB DRIVER] Warning: Current Server Discovery and Monitoring engine is deprecated, and will be removed in a future version. To use the new Server Discover and Monitoring engine, pass option { useUnifiedTopology: true } to the MongoClient constructor.  
Server up: http://localhost:3300  
Mongoose: books.ensureIndex({ isbn: 1 }, { background: true })  
(node:36170) DeprecationWarning: collection.ensureIndex is deprecated. Use createIndexes instead.*  

I could not access the site, then it started to generate below errors when I tried the second time.

<br/> 

> *ubuntu@ip-172-31-18-30:~/Books$ **node server.js**  
(node:36608) DeprecationWarning: current URL string parser is deprecated, and will be removed in a future version. To  use the new parser, pass option { useNewUrlParser: true } to MongoClient.connect.  
(node:36608) [MONGODB DRIVER] Warning: Current Server   Discovery and Monitoring engine is deprecated, and will be   removed in a future version. To use the new Server Discover  and Monitoring engine, pass option { useUnifiedTopology: true } to the MongoClient constructor.  
Server up: http://localhost:3300  
Mongoose: books.ensureIndex({ isbn: 1 }, { background: true })   
(node:36608) DeprecationWarning: collection.ensureIndex is deprecated. Use createIndexes instead.  
express deprecated res.sendfile: Use res.sendFile instead apps/routes.js:35:9  
**`Error: ENOENT: no such file or directory, stat '/home/ubuntu/Books/apps/public/index.html'`**  
Error: ENOENT: no such file or directory, stat '/home/ubuntu/Books/apps/public/index.html'    
Error: ENOENT: no such file or directory, stat '/home/ubuntu/Books/apps/public/index.html'*    

<br/>

### PROBLEM:  
> *ubuntu@ip-172-31-18-30:~/Books/public$ ls -l  
total 8  
-rw-rw-r-- 1 ubuntu ubuntu 1239 Apr 26 10:11 **index.js** `<---- It was supposed to be `**index.html**`, I made a mistake here`  
-rw-rw-r-- 1 ubuntu ubuntu 1058 Apr 25 20:27 script.js*

### SOLUTION:

> *ubuntu@ip-172-31-18-30:~/Books/public$ **mv index.js index.html**  
ubuntu@ip-172-31-18-30:~/Books/public$ ls -l   
total 8  
-rw-rw-r-- 1 ubuntu ubuntu 1239 Apr 26 10:11 index.html
-rw-rw-r-- 1 ubuntu ubuntu 1058 Apr 25 20:27 script.js*

### OUTPUT:

> *ubuntu@ip-172-31-18-30:~/Books/public$ curl -s http://localhost:3300*

```
<!doctype html>
<html ng-app="myApp" ng-controller="myCtrl">
  <head>
    <script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.6.4/angular.min.js"></script>
    <script src="script.js"></script>
  </head>
  <body>
    <div>
      <table>
        <tr>
          <td>Name:</td>
          <td><input type="text" ng-model="Name"></td>
        </tr>
        <tr>
          <td>Isbn:</td>
          <td><input type="text" ng-model="Isbn"></td>
        </tr>
        <tr>
          <td>Author:</td>
          <td><input type="text" ng-model="Author"></td>
        </tr>
        <tr>
          <td>Pages:</td>
          <td><input type="number" ng-model="Pages"></td>
        </tr>
      </table>
      <button ng-click="add_book()">Add</button>
    </div>
    <hr>
    <div>
      <table>
        <tr>
          <th>Name</th>
          <th>Isbn</th>
          <th>Author</th>
          <th>Pages</th>

        </tr>
        <tr ng-repeat="book in books">
          <td>{{book.name}}</td>
          <td>{{book.isbn}}</td>
          <td>{{book.author}}</td>
          <td>{{book.pages}}</td>

          <td><input type="button" value="Delete" data-ng-click="del_book(book)"></td>
        </tr>
      </table>
    </div>
  </body>
</html>
```

`Screenshot of site below:`

![Book register page 1](https://user-images.githubusercontent.com/70076627/116230832-064f1380-a750-11eb-9bc4-6d39243f2530.PNG)

![Book register page 2](https://user-images.githubusercontent.com/70076627/116230917-18c94d00-a750-11eb-9ce6-f91fd7bbcbdd.PNG)