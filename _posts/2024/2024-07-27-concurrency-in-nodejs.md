---
layout: post
title: "Concurrency in Node.js: Understanding and Implementing"
date: 2024-07-27 12:18:00 +0100
categories: nodejs javascript concurrency
permalink: /concurrency-nodejs
---

In my latest exploration of Node.js, I delved into how this popular runtime handles concurrency. Building scalable APIs with Node.js has been an enlightening journey, and understanding concurrency is a crucial part of making your applications efficient and responsive.

Even though Node.js operates on a single thread, its event-driven, non-blocking I/O model allows it to handle multiple operations simultaneously. This is achieved through several mechanisms:

## Event Loop
The event loop is the core of Node.js's concurrency model. It continuously checks the call stack and the callback queue for tasks to execute. When the call stack is empty, the event loop moves tasks from the callback queue to the call stack.

## Callbacks
These are functions provided as arguments to other functions and executed when an asynchronous operation completes. This approach has been the traditional way to manage asynchronous operations in Node.js.

```
const fs = require('fs');

fs.readFile('file.txt', 'utf8', (err, data) => {
  if (err) {
    console.error(err);
    return;
  }
  console.log(data);
});
```

## Promises
These modern constructs provide a cleaner and more manageable way to handle asynchronous code, making it easier to write and understand.

```
const fs = require('fs').promises;

async function readFile() {
  try {
    const data = await fs.readFile('file.txt', 'utf8');
    console.log(data);
  } catch (err) {
    console.error(err);
  }
}

readFile();
```

## Worker Threads
For tasks that are CPU-intensive and might block the event loop, the worker_threads module allows you to run code in parallel threads, thereby avoiding bottlenecks.

```
const { Worker } = require('worker_threads');

const worker = new Worker('./worker.js');

worker.on('message', (message) => {
  console.log('Message from worker:', message);
});

worker.postMessage('Start');
```

## Cluster
To take advantage of multi-core processors, Node.js provides the cluster module, which enables you to create multiple processes that can share the same port, thus improving performance and scalability.

```
const cluster = require('cluster');
const http = require('http');
const numCPUs = require('os').cpus().length;

if (cluster.isMaster) {
  for (let i = 0; i < numCPUs; i++) {
    cluster.fork();
  }

  cluster.on('exit', (worker, code, signal) => {
    console.log(`Worker ${worker.process.pid} died`);
  });
} else {
  http.createServer((req, res) => {
    res.writeHead(200);
    res.end('hello world\n');
  }).listen(8000);
}
```