---
title: Blog
identifier: blog
permalink: '/blog/'
---
{% assign polish_posts = site.posts | where: "language", "polish" %}
{% for post in polish_posts %}
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
