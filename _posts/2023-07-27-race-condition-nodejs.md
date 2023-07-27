---
layout: post
title:  "Race Conditions in Nodejs"
date:   2023-07-27 12:18:00 +0100
categories: nodejs javascript raceconditions
permalink: /race-conditions-nodejs
---

Recently I've implemented a backend API project by FreeCodeCamp, which is url shortener. You can 
see the project [here](https://github.com/marialobillo/fcc-urlshortener)


The thing is even when we know Javascript in this case Nodejs is single-threaded we could have 'Race 
condition' happening in your project. The reason for this is because we are using a database to store our data, ( well for this project we were using mongodb and redis ), and the database is not single-threaded. So, we could have several simultaneous requests to our application. 

I found two articles about race condition on Nodejs that I found very interesting and I wanted to share with you. One of them is the Luciano Mammino [article](https://www.nodejsdesignpatterns.com/blog/node-js-race-conditions/), which is very well explained, giving us an idea about race condition on nodejs could be posible, picturing the way that 2 transactions could go, a beautiful example about olives and grapes which one solution could be forcing to the second process to wait for the frist one to finish.

```javascript
// source  -> https://www.nodejsdesignpatterns.com/blog/node-js-race-conditions/
async function main () {
  await sellGrapes() // <- schedule the first transaction and wait for completion
  await sellOlives() // <- when it's completed, we start the second transaction 
                     //    and wait for completion
  const balance = await loadBalance()
  console.log(`Final balance: ${balance}`)
}

```

The second one is about a real case described Matilde Duboille [here](https://blog.theodo.com/2019/09/handle-race-conditions-in-nodejs-using-mutex/), talking how she solved a race condition problem using mutex. In the article is defining race condition, and putting us in context about the problem that she was facing. 

You can find the code that she used to solve the problem [here](https://blog.theodo.com/2019/09/handle-race-conditions-in-nodejs-using-mutex/)

I hope you find this articles interesting and useful.







