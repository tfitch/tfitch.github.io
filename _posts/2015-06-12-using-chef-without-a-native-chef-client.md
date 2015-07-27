---
title: Using Chef without a native chef-client
date: '2015-06-12T10:32:12-07:00'
author: Tyler Fitch
categories:
- chef
---
What can you do when you have a piece of IT infrastructure that doesn’t have native support for the chef-client?

## TL;DR
Put a vanilla linux machine in the middle to manage the device using the device's native APIs.

### Options
A) If the device has Ruby available on it, go tough guy mode and add support for the device to the client.  This can be both highly lucrative and highly painful.

B) If the device has APIs exposed for managing it, then use that API!

C) ¯\\\_(ツ)_/¯  There’s more are multiple ways to tackle these types of problems!

In this post we’re going to focus on Option B.  Which from a Chef perspective looks a little this.

![Network Diagram]({{ site.url }}/assets/network-switches.png)

We’re going to need one machine to sit in the middle of this process and be the registered Chef client(s), and then interact with one or more network devices via their APIs.  In this example we’ll have five network switches we want managed by Chef.  To do this we will stick a vanilla Linux machine in between the Chef Server and the network switches as the "controlling node".

We’re going to end up bootstrapping the controlling node five times, generating different client.rb files each time.  Each client.rb represents a network switch that is going to be controlled by our cookbook via the controlling node.

Then we can use the `-c` flag of the chef-client executable to identify which client the controlling node will be running as.  It would look like this `chef-client -c /data/chef/client-switch-one.rb` or `chef-client -c /data/chef/client-switch-two.rb` and so on.  Remember this because if you’re going to setup the chef-client as a scheduled task, you’ll need to set it up five separate times and use this `-c` flag as the key differentiator.

Now that that clients are all setup we can being to write our cookbook in Ruby to make API calls to the individual switches and adjust their configurations as needed.  This isn’t necessarily easy and it means you’ll be working outside of [native Chef resources](https://docs.chef.io/resources.html) but you’ll be bringing more devices in line with being managed by Chef as a consistent interface.

The end result is, if you did a `knife node list` you’d get back five results for the switches you’re now managing as Chef nodes via your custom cookbook.  And you can adjust those five switches independently, even if there is just that single controlling node doing all the work.
