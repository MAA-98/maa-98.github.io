---
layout: default
title: "Blog of MAA-98"
permalink: "/"
---

<p>By Marek Alexander Anthony <small>(né Marek Antoni Kurczyński)</small></p>

## Posts

<ul>
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a> - {{ post.date | date: "%B %d, %Y" }}
    </li>
  {% endfor %}
</ul>
