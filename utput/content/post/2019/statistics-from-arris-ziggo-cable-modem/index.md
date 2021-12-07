---
author: Dennis Kruyt
categories:
- connectbox
- ziggo
- docsis
- influxdb
- arris
- cable
- modem
- statistics
- grafana
date: "2019-08-16T14:53:00Z"
description: ""
draft: false
image: /images/2019/10/photo-1568127558771-dad1b9d93a08_2.jpeg
slug: statistics-from-arris-ziggo-cable-modem
tags:
- connectbox
- ziggo
- docsis
- influxdb
- arris
- cable
- modem
- statistics
- grafana
title: Docsis stats from ARRIS Ziggo modem
---


I have at home a Ziggo cable internet connection. The modem that Ziggo provides is a Arris modem. I have this modem in bridge mode because I have my own router. But I like to get some DOCSIS statistics from this modem. The modem provide these via a kind of web snmp output, but this is not very useful.

{{< figure src="/images/2019/08/1493642544207.png" caption="Ziggo Connect box" >}}

To parse this output into something more readable I create a crude oneliner script. This scripts export the DOCIS statistics from the ARRIS cable modem to CSV. Replace the IP with your cable modem IP. If you are using bridge mode then probably you have the same ip as in the script below.

Place the file in the directory _/etc/telegraf/telegraf.d/arrisstatus.sh_ and make it executable (chmod +x arrisstatus.sh) __

```bash
#!/bin/bash

echo "measurement,channel,value"
curl -s 'https://192.168.100.1/getRouterStatus' --compressed --insecure | sed s/1.3.6.1.2.1.10.127.1.1.1.1.1/docsIfDownChannelId/ | sed s/1.3.6.1.2.1.10.127.1.1.1.1.2/docsIfDownChannelFrequency/ | sed s/1.3.6.1.2.1.10.127.1.1.1.1.4/docsIfDownChannelModulation/ | sed s/1.3.6.1.2.1.10.127.1.1.1.1.6/docsIfDownChannelPower/ | sed s/1.3.6.1.2.1.10.127.1.1.2.1.1/docsIfUpChannelId/ | sed s/1.3.6.1.2.1.10.127.1.1.2.1.2/docsIfUpChannelFrequency/ | sed s/1.3.6.1.2.1.10.127.1.1.2.1.3/docsIfUpChannelWidth/ | sed s/1.3.6.1.2.1.10.127.1.1.2.1.15/docsIfUpChannelType/ | sed s/1.3.6.1.4.1.4115.1.3.4.1.9.2.1.2/arrisCmDoc30IfUpChannelExtendedSymbolRate/ | sed s/1.3.6.1.4.1.4115.1.3.4.1.9.2.1.3/arrisCmDoc30IfUpChannelExtendedModulation/ | sed s/1.3.6.1.4.1.4491.2.1.20.1.2.1.1/docsIf3CmStatusUsTxPower/ | sed s/1.3.6.1.4.1.4491.2.1.20.1.2.1.2/docsIf3CmStatusUsT3Timeouts/ | sed s/1.3.6.1.4.1.4491.2.1.20.1.2.1.3/docsIf3CmStatusUsT4Timeouts/ | sed s/1.3.6.1.4.1.4491.2.1.20.1.24.1.1/docsIf3SignalQualityExtRxMER/ | sed s/1.3.6.1.2.1.10.127.1.1.4.1.3/docsIfSigQCorrecteds/ | sed s/1.3.6.1.2.1.10.127.1.1.4.1.4/docsIfSigQUncorrectables/ | sed s/1.3.6.1.2.1.10.127.1.1.4.1.5/docsIfSigQSignalNoise/ | sed s/1.3.6.1.2.1.69.1.5.8.1.2/DevEvFirstTimeOid/ | sed s/1.3.6.1.2.1.69.1.5.8.1.5/DevEvId/ | sed s/1.3.6.1.2.1.69.1.5.8.1.7/DevEvText/ | sed s/1.3.6.1.2.1.126.1.1.1.1.1/docsBpi2CmPrivacyEnable/ | sed s/1.3.6.1.4.1.4491.2.1.21.1.3.1.8/docsQosServiceFlowPrimary/ | sed 's/"//g' | sed 's/,$//g' | sed 's/\./,/' | sed 's/:/,/' | grep "^[a-z]"
```

Then create a cronjob for it that writes the CSV output every minute to a tmp file.

```
* * *     root	/etc/telegraf/telegraf.d/arrisstatus.sh > /tmp/arris-status.csv
```

I then use [Telegraf](https://www.influxdata.com/time-series-platform/telegraf/) to parse the CSV output and store the values in a [InfluxDB](https://www.influxdata.com/products/influxdb-overview/) database. The following config can be placed in the _/etc/telegraf/telegraf.d_ directory. If you don't have the TIG stack running, see [here](https://www.howtoforge.com/tutorial/how-to-install-tig-stack-telegraf-influxdb-and-grafana-on-ubuntu-1804/) for instructions to het this up and running.

```
[[inputs.file]]
  files = ["/tmp/arris-status.csv"]
  data_format = "csv"
  csv_tag_columns = ["channel"]
  csv_header_row_count = 1
  csv_measurement_column = "measurement"
```

The following Grafana dashboard can be used to show all the graphs. [**Download the JSON file**](https://gist.githubusercontent.com/dkruyt/c79dfb26ed771b14ed9ed43ff3aba8f8/raw/24e90a36dbc308e6ed3680a078ab5b0d453e3244/Ziggo_Arris_Modem-Grafana.json) and import into [Grafana](https://grafana.com/). If you want to test the dashboard see a [demo page](https://snapshot.raintank.io/dashboard/snapshot/whlaUYGSWr8ZUx1zLoS7JJKT97ErPH0v?orgId=2) on raintank.io.

{{< figure src="/images/2019/08/screencapture-192-168-66-168-3000-d-27HkjgiWk-ziggo-arris-modem-2019-08-16-07_51_04.png" caption="Grafana dashboard" >}}

Now you can keep an eye on the signal values 😉. Below you can see the downstream power signals changing after a scheduled Ziggo maintenance in my neighbourhood.

{{< figure src="/images/2019/08/Screenshot-2019-08-16-at-16.20.31.png" caption="Ziggo maintenance" >}}





