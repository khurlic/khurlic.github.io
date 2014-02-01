---
layout: post
title: "xen delete my vm"
date: 2014-01-31 17:00:49 -0800
comments: true
categories: 
---

This blog post will show you how to delete the vm of your chooseing and it's 
primary disk from a Citrix Xen Server. A few things to keep in mind. Some of 
the disk will show up as xvda vs hda. This depends on how you have configured
your server. 

{% codeblock lang:bash %}
[root@xenserver ~]% xe vm-list name-label=testserver params=uuid
uuid ( RO)    : 3df485ee-0e99-2851-cf6c-e0c7517e68fd

[root@xenserver ~]% xe vm-shutdown uuid=3df485ee-0e99-2851-cf6c-e0c7517e68fd

[root@xenserver ~]% xe vbd-list vm-uuid=3df485ee-0e99-2851-cf6c-e0c7517e68fd device=hda params=uuid
uuid ( RO)    : bfdb0ba5-b397-e6f5-3ba1-5aa9df6dd0ce

[root@xenserver ~]% xe vdi-list vbd-uuids=bfdb0ba5-b397-e6f5-3ba1-5aa9df6dd0ce params=uuid
uuid ( RO)    : 07d5c7f8-40d1-4276-ba0c-5fde960ab527

[root@xenserver ~]% xe vdi-destroy uuid=07d5c7f8-40d1-4276-ba0c-5fde960ab527

[root@xenserver ~]% xe vm-destroy uuid=3df485ee-0e99-2851-cf6c-e0c7517e68fd
{% endcodeblock %}
