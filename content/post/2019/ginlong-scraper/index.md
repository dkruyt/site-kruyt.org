---
author: Dennis Kruyt
categories:
- ginlong
- pvoutput
- mqtt
- python
- solis
- pv
- solar
- influxdb
- Solarman
- Trannergy
- Omnik
date: "2019-05-13T19:07:58Z"
description: ""
draft: false
cover:
  image: /images/2019/10/photo-1545209463-e2825498edbf.jpeg
slug: ginlong-scraper
tags:
- ginlong
- pvoutput
- mqtt
- python
- solis
- pv
- solar
- influxdb
- Solarman
- Trannergy
- Omnik
title: Solis Ginlong Inverter Statistics Scraper
---


I have a Solis 5K-G Single Phase Inverter with a Ginlong LAN stick. This stick logs statistics of this inverter every 5 minutes to the Ginlong monitoring pages.

{{< figure src="/images/2019/09/4gsolis-1.jpeg" caption="Ginlong Solis Inverter" >}}

I want use those statistics on the Ginlong site for my own analyses en domotica. To get those statistics to my own systems I created a python script that scrapes PV and inverter statistics from the Ginlong monitor pages every 5 min. And then outputs it to influxdb, [pvoutput](https://pvoutput.org/list.jsp?userid=70678) and/or mqtt.

Possible it also works with the following inverters: Omnik Solar, Solarman and Trannergy inverters, they are using the same kind of stick. Let me know.

{{< figure src="/images/2019/05/grafana-dashboard-ginlong-small.png" >}}

## Install

Go to [https://github.com/dkruyt/ginlong-scraper](https://github.com/dkruyt/ginlong-scraper) and clone/download the ginlong-scaper.py script.

Install necessary python modules.

```
pip install paho-mqtt
pip install influxdb

```

Adjust the config. Set the outputs that are not needed to false.

```bash
# solis/ginlong portal config
username		= 'user@name' #your portal username
password 		= 'password' #your portal password
domain 			= 'monitoring.csisolar.com' #domain ginlong used multiple domains with same login but different versions, could change anytime. monitoring.csisolar.com, m.ginlong.com
lan 			= '2' #lanuage (2 = English)
deviceId        	= 'deviceid' # your deviceid, if set to deviceid it will try to auto detect, if you have more then one device then specify.

### Output ###

# Influx settings
influx 			= 'true' # output result to influx set to false if you dont want to use
influx_database 	= 'dbname'
influx_server 		= 'localhost'
influx_port 		= '8086'
influx_measurement 	= 'PV'

# pvoutput
pvoutput 		= 'true' # output result to pvoutput set to false if you dont want to use
pvoutput_api 		= 'apikey'
pvoutput_system 	= 'pvsystem'

# MQTT
mqtt 			= 'true' # output result to mqtt set to false if you dont want to use
mqtt_client 		= 'pv'
mqtt_server 		= 'localhost'
mqtt_username 		= 'username'
mqtt_password 		= 'password'

###

```

Create a cron entry, every 5 min is ok, because the inverter logs also every 5 min.

```
*/5 *     * * *     user	/opt/solis-influx/ginlong-scraper.py > /dev/null 2>&1

```

## 



