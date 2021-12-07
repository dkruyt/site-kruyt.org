---
author: Dennis Kruyt
categories:
- fortinet
- fortiadc
- grafana
- telegraf
- influxdb
- loadbalancer
- network
date: "2019-08-21T06:44:10Z"
description: ""
draft: false
image: /images/2019/10/photo-1511872638242-f1652016540e--1-.jpeg
slug: fortinet-fortiadc-in-grafana
tags:
- fortinet
- fortiadc
- grafana
- telegraf
- influxdb
- loadbalancer
- network
title: Fortinet FortiADC in Grafana
---


FortiADC is is a application delivery controllers (loadbalancer). The devices metrics are availalbe via SNMP. So it's quite easy to collect those and display them in Grafana.

![1037687777](/images/2019/10/1037687777.jpg)
### Pre Install

Make sure you have installed [InfluxDB](https://portal.influxdata.com/downloads) as the time-series database [Telegraf](https://portal.influxdata.com/downloads) as collector first.
Optional you can also include the FortiADC logs when they are in elasticsearch. I use the Greylog sollution for this.

### Quick Start

Get the latest files, at [my GitHub page](https://github.com/dkruyt/fortiadc_grafana)

* Enable SNMP on your FortiADC
* Put fortiadc.conf in your `/etc/telegraf/telegraf.d` directory
* Edit the community string as appropriate
* Edit the SNMP verion as appropriate
* Edit the 'agents' list to include all of your monitored FortiADC loadbalancers
* Put the files in the mibs dir `/usr/share/snmp/mibs`
* Import the Grafana dashboard

### Screenshot

After you setup all, you should have a dashboard like this.

![screencapture-fortiadc-blur](/images/2018/07/screencapture-fortiadc-blur.png)

