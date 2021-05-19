---
title: "Keyboard Navigator: Navigation between links"
date: 2021-05-12 20:00:00.000000000 +02:00
identifier: keyboard-navigator-part-2
categories:
- en
- frontend
tags:
- chrome
- extension
- keyboard-navigator
excerpt:
  In the last month I finally made a huge step in developing Keyboard Navigator!
  Now you can easily navigate between highlights and simulate click when
  pressing Enter. It still needs improvements, but you can already find it
  usable in some daily tasks.
---
After around a month since the last post about 
[Keyboard Navigator](https://github.com/Nitrooos/chrome-keyboard-navigator){:target="_blank"},
I want to  share a little update with you. I had few free evenings to work on
it and here's the summary what I achieved during that time.

There were 2 pull requests I merged:

- [navigation & simulating click events](https://github.com/Nitrooos/chrome-keyboard-navigator/pull/2){:target="_blank"}
- [selecting input elements and handling Escape](https://github.com/Nitrooos/chrome-keyboard-navigator/pull/3){:target="_blank"}

Now I want to shortly describe these changes and present the results to you :)

## Navigation & simulating click events

The whole code in *contentScript.js* was divided into 3 modules (still in the
meaning of the JS module pattern):

- navigatorModule
- domHighlightModule
- appModule

The *highlightsModule* disappeared, and its functions are now a part of *domHighlightModule*.
Why did I decide on such code splitting?

### navigatorModule

By adding a functionality of navigating between highlights, it made sense to
group such logic together. What I have in mind is finding nearest highlights
from the given one and selecting the central highlight on the page (which is a
starting point when user turns the extension on). 

### domHighlightModule

Here lie all operations working on DOM tree, like creating highlights, changing
background when given highlight is selected by the user, showing and hiding
highlights on the page.

### appModule

This is the main module for the extension - it keeps the whole state, like the
highlights array, selected highlight, state of the highlights (visible/hidden).
It initializes the extension by creating event listeners on keydown events and
calling proper functions from both *navigatorModule* and *domHighlightModule*.

## Selecting input elements and handling Escape presses

I started using the extension after merging the changes described above, but
after then realized few problems with it:

- I often wanted to type into some input/textarea fields on the page, but the
script didn't recognize them as clickable elements
- the highlights appear accidentaly when typing some data with keyboard (when
user presses the "f" letter), so it should be easy to hide them immediately

The next PR was just to address these issues.

Some elements like *&lt;select&gt;*, *&lt;textarea&gt;* and *&lt;input&gt;* are
now recognized as clickable, so they receive their highlights when extension is
on. Also, for all such elements (except some types of *&lt;input&gt;*), the 
extension simulates the *focus* event when pressing Enter, not *click*. This is
because emitting *focus* on them enables us to start typing text immediately
after pressing Enter!

Also, the new one event listener was added - now the extension listens to Escape
keydown and hides immediately all highlights on the page (if visible).

## Video presentation!

<video controls>
  <source src="{{ site.baseurl }}/assets/videos/2021-05-12/result.mp4" type="video/mp4">
  Your browser does not support HTML video.
</video>

## What needs improvement?

When I'm developing this extension, I have more and more topics in my head
what could be improved or implemented. For now, the most important issues are
(and I'm working on some of them already!):

* improving the navigation algorithm - the current one sometimes cannot find a
way to go to some highlights
* remembering the position of the last selected highlight - currently turning
the extension on always searches for the central clickable element on the page
* pressing Shift + Enter should open link in a new tab
* implement TypeScript support to make code more maintainable

I even made a "Project" on Github with the
[board](https://github.com/Nitrooos/keyboard-navigator/projects/1){:target="_blank"}
containing the list of all such issues and their statuses!

Stay tuned and wait for the next updates of the Keyboard Navigator development!
