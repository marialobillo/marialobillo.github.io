---
layout: post
title:  "Heroku is not your localhost"
date:   2020-08-29 19:04:20 +0100
categories: deploy heroku nodejs 
permalink: /heroku-is-not-localhost
---

It was my fault, I was trying to connect my backend and my frontend Stock trade app, and It wasn't working. When you try to deploy your app to Heroku there are some changes that you need to do. Everything looked perfect, no errors on my Heroku logs, but my Frontend was pointing out a different url, something on my localhost.

The way that I normally deploy to Heroku is using a buildpack, is very easy. Everything is done by default, you just need to pick up your buildpack and join your code, at least for a project using nodejs and reactjs.