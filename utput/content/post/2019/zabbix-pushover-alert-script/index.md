---
author: Dennis Kruyt
categories:
- pushover
- notification
- bash
- dark
- theme
- zabbix
- monitoring
- alert
date: "2019-05-14T16:45:52Z"
description: ""
draft: false
image: /images/2019/10/audio-blue-sky-daylight-776153_s.jpg
slug: zabbix-pushover-alert-script
tags:
- pushover
- notification
- bash
- dark
- theme
- zabbix
- monitoring
- alert
title: Zabbix Pushover Alert script
---


At home and at work we are running Zabbix for monitoring, for alerting I use Pushover. There are some scripts that have Zabbix and Pushover integration on share.zabbix.com, but none of them have priority support and have inline image for Pushover.

On my [GitHub](https://github.com/dkruyt/zabbix-pushover) you can find a alertscript for Zabbix and Pushover. Pushover has also a dark theme, but the Zabbix graphs are still light. So there is an option in the script to invert the graphs. So it wont hurt your eyes so much at night when you have oncall.

<div class="container">
    <div style="float:left;width:49%">
	    <img src="https://github.com/dkruyt/resources/raw/master/pushover_ios_small01.png">
    </div>
    <div style="float:right;width:49%">
	    <img src="https://github.com/dkruyt/resources/raw/master/pushover_ios_small03.png">
    </div>
</div>



## cli usage

```bash
notify_pushover.sh 'Pushover_Userkey|Pushover_apptoken' 'subject' 'message'

```

## Install

Download or Clone the notify-pushover.sh from my [GitHub](https://github.com/dkruyt/zabbix-pushover) and copy it  to the AlertScriptsPath from /etc/zabbix/zabbix_server.conf

Adjust the following configs in the script

```
# Zabbix address
ZBX_URL="https://zabbixserver"

# Zabbix credentials to login
USERNAME="zabbixuser"
PASSWORD="password"

```

Optional adjust the following

```bash
# Image time and size
PERIOD=24h
WIDTH=800
# invert image, better for black theme pushover and better for your eyes at night.
INV=true

```

## Configure Actions

Under _configuration -> actions_ in Zabbix create or change the notification messages to the following. The subject wil be used for the Pushover Priority. These Priority levels can be adjusted if needed in the bash script. Item Graphic will be used to extract an image from Zabbix en added to the Pushover message.

#### Operations

Default subject:

```
{TRIGGER.SEVERITY}: {EVENT.NAME}

```

Default message:

```
Hostname: {HOSTNAME}
Problem: {TRIGGER.NAME}:
Problem started at {EVENT.TIME} on {EVENT.DATE}

Severity: {EVENT.SEVERITY}

Last tested value: {{HOSTNAME}:{TRIGGER.KEY}.last(0)}
Item values: {ITEM.NAME1} ({HOST.NAME1}): {ITEM.VALUE1}

Item Graphic: [{ITEM.ID1}]

```

#### Recovery operations

Default subject:

```
Resolved: {EVENT.NAME}

```

Default message:

```
Hostname: {HOSTNAME}
Problem name: {EVENT.NAME}
Problem status: {STATUS}

Last tested value: {{HOSTNAME}:{TRIGGER.KEY}.last(0)}
Item values: {ITEM.NAME1} ({HOST.NAME1}): {ITEM.VALUE1}

```

#### Update operations

Default subject:

```
Updated problem: {EVENT.NAME}

```

Default message:

```
{USER.FULLNAME} {EVENT.UPDATE.ACTION} problem at {EVENT.UPDATE.DATE} {EVENT.UPDATE.TIME}.

{EVENT.UPDATE.MESSAGE}

Current problem status is {EVENT.STATUS}, acknowledged: {EVENT.ACK.STATUS}.

```

## Configure media type

Under _Administration -> Media_ in Zabbix add a new media. Specify the name of the script in script name and check that the parameters are correct.

{{< figure src="https://github.com/dkruyt/resources/raw/master/zabbix-mediatype.png" >}}

## Configure user media

You will need then to add the media to your users. For this just edit an user and add a media selecting the one you just created before. Specify the UserKey and AppToken in the Send to field, separated by a | .

{{< figure src="https://github.com/dkruyt/resources/raw/master/zabbix-usermedia.png" >}}

