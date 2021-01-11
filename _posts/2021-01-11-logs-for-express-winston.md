---
layout: post
title:  "Logs for Expressjs using Winston"
date:   2021-01-11 10:33:30 +0100
categories: logs logger winston expressjs 
permalink: /logs-using-winston
---

On [stock-trade](https://github.com/marialobillo/stock-trade) application I wanted to put all my logs on a file, there is a package, Winston, that make it very easy. So, for install the package, the project was already initialized, just typing this for installing the Winston package:

`npm install winston`

The easy way to use Winston package, you can import/require the package on index.js like this:

`const winston = require('winston')`

And then in order to use it you can type something like this:

`winston.info('Listening on port 3000')`

But we could go a little bit further, so we could make a module for logs in our utils directory like this, here the [code](https://github.com/marialobillo/stock-trade) and export the module like this:

```
const winston = require('winston')

module.exports = winston.createLogger({
  transports: [
      new winston.transports.Console({
          level: 'debug',
          handleExceptions: true,
          format: winston.format.combine(
              winston.format.colorize(),
              winston.format.simple()
          )
      }),
      new winston.transports.File({
          level: 'info',
          handleExceptions: true, 
          format: winston.format.combine(
              winston.format.simple()
          ),
          maxsize: 5120000, // 5 Mb
          maxFiles: 5,
          filename: `${__dirname}/../logs/application-logs.log`
      })
  ]
})

```
Here we are setting up the logs for console and for files, our files are going to be max 5 and the size of the file will be 5 Mb, also we are setting up the directory where our files will be created.