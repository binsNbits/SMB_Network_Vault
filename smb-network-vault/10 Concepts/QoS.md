---
tags: [concept, operations]
aliases: [Quality of Service, Traffic Shaping, DSCP, Traffic Prioritization]
---

# QoS

Quality of Service prioritizes latency-sensitive traffic (voice, video) over bulk traffic (backups, downloads) when a link is congested. **QoS only matters at bottlenecks** — usually the WAN uplink; a quiet gigabit LAN needs none.

## When an SMB actually needs it

| Situation | QoS needed? |
|---|---|
| [[VoIP Phones and PBX|VoIP]] + limited WAN bandwidth | **Yes** — the canonical case |
| Video conferencing complaints during backup windows | Yes — or reschedule backups ([[Backup Strategy]]) |
| Symmetric gigabit fiber, 10 users | Probably not — solve congestion with bandwidth first |
| Guest Wi-Fi hogging the uplink | Rate-limit the guest [[VLANs|VLAN]] instead — simpler than full QoS |

**Rule: bandwidth first, QoS second.** QoS shares scarcity fairly; it cannot create capacity.

## The moving parts

1. **Classification/marking** — tag packets with DSCP values at the edge: voice = **EF (46)**, call signaling = CS3, video = AF41. [[VoIP Phones and PBX|IP phones]] mark their own; trust their marks on the switch port.
2. **Trust boundary** — [[Managed Switch|switches]] honor (or strip) incoming marks. Trust phones/APs; don't trust user PCs (anything could mark itself EF).
3. **Queuing/shaping at the bottleneck** — the [[Router]]/[[Firewall (NGFW)]] must know the **real** WAN speed (set shaper to ~95% of measured upload) so *it* becomes the queue manager instead of the ISP's dumb buffer (bufferbloat).

```text
interface gi0/5                 ! Cisco: trust the phone's marks
 auto qos voip cisco-phone     ! macro: trust + queue templates
```
pfSense: *Firewall → Traffic Shaper* wizard (PRIQ/HFSC) — enter true up/down speeds, prioritize VoIP by port/DSCP.

## Wi-Fi side

WMM (Wi-Fi Multimedia) is QoS for airtime — enabled by default on business [[Wireless Access Point|APs]] and *required* by Wi-Fi 6. Map SSID/VLAN to WMM classes in the [[Wireless LAN Controller|controller]].

## Verifying it works

- Symptom fixed = choppy calls during file transfers stop.
- `show policy-map interface` (Cisco) — queue drop counters: bulk queue drops under load = working as designed; **voice queue drops = misconfigured**.
- Measure bufferbloat: waveform.com/tools/bufferbloat before/after shaping.

Related: [[VoIP Phones and PBX]] · [[Slow Network Playbook]] · [[SD-WAN Appliance]]
