---
title: Nitrooos | Myśli programisty
identifier: index
permalink: '/'
---

<aside class="authors__image">
  <figure>
    <img
      src="{{ site.baseurl }}/assets/img/nitrooos.png"
      alt=""
      width="188"
      height="188" />
    <br />
  </figure>
</aside>

# Cześć! Jestem Nitrooos.

Od niemal 5 lat zawodowo zajmuję się tworzeniem stron i aplikacji internetowych.
Aktualnie jako Senior Fullstack Developer w
<a href="https://stxnext.com" target="_blank">STX&nbsp;Next</a>.

Nic więc dziwnego, że nie znalazłem czasu na stworzenie "super fancy" strony
głównej. Prowadzę za to programistycznego <a href="/blog/">bloga</a> po
godzinach.

Moją pełną ścieżkę zawodową można prześledzić na
<a
  href="https://www.linkedin.com/in/bartosz-kostaniak-623b8bb0/"
  target="_blank">
  LinkedIn
</a>.
A więcej o mnie poczytać <a href="/authors/nitrooos">tutaj</a>.

# Ostatnio na blogu
{% assign polish_posts = site.posts | where: "language", "polish" %}
{% for post in polish_posts limit: 3 %}
  <article class="blog__post-lead">
    <h3 class="blog__post-title">
      <a
        href="{{ post.url }}">
        {{ post.title }}
      </a>
    </h3>
    <p>{{ post.excerpt }}</p>
  </article>
  <hr/>
{% endfor %}

{% assign blog_page = site.data.navigation.polish | where: name, 'blog' | first %}
<div class="index__posts-below">
  <a class="app__button" href="{{ blog_page.link }}">Więcej postów</a>
</div>
