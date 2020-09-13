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
            </p>
        {% endfor %}
    </div>

    <div class="col-md">
        <ul>
        {% for post in site.posts %}
            {% assign currentdate = post.date | date: "%B %Y" %}
            {% if currentdate != date %}
                <li id="y{{currentdate}}">{{ currentdate }}</li>
                {% assign date = currentdate %} 
            {% endif %}
                <li><a href="{{ post.url }}">{{ post.title }}</a></li>
        {% endfor %}
        </ul>
    </div>
</div>



