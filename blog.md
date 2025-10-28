---
layout: default
title: Blog
---

# Blog

<ul class="post-list">
  {% assign posts = site.posts | sort: "date" | reverse %}
  {% for post in posts %}
    <li>
      <span class="post-date-tag">{{ post.date | date: "%B %Y" }}</span>
      <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
    </li>
  {% endfor %}
</ul>
