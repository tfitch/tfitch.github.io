---
title: Wherein one learns they're doing Chef cookbook dependency management manually
date: '2015-02-10T14:05:00-07:00'
author: Tyler Fitch
categories:
- chef
- berkshelf
---
## TL;DR
The [Chef DK](https://downloads.chef.io/chef-dk/) has [Berkshelf](http://berkshelf.com/) bundled in it to handle cookbook dependency management, so take advantage of that being available to you.

## Question:
How do I ensure all my cookbook’s dependencies are on the Chef server?

Using cookbook upload:
{% highlight bash %}
knife deps  cookbooks/alpha  | xargs knife upload

Updated cookbooks/alpha
{% endhighlight %}
As an aside to this, we have looked to place community cookbook source into a separate directory, and set the cookbook path in the knife config.  But the knife deps command does not seem to use that configuration. For example chef-client cookbook and related are in community_cookbooks dir rather than cookbooks folder.
{% highlight bash %}
knife deps  cookbooks/mongo —tree

cookbooks/mongo
--cookbooks/chef-client
----cookbooks/cron
----cookbooks/logrotate
----cookbooks/windows
------cookbooks/chef_handler
{% endhighlight %}

Does `knife deps` make use of the knife config(cookbook_path) and repo config or just use the metadata found in each of the cookbooks?

## Answer:
The command above is basically come straight out of our Chef docs from [https://docs.chef.io/knife_deps.html](https://docs.chef.io/knife_deps.html)

It is great that the instructions “from the source” are being used, *BUT*

When you're using `knife deps` it is definitely using the knife.rb to determine the chef-repo-path value.  I have not actually used `knife deps` myself to see how it'd handle two different directories as cookbook sources, but what I have read says `knife` will not handle that.  But this brings up a point I want to highlight which is the dependencies on the Chef Server vs. the local development machine.  `deps` is thinking like the server (or the client), and on the server the cookbooks are all in a single Org.  But on your local development machine, this correlation is not expected, or even required.  Local cookbook directories can be anywhere.  Dependencies like a community cookbook especially could be anywhere (or nowhere).  I store mine in *~/source/gh/opscode-cookbooks/cron* for example and my personal cookbook in *~/source/gh/tfitch/why-run-alerting* depends on cron, but the cron cookbook on my machine is no where "near it".

The `knife deps  cookbooks/mongo  | xargs knife upload ` command looks like you're trying to find all dependencies for the cookbook and upload them, right?  Then I'd like you to meet [Berkshelf](http://berkshelf.com/)!  Conveniently bundled in the Chef DK.

Berkshelf can pull your cookbook's dependent cookbooks down locally and upload them to your server without you actually having to see them.  Think Maven pulling down Java libraries or npm pulling down JS packages as an analogy.  Community cookbooks will get stored by default in a *.berkshelf* directory in your user's home directory, and shared as needed.  So if you have 5 cookbooks depending on the community cron cookbook, berkshelf just manages that for you, stores the one copy locally and when `depends 'cron'` comes up in any cookbook's metadata.rb, berks upload will make sure that requirement is met automatically on the Chef Server.

Potential caveat here, if you're modifying the [community cookbooks](https://supermarket.chef.io) after downloading from the Internet, then make sure you're using the `path: '/where/you/store/community_cookbooks'` setting for cookbooks in the *Berksfile*.  It'll prevent you from automatically using the community cookbook with the same name.  And under the hood, if you have a custom cron cookbook and also use the community cron cookbook, Berkshelf handles this in the *.berkshelf* directory for you so you can still use both.

Finally, Berkshelf supports custom installs of the [Chef Supermarket](https://github.com/opscode-cookbooks/supermarket) as a "lookup location" for community cookbooks.  Meaning, if you're running an install of the Supermarket behind your firewall and it has a couple cookbooks, maybe 'ntp' and 'hostsfile' that have the standard settings for all corporate servers.  Well, you can configure the *Berksfile* to know that when you say you need the ntp cookbook, it'll look at your Supermarket first, find it and use it.  This can be very helpful to grow usage of shared things in your company as you all grow with Chef, and save people the trouble of reinventing the wheel to solve typical configuration settings people encounter at your company.