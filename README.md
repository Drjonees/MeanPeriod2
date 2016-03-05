# MeanPeriod2


* __Why would you consider a Scripting Language as JavaScript as your Backend Platform.__

The line between scripting language and 'normal' compiled programming language are getting more and more blurred, since hardware are getting better and better, the compilers gets faster and faster.
Javascipt is being interpreted by the V8 engine. That means that it runs inside of another program. Often a scripting language is faster to learn and use. You can often do more with less code.

Another reason to use javascript is that in frontend, we almost always use javascript already. By having the same language in frontend and backend, we can reuse and optimize the developer resources in a company.
A backend programmer can help with the frontend if needed and vice versa.



* __Explain Pros & Cons in using Node.js + Express to implement your Backend compared to a strategy using for example Java/JAX-RS/Tomcat__

    * __Pros__
    
        * Same language in both frontend and backend
        * Unified data format: JSON.
        * Not having to deal with threads because of node.js non-blocking event queue.
        * Speed. Node.js is often faster than the Java, JAX-RS, Tomcat stack.
        * Fairly new technology, so other cool technologies such as websockets are easy to implement.
        * Scalability.
        * NPM
        * ..To be continued
        
    
    * __Cons__
    
        * CPU heavy operations. Since Node.js is single threaded, a CPU heavy coperation could slow down the whole server.(Use Cluster module).
        * Fairly new technology. Hard to keep being updated with all the new updates and packages available.
        * Unhandled error will shutdown the server.
        * ..To be continued

