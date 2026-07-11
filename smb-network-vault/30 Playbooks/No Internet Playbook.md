---
tags: [playbook, troubleshooting]
aliases: [Internet Down, Total Outage, Connectivity Triage]
---

# No Internet Playbook

Symptom-driven triage for "the internet is down." Method: [[OSI Model]] bottom-up, halving the search space each step. Commands: [[CLI Quick Reference]].

## Step 0 — Scope the blast radius (30 seconds, saves an hour)

| Who's affected? | Points at |
|---|---|
| One user | Their device → [[Desktops and Laptops]] endpoint ladder |
| One area / one switch's users | That [[Managed Switch]] or its uplink |
| Wi-Fi only, wired fine | [[Wireless Access Point|APs]]/[[Wireless LAN Controller|controller]] → [[Wi-Fi Problems Playbook]] |
| One [[VLANs|VLAN]] | That VLAN's gateway/DHCP/trunk config |
| **Everyone, everything** | Edge: [[Modem]] / [[Firewall (NGFW)]] / ISP → continue below |

## Step 1 — From any affected client

```text
ipconfig /all      → valid IP? (169.254.x.x = DHCP failure → [[DHCP]], stop here and fix that)
ping <gateway>     → fails: L1/L2 problem inside → Step 4
ping 8.8.8.8       → works but names don't → [[DNS]] — you're done, fix DNS
nslookup google.com
```
**Gateway pings + 8.8.8.8 fails** = the edge or ISP → Step 2.

## Step 2 — The edge chain, in order

1. **[[Modem]] lights:** not "online" → ISP-side. Power-cycle modem, wait full sync, then router. Still down → check ISP status page/app (via phone LTE), call with your signal-level evidence → [[Modem]].
2. **Firewall WAN status** ([[Firewall (NGFW)]] dashboard): no WAN IP → renew; PPPoE creds; modem handoff. WAN IP is private/CGNAT → double-NAT situation changed → [[NAT]].
3. **From the firewall itself** (Diagnostics → Ping): ping 8.8.8.8 *sourcing WAN*. Works from firewall but not LAN → firewall LAN-side: rules changed recently? States stuck? NAT config? → [[Firewall Concepts]].
4. **Default route present?** `show ip route` / Status → Routes — a missing default after changes = nothing leaves → [[Routing]].

## Step 3 — "It's the ISP" checklist before you call

- Modem stats screenshot (levels/SNR vs your healthy baseline → [[Modem]])
- Bypass test: laptop direct to modem (brief — it's unfirewalled) — works = **not** the ISP, back to Step 2.3
- Outage confirmation via their status page; circuit ID ready → [[Site Note Template]]

## Step 4 — Inside faults (gateway unreachable)

1. Link lights / port state at both ends → [[Network Rack and Patch Panel]] L1 checks
2. Switch reachable? All ports err-disabled? Recent loop? → [[Spanning Tree Protocol]]
3. `show interfaces` errors, `show vlan brief` membership → [[Managed Switch]]
4. Whole switch dark → power/[[UPS]]? (Rack on battery beeping = the real story is electrical)
5. Gateway device itself hung → console access → [[Device Access Methods]], power-cycle as last resort

## Step 5 — After restoration

- Write the timeline + root cause in the change log → [[Documentation Standards]]
- If the fix was a reboot with no root cause: schedule follow-up before it recurs (check logs, firmware, [[SNMP and Monitoring]] graphs around the incident)
- If ISP-caused and business-painful: this is the [[High Availability|dual-WAN]] business case, quantified

> [!tip] Prevention beats triage
> [[SNMP and Monitoring]] with WAN + gateway + switch checks turns most of this playbook into an alert that names the failing layer before users call.
