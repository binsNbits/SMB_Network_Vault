---
tags: [template]
aliases: []
---

# Site Note Template

> Copy per location. This is the note you open during a 2 AM outage — everything needed to act must be reachable from here. Keep an offline/printed copy per [[Configuration Backup Procedure]] logic.

---

# Site: <NAME>

**Address / access hours / who has keys:**

## Internet service

| Field | Value |
|---|---|
| ISP + support phone | |
| Account # / Circuit ID | |
| Service type & speeds | → [[Modem]] |
| Static IPs (block, gateway) | |
| Secondary WAN | → [[High Availability]] |

## IP plan

| VLAN | Name | Subnet | Gateway | DHCP pool | Notes |
|---|---|---|---|---|---|
| | | | | | → [[Private IP Ranges]] scheme |

## Device inventory (link each to a [[Device Note Template|device note]])

| Hostname | Class | Mgmt IP | Location |
|---|---|---|---|
| | [[Firewall (NGFW)]] | | |
| | [[Managed Switch]] | | |
| | [[Wireless Access Point]] | | |

## Key dependencies

- DHCP runs on: · DNS runs on: → [[DNS DHCP Print Servers]]
- Wireless controller: → [[Wireless LAN Controller]]
- Backup system + off-site target: → [[Backup Strategy]]
- UPS + battery install date: → [[UPS]]

## Emergency contacts & procedures

| Who | For | Contact |
|---|---|---|
| ISP | WAN down | |
| Electrician / landlord | Power, access | |
| IR / cyber-insurance hotline | → [[Security Incident Basics]] | |
| Key vendor(s) (PBX, POS, cameras) | | |

## Diagrams & docs

- Physical diagram: · Logical diagram: · Port map: → [[Documentation Standards]]
- Config backups location: → [[Configuration Backup Procedure]]
