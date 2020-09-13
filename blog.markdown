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
    
       <div>
        {% for post in site.posts %}
            {% assign currentdate = post.date | date: "%B %Y" %}
            {% if currentdate != date %}
                <p id="y{{currentdate}}">{{ currentdate }}</p>
                {% assign date = currentdate %} 
            {% endif %}
                <p>
                    <a href="{{ post.url }}">
                        {{ post.title }}
                    </a>
                </p>
        {% endfor %}
        </div>
    </div>
</div>



