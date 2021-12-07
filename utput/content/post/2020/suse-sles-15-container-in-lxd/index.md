---
author: Dennis Kruyt
categories:
- Suse
- lxd
- container
- SLES
date: "2020-01-20T19:48:06Z"
description: ""
draft: false
image: /images/2020/01/logo3.resized.jpg
slug: suse-sles-15-container-in-lxd
tags:
- Suse
- lxd
- container
- SLES
title: SUSE SLES 15 Container in LXD
---


I am a big fan of LXD and lot of my 'home infra' and VPS's are all running LXD container. Also I am using LXD at work to create test setup for al sorts of Linux OS's and applications. Most "free" Linux OS's are already in the LXD image repository. But sometime's I need a non free Linux OS. Such as SUSE Linux Enterprise Server, or SLES for short. Off-course I can run this in a VPS just by installing it from an ISO. But this takes too long and uses too much resources.

So I wanted to have SLES also available as a local container for test purposes that I can spin up in seconds.

## Create SLES chroot

To create an LXD image we need first todo a SLES install in a chroot on a SLES install. In my case I installed SLES in a VPS. Here I create a chroot dir for the install, and with zypper I install a set of minimal packages into this chroot.

```
mkdir /mnt/chroot

zypper --installroot /mnt/chroot in aaa_base bash systemd-sysvinit sysvinit-tools iproute2 sysconfig-netconfig netcfg wget zypper vim
```

After the zypper install we need to copy some files to the chroot. We need the network interface config file. So networking will startup in the LXD container. And we copy the zypper repo from the VPS install to the chroot environment.

```
cp /etc/sysconfig/network/ifcfg-eth0 /mnt/chroot/etc/sysconfig/network/

cp -r /etc/zypp/repos.d/ /mnt/chroot/etc/zypp/
```

Now the basic chroot environment is finished and we can create a tarball of the chroot.

```
 tar -cvzf /tmp/suse15sp1.tgz -C /mnt/chroot .
```

Then we copy the tarball to the system that will run the LXD containers.

## Import LXD image

On our LXD host we need to create a metadata.yaml file. This file describes things like image creation date, name, architecture and description.

```yaml
architecture: "x86_64"
creation_date: 1579536701 # To get current date in Unix time, use `date +%s` command
properties:
architecture: "x86_64"
description: "SUSE 15 SP1"
os: "SUSE"
release: "15SP1"
```

After creating the file we need create a tarball of this.

```
 tar -cvzf metadata.tar.gz metadata.yaml
```

After creating them, let's import our two tarballs, the metadata en the SLES chroot we copied earlier to this system as a LXD image.

```
 lxc image import metadata.tar.gz suse15sp1.tgz --alias suse-15sp1
```

We can check if the image is in our image catalog.

```
dennis@thinkpad:~> lxc image list suse-15sp1
+------------+--------------+--------+-------------+--------+-----------+----------+------------------------------+
|   ALIAS    | FINGERPRINT  | PUBLIC | DESCRIPTION |  ARCH  |   TYPE    |   SIZE   |         UPLOAD DATE          |
+------------+--------------+--------+-------------+--------+-----------+----------+------------------------------+
| suse-15sp1 | fe5f49751f18 | no     |             | x86_64 | CONTAINER | 129.21MB | Jan 20, 2020 at 7:52pm (UTC) |
+------------+--------------+--------+-------------+--------+-----------+----------+------------------------------+

```

It's only a 129mb, pretty small ;)

### Test LXD SLES container

And lets test the SLES LXD container by launching it and execute bash in it.

```bash
dennis@thinkpad:~> lxc launch suse-15sp1 suse-test
Creating suse-test
Starting suse-test                        
dennis@thinkpad:~> lxc exec suse-test bash
suse-test:~ # cat /etc/os-release
NAME="SLES"
VERSION="15-SP1"
VERSION_ID="15.1"
PRETTY_NAME="SUSE Linux Enterprise Server 15 SP1"
ID="sles"
ID_LIKE="suse"
ANSI_COLOR="0;32"
CPE_NAME="cpe:/o:suse:sles:15:sp1"
suse-test:~ # uname -a
Linux suse-test 5.3.0-26-generic #28-Ubuntu SMP Wed Dec 18 05:37:46 UTC 2019 x86_64 x86_64 x86_64 GNU/Linux
suse-test:~ #
```

Now you can install with zypper other applications. Also it is possible to register with SUSEConnect the container or use it with a RMT server.

