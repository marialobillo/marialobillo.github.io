---
layout: page
title: Blog
permalink: /blog/
---

<div class="clearfix"></div>
<div class="row">
    <br/>
    <div class="jumbotron">
    <h2 class="display-3">Posts</h2>

    {% for post in site.posts %}
                <p class="lead">
                    <span>{{post.date | date_to_string }}
                    </span> &raquo;
                    <a href="{{post.url}}">{{post.title}}</a>
                    <div class="card border-info mb-3" style="max-width: 20rem;">
  
                </li>
            {% endfor %}


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
                <p class="lead">
                    <span>{{post.date | date_to_string }}
                    </span> &raquo;
                    <a href="{{post.url}}">{{post.title}}</a>
                    <div class="card border-info mb-3" style="max-width: 20rem;">
  
                </li>
            {% endfor %}
        </ul>
    </p>
    
    

    <hr class="my-4">
    
</div>


