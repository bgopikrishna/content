---
path: /blog/Using-the-Node-JS-Cluster-Module
date: 2020-07-11T16:32:00.062Z
title: "Using the Node JS Cluster Module - Exploring Node JS Modules #1"
tags:
  - Node
  - JavaScript
description: "Usually, most of the processors we use are of having multiple threads and multiple cores. So to take advantage of these multi-threaded/multi-core processors, we'll use the cluster module."
cover: /assets/cluster-cover.jpg
coverImageCredits: '<span>Photo by <a href="https://unsplash.com/@ianjbattaglia?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Ian Battaglia</a> on <a href="https://unsplash.com/s/photos/server-network?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Unsplash</a></span>'     
---
## Introduction

As we know Node JS is single-threaded. Usually, most of the processors we use are of having multiple threads and multiple cores. So to take advantage of these multi-threaded/multi-core processors, we'll use the cluster module. 

According to Node JS docs 

> A single instance of Node.js runs in a single thread. To take advantage of multi-core systems, the user will sometimes want to launch a cluster of Node.js processes to handle the load. The cluster module allows easy creation of child processes that all share server ports.

In a nutshell with the help of cluster module, we can spin up multiple instances of a Node JS process which are called worker processes to handle the incoming requests.

![](https://mermaid.ink/img/eyJjb2RlIjoiZ3JhcGggVEJcbmFbSW5jb21pbmcgcmVxdWVzdHNdIC0tPiBiW1wiQ2x1c3RlciBNYW5hZ2VyIChNYXN0ZXIgcHJvY2VzcylcIl1cbmIgLS0-IGNbXCJ3b3JrZXIgIzEgKENoaWxkIHByb2Nlc3MpXCJdXG5iIC0tPiBkW1wid29ya2VyICMyIChDaGlsZCBwcm9jZXNzKVwiXVxuYiAtLT4gZVtcIndvcmtlciAjMyAoQ2hpbGQgcHJvY2VzcylcIl1cbmIgLS0-IGZbXCJ3b3JrZXIgIzQgKENoaWxkIHByb2Nlc3MpXCJdIiwibWVybWFpZCI6eyJ0aGVtZSI6ImRlZmF1bHQifSwidXBkYXRlRWRpdG9yIjpmYWxzZX0)

Here,

* The cluster manager is the parent process, which manages the workers. 
* Cluster manager doesn't execute any application code.
* Cluster manager distributes the incoming requests to worker processes. 
* All the network requests (Like API calls) are handled by the workers.
* All the workers are independent processes, they can be killed re-spawned without affecting other workers.

## Using the Cluster Module

As an example let's create a simple express app which calculates Fibonacci number.

Let's start by creating a folder named `cluster-example`.

Open it inside your terminal and run the following commands

```bash
$ npm init -y
```

It creates the `package.json` file.

Now install express using

```bash
$ npm install express
```

#### Without Cluster Module

Let create two files, First one `without-cluster.js` which doesn't use the cluster module.

```javascript
//without-cluster.js

const express = require('express');
const app = express();
const port = 5000;

//Calculate Fibonacci Number
function fib(n) {
  if (n < 2) {
    return n;
  }
  return fib(n - 1) + fib(n - 2);
}

app.get('/', (req, res) => {
  const result = fib(30);
  return res.send({ data: result });
});

app.listen(port, () => console.log(`App listening on port ${port}!`));
```

If you look at the above code, it's a simple express app which calculates the Fibonacci number and sends the result. To keep this post simple I'm not going to explain the Fibonacci number.

To start the app just run

```bash
$ node without-cluster.js
```

Now,  When you go to `localhost:5000/` you'll get the Fibonacci number as follows

```json
{"data":832040}
```

#### With Cluster Module

Let's create another file named `with-cluster.js` which uses cluster module.

```javascript
//with-cluster.js

const cluster = require("cluster");
const express = require("express");
const app = express();
const port = 5000;

//To check whether it's a cluster manager(Master Process)
if (cluster.isMaster) {
    //This method creates the worker processes
    cluster.fork();
    cluster.fork();
    cluster.fork();
    cluster.fork();  //Here we created four worker processes
} else {
   
    {/** Application Logic Starts **/}
    function fib(n) {
        if (n < 2) {
            return n;
        }

        return fib(n - 1) + fib(n - 2);
    }

    app.get("/", (req, res) => {
        const result = fib(30);
        return res.send({ data: result });
    });

    app.listen(port, () =>
        console.log(`App listening on port ${port} ${process.pid}!`)
    );
    {/** Application Logic Ends **/}

}
```

If you look at the above code, it's the same app which calculates the Fibonacci number but uses the cluster module. We're just wrapping our application code inside an if-else block. 

First, we're checking if it's a cluster manager(master process) or not by using the `cluster.isMaster` property. If it's a cluster manager, we're going spin up workers(child process) using `cluster.fork()` method. 

The number of workers created will be equal to how many times you called `cluster.fork` method. Ideally, for better performance, the number of workers should be created needs to be equal to the number of [logical cores](https://unix.stackexchange.com/questions/88283/so-what-are-logical-cpu-cores-as-opposed-to-physical-cpu-cores) of the processor (Mine has 8 logical cores, you can check that in your task manager or using `os` module by `os.cpus().length` ).

To start the app just run (Don't forget to kill the previous one)

```bash
$ node with-cluster.js
```

Now,  When you go to `localhost:5000/`  you'll get the exact Fibonacci number as the previous one.

```json
{"data":832040}
```

## Benchmarking Using Apache Bench (ab) tool

Now you might be thinking about what performance improvements we're getting by using cluster modules. To check that we'll be using [apache bench tool](https://httpd.apache.org/docs/2.4/programs/ab.html). 

As I'm using Ubuntu(WSL 2), I can install it by running

```bash
$ sudo apt-get install apache2-utils
```

### Benchmarking for the app which doesn't use cluster module

First, we'll check for the app which doesn't use cluster module.

To start the app

```bash
$ node without-cluster.js
```

Suppose we want to see how fast our server can handle 100 requests, with a maximum of 10 requests running concurrently. Now open another terminal and run the following command.

```bash
$ ab -n 100 -c 10 localhost:5000/ #Dont forget the trailing slash
```

Let's break down the above command.

* `-n`: Total number of requests to be made.
* `-c`: Maximum number of requests to run concurrently.

After running the above command I'm getting the following results on my machine. Note, it might be different on your machine. Results will be based on your processor, programs running on your machine etc.

Sorry for my poor editing skills 😁

![without-cluster.js results](/assets/inkedwith-out-cluster_li.jpg)

### Benchmarking for the app which uses cluster module

Now let's test for the app which uses the cluster module. Start the app by running

```bash
$ node with-cluster.js
```

Run the apache bench(ab)  tool.

```bash
$ ab -n 100 -c 10 localhost:5000/ #Dont forget the trailing slash
```

After running the apache bench tool again, I'm getting the following results.

![with cluster results](/assets/inkedwith-cluster_li.jpg)

### Comparison

Now let's compare both results (which are taken for this sample app on my machine)

|                                                                            | **With Out Cluster Module** | **With Cluster Module** |
| -------------------------------------------------------------------------- | --------------------------- | ----------------------- |
| Time Taken for tests to complete                                           | 1.589 seconds               | 0.644 seconds           |
| Requests per second                                                        | 62.95 req/sec               | 155.35 req/sec          |
| Time per request ([mean](https://www.purplemath.com/modules/meanmode.htm)) | 158.8 ms                    | 64.372 ms               |
| Time per request (mean, across all concurrent requests)                    | 15.887                      | 33.53                   |

Here we're talking only a few properties from the results. You can clearly see, how many more requests we can handle by in a second by using the cluster module. The more requests we're handling, the lesser time the users have to wait to process their requests.

Similarly, you can compare other properties as well. 

> Note: Generally,  For clustering we can use process managers like [pm2](https://pm2.keymetrics.io/) which provides some extra features over the cluster module.

Thank You.

## References

[Cluster Module](https://nodejs.org/dist/latest-v12.x/docs/api/cluster.html) - Node JS Docs

[Useful Tools: ApacheBench (ab)](https://dev.to/matthias/useful-tools-apachebench-ab-2lmo) By [**Matthias** ](https://dev.to/matthias)