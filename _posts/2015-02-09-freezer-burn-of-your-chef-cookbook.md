---
title: Freezer burn of your Chef cookbook
date: '2015-02-09T14:05:00-07:00'
author: Tyler Fitch
categories:
- chef
---
## TL;DR
Using the freeze command when uploading a cookbook is great!  But as soon as you do you *must* change the version of the cookbook in `metadata.rb` to the next release number or you’ll get upload errors like you’re seeing in the question here.

![Freezer Burn]({{ site.url }}/assets/freezer-burn.jpg)

## Question:
Imagine a  Chef cookbook, or role cookbook, with dependencies defined in metadata.rb and that each version of the cookbooks are to be frozen on the Chef server, e.g. alpha > delta.

Using cookbook upload:
{% highlight bash %}
knife cookbook upload alpha --freeze --include-dependencies
Uploading alpha      [0.2.2]
Uploading delta      [0.2.0]

ERROR: Version 0.2.0 of cookbook delta is frozen. Use --force to override.
WARNING: Not updating version constraints for delta in the environment as 
the cookbook is frozen.
WARNING: Uploaded 1 cookbook ok but 1 cookbook upload failed.
{% endhighlight %}
Here you see an error, but also the knife command completed successfully, which makes it hard to identify the error if used in a tool/pipeline.

## Answer:
Getting the upload error isn’t really a terrible thing.  Freezing your cookbooks means that version has been released and you’re ready to start working on the next one.  Perfectly inline with software development processes.  Freezing and releasing versions of your cookbooks is a good thing!  I like that you’re doing it.

The keys for good freezing and releasing of a cookbook comes down to following steps.

1. Tagging the release in source control.  In your example you would have tagged the delta cookbook as “v0.2.0”.
2. Upload the version of the tagged repo to the Chef server with the —freeze flag
3. Now, immediately bump the version to 0.3.0 (or 0.2.1 depending on your needs) and check it to source control.

Three steps to repeat often, sounds ripe for automation eh?  And there are a couple tools in the Chef ecosystem to automate this task.  First up a look at Knife Spork.  [http://jonlives.github.io/knife-spork/](http://jonlives.github.io/knife-spork/) It will be installed as part of the Chef DK ([https://downloads.chef.io/chef-dk/](https://downloads.chef.io/chef-dk/)).  Secondly is thor-scmversion from RiotGames.  [https://github.com/RiotGamesMinions/thor-scmversion](https://github.com/RiotGamesMinions/thor-scmversion) A description of what thor-scmversion can do is #3 on this article.  [https://www.rallydev.com/community/engineering/6-things-every-chef-user-should-do](https://www.rallydev.com/community/engineering/6-things-every-chef-user-should-do)

The #1 rule here, even if you’re *not* freezing your cookbooks when you release them (make the final upload to the Chef Server), because not everyone does and it’s not the default behavior of an upload, is to always bump up the version number as the first change after any cookbook release.  Always, and forever, bump the version number first!  What do we do first after releasing a cookbook?  Bump the version # and check it in.

One way to alleviate the upload errors during local dev if you’re able to work with VMs locally is by using Chef Zero ([https://www.chef.io/blog/2014/06/24/from-solo-to-zero-migrating-to-chef-client-local-mode/](https://www.chef.io/blog/2014/06/24/from-solo-to-zero-migrating-to-chef-client-local-mode/)).  Then you won’t actually be hitting the live Chef Server and needing to worry about uploading conflicts.  This will also technically mask any errors around editing a frozen cookbook, but it’ll also speed up your local development until you’re ready to upload to the server.  Then you’ll get the error about the frozen cookbook and then make your tweak to the version value in metadata.rb.

## Automate the automated release task

Finally on releasing cookbooks, if you’re able to have your CI machine have write access to the cookbook repos, you can have CI jobs do the Spork/thor-scmversion tasks for you.  You’ll create the job once and when it’s time to release a cookbook you just a push a button and all tasks are completed for you.

This is totally a “down the road” goal, but something to think about as you do your current work and make plans to go faster.  If you can’t have automated machines editing repos don’t worry about it.  I did *not* have this luxury at my previous job, so it wouldn’t surprise me if you don’t either.  Still manually running `knife spork` will be better than doing the three steps manually, as they can be error prone commands to type of “just right” with every release. I’ll look to expand this in to it’s own post/tutorial in the future.