Node.js:
![Node.js Performance](http://i.imgur.com/cOfu8HZ.png)


Java on Tomcat:
![Java on Tomcat Performance](http://i.imgur.com/Ya72Zmq.png)

[Source](http://josh.zeigler.us/technology/web-development/experiences-with-node-js-porting-a-restful-service-written-in-java/ "Source")






* __Node.js uses a Single Threaded Non-blocking strategy to handle asynchronous task. Explain strategies to implement a Node.js based server architecture that still could take advantage of a multi-core Server.__

Last semester, we talked alot about why we should be using threads. Since CPU's speed have more or less stopped increasing, the number of cores has increased. By using threads, we could really take advantage of the multi-core system.

##### Multiple Cores

Since Node.js is single threaded, it is only running on one core. 
To take advantage of the multi-core system, we need to launch a cluster of Node.js processes.
Node.js comes with a built in [Cluster Api](https://nodejs.org/api/cluster.html "Cluster Api").

By using the Cluster module, we can spawn a pool of workers, running under a parent Node process. 
The parent/master process is in charge of initiating and controller workers.


The cluster module lets us create multiple processes that all uses the same server port:

```javascript
var cluster = require('cluster');
var http = require('http');
var numCPUs = require('os').cpus().length;

if (cluster.isMaster) {
    for (var i = 0; i < numCPUs; i++) {
        cluster.fork();
    }
} else {
    http.createServer(function(req, res) {
        res.writeHead(200);
        res.end('process ' + process.pid + ' says hello!');
    }).listen(8000);
}
```

##### Multiple Servers

If needed, node can scale even more by using a load balancer handling multiple servers, running multiple processes:


![Multiple Servers](http://js2016.azurewebsites.net/node1/images/NodeJS-Architecture2.png)

 
 
 
 
* __Explain, using relevant examples, concepts related to the testing a REST-API using Node/JavaScript + relevant packages__

If the tests, so not need to be automated and you just want to see that your REST-API works, you can simply use the request module.

The ['Request' module](https://www.npmjs.com/package/request "'Request' module") allows us to make http requests from our node.js application. 
This can be used to 'test' our RESTful api:

Get:
```javascript
request('http://localhost:3000/api/joke', function (error, response, body) {
    if (!error && response.statusCode == 200) {
        console.log(body);
    }
});
```

Post:
```javascript
request({
        url: "http: //localhost:3000/api/joke",
        method: "POST",
        json: true,
        body: {
            joke: "I'm a joke"
        }
    },
    function (error, res, body) {
        if (!error && res.statusCode == 200) {
            console.log(body);
        }
    });
```

However, if we want to make 'real' tests, we have to use mocha and chai.

* __Explain, using relevant examples, the Express concept; middleware.__

Alot of the work we're doing on a web server, is happening between the request and respond, for instance processing the http request, and building the appropriate response for that request.

In Express, we use middleware to do just that.
Middleware is just code that is being executed between two layers of software. In our case, it's between the request and response.

In Express the middleware functions are given access to the request object, response object and the next middleware function in the application's cycle.
We can use middleware to log what type of http methods gets through our server:

```javascript
app.use(function (req, res, next) {
  console.log('Request Type:', req.method);
  next();
});
```
Notice that no URL was given for this middleware, so it will be executed for all incoming requests.

We can use a URL if we want our middleware to only be executed from certain pages.
We can for instance make a middleware function that logs at what time someone access the 'secret page' url:

```javascript
app.use('/secretPage',function (req, res, next) {
  console.log('Someone was at the secret page at:', Date.now());
  next();
});
```

It's important to call 'next()', if the middleware is not ending the request-response cycle.

Often we're using third party middleware.
We can use 'serve-favicon' to handle the favicon of our site:
```javascript
app.use(favicon(__dirname + '/public/favicon.ico'));
```

Or we can use middleware to handle routing:
```javascript
var users = require('./routes/users');
var login = require('./routes/login');

app.use('/users', users);
app.use('/login', login);
```



* __Explain, using relevant examples, how to implement sessions, and the legal implications of doing this.__

There are various ways of implementing sessions, but the most used one is by using the module called "express-session".
The way it works (on the surface), is by saving a cookie holding the Session ID. When ever a request comes in, the session middleware will look at the session ID found in the cookie,
and make the session object available for us. 

It can be used like this to check if a user is logged in with a username:

```javascript
var express = require('express');
var session = require("express-session");
var app = express();

//Alot of other middleware stuff is going on here


app.use(session({
    secret: 'secretWordIsSecret',
    saveUninitialized: true,
    resave: true
}));


app.use(function (req, res, next) {
    //Taking the session object from the request object.
    var sess = req.session; 


    //If the session object has a userName, the request is allowed access to the rest of the site and we call next() to go to the next middleware in the express stack.
    if (sess.userName) {
        return next();
    }
    
    //Else if the requests body has a userName, that means that it's a login attempt, so we will save the username to the session and respond with a redirect to '/' path.
    else if (req.body.userName) {
        sess.userName = req.body.userName;
        return res.redirect('/');
    } 
    
    //Else the user is not logged in and is not trying to login, so we do not want to give access to the rest of the site so we redirect to the '/login' page.
    else {
         return res.redirect('/login');
    }
});
```


The express-session module comes with a 'MemoryStore', but in a production environment i'd properbly use something like a seperate mongodb or redis database. This would make it possible to scale up and use multiple node.js servers:
```javascript
var express = require('express');
var RedisStore = require('connect-redis')(session);

var app = express();

app.use(session({
    store: new RedisStore({
        url: 'redis://redistogo:87b3f59d153dc010dc97257b6a270ae5@tarpon.redistogo.com:10768'
    }),
    secret: 'secretWordIsSecret'
}));
```



Since we're using a cookie to store our Session ID, we will have to let the users know that we're using cookies, why we're using them and how.

* __Compare the express strategy toward (server side) templating with the one you used with Java on second semester.__

On 2nd semester, we used a MVC pattern, using servlets, java classes and jsp pages.
The servlet(with other controller classes), took the data from our java objects(from db -> mapping to objects -> facade) and put the data into the jsp pages, which was then generated to html and send back to the user.

With node.js server side templating, we're using the same principles but other technologies. 
We have a view engine that takes some data, and puts it into our views and returns some generated html.
Since we're using a view engine, we can easily change which engine to use, and therefore we have more oppotunities.
The most used ones are Jade, EJS and Handlebars.


* __Explain, using a relevant examples, your strategy for implementing a REST-API with Node/Express and show how you can "test" all the four CRUD operations programmatically using for example the Request package.__

The ['Request' module](https://www.npmjs.com/package/request "'Request' module") allows us to make http requests from our node.js application. 
This can be used to 'test' our RESTful api:

Get:
```javascript
request('http://localhost:3000/api/joke', function (error, response, body) {
    if (!error && response.statusCode == 200) {
        console.log(body);
    }
});
```

Post:
```javascript
request({
        url: "http: //localhost:3000/api/joke",
        method: "POST",
        json: true,
        body: {
            joke: "I'm a joke"
        }
    },
    function (error, res, body) {
        if (!error && res.statusCode == 200) {
            console.log(body);
        }
    });
```

Put:
```javascript
request({
        url: "http: //localhost:3000/api/joke/3",
        method: "PUT",
        json: true,
        body: {
            joke: "I'm an updated joke"
        }
    },
    function (error, res, body) {
        if (!error && res.statusCode == 200) {
            console.log(body);
        }
    });
```

Delete:
```javascript
request({
        url: "http: //localhost:3000/api/joke/3",
        method: "Delete",
    },
    function (error, res, body) {
        if (!error && res.statusCode == 200) {
            console.log(body);
        }
    });
```


* __Explain, using relevant examples, about testing JavaScript code, relevant packages (Mocha etc.) and how to test asynchronous code.__

TODO

* __Explain, using relevant examples, different ways to mock out databases, HTTP-request etc.__

TODO


##Resources
http://www.yorku.ca/nmw/facs1939f13/javascript_all/js_scriptingVSprogramming.html
http://stackoverflow.com/questions/101055/when-is-a-language-considered-a-scripting-language
https://www.toptal.com/nodejs/why-the-hell-would-i-use-node-js
http://blog.modulus.io/top-10-reasons-to-use-node
http://josh.zeigler.us/technology/web-development/experiences-with-node-js-porting-a-restful-service-written-in-java/
http://mobilenext.net/scaling-node-js-multi-core-systems/
http://www.sitepoint.com/how-to-create-a-node-js-cluster-for-speeding-up-your-apps/
http://cjihrig.com/blog/scaling-node-js-applications/
https://nodejs.org/api/cluster.html