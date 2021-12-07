---
author: Dennis Kruyt
categories:
- huawei
- wlan
- grafana
- influxdb
- telegraf
- FAT
date: "2018-01-11T22:20:20Z"
description: ""
draft: false
cover:
  image: /images/2018/01/Huawei-AP6010DN-AGN-802-11n-Outdoor-Access-Points.jpg
slug: huawei-fat-wlan-access-points-in-grafana
tags:
- huawei
- wlan
- grafana
- influxdb
- telegraf
- FAT
title: Huawei FAT WLAN Access Points in Grafana
---


With Huawei Enterprise Access points you have different firmware versions for different deployment scenarios. A FIT version for when you have an Wireless Access Controller (AC). This AC will managed all your AP and you have metrics available from within the AC. Huawei also offers a FAT version of the firmware for the AP. This is an standalone verion and there is no central management and overview of your AP's with the FAT version.

To have atleast a central overview of all your FAT AP's metrics and performance I created a templated dashboard for these AP's in Grafana.

### Pre Install

Make sure you have installed [InfluxDB](https://portal.influxdata.com/downloads) as the time-series database [Telegraf](https://portal.influxdata.com/downloads) as the collector first.

### Quick Start

For all latest files, see [My GitHub page](https://github.com/dkruyt/Huawei_FatAP_Grafana)

* Enable SNMP on your Huawei WLAN FAT Accespoint
* Put hw_fatap.conf in your `/etc/telegraf/telegraf.d` directory
* Edit the community string as appropriate
* Edit the 'agents' list to include all of your monitored Fat WLAN Access points
* Put the files in the mibs dir `/usr/share/snmp/mibs`
* Import the Grafana dashboard json file

### Screenshot

After you setup all, you should have a dashboard like this.

![Grafana screenshot Huawe FAT AP](https://raw.githubusercontent.com/dkruyt/Huawei_FatAP_Grafana/master/Huawei%20FAT%20AP%20templated.png)

