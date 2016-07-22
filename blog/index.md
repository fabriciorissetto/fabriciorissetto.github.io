---
layout: page
title: Últimos Posts
excerpt: "An archive of blog posts sorted by date."
search_omit: true
---

<style>
  .animated {
    -webkit-animation-name: none!important;
    -moz-animation-name: none!important;
    -o-animation-name: none!important;
    animation-name: none!important;  
    display: block!important;  
  }
</style>

<ul class="post-list">
{% for post in site.categories.blog %} 
  <li><article><a href="{{ site.url }}{{ post.url }}">{{ post.title }} <span class="entry-date"><time datetime="{{ post.date | date_to_xmlschema }}">{{ post.date | date: "%B %d, %Y" }}</time></span>{% if post.excerpt %} <span class="excerpt">{{ post.excerpt | remove: '\[ ... \]' | remove: '\( ... \)' | markdownify | strip_html | strip_newlines | escape_once }}</span>{% endif %}</a></article></li>
{% endfor %}
</ul>
