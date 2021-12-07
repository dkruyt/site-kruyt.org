---
author: Dennis Kruyt
categories:
- huawei
- grafana
- metrics
- influxdb
- telegraf
- snmp
date: "2017-07-22T19:27:02Z"
description: ""
draft: false
cover:
  image: /images/2019/10/photo-1471079688237-3ac9a55f1d6f--1-.jpeg
slug: oceanstor_grafana
tags:
- huawei
- grafana
- metrics
- influxdb
- telegraf
- snmp
title: Huawei OceanStor metrics in Grafana
---


I manage a couple of storages at work. I got a few Huawei OceanStor storages. v2 as wel v3 storages. There is some commercial tooling from Huawei available to gather metrics, but that didn't fit my needs. I found out that all or almost all metrics that I need are available via SNMP. So it was not hard to setup a Grafana dashboard with all metrics.
 
### Pre install

Make sure you have installed [InfluxDB](https://portal.influxdata.com/downloads) as the time-series database [Telegraf](https://portal.influxdata.com/downloads) as the collector first.

### Quick Start

Get the all files, at [My GitHub page](https://github.com/dkruyt/OceanStor_Grafana).

* Enable SNMP on your Huawei OceanStor storage.
* Put hw_stor.conf in your `/etc/telegraf/telegraf.d` directory.
* Edit the community string as appropriate.
* Edit the SNMP version as appropriate, please note that verion 2 is disabled by default on the Dorado storage, check with `show snmp version` on the Storage.
* Edit the 'agents' list to include all of your monitored OceanStor storages.
* Put the files in the mibs dir on the server running telgraf. With Ubuntu it is in `/usr/share/snmp/mibs` other Linux distributions can differ.
* Import the Grafana dashboard in Grafana.

### Screenshot

After you setup all, you should have some dashboard like this.

> Note, on the latest version on GitHub CIFS and NFS stats has been added, not on this screenshot yet.

![Grafana screenshot zswap](https://github.com/dkruyt/OceanStor_Grafana/raw/master/Grafana___Huawei_Storage_edit.jpg)

