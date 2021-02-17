---
title: Comments on static website? That's possible with utteranc.es!
date: 2021-02-16 20:00:00.000000000 +02:00
identifier: utterances-comments
categories:
- en
- frontend
tags:
- utterances
excerpt:
  Do you have a technical, programming blog and want to add commenting
  functionality to it? If yes, then this post will be ideal for you!
---
Of course, when you make a personal blog, the commenting functionality is often
offered out of the box, e.g. [Wordpress](https://wordpress.com){:target="_blank"}
has it without any plugin needed. But what to do when your website is static,
without any server side processing? Then you can use one of many available
services, created exactly for handling such cases!

## What's available on the market?

When I was wondering how to add a comment section below the posts on this blog,
I knew only about [Disqus](https://disqus.com/){:target="_blank}, which is
definitely the most popular solution. It has even the free plan for personal
websites, but it displays ads to the visitors, which was not acceptable for me.
Also, injecting ads by third-party service slows your website down. Another
issue seems to be privacy concerns for users - Disqus profiles them for later
ads targeting. A quite good summary of all its problems can be found e.g.
[here](https://fatfrogmedia.com/delete-disqus-comments-wordpress/){:target="_blank}.
Thanks, just **no**.

An interesting alternative which doesn't track users nor display ads and it's
focused on performance is [Hyvor Talk](https://talk.hyvor.com/){:target="_blank"}.
As I wrote, it avoids most of Disqus flaws, but in the cost not being free to
use. And it's totally fine and understandable for me - I was ready to pay, but
I also realized the traffic on my personal blog is quite small. So I didn't 
want to end up paying for commenting system which serves 2 comments per year :D

Finally, I read about [utteranc.es](https://utteranc.es/){:target="_blank"} -
the comments widget, powered by Github. Or Github Issues, to be precise :) I 
swear, whoever invented this, is a genius!. Utteranc.es uses Github
Issues API to store comments and fetch them before displaying on your website. 
All you need to do is to embed a small script with few parameters,
like the name of public repo on Github (so utteranc.es can easily fetch/post
comments). Every page of your website generates a separate issue on given repo
(as soon as there's at least one comment posted). **Creating a comment on your
website creates a comment on proper issue in Github** repo and vice versa. And
that's it! You have everything as the comments on Github issues (like 
reactions) and totally for free :)

## How to enable utteranc.es on your website?

That's really simple, you start with adding a script tag to your website, which 
looks like this:

{% highlight html %}
<script
  src="https://utteranc.es/client.js"
  repo="<github-user-name>/<repo-name>"
  issue-term="pathname"
  label="comment"
  theme="github-light"
  crossorigin="anonymous"
  async>
</script>
{% endhighlight %}

Please remember to replace 
    
    <github-user-name>/<repo-name>

with actual names.

There are also few options to configure utterances, like the "issue-term",
"label" and "theme" attributes.

The "issue-term" tells utterances how to create the issue title - it can be
based on pathname, URL, page title, issue number, specific term etc.

    issue-term="pathname" | "url" | "title" | "og:title" | "<issue-term>"
    issue-number="<issue-number>"

The "label" attribute is optional and attaches a specific label to created issue
on Github.

The "theme" attribute is all about appearance and has few available options:

    theme="github-light" | "github-dark" | "github-dark-orange" | "icy-dark" | "dark-blue" | "photon-dark" | "boxy-light"

But, as always, the best is to read the [official docs](https://utteranc.es/){:target="_blank"}.

### Installing utteranc.es app on Github

The next step is to install utteranc.es app on Github, you can find it 
[here](https://github.com/apps/utterances){:target="_blank"}. After adding it
to your Github account, you need to select the repositories the utterances will
have access to in order to create issues and comments. And that's all! Your site
should now display comment section exactly where you put the &lt;script&gt; tag.

## What about disadvantages?

As everything, this solution to add comments to your website has also some
drawbacks. The most important is that it forces user who want to comment to have
the Github account. When you're a programmer and create a technical blog it's
probably not a problem (like for me on this blog). But when your audience is not from
IT world, you shouldn't use utterances. Most people don't know Github, won't
trust it and create an account specially for you :)

## Summary

In my opinion, utteranc.es is a great option to integrate comments on static
website as long as it's a programmer's blog. It's free, open source, without any 
ads nor user tracking addons, which compensates limited usage possibilities. But
still, for me, for now, is more than enough!
