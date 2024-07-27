---
layout: post
title:  "Updating Stock Trade App"
date:   2021-02-13 13:25:30 +0100
categories: nodejs expressjs reactjs update
permalink: /updating-stock-trade-app
---

I've updated my portfolio app 'Stock Trade App' which is built using Expressjs for the backend and Reactjs for frontend. I've changed the Postgresql database in the first version for Mongodb in the current version. I just want to try mongoose package and see how it goes. It's pretty simple, I like it.

So the structure of the API now is more focus to resources and organize them: users, holdings and symbols. And I have files for models and controllers, of course for routes and testing those routes. I have one more for validate the users and for errors.

<img src="{{site.baseurl}}/images/posts/resources-users.png" alt="user resources" />

[Here](https://github.com/marialobillo/stock-trade/tree/master/api/resources/users) you can see that structure for User.


