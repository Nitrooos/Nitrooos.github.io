---
title: Blog
identifier: blog
permalink: '/en/blog/'
---
{% assign english_posts = site.posts | where: "language", "english" %}
{% for post in english_posts %}
  <article class="blog__post-lead">
    <h1 class="blog__post-title">
      <a
        href="{{ post.url }}">
        {{ post.title }}
      </a>
    </h1>
    <p>{{ post.excerpt }}</p>
  </article>
  <hr/>
{% endfor %}
