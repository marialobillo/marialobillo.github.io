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
    <p class="lead">
        <ul class="col-md-10">
            {% for post in site.posts %}
                <li>
                    <span>{{post.date | date_to_string }}
                    </span> &raquo;
                    <a href="{{post.url}}">{{post.title}}</a>
                    <div class="card border-warning mb-3" style="max-width: 20rem;">
                    <div class="card-header">Header</div>
                    <div class="card-body">
                        <h4 class="card-title">Warning card title</h4>
                        <p class="card-text">Some quick example text to build on the card title and make up the bulk of the card's content.</p>
                    </div>
                    </div>
                </li>
            {% endfor %}
        </ul>
    </p>
    
    

    <hr class="my-4">
    
</div>


