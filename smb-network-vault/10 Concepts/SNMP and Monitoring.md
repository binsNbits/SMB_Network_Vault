---
tags: [concept, operations]
aliases: [Monitoring, SNMP, Syslog, NetFlow, Alerting]
---

# SNMP and Monitoring

You cannot manage what you cannot see. Monitoring converts "users are calling" into "port gi0/24 errors spiked at 09:41."

## The four telemetry channels

| Channel | Ports → [[Common Ports]] | Gives you |
|---|---|---|
| **SNMP** (poll) | UDP 161 | Metrics: bandwidth/port, CPU, memory, temperature, [[Power over Ethernet|PoE]] draw, [[UPS]] battery |
| **SNMP traps** (push) | UDP 162 | Instant events: link down, PSU failure |
| **Syslog** | UDP 514 | Every device's log lines, centralized and searchable — configure on ALL devices |
| **NetFlow/sFlow** | varies | *Who* is using the bandwidth (per-conversation) — answers [[Slow Network Playbook]] questions |

## SNMP versions — decision

- **v2c:** community string ("password") in cleartext. Acceptable read-only, on the MGMT [[VLANs|VLAN]] only, with a non-default community (never `public`).
- **v3:** auth + encryption. Use when supported. Never enable SNMP *write* unless a tool truly needs it.

```text
snmp-server community S3cr3tRO ro 99      ! Cisco: RO community + ACL 99
access-list 99 permit 10.0.10.50          ! only the monitoring server may poll
logging host 10.0.10.50                   ! syslog target
service timestamps log datetime msec      ! useful log timestamps…
ntp server 10.0.10.1                      ! …require NTP everywhere
```

**NTP is part of monitoring:** unsynced clocks make cross-device log correlation impossible.

## SMB-appropriate monitoring stacks

| Tool | Character |
|---|---|
| **LibreNMS** | Open source, auto-discovering, SNMP-centric — the classic pick |
| **Zabbix** | Open source, powerful, steeper setup |
| PRTG | Commercial, polished, free ≤100 sensors |
| Uptime Kuma | Dead-simple service up/down checks |
| Vendor clouds (UniFi/Meraki) | Free with the ecosystem; single-vendor view only |

Run it on a small VM or the [[Server]] — and remember the monitor itself needs monitoring (a second, tiny external check).

## What to alert on (start small — alert fatigue kills monitoring)

1. Device down (gateway, switches, [[Firewall (NGFW)]], [[Server]], [[NAS]]) — page immediately
2. WAN down / failover engaged
3. [[UPS]] on battery
4. Disk >85% on server/NAS; RAID degraded
5. Port utilization sustained >80% (capacity signal)
6. Interface error counters climbing (cable/duplex → [[Switching and ARP]])
7. Backup job failed → [[Backup Strategy]]

Everything else: dashboard, not alert.

Related: [[Documentation Standards]] · [[High Availability]] · [[Slow Network Playbook]]
