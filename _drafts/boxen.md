---
layout: post
category: 
tagline: 
tags: [boxen, settings, software, automation, provisioning, puppet, homebrew]
---
{% include JB/setup %}

One of the main reasons to not reinstall or restart with a new machine is that you lose al your customisation settings.
Most notably your dot files, OS and program settings.

[Boxen](http://boxen.github.com/) is a tool to manage provisioning of development machines. (Only OSX out of the box, pun intended.)
Under the hood it uses Puppet and Homebrew to make sure all the files and programs are present. It was originally developed for in-house use at GitHubb and later released as open source.

With a declarative configuration you state which conditions your machine has to meet and boxen and Puppet make sure it is configured. In more technical terms, it creates a Directed Acyclic Graph (DAG) denoting which component depends on what other. Then it starts making sure each dependency is installed and configured before the next until everything is as you configured.
As a consequence of this, you can run boxen as many times as you like and depending on your configuration it also updates everything to the latest version. For example older software versions are updated and the latest version of your dot file repository is fetched.

Boxen is basically a wrapper around puppet targeting users machines. It makes it easy to define a default set of tools and configuration and makes the deployment of them reproducible. 

Let's get to business.
To get started you can checkout the [template](https://github.com/boxen/our-boxen) from the creators. See the [Bootstrapping](https://github.com/boxen/our-boxen/#bootstrapping) section to get you going.
For archival purposes I've put the command here as well:

```bash
sudo mkdir -p /opt/boxengit 
sudo chown ${USER}:staff /opt/boxen
git clone https://github.com/boxen/our-boxen /opt/boxen/repo
cd /opt/boxen/repo
script/boxen
```

In `Puppetfile` you find the programs that are deployed by default using `github-<programname>`


- why boxen
- uses
- quick wins:
  - dotfiles
  - symlinks
  - 