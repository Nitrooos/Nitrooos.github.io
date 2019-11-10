---
title: Nitrooos | Myśli programisty
---

<aside class="authors__image">
  <figure>
    <img
      src="{{ site.baseurl }}/assets/nitrooos.png"
      alt=""
      width="188"
      height="188" />
    <br />
  </figure>
</aside>

# Cześć! Jestem Nitrooos.

Od ponad 4 lat zawodowo zajmuję się tworzeniem stron i aplikacji internetowych.
Aktualnie jako Senior Fullstack Developer w <a href="https://stxnext.com">
STX&nbsp;Next</a>.

Nic więc dziwnego, że nie znalazłem czasu na stworzenie "super fancy" strony
głównej. Prowadzę za to programistycznego <a href="/blog/">bloga</a> po
godzinach.

Moją pełną ścieżkę zawodową można prześledzić na
<a href="https://www.linkedin.com/in/bartosz-kostaniak-623b8bb0/">LinkedIn</a>.
A więcej o mnie poczytać <a href="/authors/nitrooos">tutaj</a>.

# Ostatnio na blogu
{% for post in site.posts limit:3 %}
  <article class="blog__post-lead">
    <h3 class="blog__post-title">
      <a
        class="app__link"
        href="{{ post.url }}">
        {{ post.title }}
      </a>
    </h3>
    <p>{{ post.excerpt }}</p>
  </article>
{% endfor %}
