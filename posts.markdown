---
layout: page
title: Posts
permalink: /posts/
---

<div class="home">
  {%- if site.posts.size > 0 -%}
    <!-- <h2 class="post-list-heading">{{ page.list_title | default: "Posts" }}</h2> -->
    <ul class="post-list">
      {%- for post in site.posts -%}
      <li>
        {%- assign date_format = site.minima.date_format | default: "%b %-d, %Y" -%}
        <span class="post-meta">{{ post.date | date: date_format }}</span>
        <h3 class="post-title">
          <a class="post-link" href="{{ post.url | relative_url }}">
            {{ post.title | escape }}
          </a>
        </h3>

      {{ post.excerpt | strip_html | strip_newlines | strip }}
      </li>
      {%- endfor -%}
    </ul>
  {%- endif -%}

</div>
