# Building a Gov Uk Frontend Service
Frontend services on the Multichannel Digital Tax Platform (MDTP) use a lot of libraries containing HTML, JavaScript and CSS to standardise how the UI looks so that it is consistent across Government services.
They are based off the [Government Design System](https://design-system.service.gov.uk).

## This Tutorial
This tutorial walks you through the creation of a Frontend service that can utilise common Gov UK shared UI elements. It covers these main concepts and technologies, some more in depth than others:
* Scala
* Using the [Play UI](https://github.com/hmrc/play-ui) library and [Assets Frontend](https://github.com/hmrc/assets-frontend)
* Creating a Frontend service with Play Framework
* Model View Controller pattern
* SBT

## Previous Knowledge
Its recommended you have some experience with the following, although it isn't completely required:
* Scala (tutorial [here](https://www.scala-exercises.org/scala_tutorial)), Java or similar
* [HTML](https://html.com/)

Some of the sections are purposefully vague and get more challenging as you progress. This is intended - your colleagues, Slack, Confluence, GitHub are all valuable resources!

## Why Scala & Play?
Reactive Programming is based around building applications that can meet the diverse demands of modern environments. Four main characteristics:
* Responsiveness: high-performance, with consistent and fast response times
* Resilience: fails gracefully, and remains responsive during failures
* Elasticity: remains responsive during varying workload
* Message Driven: non-blocking, back-pressure capable, asynchronous

## Play Framework
[Play! Framework](https://playframework.com) is an open source web app framework. It’s similar to other web application frameworks you may be familiar with like Spring MVC and Rails:
* model, view, controller-based
* comes with a lot of built-in support tooling (scaffolding, execution, dependency management, etc)
* based on the principle of convention over configuration

In other ways, it differs from those frameworks: it’s 100% stateless and is built to run on [Netty](http://netty.io) – a non-blocking, asynchronous application framework.

## SBT
[SBT](https://www.scala-sbt.org/0.13/docs/Getting-Started.html) (Simple Build Tool) supports the compilation of Scala code and integration with other frameworks we're using. It is similar to Maven which you may be familiar with. 

## Sections
* [Part 1](Part1.md) 
* [Part 2](Part2.md) 
* [Part 3](Part3.md) 