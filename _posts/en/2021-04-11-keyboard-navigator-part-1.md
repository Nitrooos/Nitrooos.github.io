---
title: "Writing Chrome extension part I: Introduction"
date: 2021-04-11 20:00:00.000000000 +02:00
identifier: keyboard-navigator-part-1
categories:
- en
- frontend
tags:
- chrome
- extension
- keyboard-navigator
excerpt: 
  Do you wonder how to write a browser's extension? I too, so decided to learn
  it and share my experience in this topic with you. Let's start a mini cycle
  about writing Chrome extension called Kayboard Navigator!
---
In my daily work as a web developer I spend much time (if not most) inside the
browser's window. The problem I still have is navigating the page with keyboard
only. The purpose for it is saving time and my personal comfort, so
recently I decided to do sth with it. The post you're reading is the result of
my first attempts to solve that problem.

In the following posts we'll write a simple Chrome extension called Keyboard
Navigator. Its task will be to simplify page navigation using keyboard. The
final code from this post is available on my
[Github](https://github.com/Nitrooos/chrome-keyboard-navigator/tree/KEY-1_poc){:target="_blank"}
account, inside a dedicated repo called "keyboard-navigator", on a 
"KEY-1_poc" branch.

## Wait a while! I have some questions...

### Why did you choose Chrome?

Basically because Chrome is the browser I use everyday in work and it's the 
most popular one, so the knowledge gained by writing an extension is the most
beneficial.

### Aren't there any existing solutions to your problem?

Ehhh... Probably yes, but after a quick research I found only
[Vimium](https://chrome.google.com/webstore/detail/vimium/dbepggeogbaibhgnhhndojpepiihcmeb){:target="_blank"}
and even gave it a try. It allows you to press F key and see all links on the
page. Basically, there are small rectangles with key combinations displayed near
to each of them, e.g. "QW", "FP", "SN" and so on, generated probably randomly.
All you need to do is to press displayed combination and browser simulates 
clicking in given link/button. The problem with this approach is that you need to
read proper combination, then take a look on a keyboard, type it and go back with
a look on a monitor to actually see the effect. I want to simplify this pattern.

But yes, I could actually find sth better if I had searched a little bit longer.
Or, I can just write a new one :)

## How to start?

The best option is to start with an official
[tutorial](https://developer.chrome.com/docs/extensions/mv3/getstarted/){:target="_blank"}
from Google. Please refer that page to get a basic understanding how the
Chrome extension actually works. Here we start with the code of the example 
extension they present in the tutorial and which can be downloaded
[here](https://storage.googleapis.com/chrome-gcs-uploader.appspot.com/file/WlD8wC6g8khYWPJUsQceQkhXSlv1/SVxMBoc5P3f6YV3O7Xbu.zip){:target="_blank"}.

## What we want to achieve in this post?

Of course not the finished extension, but at least the script which listens to
F key press events and toggles highlighting of clickable element on the page
we visit.

### Adding content script

We need to start with adding a content script, so the script being executed 
directly on the page we visit. Its responsibility will be to listen to F key
press events and toggle highlights of clickable elements on the page. At least
for the time of this first post.

We add it by creating a file *contentScript.js* in the main dir. Next, we include

{% highlight json %}
"content_scripts": [
  {
    "matches": ["*://*/*"],
    "js": ["contentScript.js"]
  }
]
{% endhighlight %}

in our *manifest.json* file to inform browser to execute *contentScript.js*
on each page we visit with extension enabled.

### Finding clickable elements on the page

In order to locate all clickable elements on the page we create a simple function

{% highlight javascript %}
function queryClickableAll() {
  const clickableSelector = "a, button, input[type=\"button\"], input[type=\"submit\"], input[type=\"reset\"]";
  return Array.from(domDocument.querySelectorAll(clickableSelector));
}
{% endhighlight %}

and put it in *contentScript.js* file. The searched elements are all &lt;a&gt;
and &lt;button&gt; tags, but also &lt;input&gt;s of type "button", "submit" or
"reset". For now it's sufficient.

### Creating a highlight for given element

Then, for each of the clickable element found, we need to create a "highlight"
element. What it is? For now it's a simple, transparent rectangle with black borders,
positioned over clickable element on the page. To do it, we write a simple
mapping function:

{% highlight javascript %}
function createHighlightFromClickable(clickableElement) {
  const elementPosition = clickableElement.getBoundingClientRect();
  const highlight = domDocument.createElement("div");
  Object.assign(highlight.style, {
    background: "transparent",
    border: "2px solid black",
    height: elementPosition.height + "px",
    left: (elementPosition.left + domDocument.documentElement.scrollLeft) + "px",
    position: "absolute",
    top: (elementPosition.top + domDocument.documentElement.scrollTop) + "px",
    width: elementPosition.width + "px",
    zIndex: 999999999
  });
  return highlight;
}
{% endhighlight %}

All it does is creating a &lt;div&gt; element with black borders and transparent
background, with the same sizing and position as the element given in a parameter.
*z-index* set to such high value is to ensure (or at least to increase the 
probability ;) of actually displaying the highlight.

Now, we can search for all clickable elements on the page and create 
related highlights for each of them:

{% highlight javascript %}
const highlights = queryClickableAll().map(createHighlightFromClickable);
{% endhighlight %}

### Show and hide highlights on demand

That's still easy - we can just write a function for toggling highlights state
(shown/hidden) and a dedicated functions for actually showing/hiding them on the
page:

{% highlight javascript %}
function toggleHighlights() {
  self.highlightsVisible ? hide(self.highlights) : show(self.highlights);
  self.highlightsVisible = !self.highlightsVisible;
}

function show(highlights) {
  highlights.forEach(highlight => document.body.appendChild(highlight));
}

function hide(highlights) {
  highlights.forEach(highlight => highlight.remove());
}
{% endhighlight %}

Wait, what's *self.highlightsVisible*? Remember, we need to store the state of
highlights somewhere, and the *self* object is just an object for all the state
we need regarding the highlights. For now, it contains only a boolean flag 
which tells whether highlights are currently visible (= true) or hidden (= false).

### Listening for a key press event

I arbitrarily chose a F key press, but you can use whichever you want of course:

{% highlight javascript %}
window.document.addEventListener("keydown", (event) => {
  switch (event.key) {
    case "f": toggleHighlights();
  }
});
{% endhighlight %}

Here we register an event listener on "keydown" event, checking what key was
pressed and calling "toggleHighlights" function when the pressed key was F.
Then, the function adds or removes proper elements on/from the HTML page.

### Use JavaScript module pattern to wrap code

I know I could use class syntax or split the code between files and use ES6
import/export, write the code in TypeScript and create a build process... But
at the moment we have only few simple functions so I decided to only use a
well known JavaScript module pattern and enclose the state and functions in a
single "highlightsModule" object. You can read about that pattern more on a
Cory Rylan's [blog](https://coryrylan.com/blog/javascript-module-pattern-basics){:target="_blank"}.
By the way, I can recommend his site to all of you who want to learn interesting
stuff from the frontend world.

## So, what do we already have?

I recorded a short video with the extension in the state from this post in
action! It can't do anything useful now, but we can clearly see the script
properly recognizes clickable elements and highlights each of them. I think
it's a good start for the next article :)

<video controls>
  <source src="{{ site.baseurl }}/assets/videos/2021-04-11/result.mp4" type="video/mp4">
  Your browser does not support HTML video.
</video>

## Summary

It's only the beginning, but we've already written sth which produces visible
effects. I wanted to try creating Chrome extension and have to say I really
enjoyed writing the post you're reading now - can't wait to inform you
about next progresses!
