---
layout: post
category:
tagline:
tags: [scala, java, code coverage, programming, qa, quality, sonarqube, sonar, maven, multi-module, scoverage, scalastyle]
title: Scala Amsterdam Meetup talk on mixing Java and Scala in a single module
---
{% include JB/setup %}

Last night I gave a presentation on some of the work I have been doing on incorporating Scala into an existing Java Maven project. I did a step by step approach on this while keeping code quality up. A large part was about how to achieve meaningful code coverage on the both of Java and Scala source files.
Tools such as SonarQube can help you to store metrics such as code coverage and issues from FindBugs and ScalaStyle.
Also it is useful to enable compiler flags that warn and help writing better code.

Here you can find the links to the slides and code:

- Slides - <http://blog.timmybankers.nl/scala-java-maven-slides>

- Code - <https://github.com/TimSoethout/scala-java-maven-code>

If you're setting up your own Scala/Java mix up, feel free to use these resources:

- scala-sonarqube-docker - <https://github.com/TimSoethout/scala-sonarqube-docker>

- sonar-scala-plugin <https://github.com/TimSoethout/sonar-scala/releases>

- xml-transform-maven-plugin - <https://github.com/TimSoethout/transform-xml-maven-plugin>

You can also see my previous blog post on the (older) details on the setup: <http://blog.timmybankers.nl/2015/06/07/Mixing-Java-Scala-With-Sonar>
