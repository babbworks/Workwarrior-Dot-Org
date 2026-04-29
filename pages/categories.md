---
layout: page
title: Categories
eyebrow: Browse Updates
permalink: /categories
nav_exclude: true
---

<div class="categories-index">
  {% assign sorted_cats = site.categories | sort %}
  {% for category in sorted_cats %}
    {% assign cat_name = category[0] %}
    {% assign cat_posts = category[1] %}
    <div class="cat-group">
      <div class="cat-group-header">
        <a href="/categories/{{ cat_name | slugify }}">{{ cat_name }}</a>
        <span class="cat-count">{{ cat_posts.size }} {% if cat_posts.size == 1 %}post{% else %}posts{% endif %}</span>
      </div>
      <ul class="cat-preview-list">
        {% for post in cat_posts limit:3 %}
        <li>
          <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
          <span class="post-date">{{ post.date | date: "%Y-%m-%d" }}</span>
        </li>
        {% endfor %}
        {% if cat_posts.size > 3 %}
        <li class="cat-more">
          <a href="/categories/{{ cat_name | slugify }}">+{{ cat_posts.size | minus: 3 }} more →</a>
        </li>
        {% endif %}
      </ul>
    </div>
  {% endfor %}
</div>
