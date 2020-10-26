---
title: Using icons in your webapp - bitmaps, spritesheet or SVG?
date: 2020-10-25 20:00:00.000000000 +02:00
identifier: icons-internet-aplications
categories:
- en
- frontend
tags:
- bitmaps
- icons
- svg
excerpt:
  "This time I wrote few words about using icons in web applications from
  developer's perspective: should we use bitmaps, SVG files or maybe a dedicated
  file (spritesheet)? Let's analyze available options!"
---
Almost every web application, soon or later, will need to display some icons.
They act as a simple graphical representation of actions the user can do. So,
when you see a pencil near to each record in a table, you know you can edit
the record. Question mark icon often means you can hover cursor above it and you'll
get some additional information, often in a form of a tooltip. As we see, icons
make an important role in your application. But how to store them in an
efficient way, so they become easily configurable and reusable?

## Available options

It turns out we can reach the same effect on the page in many different ways. In
this post I describe few options, listed below:

1. every icon as a separate image (bitmap)
1. one image with all icons (spritesheet)
1. one font file, containing icons as a single characters (often referred to as
*glyphs*)
1. one SVG file, defining all icons being used in application

Each of the method above (maybe beside the first one ;) has its advantages and
disadvantages, which I cover in this post.

## Separate bitmap for every icon

The simplest of available options, used by less experienced frontend developers.
The trap of creating new image per every new icon is easy to fall into -
it's sufficient that developer doesn't know what is the direction the
application will evolve. When we use only few icons that's ok, but when we start
adding more and more of them, we'll quickly spot an issue:

### Cons: Increasing the number of requests to the server

