---
author: Dennis Kruyt
categories:
- scaleway
- hetzner
- VPS
- test
- compare
date: "2018-07-19T18:59:35Z"
description: ""
draft: false
cover:
  image: /images/2018/07/electrical-2476782_1920-1.jpg
slug: scaleway-vs-hetzner
summary: I am looking for a new not so expensive VPS provider. When I was looking
  arround for something that would suit my needs I came up with ScaleWay and Hetzner
  as possible candidates.
tags:
- scaleway
- hetzner
- VPS
- test
- compare
title: ScaleWay vs Hetzner
---


I am looking for a new not so expensive VPS provider. When I was looking arround for something that would suit my needs I came up with ScaleWay and Hetzner as possible candidates. 
![scalewayvshetzner](/images/2018/07/scalewayvshetzner.png)
I need a VPS with 4GB of RAM, so the following offers would suit me and the price is what I want to pay.
| Specs      | ScaleWay VC1M       | Hetzner CX21 |
| ------------- |---------------| --------|
| CPU       | **4**       | 2 |
| Memory       | **3941 MB**       | 3848 MB |
| Disk       | **50 GB**       | 40 GB |
| Price       | **€4.99**       | €5.93 |

We can see that on paper ScaleWay specs are better and for a lower price. Both claim on their site 4GB of memory, but ScaleWay has a little more if we look at the total memory available to the OS. But specs dont say everything. 

So I want to test, you can pay by the hour, so no harm there. I deploy them both with their Ubuntu 18.04 LTS default install, installed latest patches and kernel 4.15.0-23-generic x86_64. So in this way they are the same on OS and Kernel. I noticed when deploying that Hetzner system was online is second, ScaleWay took a little longer, maybe a minute or so. That would be a plus for Hetzner. But I only deploy a system one time, so no real gain here for me.

###CPU

First I tested CPU and RAM with [GeekBench](https://www.geekbench.com/). This shows that ScaleWay gives better performance on single thread and multi thread. Als I compiled fio, a tool I later wil use to test disk performance. This compiles faster on ScaleWay.

| Test       | ScaleWay       | Hetzner |
| ------------- |---------------| --------|
| Geekbench Single       | **[3073](https://browser.geekbench.com/v4/cpu/9072940)**       | [2482](https://browser.geekbench.com/v4/cpu/9072938) |
| Geekbench Multi       | **[7443](https://browser.geekbench.com/v4/cpu/9072940)**       | [4652](https://browser.geekbench.com/v4/cpu/9072938) |
| Fio Compile       | **18.843s**       | 21.443s |

###Disk
To test Disk performance I used [fio](https://github.com/axboe/fio) to test read and write performance. With random read/write it is a tie. With read ScaleWay wins and with Write Hetner.

| Test       | ScaleWay       | Hetzner |
| ------------- |---------------| --------|
| Random read/write       | 59223/19711       | 59457/19782 |
| Random read       | **129198**       | 83405 |
| Random write       | 37007       | **75129** |

Fio parameters used for this test

```bash
#Random Read Write
./fio --randrepeat=1 --ioengine=libaio --direct=1 --gtod_reduce=1 --name=test --filename=test --bs=4k --iodepth=64 --size=4G --readwrite=randrw --rwmixread=75

#Random Write
./fio --randrepeat=1 --ioengine=libaio --direct=1 --gtod_reduce=1 --name=test --filename=test --bs=4k --iodepth=64 --size=4G --readwrite=randread

#Random Read
./fio --randrepeat=1 --ioengine=libaio --direct=1 --gtod_reduce=1 --name=test --filename=test --bs=4k --iodepth=64 --size=4G --readwrite=randwrite
```

###Conclusion

After this test I decided to give ScaleWay a try, will migrating some services to them soon.


***update 8 Sept 2018**, running now on ScaleWay for about 6 weeks, found no problems and everything is performing well.*

