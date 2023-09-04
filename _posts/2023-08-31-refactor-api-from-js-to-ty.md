---
layout: post
title:  "Refactoring API from Javascript to Typescript"
date:   2023-08-31 18:30:00 +0100
categories: programming typescript javascript nodejs refactoring 
permalink: /refactoring-api-from-js-to-ts
---


I've been refactoring my old project [stock-trade](https://github.com/marialobillo/stock-trade) ( this is the 2nd time refactoring this project ) the first version of the project was written in Javascript, and using Postgresql as database. For the second version I decided to use a better structure (lib and resources folders), and I also decided to use mongodb as database, that was a huge change.

For this time I wanted to make the refactor more clear to work with it, so I decided to use Typescript, and I'm really happy with the result. I'm going to explain the structure of the project, and how I've been working with it.


```md
├── src
│   ├── controller
│   ├── interfaces
│   ├── middleware
│   ├── models
│   ├── routes
│   ├── services
│   ├── utils
└──-├──app.js
```

The project is in the src folder, and the app.ts is the entry point of the project. We could start with interfaces that define all the types and attributes that we are going to use in the project. That interfaces are used on the model, and from there we implement the services that we are going to use on the controllers, and later define the routes. That is basically the structure of the project.

It is a really big change using typescript instead of javascript because at least for me everything seems to be more organized, and I can see the types of the variables, and the attributes of the objects. I think that is a really good way to learn more about typescript, and how to use it in a project while you refactor an old project.

I'm going to leave the link of the project here, and I hope you can find it useful. If you have any question, or you want to talk about it, you can contact me on [twitter](https://twitter.com/maria_lobillo).
