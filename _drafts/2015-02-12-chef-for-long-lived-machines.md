---
title: Test Chef cookbooks for long lived machines
date: '2015-02-12T14:05:00-07:00'
author: Tyler Fitch
categories:
- chef
- testing
---
## Question:
How does one test Chef cookbooks for long lived machines.

Those unicorns, with the Cloud provisioners and short lived machines aren't entirely mythical, but they aren't wide spread either.

Corporate IT departments spin up a machine, and it's going to be your machine "forever".

It is easy to test and prove that version `1.4.3` of your cookbook will work on a new/clean machine.  But how do you know that version `1.4.3` of your cookbook is going to work on a node coming from version `1.4.2`?

## Answer:
Yeah, a simple answer would be pretty powerful.