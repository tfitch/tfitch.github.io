---
title: Sending name/value pairs from Chef cookbook attributes to be dynamic Resource attributes
date: '2014-09-01T14:05:00-07:00'
author: Tyler Fitch
categories:
- chef
---
One of my customers had a question about using name/value pairs from his cookbook’s attributes as settings for a Resource.

## TL;DR
The name/value pairs in a cookbook Resource object are more than just a combination of attribute + value. More precisely, they’re a method_name + value combination. So making the method_name come from a dynamic source requires you use `send()` method of a Ruby Object to invoke the method, not just output a String equal to what the cookbook would look like if everything was hard coded.

### Long form:
Let’s start with the hardcoded cookbook recipe and see what that would look like.
{% highlight ruby linenos %}
# if nothing was dynamic
iis_pool 'MyAppPool' do
  runtime_version '12'
  max_proc 4
  thirty_two_bit false
  action :config
end
{% endhighlight %}

Now to convert all those Resource attributes to be dynamic let’s convert them in to Attributes of the cookbook
{% highlight ruby linenos %}
# attributes for the iis_pool
default['config']['setting']['runtime_version'] = '12'
default['config']['setting']['thirty_two_bit'] = false
default['config']['setting']['max_proc'] = 4
{% endhighlight %}

A first attempt at setting the attributes looked liked this, but it did not work
{% highlight ruby linenos %}
# outputs the same as hardcoded recipe but does *not* work
iis_pool 'MyAppPool' do
  node['config']['setting'].each do |setting, value|
    "#{setting}" "#{value}"
  end
  action :config
end 
{% endhighlight %}

At first pass this all looked correct to me too. I was stumped until my brilliant co-worker Steve Danna ([@SteveDanna](https://twitter.com/stevedanna)) shared some great Ruby/Chef knowledge with me. The correct way to achieve the goal would be this
{% highlight ruby linenos %}
# same result as hardcoded recipe but now driven by Attributes
iis_pool 'MyAppPool' do
  node['config']['setting'].each do |setting, value|
    send(setting, value)
  end
  action :config
end 
{% endhighlight %}

The key here is the `send()` method. [http://ruby-doc.org/core-2.1.2/Object.html#method-i-send](http://ruby-doc.org/core-2.1.2/Object.html#method-i-send)

When you do `#{setting} #{value}` then setting is just returned as a string and not *calling* the relevant setter method of the Resource object (iis_pool in this case).

I guess this surprised me, because when the cookbook is hardcoded you see the line thirty_two_bit false and it reads like two strings and not that thirty_two_bit is actually a method name being called.

Alas, now I know and you do too.
