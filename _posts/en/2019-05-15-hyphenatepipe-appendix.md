---
title: HyphenatePipe - the appendix
date: 2020-02-12 20:00:00.000000000 +02:00
identifier: hyphenation-appendix
categories:
- en
- frontend
tags:
- angular
- hyphenate
- pipe
excerpt:
  An appendix to one of the previous posts - what to do when using pipe in your
  Angular application suddenly decreases its performance? How exactly the pipe
  works in Angular?
---
One of my latest
[post](/en/frontend/2020/02/auto-splitting-words-hyphenation){:target="_blank"}
was about hyphenation (valid in the grammatical sense splitting the words in
syllables). The goal was there to acheived by implementing custom pipe, so the
*HyphenatePipe* class. I decided to create a small appendix to that post,
because the implementation provided there has some lacks. Keep calm, it's not a
critical thing, but you can be frustrated when you'll find it by yourself ;)
So what's actually going on?

**TLDR;**
An operation of creating the pipe from previous post is time- and
memory-consuming. Using this util intensively means Angular (potentially)
repeats this expensive operation, completely unnecessary. The provided
implementation needs to be modified in such way, that creating a new instance
of *HyphenatePipe* will consume less resources.

## How the pipe utility in Angular actually works?

As you can read in the official
[docs](https://angular.io/guide/pipes){:target="_blank"}, in Angular we can
distinguish two types of pipes:

* *pure pipe*, so the default one, by definition it doesn't contain any internal
state, which results in using **one instance** for all usages in the same view.
It means saving memory and, what's important in our case, the time needed to
create any new instances.
* *impure pipe*, which is a pipe with the possibility to keep an internal state.
We can make any pipe impure explicitly by just adding a flag { pure: false }
in the @Pipe decorator. The disadvantage is of course the need to create a new
instance for every usage, so it's definitely more memory- and time-consuming
option. Use it with caution!

### The problem definition

The pipe I described (*HyphenatePipe*) is *pure*, but it's easy to overlook one
fact. **Using pipes inside the children components** (even if they are part
of the same view in the application, and the pipes are pure) **will cause
creating the separate instances of pipe**. Each of them will handle a single use
in the view. Why it's so important? In order to understand this, you
need to think...

## What actually does the constructor of HyphenatePipe?

It creates a new instance *Hypher* class (the *hyphenator* field). This
operation requires importing the proper language pattern for splitting the
words and also parsing it. In the case of german language, the pattern is quite
big (about 70kB of data), in general they can be from few to almost 100kB.
Combining it with creating a new *HyphenatePipe* instance for every usage of
"... | hyphenate" construction delays the loading of an application :/

Unfortunately, it's so visible, that this problem was the major factor
responsible for long loading times of my application. In my case the pipe was
created 8 times in the homepage, every time loading and parsing the language
pattern file.

## HyphenatePipe problem solving

The proper solution is of course to import the language pattern only once. We
can acheive that by extracting the operation of creating the *Hypher* class
object into a dedicated service. In our case it will be named
*HyphenationPatternsService*. An implementation is fairly easy, I can post it
in one piece of code:

{% highlight typescript %}
import { Injectable } from '@angular/core';
import * as Hypher from 'hypher';
import * as german from 'hyphenation.de';

@Injectable()
export class HyphenationPatternsService {
  private hyphenator: Hypher = null;

  constructor() {
    this.hyphenator = new Hypher(german);
  }

  public hyphenate(word: string) {
    return this.hyphenator.hyphenate(word);
  }
}
{% endhighlight %}

At the end, we have to inject this service into the *HyphenatePipe* class:

{% highlight typescript %}
constructor(private hyphenationService: HyphenationPatternsService) { }
{% endhighlight %}

From now, creating new instances of *HyphenatePipe* is very fast and doesn't
affect visibly loading times of an application. The language pattern is always
loaded and parsed only once. Good job!
