---
author: Dennis Kruyt
categories:
- fortigate
- fortinet
- grafana
date: "2020-06-21T17:34:12Z"
description: ""
draft: false
cover:
  image: /images/2020/09/gate-4060998_1280--1-.webp
slug: fortigate_grafana
tags:
- fortigate
- fortinet
- grafana
title: Fortinet Fortigate in Grafana
---




{{< figure src="/images/2019/08/fortigate-grafana.png" >}}

To get metrics from your FortiGate in Grafana using Telegraf and InfluxDB.

* Enable SNMP on your FortiGate
* Put the following telegraf config in your `/etc/telegraf/telegraf.d` directory
* Edit the the agent IP and community string as appropriate

```
# # Retrieves SNMP values from remote agents
[[inputs.snmp]]
   agents = [ "192.168.1.1:161" ]
   timeout = "10s"
   retries = 3
   version = 2
   
   community = "monitoring"
#   ## SNMPv3 auth parameters
   sec_name = "sec_name"
   auth_protocol = "SHA"      # Values: "MD5", "SHA", ""
   auth_password = "auth_pass"
   sec_level = "authPriv"   # Values: "noAuthNoPriv", "authNoPriv", "authPriv"
#   #context_name = ""
   priv_protocol = "AES"         # Values: "DES", "AES", ""
   priv_password = "priv_pass"

  name = "FortiGate"
  [[inputs.snmp.field]]
    name = "hostname"
    oid = "SNMPv2-MIB::sysName.0"
  [[inputs.snmp.field]]
    name = "sysLocation"
    oid = "SNMPv2-MIB::sysLocation.0"
  [[inputs.snmp.field]]
    name = "uptime"
    oid = "DISMAN-EXPRESSION-MIB::sysUpTimeInstance.0"
  [[inputs.snmp.field]]
    name = "fnSysSerial"
    oid = "FORTINET-CORE-MIB::fnSysSerial.0"
  [[inputs.snmp.field]]
    name = "fgSysVersion"
    oid = "FORTINET-FORTIGATE-MIB::fgSysVersion.0"
  [[inputs.snmp.field]]
    name = "fgSysUpTime"
    oid = "FORTINET-FORTIGATE-MIB::fgSysUpTime.0"

  [[inputs.snmp.field]]
    name = "fgSysMemUsage"
    oid = "FORTINET-FORTIGATE-MIB::fgSysMemUsage.0"
  [[inputs.snmp.field]]
    name = "fgSysCpuUsage"
    oid = "FORTINET-FORTIGATE-MIB::fgSysCpuUsage.0"
  [[inputs.snmp.field]]
    name = "fgSysMemCapacity"
    oid = "FORTINET-FORTIGATE-MIB::fgSysMemCapacity.0"
  [[inputs.snmp.field]]
    name = "fgSysDiskUsage"
    oid = "FORTINET-FORTIGATE-MIB::fgSysDiskUsage.0"
  [[inputs.snmp.field]]
    name = "fgSysDiskCapacity"
    oid = "FORTINET-FORTIGATE-MIB::fgSysDiskCapacity.0"
  [[inputs.snmp.field]]
    name = "fgSysSesCount"
    oid = "FORTINET-FORTIGATE-MIB::fgSysSesCount.0"
  [[inputs.snmp.field]]
    name = "fgSysLowMemUsage"
    oid = "FORTINET-FORTIGATE-MIB::fgSysLowMemUsage.0"
  [[inputs.snmp.field]]
    name = "fgSysLowMemCapacity"
    oid = "FORTINET-FORTIGATE-MIB::fgSysLowMemCapacity.0"

# IF-MIB::ifXTable contains newer High Capacity (HC) counters that do not overflow as fast for a few of the ifTable counters
  [[inputs.snmp.table]]
    name = "FortiGate-interface"
# Interface tag - used to identify interface in metrics database
    [[inputs.snmp.table.field]]
      name = "ifName"
      oid = "IF-MIB::ifName"
      is_tag = true
    [[inputs.snmp.table.field]]
      name = "ifHCOutOctets"
      oid = "IF-MIB::ifHCOutOctets"
    [[inputs.snmp.table.field]]
      name = "ifHCInOctets"
      oid = "IF-MIB::ifHCInOctets"

#FORTINET-FORTIGATE-MIB::fgVdTable
  [[inputs.snmp.table]]
    name = "FORTINET-FORTIGATE-MIB::fgVdTable"
    inherit_tags = [ "hostname" ]
    oid = "FORTINET-FORTIGATE-MIB::fgVdTable"

#FORTINET-FORTIGATE-MIB::fgVdEntName
    [[inputs.snmp.table.field]]
      name = "fgVdEntName"
      oid = "FORTINET-FORTIGATE-MIB::fgVdEntName"
      is_tag = true

#VPN
  [[inputs.snmp.table]]
    ## measurement name
    name = "fgVpnTun"
    [[inputs.snmp.table.field]]
      name = "fgVpnTunEntPhase1Name"
      oid = "FORTINET-FORTIGATE-MIB::fgVpnTunEntPhase1Name"
      is_tag = true
    [[inputs.snmp.table.field]]
      name = "fgVpnTunEntPhase2Name"
      oid = "FORTINET-FORTIGATE-MIB::fgVpnTunEntPhase2Name"
      is_tag = true
    [[inputs.snmp.table.field]]
      name = "fgVpnTunEntInOctets"
      oid = "FORTINET-FORTIGATE-MIB::fgVpnTunEntInOctets"
    [[inputs.snmp.table.field]]
      name = "fgVpnTunEntOutOctets"
      oid = "FORTINET-FORTIGATE-MIB::fgVpnTunEntOutOctets"
    [[inputs.snmp.table.field]]
      name = "fgVpnTunEntStatus"
      oid = "FORTINET-FORTIGATE-MIB::fgVpnTunEntStatus"

#HA
  [[inputs.snmp.table]]
    ## measurement name
    name = "fgHaStats"
    [[inputs.snmp.table.field]]
      name = "fgHaStatsHostname"
      oid = "FORTINET-FORTIGATE-MIB::fgHaStatsHostname"
      is_tag = true
    [[inputs.snmp.table.field]]
      name = "fgHaStatsSyncStatus"
      oid = "FORTINET-FORTIGATE-MIB::fgHaStatsSyncStatus"
    [[inputs.snmp.table.field]]
      name = "fgHaStatsCpuUsage"
      oid = "FORTINET-FORTIGATE-MIB::fgHaStatsCpuUsage"
    [[inputs.snmp.table.field]]
      name = "fgHaStatsSerial"
      oid = "FORTINET-FORTIGATE-MIB::fgHaStatsSerial"
    [[inputs.snmp.table.field]]
      name = "fgHaStatsMemUsage"
      oid = "FORTINET-FORTIGATE-MIB::fgHaStatsMemUsage"
    [[inputs.snmp.table.field]]
      name = "fgHaStatsNetUsage"
      oid = "FORTINET-FORTIGATE-MIB::fgHaStatsNetUsage"
    [[inputs.snmp.table.field]]
      name = "fgHaStatsSesCount"
      oid = "FORTINET-FORTIGATE-MIB::fgHaStatsSesCount"
    [[inputs.snmp.table.field]]
      name = "fgHaStatsPktCount"
      oid = "FORTINET-FORTIGATE-MIB::fgHaStatsPktCount"
    [[inputs.snmp.table.field]]
      name = "fgHaStatsByteCount"
      oid = "FORTINET-FORTIGATE-MIB::fgHaStatsByteCount"
    [[inputs.snmp.table.field]]
      name = "fgHaStatsIdsCount"
      oid = "FORTINET-FORTIGATE-MIB::fgHaStatsIdsCount"
    [[inputs.snmp.table.field]]
      name = "fgHaStatsAvCount"
      oid = "FORTINET-FORTIGATE-MIB::fgHaStatsAvCount"

```



