---
layout: page
title: Blog
permalink: /blog/
---

<div class="clearfix"></div>
<div class="row">
    <br/>
    <div class="jumbotron">
    <h2 class="display-3">About me</h2>
    <p class="lead">
        Hi, my name is Maria I am a Full Stack Developer passionate 
        for Javascript and clean code. I have experience with Node.js, 
        React.js, mysql, postgresql and Jest.
    </p>
    <p class="lead">
    I love building things. I really enjoy making applications and 
    websites, as I love doing sports and search new healthy recipes, 
    as easier as better.
    </p>
    <p class="lead">
    When I write code I like listen music like 
    <a href="https://www.youtube.com/watch?v=uqLEI_ht_fY" target="_blank">this</a>,
    because let me to concentrate on what I'm doing and not in all 
    noises around.
    </p>

    <hr class="my-4">
    <p>
        This website is in construction, so I think I'll be adding new content...
    </p> 
</div>

<div class="clearfix"></div>
<div class="row">
    <br/>
    <div class="jumbotron">
    <h2 class="display-3">Posts</h2>
    <p class="lead">
        <ul class="col-md-10">
            {% for post in site.posts %}
                <li>
                    <span>{{post.date | date_to_string }}
                    </span> &raquo;
                    <a href="{{post.url}}">{{post.title}}</a>
                    <div class="card border-info mb-3" style="max-width: 20rem;">
  <div class="card-header">Header</div>
  <div class="card-body">
    <h4 class="card-title">Info card title</h4>
    <p class="card-text">Some quick example text to build on the card title and make up the bulk of the card's content.</p>
  </div>
</div>
                </li>
            {% endfor %}
        </ul>
    </p>
    
    

    <hr class="my-4">
    
</div>


