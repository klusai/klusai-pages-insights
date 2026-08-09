---
layout: page
title: "Blog"
permalink: /blog/
---

<p class="eyebrow">Field notes from building kOS</p>

Engineering, decisions and measurements from the platform our use cases run on,
and the vertical products on top of it.

<ul class="post-list">
  {%- for post in site.posts -%}
  <li>
    <span class="post-meta">{{ post.date | date: "%b %-d, %Y" }}</span>
    <h3><a href="{{ post.url | relative_url }}">{{ post.title }}</a></h3>
    {%- if site.show_excerpts -%}{{ post.excerpt }}{%- endif -%}
  </li>
  {%- endfor -%}
</ul>
