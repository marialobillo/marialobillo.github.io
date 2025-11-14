---
layout: default
title: Blog
permalink: /blog/
---

<div class="space-y-2 mb-8">
  <h1 class="text-4xl font-bold">Technical Blog</h1>
  <p class="text-gray-400 text-lg">Writing about software architecture, backend systems, and building scalable applications.</p>
</div>

<div class="border-t border-gray-800 pt-8"></div>

<div class="space-y-12 mt-8">
{% for post in site.posts limit:10 %}
<article class="pb-8 border-b border-gray-800 last:border-0">
  <h2 class="text-2xl font-semibold mb-2">
    <a href="{{ post.url }}" class="hover:text-blue-400 transition-colors">
      {{ post.title }}
    </a>
  </h2>
  <div class="text-sm text-gray-500 mb-3">
    {{ post.date | date: "%B %d, %Y" }}
    {% if post.tags %}
      <span class="mx-2">•</span>
      {% for tag in post.tags %}
        <span class="text-blue-400">#{{ tag }}</span>{% unless forloop.last %} {% endunless %}
      {% endfor %}
    {% endif %}
  </div>
  <p class="text-gray-400 mb-3">{{ post.excerpt | strip_html | truncatewords: 40 }}</p>
  <a href="{{ post.url }}" class="text-blue-400 hover:underline text-sm inline-block">Read more →</a>
</article>
{% endfor %}
</div>

<div class="border-t border-gray-800 mt-12 pt-12">
  <h2 class="text-2xl font-semibold mb-6">Archive by Year</h2>
  
  {% assign postsByYear = site.posts | group_by_exp: "post", "post.date | date: '%Y'" | sort: "name" | reverse %}
  {% for year in postsByYear %}
  <div class="mb-8">
    <h3 class="text-xl font-semibold mb-3 text-blue-400">{{ year.name }}</h3>
    <ul class="space-y-2 text-gray-400">
    {% for post in year.items %}
      <li class="flex justify-between items-baseline">
        <a href="{{ post.url }}" class="hover:text-blue-400 transition-colors">
          {{ post.title }}
        </a>
        <span class="text-sm text-gray-600 ml-4 flex-shrink-0">{{ post.date | date: "%b %d" }}</span>
      </li>
    {% endfor %}
    </ul>
  </div>
  {% endfor %}
</div>