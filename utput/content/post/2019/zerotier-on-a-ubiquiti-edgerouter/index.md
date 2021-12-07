---
author: Dennis Kruyt
categories:
- edgerouter
- zerotier
- sdwan
- networking
- edgeos
- Ubiquiti
- network
- vyos
date: "2019-08-31T17:36:31Z"
description: ""
draft: false
image: /images/2019/10/photo-1477519242566-6ae87c31d212--1-.jpeg
slug: zerotier-on-a-ubiquiti-edgerouter
summary: 'With the new EdgeOS version 2 firmware for the EdgeRouter it is now possible
  to install ZeroTier on this router. '
tags:
- edgerouter
- zerotier
- sdwan
- networking
- edgeos
- Ubiquiti
- network
- vyos
title: ZeroTier on a Ubiquiti EdgeRouter
---


With the new EdgeOS version 2 firmware for the Ubiquiti [EdgeRouter](https://www.ui.com/edgemax/edgerouter/) it is now possible to install [ZeroTier](https://www.zerotier.com/) on this router. This post wil give you instructions on how to do this.

I use ZeroTier on this router to create connections the my containers on my containers hosts in the cloud. With ZeroTier installed on my EdgeRouter Lite I can connect to them directly from all my machines at home. My connections are managed routes with ZeroTier. Bridging should work also, but I didn't try it, it will take a little more work.

{{< figure src="/images/2019/08/ubiquiti-edgemax-edgerouter-lite-erlite-3-server-1707-13-server@13-1.jpg" caption="edgeRouter Lite" >}}

### What is ZeroTier

> ZeroTier is a smart Ethernet switch for planet Earth.

> It’s a distributed network hypervisor built atop a cryptographically secure global peer to peer network. It provides advanced network virtualization and management capabilities on par with an enterprise SDN switch, but across both local and wide area networks and connecting almost any kind of app or device.

{{< figure src="/images/2019/08/150px-ZeroTier_Logo.png" caption="ZeroTier logo" >}}

## Instructions

### Install ZeroTier on EdgeRouter

First login on your EdgeRouter en sudo to root, then use curl to get the install script from ZeroTier itself.

```bash
admin@edgerouter# sudo -i
root@edgerouter:~# curl -s https://install.zerotier.com | sudo bash

*** ZeroTier One Quick Install for Unix-like Systems

*** Tested distributions and architectures:
***   MacOS (10.7+) on x86_64 (just installs ZeroTier One.pkg)
***   Debian (7+) on x86_64, x86, arm, and arm64
***   RedHat/CentOS (6+) on x86_64 and x86
***   Fedora (16+) on x86_64 and x86
***   SuSE (12+) on x86_64 and x86
***   Mint (18+) on x86_64, x86, arm, and arm64

*** Please report problems to contact@zerotier.com and we will try to fix.

*** Detecting Linux Distribution

*** Found Debian "stretch" (or similar), creating /etc/apt/sources.list.d/zerotier.list
OK

*** Installing zerotier-one package...
<cut>
*** Enabling and starting zerotier-one service...
Synchronizing state of zerotier-one.service with SysV service script with /lib/systemd/systemd-sysv-install.
Executing: /lib/systemd/systemd-sysv-install enable zerotier-one

*** Waiting for identity generation...

*** Success! You are ZeroTier address [ 5fca9e9471 ].
```

If the install successful you will get a ZeroTier address. If not, take a look at your firewall logs/settings. If successful you can join your ZeroTier network.

### Persistent after firmware upgrade

To make ZeroTier config persistent after firmware updates, we need to move the _/var/lib/zerotier_ to _/config/script._

```bash
root@edgerouter:/etc# cd /var/lib
root@edgerouter:/var/lib# mv /var/lib/zerotier-one /config/scripts/
root@edgerouter:/var/lib# ln -s /config/scripts/zerotier-one
```

Possible after a firmware upgrade you may need to recreate the symlink again. But your ZeroTier networks will be preserved.

### Map your ZeroTier interface

The part below is not necessary any more if you install my package to use the EdgeOS CLI.

{{< bookmark url="https://blog.kruyt.org/zerotier-on-edgerouter-p2/" title="ZeroTier on a Ubiquiti EdgeRouter (part 2)" description="In my previous post I showed how to install ZeroTier on the EdgeRouter, after\ninstalling we need to rename the ZeroTier interface to ethx interface to show up\nin EdgeOS. ZeroTier on a Ubiquiti EdgeRouterWith the new EdgeOS version 2 firmware for the\nEdgeRouter it is now possible to install ZeroTier…" icon="https://blog.kruyt.org/favicon.ico" author="Dennis Kruyt" publisher="Kruyt.org" thumbnail="/images/2019/09/photo-1465447142348-e9952c393450.jpeg" caption="" >}}

The EdgeOS won't show and interact with your ZeroTier interface properly. This because it doesn't know how the handle a ztxxxxx interface. To work around we need to change the ztxxxxx interface name to a ethx name. To do this create a new file _/var/lib/zerotier-one/devicemap_ with the following content.

Replace _<zerotiernetworkid>_ with your ZeroTier network. If you have more network's then create a entry for each network with a unique ethx interface.

```bash
<zerotiernetworkid>=eth4
```

> If you have an other Edgerouter model with more then 3 ethx ports, then increase the number 4 to a free one.

Then restart ZeroTier to create the new mapped interface.

```bash
root@edgerouter:~# /etc/init.d/zerotier-one restart
[ ok ] Restarting zerotier-one (via systemctl): zerotier-one.service.
root@edgerouter:~# ifconfig eth4
eth4: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 2800
        inet 10.122.12.95  netmask 255.255.255.0  broadcast 10.147.17.255
        inet6 fe80::6cb5:75ff:fe9a:e2  prefixlen 64  scopeid 0x20<link>
        ether 6e:b5:75:9a:00:e2  txqueuelen 1000  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 7  bytes 738 (738.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

### Enable new ethx interface in EdgeOS

Then to enable the interface in EdgeOS, do the following per interface. And change the description to your network id.

```bash
admin@edgerouter# set interfaces ethernet eth4 description "ZeroTier 1d73947417ceeb6e"
[edit]
admin@edgerouter# commit
[edit]
admin@edgerouter# save
Saving configuration to '/config/config.boot'...
Done
```

After this, login the web interface. Now you will see a eth4, your ZeroTier interface. It will also show up now in your firewall rules. So now you can your ZeroTier interface as any other interface physical on your router.

{{< figure src="/images/2019/08/EdgeOS_-_edgerouter.png" >}}

Also the UNMS interface shows the ZeroTier interface, but it shows the MTU with a red underline, But this is the correct value for a ZeroTier interface, so don't change it.

{{< figure src="/images/2019/09/UNMS_1_0_1-2.jpg" >}}

### Performance

I did some testing with _iperf_ trough my EdgeRouter Lite. I was able to get between 20 and 30 Mbits/sec. The bottleneck is that the ZeroTier process on the EdgeRouter Lite is maxed out due to the _small_  _CPU_ on the EdgeRouter Lite. Other bigger EdgeRouter models may perform better.

After I installed ZeroTier on the router I noticed a increase in CPU usage, even there is barely any traffic on the ZeroTier interface.

{{< figure src="/images/2019/09/Mblq4iTx3TuphYfj.png" caption="CPU" >}}

{{< figure src="/images/2019/09/B93FoM8epRqGS4hw.png" caption="ZeroTier interface traffic" >}}



