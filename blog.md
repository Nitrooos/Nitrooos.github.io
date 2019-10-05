---
title: Blog
---
{% for post in site.posts %}
  <article class="blog__post-lead">
    <h1 class="blog__post-title">
      <a
        class="app__link"
        href="{{ post.url }}">
        {{ post.title }}
      </a>
    </h1>
    <p>{{ post.excerpt }}</p>
  </article>
{% endfor %}