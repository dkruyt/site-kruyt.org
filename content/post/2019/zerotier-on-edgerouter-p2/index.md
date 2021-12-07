---
author: Dennis Kruyt
categories:
- zerotier
- vyos
- edgerouter
- edgeos
date: "2019-09-27T14:30:27Z"
description: ""
draft: false
cover:
  image: /images/2019/10/photo-1465447142348-e9952c393450--1-.jpeg
slug: zerotier-on-edgerouter-p2
tags:
- zerotier
- vyos
- edgerouter
- edgeos
title: ZeroTier on a Ubiquiti EdgeRouter (part 2)
---


In my previous post I showed how to install ZeroTier on the EdgeRouter, after installing we need to rename the ZeroTier interface to ethx interface to show up in EdgeOS.

{{< bookmark url="https://blog.kruyt.org/zerotier-on-a-ubiquiti-edgerouter/" title="ZeroTier on a Ubiquiti EdgeRouter" description="With the new EdgeOS version 2 firmware for the EdgeRouter it is now possible to install ZeroTier on this router." icon="https://blog.kruyt.org/favicon.ico" author="Dennis Kruyt" publisher="Kruyt.org" thumbnail="/images/2019/08/51gXs2qWNvL._SL1280_-1.jpg" caption="" >}}

Now I adopted EdgeOS (which is basically VyOS/Vyatta running on the EdgeRouter) to support ZeroTier from the native CLI with tab completion.

### Demo

<script src="https://asciinema.org/a/271169.js" id="asciicast-271169" async data-t="2" data-size="small" data-speed="2"></script>

### Install

> I have tested this only with EdgoOS version 2.0.6, I have not tested it with other versions of EdgeOS. So be **warned** especially with 1.x version.

Install ZeroTier as described in my previous post but skip the interface mapping.

After that create a backup of the _Interface.pm_ file.  This is the only file that wil be modified so it is good to keep a backup.

```
root@edgerouter:~# cp /opt/vyatta/share/perl5/Vyatta/Interface.pm /opt/vyatta/share/perl5/Vyatta/Interface.pm.backup
```

Download the tgz from [https://github.com/dkruyt/resources/raw/master/zerotier-edgeos.tgz](https://github.com/dkruyt/resources/raw/master/zerotier-edgeos.tgz)

Extract all files to their absolute path.

```
root@edgerouter:/# tar -C / -xvzf /home/admin/zerotier-edgeos.tgz
opt/
opt/vyatta/
opt/vyatta/share/
opt/vyatta/share/vyatta-op/
opt/vyatta/share/vyatta-op/templates/
opt/vyatta/share/vyatta-op/templates/disconnect/
opt/vyatta/share/vyatta-op/templates/disconnect/zerotier/
opt/vyatta/share/vyatta-op/templates/disconnect/zerotier/node.tag/
opt/vyatta/share/vyatta-op/templates/disconnect/zerotier/node.tag/node.def
opt/vyatta/share/vyatta-op/templates/disconnect/zerotier/node.def
opt/vyatta/share/vyatta-op/templates/connect/
opt/vyatta/share/vyatta-op/templates/connect/zerotier/
opt/vyatta/share/vyatta-op/templates/connect/zerotier/node.tag/
opt/vyatta/share/vyatta-op/templates/connect/zerotier/node.tag/node.def
opt/vyatta/share/vyatta-op/templates/connect/zerotier/node.def
opt/vyatta/share/vyatta-op/templates/show/
opt/vyatta/share/vyatta-op/templates/show/zerotier/
opt/vyatta/share/vyatta-op/templates/show/zerotier/info/
opt/vyatta/share/vyatta-op/templates/show/zerotier/info/node.def
opt/vyatta/share/vyatta-op/templates/show/zerotier/peers/
opt/vyatta/share/vyatta-op/templates/show/zerotier/peers/node.def
opt/vyatta/share/vyatta-op/templates/show/zerotier/moons/
opt/vyatta/share/vyatta-op/templates/show/zerotier/moons/node.def
opt/vyatta/share/vyatta-op/templates/show/zerotier/node.def
opt/vyatta/share/vyatta-op/templates/show/zerotier/networks/
opt/vyatta/share/vyatta-op/templates/show/zerotier/networks/node.def
opt/vyatta/share/vyatta-op/templates/show/interfaces/
opt/vyatta/share/vyatta-op/templates/show/interfaces/zerotier/
opt/vyatta/share/vyatta-op/templates/show/interfaces/zerotier/node.def
opt/vyatta/share/vyatta-cfg/
opt/vyatta/share/vyatta-cfg/templates/
opt/vyatta/share/vyatta-cfg/templates/interfaces/
opt/vyatta/share/vyatta-cfg/templates/interfaces/zerotier/
opt/vyatta/share/vyatta-cfg/templates/interfaces/zerotier/node.tag/
opt/vyatta/share/vyatta-cfg/templates/interfaces/zerotier/node.tag/description/
opt/vyatta/share/vyatta-cfg/templates/interfaces/zerotier/node.tag/description/node.def
opt/vyatta/share/vyatta-cfg/templates/interfaces/zerotier/node.def
opt/vyatta/share/perl5/
opt/vyatta/share/perl5/Vyatta/
opt/vyatta/share/perl5/Vyatta/Interface.pm
```

Thats it, now you can use ZeroTier als0 in the CLI on the router and also it shows up correctly as a ZeroTier interface in the GUI.

{{< figure src="/images/2019/09/image-26.png" >}}

