---
layout: post
title: "xen find my mac"
date: 2014-01-31 15:51:36 -0800
comments: true
categories: tech xen linux
---
Not all companies will hand you a shiny new or old windows laptop. Not all companies
will purchase a Windows license to run as a VM. This is will force you to
use the cli to get your job done. The blog post will show you have to find the
mac address of a vm running on Citrix Xen Server. 

{% codeblock lang:bash %}
[root@xenserver ~]% xe vm-list name-label=<your vm name here> params=uuid
uuid ( RO)    : 3df485ee-0e99-2851-cf6c-e0c7517e68fd

[root@xenserver ~]% xe vif-list vm-uuid=3df485ee-0e99-2851-cf6c-e0c7517e68fd params=MAC
MAC ( RO)    : 3a:c3:6f:ee:ab:c8
{% endcodeblock %}


####References
**Virtual Inter Face (VIF)**: A VIF, which represents a virtual NIC on a virtual machine. VIF objects have a name and description, a globally unique UUID, and the network and VM they are connected to.



*Xen Server version 6.2.0 documentation (cli xe command vif).*  
*Retrieved from*  
*[cli xe command reference](http://docs.vmd.citrix.com/XenServer/6.2.0/1.0/en_gb/reference.html#cli-xe-commands_vif)*
