---
layout: default
title: Blog
---

# Technical Blog

Writing about software architecture, backend systems, and building scalable applications.

---

## Recent Posts

{% for post in site.posts limit:10 %}
<article class="mb-8 pb-8 border-b border-gray-800">
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
        <span class="text-blue-400">#{{ tag }}</span>
      {% endfor %}
    {% endif %}
  </div>
  <p class="text-gray-400">{{ post.excerpt | strip_html | truncatewords: 30 }}</p>
  <a href="{{ post.url }}" class="text-blue-400 hover:underline text-sm">Read more →</a>
</article>
{% endfor %}

---

## Archive by Year

{% for post in site.posts %}
  {% assign currentYear = post.date | date: "%Y" %}
  {% if currentYear != year %}
    {% unless forloop.first %}</ul>{% endunless %}
    <h3 class="text-xl font-semibold mt-6 mb-3">{{ currentYear }}</h3>
    <ul class="space-y-2 text-gray-400">
    {% assign year = currentYear %}
  {% endif %}
  <li>
    <a href="{{ post.url }}" class="hover:text-blue-400 transition-colors">
      {{ post.title }}
    </a>
    <span class="text-sm text-gray-600">- {{ post.date | date: "%b %d" }}</span>
  </li>
  {% if forloop.last %}</ul>{% endif %}
{% endfor %}