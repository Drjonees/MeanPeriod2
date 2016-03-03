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
        * ..To be continued

Node.js:
![Node.js Performance](http://i.imgur.com/is7PVIg.png)


Java on Tomcat:
![Java on Tomcat Performance](http://i.imgur.com/Ya72Zmq.png)

[Source](http://josh.zeigler.us/technology/web-development/experiences-with-node-js-porting-a-restful-service-written-in-java/ "Source")



* __Node.js uses a Single Threaded Non-blocking strategy to handle asynchronous task. Explain strategies to implement a Node.js based server architecture that still could take advantage of a multi-core Server.__

Last semester, we talked alot about why we should be using threads. Since CPU's speed have more or less stopped increasing, the number of cores has increased. By using threads, we could really take advantage of the multi-core system.
Since Node.js is single threaded, it is only running on one core. 
To take advantage of the multi-core system, we need to launch a cluster of Node.js processes.
Node.js comes with a built in [Cluster Api](https://nodejs.org/api/cluster.html "Cluster Api").
The cluster module lets us create multiple processes that all uses the same server port.

```javascript
const cluster = require('cluster');
const http = require('http');
const numCPUs = require('os').cpus().length;

if (cluster.isMaster) {
  // Fork workers.
  for (var i = 0; i < numCPUs; i++) {
    cluster.fork();
  }

  cluster.on('exit', (worker, code, signal) => {
    console.log(`worker ${worker.process.pid} died`);
  });
} else {
  // Workers can share any TCP connection
  // In this case it is an HTTP server
  http.createServer((req, res) => {
    res.writeHead(200);
    res.end('hello world\n');
  }).listen(8000);
}
```




* __Explain, using relevant examples, concepts related to the testing a REST-API using Node/JavaScript + relevant packages__



* __Explain, using relevant examples, the Express concept; middleware.__



* __Explain, using relevant examples, how to implement sessions, and the legal implications of doing this.__



* __Compare the express strategy toward (server side) templating with the one you used with Java on second semester.__



* __Explain, using a relevant examples, your strategy for implementing a REST-API with Node/Express and show how you can "test" all the four CRUD operations programmatically using for example the Request package.__



* __Explain, using relevant examples, about testing JavaScript code, relevant packages (Mocha etc.) and how to test asynchronous code.__



* __Explain, using relevant examples, different ways to mock out databases, HTTP-request etc.__


##Resources
http://www.yorku.ca/nmw/facs1939f13/javascript_all/js_scriptingVSprogramming.html
http://stackoverflow.com/questions/101055/when-is-a-language-considered-a-scripting-language