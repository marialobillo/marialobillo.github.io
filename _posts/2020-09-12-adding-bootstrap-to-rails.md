---
layout: post
title:  "I love Bootstrap and Rails together"
date:   2020-09-12 20:11:20 +0100
categories: bootstrap rails 
permalink: /i-love-bootstrap-rails
---

One of my favourites backend frameworks is Ruby on Rail. Because is simple, is powderfull, and on version 6 you can add your Javascript dependencies using yarn. So, so excited about it.

These are the notes for adding Bootstrap to Rails project:

1. Type this on the console:

<pre>
<code>
yarn add bootstrap jquery popper.js
</code>
</pre>


2. Your yourproject/config/webpack/environment.js file should look like this:

<pre>
<code>
const { environment } = require('@rails/webpacker')
const webpack = require('webpack')
environment.plugins.append("Provide", 
    new webpack.ProvidePlugin({
        $: 'jquery',
        jQuery: 'jquery',
        Popper: ['popper.js', 'default']
    })
)
module.exports = environment
</code>
</pre>

3. Now we are going to import Bootstrap on app/javascript/packs/application.js with the following line:

<pre>
<code>
import "bootstrap"
</code>
</pre>


4. On your gemfile you can add two gem more, for 'bootstrap' and 'jquery-rails', for my were these lines:

<pre>
<code>
gem 'bootstrap', '~> 4.5', '>= 4.5.2'
gem 'jquery-rails', '~> 4.4'
</code>
</pre>

I recommend go to https://rubygems.org/ and pick up the current versions.

5. Probably it's a good idea to re-start your rails server:

<pre>
<code>
rails s
</code>
</pre>
