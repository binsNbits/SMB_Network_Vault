---
tags: [concept, security]
aliases: [Firewall Rules, Stateful Inspection, ACL, Zones, DMZ]
---

# Firewall Concepts

A firewall permits or denies traffic according to **rules**. Understanding rule evaluation is prerequisite to configuring the [[Firewall (NGFW)]], [[Router]] ACLs, or inter-[[VLANs|VLAN]] policy.

## Stateful inspection — the foundation

The firewall tracks **connections**, not just packets. When an inside PC opens a session outward, the reply is matched to the state table and allowed automatically — you never write "allow return traffic" rules. Hence the default posture:

> **Outbound: mostly allowed. Inbound (unsolicited): denied.** Tighten outbound over time (egress filtering — e.g., only the mail server may use port 25; only your [[DNS]] resolver may use 53 outbound).

## Rule anatomy and evaluation

```text
action  source            destination        service/port
ALLOW   VLAN20 (staff)    VLAN60:10.0.60.10  TCP 445   "staff → NAS file shares"
ALLOW   VLAN20            any (WAN)          TCP 80,443
DENY    VLAN40 (guest)    RFC1918            any       "guests: internet only"
DENY    any               any                any       ← implicit/explicit final deny
```

- **First match wins, top-down** (nearly every vendor). Order specific→general.
- Rules apply **per interface/zone, usually on ingress** — pfSense evaluates on the interface where traffic *enters*.
- **Log the deny rules** at minimum; logs are how you debug "app X can't reach Y" → check the firewall log *first*, not last.
- Comment every rule with its business reason → [[Documentation Standards]].

## Zones — think in trust levels, not IPs

| Zone | Trust | Typical members |
|---|---|---|
| WAN | none | Internet |
| LAN/Staff | high | User VLANs |
| Servers | high, protected | [[Server]], [[NAS]] |
| Guest/IoT | none internally | [[IoT Devices]], visitor Wi-Fi |
| DMZ | low | Anything reachable from the internet |
| VPN | medium | Remote users → [[VPN Fundamentals]] |

Policy grid (write it before touching the GUI): which zone may initiate to which, on which services. This *is* [[Network Segmentation]].

## NGFW layer-7 extras

Beyond ports: application awareness (block BitTorrent even on 443), IDS/IPS (signature-based blocking — Suricata/Snort), TLS inspection (powerful; breaks cert pinning — decide deliberately), GeoIP blocking (crude but cuts log noise), web/content filtering categories.

## Common mistakes

| Mistake | Consequence |
|---|---|
| Rule above a more specific one shadows it | "Rule doesn't work" — it's never reached; use hit counters |
| Any-any-allow "just to test", left in place | No firewall at all |
| Managing from WAN enabled | Internet-wide brute force on your admin UI |
| Forgetting IPv6 rules | Entire policy bypassed over IPv6 → [[IP Addressing]] |
| No rules between VLANs | Segmentation theater |

Related: [[Firewall (NGFW)]] · [[NAT]] · [[Common Ports]] · [[Security Incident Basics]]
