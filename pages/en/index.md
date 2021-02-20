---
title: Nitrooos | Dev thoughts
identifier: index
permalink: '/en'
---

<aside class="authors__image">
  <figure>
    <img
      src="{{ site.baseurl }}/assets/img/nitrooos.png" alt="nitrooos portrait" />
    <br />
  </figure>
</aside>

# Hi! I'm Nitrooos.

From almost 6 years I have worked on creating websites and internet applications.
Currently as a Senior Fullstack Developer in
[STX&nbsp;Next](https://stxnext.com){:target="_blank"}.

It's not surprising that I didn't find a time for creating a "super fancy"
main page. At least, I created and still maintain a developer's
[blog](/en/blog/) after hours.

My full career path can be found on
[LinkedIn](https://www.linkedin.com/in/bartosz-kostaniak-623b8bb0/){:target="_blank"}.
You can also read more about me [here](/authors/en/nitrooos).

# Latest posts on blog
{% assign english_posts = site.posts | where: "language", "english" %}
{% for post in english_posts limit: 3 %}
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

{% assign blog_page = site.data.navigation.english | where: name, 'blog' | first %}
<div class="index__posts-below">
  <a class="app__button" href="{{ blog_page.link }}">More posts</a>
</div>
