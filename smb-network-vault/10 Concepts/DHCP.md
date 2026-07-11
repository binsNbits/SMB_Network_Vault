---
tags: [concept, services]
aliases: [DHCP Server, DORA, DHCP Lease, IP Reservation]
---

# DHCP

Dynamic Host Configuration Protocol hands out [[IP Addressing|IP configuration]] automatically. Runs on the [[Router]], [[Firewall (NGFW)]], a [[Server]], or a dedicated appliance → [[DNS DHCP Print Servers]]. UDP 67/68 → [[Common Ports]].

## DORA — the lease handshake

```text
Client                       Server
  │── DISCOVER (broadcast) ──▶│   "any DHCP server out there?"
  │◀── OFFER ────────────────│   "here's 10.0.20.51 + options"
  │── REQUEST (broadcast) ───▶│   "I'll take it"
  │◀── ACK ──────────────────│   lease confirmed, timer starts
```

**Why broadcast matters:** DISCOVER can't cross subnets. Each [[VLANs|VLAN]] needs either its own DHCP scope on the gateway **or** a relay (`ip helper-address <server>` on the [[Layer 3 Switch]] SVI) forwarding requests to a central server. Forgotten relay = the single most common "new VLAN doesn't work" cause.

## What a lease delivers (options)

| Option | Content | Notes |
|---|---|---|
| — | IP + subnet mask | From the scope's pool |
| 3 | Default gateway | Usually the SVI/interface IP |
| 6 | [[DNS]] servers | Point at AD DNS if you run a domain → [[Server]] |
| 15 | Domain name | |
| 42 | NTP server | Time sync |
| 66/67 | TFTP/boot file | [[VoIP Phones and PBX|Phone]] provisioning |
| 43/custom | Vendor options | e.g., UniFi controller address → [[Wireless LAN Controller]] |

## Design rules

- **One DHCP server per subnet.** A rogue second server (someone's plugged-in home router!) races yours and hands out broken configs — classic outage. Mitigate with *DHCP snooping* on the [[Managed Switch]].
- **Pool sizing:** leave room for statics per the allocation scheme in [[Documentation Standards]] (e.g., pool .50–.239).
- **Lease time:** 1–7 days for staff; **1–4 hours for guest Wi-Fi** (high churn would exhaust the pool).
- **Reservations** (MAC→IP) for anything you must find reliably: [[Network Printer|printers]], [[NAS]], [[IP Cameras and NVR|cameras]] — better than device-side statics.

## Troubleshooting

| Symptom | Meaning | Fix |
|---|---|---|
| Client has 169.254.x.x | No DHCP answer reached it | Check port VLAN, trunk allowed-list, relay, server service ([[Private IP Ranges]]) |
| Right IP, wrong subnet's IP | Port in wrong VLAN | Fix access VLAN |
| Random duplicate-IP conflicts | Static device inside pool range | Move static out or shrink pool |
| Pool exhausted | Lease too long / pool too small / MAC-randomizing phones | Shorten lease, widen pool |
| Some clients fine, new ones fail | Pool full or snooping blocking | `show ip dhcp binding`, check snooping trust |

Diagnostic commands: `ipconfig /all`, `ipconfig /release && /renew` → [[CLI Quick Reference]].

Related: [[No Internet Playbook]] · [[DNS]] · [[VLANs]]