Each of them will (probably) finish quickly (icons generally shouldn't be very
big in size), but loading a site with 50 icons, the browser makes 49 too many
requests. Keeping in mind every web browser has a limitation on a number of 
open connections per domain (you can read more about it
[here](https://docs.pushtechnology.com/cloud/latest/manual/html/designguide/solution/support/connection_limitations.html){:target="_blank"}),
this is a serious issue. Even modern browsers have a limit of 6 open connections
per domain. So in any given moment maximally 6 icons can be fetched from server,
what leads to increased page load times.

### Cons: Icon's configuration not possible

Until the icon is used only in one place in the application, that's not a
problem. But, what about e.g. calendar icon, which we want to display in 4
places, including blue and red versions, big and small? The need to modify the
icon in a *simple* way (color and size) will probably end up putting 4 separate
files on server. And this is not what you want to see:

<pre>$ ls -l
total 4
-rw-r--r-- 1 user group  520 May 20  2019 calendar.png
-rw-r--r-- 1 user group  520 May 20  2019 calendar_big.png
-rw-r--r-- 1 user group  520 May 20  2019 calendar_red.png
-rw-r--r-- 1 user group  520 May 20  2019 calendar_red_big.png</pre>

### Cons: Loss of quality after scaling

It's obvious, but you need to remember - for simple icons it's worth to consider
using SVG files, so you'll avoid problems like below:

![
  Pencil icon in normal size and scaled up
]({{ site.baseurl }}/assets/img/2019-05-31/pixelized.png)
*The loss of quality after scaling bitmap icon<br/>(pencil icon downloaded from
[https://www.stockio.com/free-icon/maria-pencil](https://www.stockio.com/free-icon/maria-pencil){:target="_blank"})*

## All icons in a dedicated file (spritesheet)

It's a step further compared to method 1., because all needed icons can be
obtained from server in one HTTP request. But still, you cannot modify the icons
in size (without quality loss) nor color. This idea was used long time ago on
many internet websites and applications, so I mention it. Also, you can still
find proper files with icons (often called *spritesheets*) in the internet even
today:

![
  Bitmap file with icons
]({{ site.baseurl }}/assets/img/2019-05-31/icon_spritesheet.png)

It's interesting how to actually use such file, because you need to cut proper
piece of it before displaying an icon. You need to define proper CSS code doing
exactly that. For example, when you want to use "/assets/spritesheet.png" file,
you start with defining a generic icon class:

{% highlight css %}
.icon {
  background: url("/assets/spritesheet.png") no-repeat;
  height: 32px;
  width: 32px;
}
{% endhighlight %}

And after that classes for particular icons:

{% highlight css %}
.icon-question {
  background-position: 0 0;
}

.icon-info {
  background-position: -32px 0;
}

...

.icon-github {
  background-position: 0 -160px;
}

...

.icon-internet-explorer {
  background-position: -96px -192px;
}
{% endhighlight %}

And so forth, and so forth, defining the proper *background-position* for all the icons. There
still exist many websites online which allow you to find proper *background-position*
values, e.g. [http://www.spritecow.com/](http://www.spritecow.com/){:target="_blank"}.

## Embedding icons in a font file

While searching for any better alternatives, we'll eventually find the first
which is really worth considering: embedding icons as a single characters
(*glyph*) in the font file. Icons are included in a single file, in addition in
a loseless format. It means we can freely scale them, without loosing quality
and set any color we want. Exactly what we wanted!

This technique is used in a popular project
[Font Awesome](https://fontawesome.com){:target="_blank"}, containing over 1.5
thousand icons in a free version. It's a really great option to use it in your
application, especially when you don't need to use the icons provided by your
client. This set of icons is so rich that you'll always find something matching
your needs.

### Wait... It must have some disadvantages, right?

Nothing's ideal and this technique isn't an exception. Editing font files isn't
simple, you need to use a dedicated program to do it. The most popular one is
paid Adobe Illustrator, but you can also use a freely available
[Font Forge](https://fontforge.github.io){:target="_blank"}, which I also use.
Still, editing font files requires you to learn another utility and it's **(in
general) time-consuming**.

Unfortunately, another problem is processing font formats by different web
browsers. If we need to support Chrome, Firefox, Safari and IE 11 (what still
isn't as rare as you think and wish), then you'll have to export such font file
to **few formats**. And you'll need to remember about updating all of them with
every change of any icon, so they look the same in all browsers. Most often
they'll be TTF, WOFF2 and EOT formats (the last one for Internet Explorer).

If you don't feel upset with that, still you'll have to face with one thing, but
this can be fixed in a relatively simple way. I talk about the fact that
browser displays text using a system font when still waits for our custom one
to be fetched from server. This leads to situation, when during a short period
of time, instead of our icons on the page, there will be some weird characters
displayed. Why? Particular icons are often defined in places suppposed for
custom usage. Personally, I met using the space from 0x800 position. In our 
file there are icons defined, but in a system, standard font, there can be any
characters. Nervertheless, this behavior can be controlled by using
*font-display* CSS property.

### How to use icons defined in a font file?

Let's assume we defined our icons on positions from 0x800 up to 0x80F. We start
by defining a font:

{% highlight css %}
@font-face {
  font-family: "icons-font";
  src: url("/fonts/icons-font.eot");
  src: url("/fonts/icons-font.eot#iefix") format("embedded-opentype"),
  url("/fonts/icons-font.woff2?40259512") format("woff2"),
  url("/fonts/icons-font.ttf?40259512") format("truetype");
  font-weight: normal;
  font-style: normal;
}
{% endhighlight %}

And next, all classes with some specific prefix should be reserved for icons
(in our case the *"icon-"* prefix):

{% highlight css %}
[class^="icon-"]:before, [class*=" icon-"]:before {
  display: inline-block;
  font-family: "icons-font";
  font-style: normal;
  font-variant: normal;
  font-weight: normal;
  line-height: 1em;
  text-transform: none;
  width: 1em;
}
{% endhighlight %}

Then we can define particular icons:

{% highlight css %}
.icon-pencil:before                      { content: "\e800"; }
.icon-book:before                        { content: "\e801"; }
.icon-cross:before                       { content: "\e802"; }

...

.icon-plus:before                        { content: "\e80E"; }
.icon-minus:before                       { content: "\e80F"; }
{% endhighlight %}

From now we can use any of these icons by just adding a simple HTML code:

{% highlight css %}
<span class="icon-pencil"></span>
{% endhighlight %}

The proper color of an icon can be reached by adding another CSS class to
HTML element and defining a rule for it. For example, when we want the pencil
icon to be displayed blue, we may add "pencil" class to it and rule:

{% highlight css %}
.pencil {
  color: blue;
}
{% endhighlight %}

## Dedicated SVG file with icons

The option with all pros of the previous one and lacking its cons it's creating
one SVG file with all icons we need (yes, SVG format allows that!). By defining
icons in SVG, we get the possibility to:

<ul>
  <li>set any size of an icon (lossless format)</li>
  <li>set any color we want</li>
  <li>edit such file in a simple way (SVG is a regular text file!)</li>
</ul>

### How can we use this method?

We start with creating a new file, e.g. *spritesheet.svg* with the following
content:

{% highlight css %}
<svg>
  <defs>
    <g id="icon-pencil">
      <!-- first icon definition -->
    </g>
    <g id="icon-book">
      <!-- second icon definition -->
    </g>
    <!-- and so forth... -->
  </defs>
</svg>
{% endhighlight %}

Particular icons are defined inside *<g&gt;* elements with unique identifiers
(*id* attribute). We can copy icon definition from one of the freely available
sources (e.g. from [https://www.flaticon.com](https://www.flaticon.com){:target="_blank"})
and paste into that file. Possible is also creating and editing SVG files in the
[Inkscape](https://inkscape.org/){:target="_blank"} program.

### Alright, but how can I display such icon on my webpage?

First of all, you need to put proper file on your webpage. That can be done by
simply pasting content of the *spritesheet.svg* file into HTML document, e.g. as
a first child of the <body&gt; element. You can also add this file as a resource
to fetch in the *<head&gt;* section:

{% highlight css %}
<link rel="preload" href="spritesheet.svg" as="image">
{% endhighlight %}

Then, everywhere in the webpage we can use any of the defined icon:

{% highlight css %}
<svg viewBox="0 0 100 100" class="icon icon-pencil">
  <use xlink:href="#icon-pencil"></use>
</svg>
{% endhighlight %}

You only need to carefully put a path calling the icon: in this case it's just
*#pencil-icon* and it's a valid value when the icon's definition is included
directly in the HTML document. If we don't paste the content of the
*spritesheet.svg* file, but instead keep it as a separate file, then we'll need
to remember about a path to it in *xlink:href* attribute:

{% highlight css %}
<svg viewBox="0 0 100 100" class="icon icon-pencil">
  <use xlink:href="/spritesheet.svg#icon-pencil"></use>
</svg>
{% endhighlight %}

### Finally, styling SVG icon in CSS

SVG icons embedded in your webpage are an integral part of the DOM tree, that's
why it's possible to style them (and even their parts!) using a *fill* rule in
CSS:

{% highlight css %}
.icon-pencil {
  fill: red;
}
{% endhighlight %}

### Is it really an ideal solution to display icons?

The answer is: of course NOT! But the disadvantage I want to mention can be
safely ignored in some cases. I talk about browsers' support for SVG files.
Until we use the method of embedding (pasting) the content of SVG file directly
into HTML document, it's really good. IE 8+, Safari 5+, iOS 4.3+ and
Android 2.3+ sounds like a sufficient support for almost every project nowadays.
It becomes worse when we decide to put this file as a separate resource in the
<link&gt; section (which is generally a good idea, because this way the browser
caches the content with the following requests to your webpage). In such
scenario, we lose support for all versions of Internet Explorer (which
still is needed in many projects). On the other hand, a proper polyfill exists:
it's [svg4everybody](https://www.npmjs.com/package/svg4everybody){:target="_blank"}
library, which is always an option to consider.

## Icons in internet applications - summary

Nowadays we have many available options to embed icons in our webpages, choosing
the right one depends on the concrete case. For professional, commercial
applications I can recommend the method of embedding icons in font file and in
a dedicated SVG file. They require more work from the developer's perspective to
use, but they also have strong advantages: the possibility to change colors
and sizes of icons without decreasing quality are the real pluses. I hope this
article helped you get to know possible options in the topic!
