---
layout: post
title: "Nepho 1.0.0 released!"
description: ""
category: "software"
tags: [nepho,release,software]
---
The HUIT Cloud Engineering team is proud to announce the release of [nepho 1.0.0](https://pypi.python.org/pypi/nepho)!

Nepho is a command-line tool that orchestrates the creation of complete working application stacks on virtual infrastructure. Initially targeting Amazon Web Services as well as Vagrant, Nepho abstracts datacenter creation, instance configuration, and application deployment into portable "cloudlets" that can be shared between developers and teams.

### Why might you want to use Nepho?

Nepho simplifies and streamlines the process of developing, testing, and deploying applications in the cloud.  For example, let's say your team is testing out a new application (a commercial Java web application, perhaps, or a Ruby on Rails application written by a contractor).  The process of provisioning hardware in your production datacenter can be onerous and time-consuming; however, infrastructure-as-a-service ([IaaS](https://en.wikipedia.org/wiki/Infrastructure_as_a_service#Infrastructure_as_a_service_.28IaaS.29)) cloud providers can offer a bewildering array of options in their attempts to provide as broad a range of services as possible.

Here's where Nepho comes to the rescue.  Nepho's cloudlets enable subject matter experts to capture technical best practices in code so that you can easily reuse these best practices in your own projects.  You don't need to be a cloud infrastructure expert in order to deploy your application in the cloud; instead, you can use a task-appropriate cloudlet to automatically set up your cloud infrastructure for you.  The HUIT Cloud Engineering team maintains several cloudlets; you can use Nepho to search for and install cloudlets.

Please take a moment to read our [Getting Started](https://github.com/huit/nepho/wiki/Getting-Started) guide, and give Nepho a try!  The guide includes a link to our issue tracker; please let us know how we can make Nepho a better tool for you to use.

{% include JB/setup %}
