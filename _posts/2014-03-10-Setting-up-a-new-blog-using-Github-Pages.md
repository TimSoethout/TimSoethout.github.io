---
layout: post
category : lessons
tagline: "gh-pages"
tags : [beginner, jekyll, blog, gh-pages]
published : true
---

{% include JB/setup %}

## A new blog

A long time ago I started out with a self-hosted Wordpress blog. It took some time to setup, but in the end I got it working on my server at home and was available together with some other sites through Apache.

It took a lot of time to correctly set up Wordpress having all the features I would want: some kind of post drafting and publication, a simple and easy web site, a good uptime.
Unfortunately the blog died a cold dead a while ago. When setting up a new blog I wanted more or less the same functionality and was about to setup a new self-hosted Wordpress blog when I first looked into alternatives.

My eye fell upon [GitHub pages](http://pages.github.com/), which is a **free** hosted service from GitHub on which you can serve simple static web pages. As long as your pages are thus statically created it is a great place to host them with good availability. Luckily they also provide a way to generate the pages using [Jekyll](http://jekyllrb.com/), which is a blog-aware static page generator. 
The blog itself is just a GitHub repository and the blog posts are files written in [markdown](http://daringfireball.net/projects/markdown/).

## Getting started

The process to get it up and running was easy:

- First create a repository on GitHub with you GitHub user: _user_.github.io (.com might also work).

- Check out your Jeckyll template of choice (I chose [Jekyll Bootstrap](https://github.com/plusjade/jekyll-bootstrap/))
  `$ git clone https://github.com/plusjade/jekyll-bootstrap/`

- Attach it to your newly created repo
  `$ git remote set-url origin git@github.com:user/user.github.io`

- And push
  `$ git push`

Your new blog is up and running in about 10 minutes.

Optionally you can also let your own domain refer to it very easily:

- Add a file `CNAME` to the root of your repository with the exact URL in the body of the file such as: `blog.timmybankers.nl`

- Configure the DNS of your domain to let the same URL point to _user_.github.io using as you might have guessed `CNAME`.

##Useful links

More useful links are:
- <pros.io>: A simple web interface that you can use to edit your markdown blog posts in the browser.
- themes: You can easily change the visual appearance of your blog by changing [themes](http://themes.jekyllbootstrap.com/).
