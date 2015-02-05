---
title: Installing Sun Java 1.5 with Chef
date: '2013-06-28T21:25:00-07:00'
author: Tyler Fitch
categories:
- chef
- java
meta:
- opscode
---
In trying to use the [Chef community cookbook for Java](http://community.opscode.com/cookbooks/java) I found it to only work with Java 6 and Java 7.

I was trying to create a cookbook to recreate an existing environment at work I need a scripted install of the [Sun 1.5 JDK](http://www.oracle.com/technetwork/java/javasebusiness/downloads/java-archive-downloads-javase5-419410.html).

So I had to bail entirely on the community cookbook and go my own route.  Here’s how it played out.

* Download the installer you need.  In my case the .bin file - without the rpm, just the bin.
* Upload the .bin to your [Artifactory](http://www.jfrog.com/home/v_artifactory_opensource_overview) server so everyone can easily get access to it when building their VMs.
* Get the installer on to the VM using Chef remote_file command with mode 0700 to ensure the installer is executable.
{% highlight ruby %}
remote_file "/install/location/path/jdk-1_5_0_XYZ-linux-i586.bin" do
  source "http://your.artifactory.example.com/artifactory/yourRepo/jdk-1_5_0_XYZ-linux-i586.bin"
  mode "0700"
  not_if { ::File.exists?('/install/location/path/jdk-1_5_0_XYZ-linux-i586.bin') }
end
{% endhighlight %}
* Run the bash command to run the installer
* The installer requires acceptance of the EULA. So scripting this was a pain. Hat tip to Kule on linuxquestions.com - [http://www.linuxquestions.org/questions/linux-newbie-8/java-jdk-silent-install-help-891313/#post4433389](http://www.linuxquestions.org/questions/linux-newbie-8/java-jdk-silent-install-help-891313/#post4433389)
* The EOH here is important - for me EOF did not work for the command attribute. 
{% highlight ruby %}
bash "install-jdk15" do
  cwd "/install/location/path"
  code <<-EOH
    ./jdk-1_5_0_XYZ-linux-i586.bin >/dev/null < <(echo yes) >/dev/null < <(echo yes)
    rm -rf /install/location/path/jdk1.5.0_XYZ/sample
    rm -rf /install/location/path/jdk1.5.0_XYZ/demo
    ln -s /install/location/path/jdk1.5.0_XYZ /install/location/path/java
  EOH
  not_if { ::File.exists?('/install/location/path/jdk1.5.0_XYZ/README.html') }
end
{% endhighlight %}
* Using the [magic_shell cookbook](https://supermarket.chef.io/cookbooks/magic_shell) - create a environment variables $JAVA_HOME just like the Java cookbook would and many Java apps now expect to exist. 
{% highlight ruby %}
magic_shell_environment 'JAVA_HOME' do
  value '/install/location/path/java'
end
{% endhighlight %}
One thing this does not do is put the java and/or javac executables in to your $PATH.  I’d do this like the Java cookbook does and do symlinks to /usr/local/bin if you need it.
