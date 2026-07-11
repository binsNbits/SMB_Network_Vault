---
tags: [device, edge]
aliases: [SD-WAN, Software-Defined WAN, Dual WAN Appliance]
---

# SD-WAN Appliance

**Role:** manages **multiple WAN links** (fiber + cable + LTE/5G + legacy MPLS) as one smart pipe: per-application path selection, sub-second failover, and auto-built site-to-site overlay tunnels — [[VPN Fundamentals|VPN]] meshes without hand-built IPsec configs.

**Position:** replaces or fronts the edge [[Router]]; often the same box as the [[Firewall (NGFW)]] (FortiGate SD-WAN, Meraki MX, VeloCloud, UniFi Site Magic, Peplink).

## Do you need SD-WAN, or just dual-WAN? (decision)

| Situation | Answer |
|---|---|
| One site, want internet failover | **Dual-WAN on the firewall** — simpler, no new box → [[High Availability]] |
| 2+ sites, want site-to-site links that self-heal | SD-WAN earns its keep |
| VoIP/video quality varies by ISP path | SD-WAN: per-app steering by measured latency/jitter/loss |
| Replacing expensive MPLS | The classic SD-WAN business case: broadband + overlay ≈ MPLS reliability at a fraction of cost |
| CGNAT/LTE-only branches | SD-WAN overlays handle outbound-only links gracefully → [[NAT]] |

## Core concepts

- **Overlay vs. underlay:** underlay = the raw ISP links; overlay = encrypted tunnels between sites riding them. Your traffic lives in the overlay; the appliance constantly probes each underlay (latency/jitter/loss) and steers flows.
- **Application-aware routing:** policies like "SIP/[[VoIP Phones and PBX|voice]] → lowest-jitter path; O365 → direct internet breakout; bulk [[Backup Strategy|backup]] replication → cheapest path."
- **Orchestrator:** cloud dashboard managing all sites — templates, zero-touch provisioning (ship a box to a branch, it phones home, pulls config).

## Deployment — step-by-step (generic, orchestrator-based)

1. **Design first:** per-site subnets that never overlap (10.<site>.0.0/16 scheme → [[Private IP Ranges]]); which apps get which path priority; local internet breakout vs. backhaul-to-HQ for filtering.
2. Register appliances in the orchestrator (serials); build a **site template**: VLANs, DHCP, firewall policy → [[Network Segmentation]].
3. Ship + cable: WAN1 = primary ISP, WAN2 = secondary/LTE, LAN → site [[Managed Switch]]. Zero-touch pulls the template.
4. Verify overlay: tunnels green between sites; probe stats per link visible.
5. **Test failover on purpose** (pull WAN1) *before* users depend on it — measure: a live VoIP call should survive or re-setup within seconds → [[High Availability]].
6. Set alerting: link degradation, failover events → [[SNMP and Monitoring]].

## Configuration surfaces

Cloud dashboard (primary) · local web UI (bring-up/rescue) · CLI on Forti/Cisco flavors (`diagnose sys sdwan health-check`, `show sdwan status`) · console for recovery ([[Device Access Methods]]).

## Conditionals & gotchas

- **Asymmetric paths:** flow leaves via WAN1, return arrives WAN2 → stateful firewalls drop it. Orchestrated SD-WAN handles this; DIY dual-WAN policy routing must pin flows per-gateway.
- **LTE/5G backup:** watch data caps — alert on failover *and on failback failure* (running on metered LTE for a week because nobody noticed = a real bill).
- **Subscription reality:** most SD-WAN is licensed; lapse can degrade or stop management (Meraki: device stops passing traffic). Calendar the renewals → [[Documentation Standards]].
- **Don't double-firewall:** if SD-WAN box sits behind the NGFW, put it in bridge/one-arm mode or you're back to double [[NAT]].

Troubleshooting: per-link probe history answers "was it the ISP?"; overlay down on one site → check that site's underlay first ([[No Internet Playbook]] locally), then orchestrator reachability.

Related: [[Firewall (NGFW)]] · [[VPN Appliance]] · [[QoS]] · [[High Availability]]
