---
layout: post
category: 
tagline: 
tags: [scala, scalatest, testing, scalacheck, unittest, programming]
title: ScalaCheck in ScalaTest
---
{% include JB/setup %}
{% include mathjs %}

Today I held a presentation at the Scala Community at my employer about ScalaCheck. 
ScalaCheck is a property based testing tool, which allows you to specify properties using predicates such as: $\forall s :: s.reverse.reverse \eq s$, which denotes that for all Strings `s` when you reverse `s` twice it should equal the original `s`.

Please see the [Slides](/PropertyBasedTestingScalaCheck/index.html), which are made in the exellent [RevealJS](https://github.com/hakimel/reveal.js/).

Maybe even more interesting are the code examples which can be found in the [code folder](https://github.com/TimSoethout/PropertyBasedTestingScalaCheck/tree/master/code) of [my github repo for the presentation](https://github.com/TimSoethout/PropertyBasedTestingScalaCheck).
There are a couple of files with accompanying tests. `PropertiesTest.scala` shows the ScalaCheck way of writing an executable test file which checks properties.
`ReverseExampleTest.scala` contain some simple properties using ScalaTest's `GeneratorDrivenPropertyChecks`, which using ScalaCheck under the hood.
`IbanExampleTest.scala` contains a more interesting example where an implementation that calculates IBANs from old bank account numbers is tested.