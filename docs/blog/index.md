---
title: Blog
permalink: /blog/
---
<p class="eyebrow">blog</p>
<h1>Blog</h1>
<p class="hero__lede">Notes on what I'm learning — SQL patterns, pipeline decisions, and the occasional job-search lesson.</p>

<ul class="post-list">
  {% for post in site.posts %}
  <li>
    <span class="post-date">{{ post.date | date: "%b %-d, %Y" }}</span><br>
    <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
  </li>
  {% else %}
  <li>No posts yet — add markdown files to <code>docs/_posts/</code> named <code>YYYY-MM-DD-title.md</code>.</li>
  {% endfor %}
</ul>
