---
layout: page
title: Updates
eyebrow: Workwarrior Updates
subtitle: Release notes, technical deep dives, and project news.
permalink: /updates
---

<ul class="updates-list">
  {% for post in site.posts %}
  <li>
    <a class="post-link" href="{{ post.url | relative_url }}">{{ post.title }}</a>
    {% if post.excerpt %}
    <p class="post-excerpt">{{ post.excerpt | strip_html | truncatewords: 30 }}</p>
    {% endif %}
    <div class="post-meta-row">
      <span class="post-date">{{ post.date | date: "%Y-%m-%d" }}</span>
      {% for cat in post.categories %}
        <a class="cat-tag" href="/categories/{{ cat | slugify }}">{{ cat }}</a>
      {% endfor %}
    </div>
  </li>
  {% endfor %}
</ul>

<div class="updates-footer-links">
  <a href="/categories">Browse by category →</a>
  <a href="/feed.xml">Subscribe via RSS →</a>
</div>
