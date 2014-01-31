---
layout: post
title: "Channel Bonding Interfaces the RedHat 6.x way"
date: 2014-01-30 20:35:32 -0800
comments: true
categories: tech, linux
---

Recently at work, I received an email from my networking team on why they were seeing the following error.

%SW_MATM-4-MACFLAP_NOTIF: Host dddd.dddd.dddd in vlan 100 is flapping between port Te1/0/1 and port Gi2/0/22.

I tracked down the mac from the message above. (mac changed for security) I logged in to the box and discovered that the nic bond was running in mode 0 instead of mode 1 which it was configured for. The root cause was Red Hat changed the way parameters for the bonding kernel module were loaded. Starting in RHEL 6.0 you’ll need to add the bonding options to the bonded interface file (ifcfg-bondX).

Here are the commands I took to correct the issue.

**Verify bonding mode**
{% codeblock lang:bash %}
# cat /sys/class/net/bond0/bonding/mode
balance-rr 0

# cat /proc/net/bonding/bond0 | grep -i mode
Bonding Mode: load balancing (round-robin)

# cat /etc/modprobe.d/bonding.conf
alias bond0 bonding
options bond0 mode=1 miimon=100
{% endcodeblock %}

**Make changes**
{% codeblock lang:bash %}
# sed -i'.bak' -e '/options.*/d' /etc/modprobe.d/bonding.conf
# echo 'BONDING_OPTS="mode=1 miimon=100"' >> /etc/sysconfig/networking-scripts/ifcfg-bond0
{% endcodeblock %}

**Restart Network**
{% codeblock lang:bash %}
# service network restart
{% endcodeblock %}

**Verify bonding mode**
{% codeblock lang:bash %}
# cat /sys/class/net/bond0/bonding/mode
active-backup 1

# cat /proc/net/bonding/bond0 | grep -i mode
Bonding Mode: fault-tolerance (active-backup)

# cat /etc/modprobe.d/bonding.conf
alias bond0 bonding
{% endcodeblock %}


Setting up nic bonding from scratch
=====
_____

Change all of the example IP addresses from 1.1.1.x to the IP addresses that work in your environment.

1.) Edit the  /etc/modprobe.d/bonding.conf
{% codeblock lang:bash %}
[root@host ~]# vi /etc/modprobe.d/bonding.conf
alias bond0 bonding
{% endcodeblock %}

2.) Edit the /etc/sysconfig/network
{% codeblock lang:bash %}
[root@host ~]# vi /etc/sysconfig/network
NETWORKING=yes
HOSTNAME=your.hostname.here
GATEWAY=1.1.1.1
{% endcodeblock %}

3.) Edit the /etc/resolv.conf
{% codeblock lang:bash %}
[root@host ~]# vi /etc/resolv.conf
search example.com
nameserver 1.1.1.2
nameserver 1.1.1.3
{% endcodeblock %}

4.) Edit the /etc/sysconfig/network-scripts/ifcfg-bond0
{% codeblock lang:bash %}
[root@host ~]# vi  /etc/sysconfig/network-scripts/ifcfg-bond0
DEVICE=bond0
USERCTL=no
BOOTPROTO=none
ONBOOT=yes
USERCTL=no
IPADDR=1.1.1.100
NETMASK=255.255.255.0
BONDING_OPTS=”mode=1 miimon=100″
{% endcodeblock %}

5.) Edit the  /etc/sysconfig/network-scripts/ifcfg-eth0
{% codeblock lang:bash %}
[root@host ~]# vi  /etc/sysconfig/network-scripts/ifcfg-eth0
DEVICE=”eth0″
SLAVE=”yes”
ONBOOT=”yes”
MASTER=”bond0″
USERCTL=”no”
BOOTPROTO=none
NM_CONTROLLED=”no”
{% endcodeblock %}

6.) Edit the  /etc/sysconfig/network-scripts/ifcfg-eth1
{% codeblock lang:bash %}
[root@host ~]# vi  /etc/sysconfig/network-scripts/ifcfg-eth1
DEVICE=”eth1″
SLAVE=”yes”
ONBOOT=”yes”
MASTER=”bond0″
USERCTL=”no”
BOOTPROTO=none
NM_CONTROLLED=”no”
{% endcodeblock %}

7.) Load the bonding kernel module
{% codeblock lang:bash %}
[root@host ~]# modprobe bonding
{% endcodeblock %}

8.) restart the network
{% codeblock lang:bash %}
[root@host ~]# service network restart
{% endcodeblock %}

9.) Verify changes
{% codeblock lang:bash %}
[root@host ~]# ping 1.1.1.1

PING 1.1.1.1 (1.1.1.1) 56(84) bytes of data.
64 bytes from 1.1.1.1: icmp_seq=1 ttl=255 time=1.89 ms
64 bytes from 1.1.1.1: icmp_seq=2 ttl=255 time=0.872 ms
64 bytes from 1.1.1.1: icmp_seq=3 ttl=255 time=2.25 ms
64 bytes from 1.1.1.1: icmp_seq=4 ttl=255 time=0.880 ms

— 1.1.1.1 ping statistics —
4 packets transmitted, 4 received, 0% packet loss, time 3003ms
rtt min/avg/max/mdev = 0.872/1.476/2.253/0.612 ms

[root@host ~]# host www.google.com
host www.google.com
www.google.com has address 74.125.224.177
www.google.com has address 74.125.224.178
www.google.com has address 74.125.224.179
www.google.com has address 74.125.224.180
www.google.com has address 74.125.224.176
www.google.com has IPv6 address 2607:f8b0:4007:800::1010

[root@host ~]# cat /proc/net/bonding/bond0 | grep -i mode
Bonding Mode: fault-tolerance (active-backup)

[root@host ~]# cat /sys/class/net/bond0/bonding/mode
active-backup 1

[root@host ~]# ifdown eth0
from another host ping your server, if the ping is good, then ssh to it. If you can reach your server via ping and ssh then the nic bonding is working as it should.

[root@host ~]# ifup eth0
{% endcodeblock %}

Reference.

[Redhat Docs for bonding](https://access.redhat.com/site/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Deployment_Guide/s2-networkscripts-interfaces-chan.html "RedHat Docs")
