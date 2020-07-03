---
layout: page
title: Blog
permalink: /blog/
---


<div class="row">
    <h1>Blog Posts</h1>
    <ul class="col-md-10">
        {% for post in site.posts %}
            <li>
                <span>{{post.date | date_to_string }}
                </span> &raquo;
                <a href="{{post.url}}">{{post.title}}</a>
            </li>
        {% endfor %}
    </ul>
    <p>Check out my <a href="/about/">About</a></p>
    <p>
        <img src="../images/me.jpg" atl="Maria" class="image-maria" />
    </p>
</div>
