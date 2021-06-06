---
title: "Keyboard Navigator: Go into TypeScript!"
date: 2021-06-05 20:00:00.000000000 +02:00
identifier: keyboard-navigator-part-3
categories:
- en
- frontend
tags:
- chrome
- extension
- keyboard-navigator
excerpt:
  Another update on Keyboard Navigator development process - this time I
  finally made a transition from Vanilla JS into TypeScript as mentioned in the
  previous post. Also, pressing Ctrl + F doesn't show highlights anymore. Read
  to check the full update!
---
The main change in May was the installation of TypeScript in the repository.
I've also splitted the code into few modules so it's no more the whole logic
stored in a single file.

There were 3 pull requests I merged:

- [install TypeScript](https://github.com/Nitrooos/keyboard-navigator/pull/4){:target="_blank"}
- [split code into modules](https://github.com/Nitrooos/keyboard-navigator/pull/6){:target="_blank"}
- [ignore key presses with modifiers](https://github.com/Nitrooos/keyboard-navigator/pull/7){:target="_blank"}

## Install TypeScript

In this [pull request](https://github.com/Nitrooos/keyboard-navigator/pull/4){:target="_blank"})
I added a simple *package.json* file with *typescript*, *webpack* and
*ts-loader* packages defined in *"devDependencies"* section.

The *[tsconfig.json](https://github.com/Nitrooos/keyboard-navigator/blob/2cbfe1cfa7a8fa2d29c605d4aeddc386e536ae29/tsconfig.json){:target="_blank"}*
file has a *"DOM"* library defined in *"lib"* setting to recognize the types
coming from Document Object Model. Also, the *"noImplicitAny"* was set to
*"false"* in order to not throw any errors about lacking types (this was
improved in the next pull request **Split code into modules**).

The *[webpack.config.js](https://github.com/Nitrooos/keyboard-navigator/blob/2cbfe1cfa7a8fa2d29c605d4aeddc386e536ae29/webpack.config.js){:target="_blank"}*
is also pretty straightforward - it's configfured to start within
*src/contentScript.ts* file and load all *.ts, *.tsx and *.js files imported
from there using *ts-loader*. The resulting, single file *main-bundle.js*,
will be saved inside *build/* local directory.

## Split code into modules

The second [pull request](https://github.com/Nitrooos/keyboard-navigator/pull/6){:target="_blank"}
is about splitting the code, located till now in a single file
*contentScript.ts*. The code there has already been divided into few modules
(in a sense of
[JS module pattern](https://coryrylan.com/blog/javascript-module-pattern-basics){:target="_blank"})
and it was reflected when splitting the code into separate files.

Beside of that, I've added typing in the codebase, defined few models like
*Highlight*, *Point* or *AppState*, so the project could actually benefit from
TypeScript.

## Ignore key presses with modifiers

Very small
[pull request](https://github.com/Nitrooos/keyboard-navigator/pull/7){:target="_blank"}
containing 1 change: keydown events are handled by the extension only when
there are no modifier keys (Alt, Ctrl, Meta or Shift) pressed alongside with
regular keys. Thanks to this change, when user presses e.g. Ctrl + F (search
shortcut), no highlights are shown on the page.

## What are the plans to do next?

This time I didn't record any video, because the changes aren't mostly visible
to the end user. But, I will summarize the plans for developing
[Keyboard Navigator](https://github.com/Nitrooos/keyboard-navigator){:target="_blank"}
in the following weeks:

* improving the navigation algorithm (which is already in an advanced phase
with proper [pull request](https://github.com/Nitrooos/keyboard-navigator/pull/8/files){:target="_blank"}
open)
* disabling the extension when user types text in inputs/textareas etc. to
prevent accidental turning on highlights
* implement remembering the location of last selected highlight

If you're interested in the latest news from developing this extension, please
check the [open source repo](https://github.com/Nitrooos/keyboard-navigator){:target="_blank"}
on Github. You can create issues or propose changes to contribute in this
project if you want :)

Stay tuned, the next updates will appear soon!
