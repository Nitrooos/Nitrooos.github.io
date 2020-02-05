---
title: Hyphenation - automatic word splitting on syllables
date: 2020-02-04 20:00:00.000000000 +02:00
identifier: hyphenation
categories:
- en
- frontend
tags:
- angular
- code
- hyphenate
- pipe
excerpt: Applications translated to some of the languages (e.g. Scandinavian,
  but also German) have to deal with properly displaying words longer than many
  sentences :) How to do that? Let's learn about hyphenation today!
---
Few times I had an opportunity to work with clients from Germany and always
the same problem appeared. Having to implement a fully responsive websites
generated a situation when some of the words, e.g. "Datenschutzerklärung",
"Zusammenfassung" or "Änderungswunsch" didn't fit into one line on mobile devices.
We could of course crop them or add a horiozntal scrollbar to make sure everything
is displayed. But, let's be honest, webiste prepared in one of such ways wouldn't
be truely professional. The solution for our problem was to implement hyphenation -
valid (in the grammatical sense) splitting of such words in syllables.

## The basic idea

We want the solution to enable mechanism for particular words or sentences
directly in HTML template, best with the possibility to pass some additional,
configuration options, like splitting only N longest words or words longer then M
characters. I will show you the implementation in Angular (using the pipe utility),
but you could easily move the idea into any other framework.

The usage of our pipe will look like this:

<pre>
    &lbrace;&lbrace; 'Datenschutzerklärung' | hyphenate &rbrace;&rbrace;
    &lt;!-- displays Da•ten•schutz•er•klä•rung -->
</pre>

where • means a special &amp;shy­­ HTML entity, not being displayed by browsers, but
defining points in which they can split the words when needed.

An actual mechanism of splitting words will be handled by the "hypher" library
[https://github.com/bramstein/hypher](https://github.com/bramstein/hypher){:target="_blank"},
designed to make it in a grammatical valid way.

## Side note: what is a pipe in "new" Angular?

Before I move into the actual implementation I want to write few words about pipes.
It's an utility which transforms some input value in a defined way, resulting
in some output value, ready to be displayed in HTML template. In order to create
a new pipe, we only have to declare a new class implementing the PipeTransform
interface (with the "transform" function). In Angular, there are some standard
transformations defined, e.g. CurrencyPipe and DatePipe, which can be used in the
following way:

<pre>
    &lbrace;&lbrace; 1.23 | currency }}
    &lt;!-- displays '$1.23' -->

    &lbrace;&lbrace; dateObj | date:'medium' }}
    &lt;!-- displays 'Jan 27, 2019, 10:40:42 PM' -->
</pre>

Our implementation will also use the pipe utility.

## Now to the implementation details!

### Installing the necessary packages

We will start with installing "hypher" library and all language patterns needed
(only German in this case):

<pre>
    npm install hypher hyphenation.de
</pre>

### Template of the HyphenatePipe class

Then we create a file with pipe's definition, e.g. hyphenate.pipe.ts, in which
we import "hypher" library and proper language pattern:

{% highlight typescript %}
import * as Hypher from 'hypher';
import * as german from 'hyphenation.de';
{% endhighlight %}

Directly below we can start defining HyphenatePipe class:

{% highlight typescript %}
/**
 * Hyphenates given text, based on Hypher library
 * @example
 *  'Finanzierungsanfrage' | hyphenate
 *  formats to: 'Fi-nan-zie-rungs-an-fra-ge'
 *  (with ­&shy; entities in place of hyphens)
 */
@Pipe({name: 'hyphenate'})
export class HyphenatePipe implements PipeTransform {
{% endhighlight %}

The @Pipe decorator tells the compiler which type of a class we want to implement.
In parentheses we pass a config object with the "name" key, meaning the name we
want to use to identify our pipe in the templates. The important thing is also
pointing an implementation of PipeTransform interface, containing a "transform"
method mentioned before. On the top I added the comment in the style of jsdoc,
which I wrote a separate blog [post](/en/frontend/2020/01/jsdoc){:target="_blank"}
about some time ago :)

