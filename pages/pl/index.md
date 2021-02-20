---
title: Nitrooos | Myśli programisty
identifier: index
permalink: '/'
---

<aside class="authors__image">
  <figure>
    <img src="{{ site.baseurl }}/assets/img/nitrooos.png" alt="nitrooos zdjęcie" />
    <br />
  </figure>
</aside>

# Cześć! Jestem Nitrooos.

Od niemal 6 lat zawodowo zajmuję się tworzeniem stron i aplikacji internetowych.
Aktualnie jako Senior Fullstack Developer w
[STX&nbsp;Next](https://stxnext.com){:target="_blank"}. Prowadzę także działalność
gospodarczą o nazwie **"Nitrosoft&nbsp;Bartosz&nbsp;Kostaniak"**, zarejestrowaną 
pod numerem NIP&nbsp;7773290104 oraz REGON&nbsp;367835571.

Nic więc dziwnego, że nie znalazłem czasu na stworzenie "super fancy" strony
głównej. Prowadzę za to programistycznego [bloga](/blog/) po godzinach.

Moją pełną ścieżkę zawodową można prześledzić na
[LinkedIn](https://www.linkedin.com/in/bartosz-kostaniak-623b8bb0/){:target="_blank"}.
A więcej o mnie poczytać [tutaj](/authors/nitrooos).

# Ostatnio na blogu
{% assign polish_posts = site.posts | where: "language", "polish" %}
{% for post in polish_posts limit: 3 %}
  <article class="blog__post-lead">
    <h3 class="blog__post-title">
      <a href="{{ post.url }}">
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
