---
layout: post
category :
tagline:
tags : [docker, cloud, container, dockercon]
---
{% include JB/setup %}

![Dockercon 2014]({{ site.url }}/assets/images/docker_wave_whale.svg)

Dockercon had an Amsterdam edition this 4th and 5th December.
This post is to write down some notes and insights for this conference.

[Docker](https://www.docker.com) is a community open source project, with a company called Docker inc doing much of the work.


# Opening words

Although the conference is not that strict with timing, the content is interesting. The keynote starts off with an overview of the state and history of Docker. 700 contributors, 65000 forks, 67 million downloads.
The adaptation is also growing, with big companies starting to use and implement it.

To accommodate the big community the whole flow of pull requests is formalised and even falls under an SLA.

The problem Docker is trying to solve is the separation of Content creation and Production. The idea is to let the engineers focus on content creation and to make the flow to production as painless as possible.

The big open question is "How to do Orchestration?".

# ING keynote

At the moment I am employed by ING, so it is interesting to see us at more and more conferences. ING tries to set up an engineering culture and also shows carry out this view to the world. One of those things is speaking at conferences about the journey. One of those conferences is DockerCon. Our Chief Architect Henk Kolk spoke about going from waterfall to agile, pointing out that a bank is an IT company. Especially the quote about throwing out all project managers and business analysts made an impact on the crowd. I had many conversations about our culture when they found out I am working at ING. Really great my working environment is quite great.

# Docker Keynotes

There were many presentation by employees of Docker inc. Most of them were from CTO Solomon Hykes, who is a very good speaker. Interesting is the way Docker inc. works on their open source product. In only 22 months, the project has grown to more than 700 committers. This gives an interesting scaling problem.

Policy and process changes are done via pull requests. Instead of horribly long change documents from the governments, you actually use a diff to specify the changes. Then people can vote in the pull requests.

To scale the actual development functionality is grouped and assigned to a group of commiters, with a tech lead. This group decides to merge a pull request if it concerns some of their code.
Also all code written by employees of Docker Inc. has to go through the same procedure, and is thus evaluated by the community as well.
I think this is a great system, which is scalable and community driven.

Another great aspect was the release of a couple of new [http://blog.docker.com/2014/12/announcing-docker-machine-swarm-and-compose-for-orchestrating-distributed-apps/](Docker products). Docker Machine, which is like [Boot2Docker](https://boot2docker.io), but also aimed at docker hosts anywhere. Docker Swarm a cluster manager for docker. Docker Compose, inspired by [Fig](https://boot2docker.io), for easily managing multi container apps. Dockerhub Enterprise, basically dockerhub on premise.
For each product one of the team members actually working on the product did the introduction and the demo.

### Takeaways

It seems the Docker ecosystem is maturing really fase. Good practices as single purpose applications, immutable servers and microservices fit nicely in the promise of Docker. Docker Cluster management is really starting to get of, and hopefully will be ready so to use in a large production environment.

What I really like is the ability to let teams have the control. Docker enables them to pick any technology they are most productive in, as long as it can run in a linux process.

And more non-functionally: Many hipsters, free T-shirts and good food and atmosphere.

## Some technology to keep in mind

- Docker Machine, Swarm, Compose and Dockerhub Enterprise
- Flocker, Docker cluster manager with a networking solution over Docker hosts
- Mesos, clustering for Docker, partnering up with Docker Inc.
- Docker-builder, streamlining building and piblishing of images
