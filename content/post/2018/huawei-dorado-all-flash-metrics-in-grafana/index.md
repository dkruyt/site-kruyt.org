---
author: Dennis Kruyt
categories:
- dorado
- huawei
- storage
- grafana
- metrics
- influxdb
- telegraf
date: "2018-01-11T21:51:48Z"
description: ""
draft: false
cover:
  image: /images/2018/04/Xiphias_gladius2.jpg
slug: huawei-dorado-all-flash-metrics-in-grafana
tags:
- dorado
- huawei
- storage
- grafana
- metrics
- influxdb
- telegraf
title: Huawei Dorado all flash metrics in Grafana
---


Recently I got my hands on a nice all flash storage from Huawei, the Dorado 5000. Off course I wanted to see if my Grafana dashboard I created for the Huawei Oceanstor series was also working on the Dorado, most is working, but some mibs are different and not all performance metrics that are availble on the Oceanstor are on the Dorado. So I adjusted the Telegraf and Grafana dashboard to work correclty with the Dorado.   
![dorado](/images/2018/01/dorado.png)
### Pre Install

Make sure you have installed [InfluxDB](https://portal.influxdata.com/downloads) as the time-series database [Telegraf](https://portal.influxdata.com/downloads) as the collector first.

### Quick Start

Get the latest files, at [my GitHub page](https://github.com/dkruyt/OceanStor_Grafana)

* Enable SNMP on your Huawei Dorado storage
* Put dorado.conf in your `/etc/telegraf/telegraf.d` directory
* Edit the community string as appropriate
* Edit the SNMP verion as appropriate, please note that verion 2 is disabled by default on the Huawei OceanStor, check with `show snmp version` on the Storage
* Edit the 'agents' list to include all of your monitored Dorado storages
* Put the files in the mibs dir `/usr/share/snmp/mibs`
* Import the Grafana dashboard

### Screenshot

After you setup all, you should have some dashboard like this.

![Grafana screenshot Dorado](https://github.com/dkruyt/Dorado_Grafana/blob/master/Huawei_Dorado_Storage.jpg?raw=true)