Then we have an initialization section:

{% highlight typescript %}
private hyphenator: Hypher = null;
private hyphenChar = '\u00AD';

constructor() {
  this.hyphenator = new Hypher(german);
}
{% endhighlight %}

Here one word of explanation: in order to use "hypher" library, we need to create
the instance of Hypher class with the language pattern as a parameter. So that's
why you can see a "hyphenator" field here. The "\u00AD" character is the so called
"soft hyphen" (&amp;shy­ HTML entity). This character is not displayed by a browser,
but it marks a place where it can split the word on syllables when needed.

{% highlight typescript %}
/**
 * Hyphenates given text
 * @param {string} text Text to hyphenate
 * @param {HyphenateOptions} options
 *  Optional. Additional options can be specified here.
 */
transform(text: string, options: HyphenateOptions = {}): string {
{% endhighlight %}

The most important method is "transform", which has 2 parameters: "value" of the
string type, so the input value for a pipe which will be transformed, and "options"
of the HyphenateOptions, so the options defining some additional behavior.

### Additional options for pipe - HyphenateOptions interface

{% highlight typescript %}
/**
 * @desc Options which can be given into hyphenate pipe
 * @prop {number} onlyNLongest Hyphenate only N longest words from given text
 * @prop {number} longerThan Hyphenate only words longer than N characters
 */
interface HyphenateOptions {
  onlyNLongest?: number;
  longerThan?: number;
}
{% endhighlight %}

This interface means that our pipe can optionally split only N longest words in
the sentence or only the words longer than M characters. These options can be
passed using the construction:

<pre>
  value | hyphenate:{ longerThan: M, onlyNLongest: N }
</pre>

### The culmination point, so "transform" method

{% highlight typescript %}
transform(text: string, options: HyphenateOptions = {}): string {
  const words = text.split(/\s+/);
  const hyphenateNLongest = Math.min(
    words.length, options.onlyNLongest || words.length
  );
  const hyphenateLongerThan = options.longerThan || 0;
  const wordsToHyphenate = words
   .concat()
   .sort((word1, word2) => word2.length - word1.length)
   .slice(0, hyphenateNLongest)
   .filter(word => word.length > hyphenateLongerThan);
  return words
    .map(word => {
      if (wordsToHyphenate.indexOf(word) !== -1) {
        return this.hyphenator.hyphenate(word).join(this.hyphenChar);
      }
      return word;
    })
    .join(' ');
}
{% endhighlight %}

Because passing any additional options is not required, by default all words in
the sentence are splitted. This is reached by setting the onlyNLongest option
to the number of the words and longerThan to 0.

By using the "concat" method on the "words" list we create its copy, which is
then sorted. This is needed, because "sort" method changes the order of
list's elements.

Then we take only N longest words and filter the list so it contains only the
words longer than M characters. The process of splitting on syllables
(the main "return" instruction) starts with mapping the "words" list from the
input. In this step (.map(word =&gt; …) operation) we check if we actually should
split given word (so if it's inside the wordsToHyphentate list). If yes, then we
call the "hyphenate" method on "this.hyphenator" object (so the object from the
library), and the resulting syllables list is joined with the character of
*soft hyphen* (&amp;shy­ entity). In the other case (so when the word shouldn't
be splitted) we return it without any modifications. The last step is to join
the list of words with a space (.join(' ')).

## Summary

I think the solution I presented in this post is quite elegant and not only I
like it :) The situations, in which such utility can be helpful are not so
rare - that's what comes from my experience. The possibility of defining custom
pipes, embedded in Angular, is a good choice for it. Additionaly, optional
parameters, which we can pass to "hyphenate" pipe makes it possible to create
an extensible solution for this problem. Thank you for reading the whole article,
have a good day!
