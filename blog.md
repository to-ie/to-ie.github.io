---
layout: default
title: Blog
---

# Blog

<ul class="post-list">
  {% assign posts = site.posts | sort: "date" | reverse %}
  {% for post in posts %}
    <li>
      <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
      <span class="post-date-tag">{{ post.date | date: "%B %Y" }}</span>
    </li>
  {% endfor %}
</ul>
