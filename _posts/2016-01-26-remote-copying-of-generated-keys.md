---
title: Remote Copying of Generated Keys
date: '2016-01-26T09:36:10-07:00'
author: Tyler Fitch
categories:
- Chef
- Artifactory
---
A customer asked me this question:

> I feel like I’m trying to accomplish something fairly common, but I can’t find any good documentation around it and I’m not sure of the correct design pattern to use here. On each machine during a chef-client run, I need to generate an SSL keypair, upload the public key to another server, and the run commands on that remote machine necessary to add the key to a TLS truststore. That other machine is managed by Chef as well, so it’s possible to deal with importing any new keys into the truststore during that machine’s chef-client run, but getting the key onto the machine is proving to be a little more complicated.
>
> Is the best way to deal with this just to issue an scp command through the execute resource that uploads the key to the TLS server or is there some more idiomatic way to upload a file to a specific location on a remote machine?

So I thought about the question for a while and came up with a possible solution to this.

##TL;DR
* Understand that the public key is an artifact and should be treated as such.
* Decided what the preferred method for artifact sharing is within your architecture.
* Use that method (probably something like [Artifactory](https://www.jfrog.com/artifactory/)) to make the public keys accessible via HTTP(S) for easy consumption by the tools that need it.

## Share the Public Key

Any server can generate its SSL key pair when it is being created.  Then when once you have the public key, add a bit of logic to the Chef cookbook (or configuration management tool of choice, but you know which way I lean) to upload the public key to your artifact server of choice.  Since I recommend Artifactory, consider using the [Artifactory gem](https://github.com/chef/artifactory-client) in your Chef cookbook to handle this piece of code.

When uploading the public key to Artifactory, use tactics to allow for systematic scripted uploading and subsequent downloading of the public keys.  I prefer to use a consistent naming convention like `/ssl_public_keys/<hostname>`.

Then on any remote machine that needs to add the public key for `<hostname_zyx>` to its TLS truststore, the remote machine will go find the public key on the Artifactory server in the `/ssl_public_keys/<hostname_zyx>` location and pull it down over HTTP(S).

Long story short (but not the TL;DR), a Chef recipe can generate the key pair on the server and a Chef recipe will be used to install the key on remote nodes, but Chef does not have to manage all the pieces of the puzzle between those two events.

## Alternate solutions
* You don't have to use Artifactory.  Instead you could use [Nexus](http://www.sonatype.org/nexus/) or just put the public key in a generic HTTP server.  Just make it a consistent pattern for retrieval.
* Check out a newer SSL key generator tool like [Let's Encrypt](https://letsencrypt.org/) that will be generate SSL keys from a recognized Certificate Authority (CA).  Which can eliminate some tools requirement to do the truststore import.
* Put the public key in a Key/Value tool like [Consul](https://www.consul.io/intro/getting-started/kv.html) for HTTP access.
* Or go all Chef and put the public key in a [Chef Data Bag](https://docs.chef.io/data_bags.html).  But like I said above, Chef doesn't have to manage all the pieces to this equation.  Using external tools like Artifactory, Nexus or Consul all users who are unfortunately not using Chef to manage all the things to still have simple HTTP access to the public keys.
* If you're all in on AWS - they just launched the [AWS Certificate Manager](https://aws.amazon.com/blogs/aws/new-aws-certificate-manager-deploy-ssltls-based-apps-on-aws/) which also looks like a solution to help deal with this.  But I'm not certain of this, so just like you should read more about it, I need to too.

## Lots of choices - find one that works for you
The list of alternate solutions is about as long as my description of how to setup one of the solutions (and I'm certain I missed some options).  So like most technology choices, it comes down to finding the solution that works for your application.  If you're not sure which one works, and that is totally okay, don't spend all your time just thinking about the options while trying to find why it won't work.  Just try one for two weeks.  It might work - it might not.  If you find the edge case where it is not going to work, then try the next option.  Eventually you will have a working solution for your needs and you'll know a lot more about the tool(s) because you have used them instead of just thought about them.
