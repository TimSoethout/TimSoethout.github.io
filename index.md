---
layout: page
title: Blog
tagline: 
published: true
---

{% include JB/setup %}

Hello, my name is Tim and this is my technology blog. I'm interested in all kinds of software related things and love to program, optimise and automate things. I like to call myself lazy, because I never want to have to do things twice.

This blog's main use is for me to not forget things I've spend way too much time figuring out and thus functions as a reminder and write down spot. If it is also of use to someone else, that's a great benefit.
Please don't hesitate to leave a message if you think I'm wrong or right, or if I'm unclear.

## Posts

<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
    {{ post.excerpt }}
  {% endfor %}
</ul>
