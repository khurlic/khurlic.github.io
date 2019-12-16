---
layout: post
title: "Provision Centos7 vagrant box with fedora31"
date: 2019-12-15 21:40:57 -0800 
categories: automation
---

Im playing around with buildah. Im still managing older centos 7 systems. My workstation is running fedora 31. 
I thought it would be easy enough to use vagrant to spin up a CentOS 7 sytstem and provision it with ansible  
from my fedora31 system. It was all going good until I received the following error message.
```
fatal: [buildah]: FAILED! => {"changed": false, "msg": "The Python 2 bindings for rpm are needed for this module. If you require Python 3 support use the `dnf` Ansible module instead.. The Python 2 yum module is needed for this module. If you require Python 3 support use the `dnf` Ansible module instead."}
```

I stopped to think about this. CentOS 7 is still using yum and python 2.x by default. 
Fedora 31 is using dnf and python3.x. Hmm... How does one fix this?  

So took this approach.  
1. Install python2 on my fedora31 system  
`$ sudo dnf install python2`
2. Updated my Vagrantfile. The ansible provision block. I added ansible.extra_vars that look like this.
```
config.vm.provision "ansible" do |ansible|
  ansible.verbose = "v"
  ansible.playbook = "ansible/playbook.yml"
  ansible.compatibility_mode = "2.0"
  ansible.extra_vars = { ansible_python_interpreter:"/usr/bin/python2" }
end
```

All set.. Working as expected now. Hope this helps someone else out.
