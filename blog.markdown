---
layout: page
title: Blog
permalink: /blog/
---
<div class="clearfix"></div>

<div class="container">
    <div class="row">
        <h4>Blog Posts</h4>
    </div>
    <div class="row">
        <ul class="col-md-10">
            {% for post in site.posts %}
                <li>
                    <span>{{post.date | date_to_string }}
                    </span> &raquo;
                    <a href="{{post.url}}">{{post.title}}</a>
                </li>
            {% endfor %}
        </ul>
    </div>
</div>
