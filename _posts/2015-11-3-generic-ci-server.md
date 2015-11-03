---
title: Generic Chef CI Instructions
date: '2015-11-03T13:15:12-07:00'
author: Tyler Fitch
categories:
- chef
- Continous Integration
- CI
- testing
---
I’ve been in discussion with a few of my ![Chef]({{ site.url }}/assets/chef-emoji.png) customers about how they are progressing on their DevOps Journeys.  A frequent point of discussion is their Continuous Integration (CI) servers, or lack thereof as the discussions highlight.

##TL;DR
I will not talk about a specific CI server platform here.  The goal of CI is always the same.  Check out source code, build, valid and deploy. How a tool gets there might have slight variances, but the steps to follow are the same.

## What’s missing and what problems will a CI server solve?

Let’s look at it from a high level.

It is not just you that needs the CI server.  It is your team, your organization and your company.  I can honestly say setting up the CI server is absolutely the best first step any team can take with automation.  Even more important than than starting to use Chef - yep I said it.

## Your CI server is the foundation of automation
I consider setting up your CI server to be a Sprint Zero task in Agile terms.  You want to start building your Minimum Viable Product on Sprint One?  Build the app how? Build the app where?

![Exactly]({{ site.url }}/assets/exactly.gif)

Feel free to finish reading this reading this article, but then you must go deploy your CI server immediately!

Here are some planning steps for your CI server.

* Don’t put the CI server on a box under your desk.  That is so 2009.
* Have lots of dedicated storage for your build artifacts. I am looking at TBs of space, especially if you are building Java apps with a 50MB WAR (or larger!) for every single build.
* Detach this storage from the build server.  Your CI server and build jobs should be scripted as part of you [Configuration Management Tooling](Chef.io) *hint* *hint*.  Your CI server should be able to come and go like all your other automated infrastructure.  But you need to keep your build artifacts!
* Better yet, along with your CI server, run an artifact server like JFrog's [Artifactory](http://www.jfrog.com/artifactory) to host all your build artifacts.

## The standard builds jobs
Your build pipelines for Java apps, Ruby apps, Chef cookbooks, etc will all look the same and have these types of jobs, in this order.

### Linting
Is your code well formatted?  Do you have unused variables?  Catch these deficiencies early on with quick linting tests.  Ideally your engineers will be running these tests locally before committing their code, but we can leave nothing to chance.

### Unit tests
Unit Tests will be longer running than the linting tests, but shorter running than Integration Test suites.  Unit tests will validate that your code says what it does (the code will install Nginx).

### Integration tests
Finally we will run our Integration Tests.  For Java, Ruby, Node, etc. applications Integration Tests will validate your compiled/converged code do what they say  In Chef this will be applying a cookbook to node and running a suite of tests against it, in other words, running the `kitchen test` command and finding out that Nginx is running as well as Nginx is listening on port 80 and 443.

### Deploy your build artifact
Once your linting, unit tests and integration tests have all passed you have a good check in and build artifact.  Automatically deploy the code to the proper location (this could be uploading a Chef cookbook to your Chef Server, a WAR file to an Artifactory instance or a Ruby Gem to rubygems.org).

## Your CI server doesn’t get bored or take shortcuts
And why do you want to take the time to set up all those build jobs I just described?  Because having humans do these tasks is problematic at best.  Humans get bored. Humans are tired. Those humans make misatkes. On the other hand, computers do the same thing you tell them to do, over and over and over again.

Even worse, humans will try circumvent security where it gets in your way.  “Let me just upload this one line change.”  Yeah - those are famous last words.  Your CI server on the other hand, won’t skip a step of your pipeline.  That “one line change” will be tested and only deployed once those tests pass.  All your rules will be followed exactly how your security teams wants them to be.  They are going to be really happy campers.

## Here is your example
I know I said this would be platform agnostic, and it was until now.  But below is a repo that will build you a CI pipeline to play with, even copy from.  All the words above are great, but having a toy to play with will definitely better meet some of your learning styles.

Your big payoff for reading this far?  You can kick start this whole process in a Chef specific way!  Check out the https://github.com/chef-solutions/pipeline repository on GitHub.  With it you can literally `kitchen converge centos-7` a functional pipeline to see an example of everything wired together.

Congrats! You can now further

![automate all the things]({{ site.url }}/assets/automate-all-the-things.jpg)
