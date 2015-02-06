---
layout: post
title: "install puppet 3.x on Amazon Linux AMI"
date: 2015-02-06 13:53:50 -0800
comments: true
categories: 
---
I came across an interesting issue with puppet today. Im using the Amazon Linux AMI  
and I discovered that it is running an older version puppet. I did not think much of  
it. So I installed the puppet labs yum repo and installed the new shiny 3.7.x version  
of puppet. When I did my test run. I received lots of errors which caused me to do  
some googling. I finally got everything working. Im posting what I did here in order  
to save someone else some time and hair.

{% codeblock lang:bash %}
[user@host ~]% sudo yum erase puppet
[user@host ~]% sudo rpm -ivh https://yum.puppetlabs.com/el/6/products/x86_64/puppetlabs-release-6-7.noarch.rpm
[user@host ~]% sudo sed -i'' -e '/[main].*/ {N; s/enabled = 0/enabled = 04/g}' /etc/yum/pluginconf.d/priorities.conf
[user@host ~]% sudo yum install puppet
[user@host ~]% sudo yum install rubygem18-json.x86_64
[user@host ~]% sudo alternatives --set ruby /usr/bin/ruby1.8
[user@host ~]% sudo puppet --version
3.7.4
{% endcodeblock %}

####References
*Puppet 3.X is now broken on Amazon AWS due to Ruby 2.0 being the default*
*Retrieved from*
*[puppet jira ticket](https://tickets.puppetlabs.com/browse/PUP-2132)*
