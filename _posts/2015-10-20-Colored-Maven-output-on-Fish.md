---
layout: post
category:
tagline:
tags: [maven, fish, terminal, shell]
title: Coloured Maven output in fish shell.
---
{% include JB/setup %}

Here is a nice gist I found which will instantaneously give you simple colored maven output in the [fish shell](http://fishshell.com):

{% gist a9b74c537f667a8dd28e %}

Just put it in `~/.config/fish/functions/mvn.fish` and it will be used when `mvn` is used on your shell.
