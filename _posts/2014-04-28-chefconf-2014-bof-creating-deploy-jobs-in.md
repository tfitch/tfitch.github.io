---
title: "#ChefConf 2014 BoF - Creating Deploy Jobs in Jenkins"
date: '2014-04-28T23:29:00-07:00'
author: Tyler Fitch
categories:
- ChefConf
---
At #ChefConf 2014 I had the honor of leading a Birds of a Feather session on “Creating Deploy Jobs in Jenkins”.

![ChefConf 2014 BoF]({{ site.url }}/assets/chefconf_2014_bof.jpg)

My interest here is in deploying a web app to my Chef managed servers.  The problem I need to solve is for when you don’t know what servers you have because in the Cloud the servers come and go.  [Servers are cattle - not pets, right?](http://www.theregister.co.uk/2013/03/18/servers_pets_or_cattle_cern/)

### So what are our options?
* Doing it manually - bleh: Hardcoding server addresses in a Jenkins deploy job won’t work.  Well, it’ll work until you shoot your cow and spin up a new one.  Manually updating the Jenkins deploy job with the current server addresses sounds about as fun as herding cats and won’t help you [Continuously Deliver](https://www.amazon.com/gp/product/0321601912/ref=as_li_ss_tl?ie=UTF8&camp=1789&creative=390957&creativeASIN=0321601912&linkCode=as2&tag=tylfit-20).
* Use the Chef Server + knife status to lookup what servers you have.  When combined with roles + environments, this gets an accurate reading of what’s out there to deploy to.  I’ve found the output of knife status is also much friendlier to command-line fu than the knife search output.
{% highlight bash %}
knife status “role:webserver AND chef_environment:stage”{% endhighlight %} will give me the following output.

{% highlight bash %}
12 mins ago, ec2-57.186.102.29-1396555868.17, ec2-57-186-102-29.us-west-2.compute.amazonaws.com, 192.31.9.43, centos 6.4.
8 mins ago, ec2-57.82.206.136-1396555883.36, ec2-57-82-206-136.compute-1.amazonaws.com, 57.82.206.136, centos 6.4.
{% endhighlight %}

So I want to cherry pick the 3rd item of the comma-separated list, the hostname.  I could use awk, but found a simple example using cut and went with it.
I loop over the output of the knife status search like so
{% highlight bash %}
for h in $(knife status “role:webserver AND chef_environment:load”| cut -d’,’ -f3); do ssh -oStrictHostKeyChecking=no myuser@$h “uptime”; done
{% endhighlight %}
The `-oStrictHostKeyChecking=no` is there because I’ve maybe never deployed to this cow before.  Change the ssh “uptime” command to the necessary scp command and you’re deploying.

But it isn’t bulletproof.  If a server crashes and doesn’t unregister itself as an active node with the Chef server you’ll try to deploy to a place that is no longer valid.  One way to mitigate this is use the timestamp column in the `knife status` result that shows the last time the server phoned home.  If all servers are checking in every 15-30 mins and it’s been an hour since the Chef client last checked with the Chef server then skip trying to deploy to it.


### Amazon AWS API options:

During the discussion two options for Amazon Web Services were identified.

* First up, use the AWS API to query for the list of active servers to deploy to.  If you tag them or have other means to identify their role in your infrastructure you’ll know what’s active at that time.  I imagine other cloud hosting providers will have similar APIs available.
* Secondly, [Kennon Kwok](https://twitter.com/kennonkwok) from Chef tipped me off to a nice library from Netflix called [Edda](https://github.com/Netflix/edda/wiki) which “polls your AWS resources via AWS APIs and records the results. It allows you to quickly search through your resources and show how they have changed over time.”  In other words, Edda is wrapper for the AWS API that adds in historical data too (among other things).  Nice. 

Both of these options are nice a would be more accurate than polling the Chef server.  But in my environment I actually have a tool called [Scalr](http://www.scalr.com/) fronting my AWS account.  So I can create servers but do not actually have the account’s AWS secret key use the AWS API or Edda!  It’s a hard knock life in the corporate world of cloud billing and measures to consolidate billing for their numerous AWS accounts.

### Lesser known options:

Also discussed were the ideas of using [Apache Zookeeper](http://zookeeper.apache.org/), [Serf from Hashicorp](http://www.serfdom.io/) or [etcd from CoreOS](https://github.com/coreos/etcd).  Zookeeper was not highly rated for WAN setups with machines not on the same networks.  And there was not enough experience with Serf or etcd in the room to give them a proper evaluation. 

### The end result for me?

I’m still proceeding with my initial plan that uses Chef Server + knife status lookup command, but now know a few more options to use in the future to solve to create my deployment jobs in Jenkins.

Thanks to Kennon, Eric from Boston, Mahesh from Dow Jones, the Behance guys and the others that I didn’t get your names but joined in the conversation for the BoF.  I had a great time and hope you all did too.
