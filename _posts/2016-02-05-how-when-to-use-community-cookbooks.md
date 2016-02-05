---
title: How/When to use Community Cookbooks
date: '2016-02-05T14:26:43-07:00'
author: Tyler Fitch
categories:
- Chef
---
A common theme my customers will ask is "How/when do I trust Community Cookbooks"?

## TL;DR
* You learn to trust [community cookbooks](https://supermarket.chef.io/) by testing the cookbooks meets your needs
* Using a community cookbook can save you from reinventing the wheel. *Edited after input from [Noah Kantrowitz](https://twitter.com/kantrn/status/695752423781040128)*


Before diving in to just using the cookbook and rolling it out to your production environment(s) we want to make sure you have a solid plan to do this.

#### Considerations:
* Cookbook License
* Maintaining changes
* Dependencies

The first consideration when working in your company is if the community cookbook’s License is acceptable to be used in your company.  [IANAL](https://en.wikipedia.org/wiki/IANAL) - but I will say most all Chef community cookbooks have either the Apache 2 or MIT license and those are friendly to you and your company.  But verify the cookbook’s license before making it part of your applications.

Next thing to consider - do you want to just use the community cookbook or potentially fork it ([make your own copy](https://blogs.atlassian.com/2013/05/git-branching-and-forking-in-the-enterprise-why-fork/#what)) to use and improve on your own schedules.  Does the cookbook do 95% of what you need and you’ll make a change to get that other 5%?  Great - do it.  Also, please consider submitting your change back to the source to improve it, but if that is not possible that is okay too.

![From http://wondrouseyesalwayshaveastory.tumblr.com/post/44403549189]({{ site.url }}/assets/great_500.gif)

Last consideration - Does the community cookbook support every platform under the sun and you only need [RHEL](https://en.wikipedia.org/wiki/Red_Hat_Enterprise_Linux) support?  Fork that cookbook and pull out all the Windows dependencies to streamline your dependency needs.  One benefit I really like here - this will improve your converge times because less dependent cookbooks will need to be downloaded on to each node from the Chef Server.

## Deploying your copy of the Community Cookbook
Now that we’ve thought about the main things that need to be thought about - let’s put the community cookbook to work for you.  How do you know the cookbook is going to work for your app?  How do you know that *your* cookbook is going to work for your app?  [BY TESTING!](http://kitchen.ci/)

Run the community cookbook through the same pipeline you are running your cookbooks through.  Here is an [example pipeline](https://github.com/chef-solutions/pipeline) if you need one.  That said, the community cookbook may not have proper working linting and unit or integration tests and fail in your standard pipeline.  If this is the case either fix the issues (and submit the fix back as a [Pull Request](http://oss-watch.ac.uk/resources/pullrequest)) or skip the failing steps.  Skipping the failing validation steps should not entirely be a deal breaker.  At the end of the day we’re more interested in the [integration tests](https://learn.chef.io/test-your-infrastructure-code/) of *your* cookbook that uses the community cookbook passing than we are interested in just the tests community cookbook.  So if the community cookbook has the functionality you want but not the tests, it may not be a blocker, just a variable to consider.

## Go Forth
In summary, I say "Do use community cookbooks from the [Supermarket](https://supermarket.chef.io/)".  But don’t blindly implement them without knowing what they do.  Read the source code, follow the license and test the cookbook in your environments.
