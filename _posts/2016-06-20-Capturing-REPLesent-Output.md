---
title: Capturing REPLesent Output
layout: post
category: null
tagline: null
tags:
  - scala
  - REPL
  - RELesent
  - AppleScript
---

{% include JB/setup %}

For my presentation at ScalaDays I used [REPLesent](https://github.com/marconilanna/REPLesent), a very neat tool that allows you to create slides in the Scala REPL and evaluate the code on the slides.

I wanted to also put this output in the resulting slidedeck for my presentation, so I wrote a script using AppleScript which captures the console window and goes through the slides, screenshotting every slide and it's REPL output.

You can find the source in the gist below. It assumes you have the presentation and REPL loaded in iTerm and that it is full screen with the tabs/toolbar hidden.

 {% gist 48c4ff62013b37e866b2e93aa676efc6 %}

 Have fun using it!
