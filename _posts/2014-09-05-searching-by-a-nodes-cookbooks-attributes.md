---
title: Searching by a Node's cookbook's attributes
date: '2014-09-05T22:15:27-07:00'
author: Tyler Fitch
categories:
- chef
- attributes
---
We’re working on a method for self discovery of a group of servers being setup to be a MongoDB replica set.  And a specific replica set among an environment with multiple MongoDB servers and replica sets at that.

## TL;DR
A cookbook’s node attributes are uploaded back to the server and available to be searched against.  Here’s the secret to searching on the Node’s attributes defined in the cookbook.  In the cookbook we had `node[‘mongodb’][‘replica_set’]` but when we wanted to search for it we did `mongodb_replica_set:ReplSet1` - chaining the full name of the attributes with underscores.  Doesn’t entirely seem intuitive, but it is the way Search works with the underlying Solr engine in the Chef Server.

Out of all the nodes in a Chef server Organization we’re going to use three keys to identify the nodes/machines of a single replica set.

1) A Role - [http://www.getchef.com/blog/2013/11/19/chef-roles-arent-evil/](http://www.getchef.com/blog/2013/11/19/chef-roles-arent-evil/) For this I like having two roles, mongodb-primary and mongodb-secondary.  The difference will be the run_list.  Both will have the `recipe[‘mongodb’]` which will install, configure and startup MongoDB on the servers.  But the mongodb-primary role will have the run_list of `recipe[‘mongodb’],recipe[‘mongodb::primary’]` to install MongoDB and configure the replicaset.

2) The Environment - [https://docs.getchef.com/essentials_environments.html](https://docs.getchef.com/essentials_environments.html)

3) A replica set name, set via a cookbook attribute `default['mongodb']['replica_set'] = 'ReplSet1'`

Not everyone in the Chef community may love Roles and Environments, but this is a great use case for them.  Treating the combination of the three like a compound key in a database table, we’re able to identify like machines to be joined together in to the replica set.

### Searching for the Nodes
Using the above three values, what do the search commands look like?  First up from knife we’d have
{% highlight bash %}
knife search node "role:mongo AND chef_environment:dev AND mongodb_replica_set:ReplSet1"
{% endhighlight %}

This is nice to verify you’ll get the desired search results without having to run a cookbook and debug the search syntax by doing a node converge.

Now we’ve got the search syntax worked out, we’ll want to actually use this in a cookbook to create a replicaset configuration file.
{% highlight ruby linenos %}
# some stuff was probably above here
 
# chef_environment might look redundant, but it will find machine in the same environment this machine is configured for
secondaries = search(:node, "role:mongodb-secondary AND chef_environment:#{node.chef_environment} AND mongodb_replica_set:#{node.mongodb.replica_set}")
 
# pass them in to the template below
template '/apps/mongodb/conf/replicaset.js' do
    owner 'mongodb'
    group 'users'
    mode '0755'
    action :create
    variables({
        :replicaset => node.mongodb.replica_set,
        :secondaries => secondaries
        })
end
 
execute "mongodb-config" do
  command "/apps/mongodb/bin/mongo localhost:27017/test /apps/mongodb/conf/replicaset.js"
  action :run
  retries 6
  retry_delay 10
end
 
# and something is probably below here, but maybe not for the primary
{% endhighlight %}

{% highlight ruby linenos %}
# replicaset.js.erb file
rsconf = {
    "_id" : "<% replicaset %>",
    "version" : 1,
    "members" : [
        <% idCounter = 1 %>
        <%- @secondaries.each do |secondary| %>
        {
            "_id" : <%= idCounter %>,
            "host" : "<%= secondary.fqdn %>:27017"
        },
            <% idCounter += 1 %>
        <%- end %>
        // in order to gracefully handle trailing commas, put the known Primary last
        {
            "_id" : 0,
            "host" : "<%= node.fqdn %>:27017"
        }
        <% end %>
    ]
}
 
rs.initiate( rsconf )
{% endhighlight %}

There we have it.  We’ve found all the machines we want to be in the same replicaset and configured the replicaset by running the JS file on the Primary machine of the MongoDB replica set.

Remember, only one machine in the replica sets needs to run this script, the primary.  AND it will need to run it last, after all the nodes in the replicaset are running (you can’t add a machine to a replica set if it isn’t running and configured to be a mongodb yet).
