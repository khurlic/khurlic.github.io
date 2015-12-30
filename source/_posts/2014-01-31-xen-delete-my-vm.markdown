---
layout: post
title: "xen delete my vm"
date: 2014-01-31 17:00:49 -0800
comments: true
categories: linux tech xen
---

This blog post will show you how to delete the vm of your choosing and it's 
primary disk from a Citrix Xen Server. A few things to keep in mind. Some of 
the disk will show up as xvda vs hda. This depends on how you have configured
your server. 

```bash
[root@xenserver ~]% xe vm-list name-label=testserver params=uuid
uuid ( RO)    : 3df485ee-0e99-2851-cf6c-e0c7517e68fd

[root@xenserver ~]% xe vm-shutdown uuid=3df485ee-0e99-2851-cf6c-e0c7517e68fd

[root@xenserver ~]% xe vbd-list vm-uuid=3df485ee-0e99-2851-cf6c-e0c7517e68fd device=hda params=uuid
uuid ( RO)    : bfdb0ba5-b397-e6f5-3ba1-5aa9df6dd0ce

[root@xenserver ~]% xe vdi-list vbd-uuids=bfdb0ba5-b397-e6f5-3ba1-5aa9df6dd0ce params=uuid
uuid ( RO)    : 07d5c7f8-40d1-4276-ba0c-5fde960ab527

[root@xenserver ~]% xe vdi-destroy uuid=07d5c7f8-40d1-4276-ba0c-5fde960ab527

[root@xenserver ~]% xe vm-destroy uuid=3df485ee-0e99-2851-cf6c-e0c7517e68fd
```


####References
__Virtual Block Device (VBD)__: A VBD is a software object that connects a VM to the VDI, which represents the contents of the virtual disk. The VBD has the attributes which tie the VDI to the VM (is it bootable, its read/write metrics, and so on), while the VDI has the information on the physical attributes of the virtual disk (which type of SR, whether the disk is shareable, whether the media is read/write or read only, and so on).

__Virtual Disk Image (VDI)__: A VDI is a software object that represents the contents of the virtual disk seen by a VM, as opposed to the VBD, which is a connector object that ties a VM to the VDI. The VDI has the information on the physical attributes of the virtual disk (which type of SR, whether the disk is shareable, whether the media is read/write or read only, and so on), while the VBD has the attributes which tie the VDI to the VM (is it bootable, its read/write metrics, and so on).

*Xen Server version 6.2.0 documentation (cli xe command vbd).*  
*Retrieved from*  
[cli xe command reference](http://docs.vmd.citrix.com/XenServer/6.2.0/1.0/en_gb/reference.html#cli-xe-commands_vbd)

