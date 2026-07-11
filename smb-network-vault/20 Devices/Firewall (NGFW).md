---
tags: [device, edge, security]
aliases: [Firewall, NGFW, UTM, pfSense, FortiGate]
---

# Firewall (NGFW)

**Role:** the security brain of the network. Modern SMB practice: one appliance is edge [[Router]], [[Firewall Concepts|stateful firewall]], [[NAT]] engine, [[VPN Fundamentals|VPN]] terminator, [[DHCP]]/[[DNS]] server, and often IDS/IPS + content filter (then it's called UTM/NGFW).

**Position:** `[[Modem]] → FIREWALL → [[Managed Switch]]` — the chokepoint every inter-zone packet crosses; which is why [[Network Segmentation]] policy is enforced here.

**Common SMB platforms:** pfSense/OPNsense (open source), FortiGate, Sophos, WatchGuard, SonicWall, UniFi Gateway, Meraki MX. Examples below use pfSense + FortiGate flavor.

## Prerequisites

[[Firewall Concepts]] · [[NAT]] · [[VLANs]] · [[Network Segmentation]] · [[Common Ports]]

## Access ([[Device Access Methods]])

| Method | pfSense | FortiGate |
|---|---|---|
| Browser | `https://192.168.1.1` (admin/pfsense first boot) | `https://192.168.1.99` (admin/blank first boot) |
| SSH/CLI | menu-driven console + shell | full FortiOS CLI |
| Console | serial/VGA — interface assignment, password reset | serial 9600 8-N-1 |

**Never enable WAN-side management.** Admin via MGMT [[VLANs|VLAN]] or VPN only.

## Initial deployment — step-by-step (pfSense example, with reasons)

1. **Install/boot, assign interfaces** (console): WAN = ISP-facing NIC, LAN = inside NIC. *Why at console: mis-assigning interfaces over the network locks you out.*
2. Web UI setup wizard: hostname per [[Documentation Standards]], DNS, timezone/NTP, **new admin password**.
3. **WAN:** DHCP/PPPoE/static per ISP. Leave the defaults *Block private networks* + *Block bogons* on WAN. *Why: no legitimate WAN traffic sources from private space — dropping it kills spoofing cheaply.*
4. **LAN + VLANs:** Interfaces → Assignments → VLANs — create tags 10/20/30/40/50/60 on the LAN parent NIC per your [[Private IP Ranges]] plan; assign, enable, address each (e.g., `10.0.20.1/24`). The switch port toward the firewall becomes a **trunk** → [[VLANs]].
5. **DHCP per VLAN:** Services → DHCP Server → per interface: pool, DNS, reservations. *Why here: the firewall sees every VLAN, so it's the natural scope host in small nets.*
6. **Firewall rules per interface** — implement your policy grid from [[Network Segmentation]]:
   - GUEST: allow → any WAN; **block → all RFC1918**
   - IOT: allow → NVR + vendor cloud; block all else
   - STAFF: allow → SERVERS on named ports; allow → WAN
   - Enable logging on default-deny rules. *Why: the log is your debugging tool.*
7. **VPN:** WireGuard or IPsec per [[VPN Fundamentals]]; VPN clients land in their own zone with explicit rules.
8. **Optional NGFW services** — enable one at a time, watching CPU: IDS/IPS (Suricata: run IDS-mode first, review alerts ~1 week, then block), DNS filtering (pfBlockerNG), traffic shaping → [[QoS]].
9. **Backups:** Diagnostics → Backup/Restore — export XML now; enable AutoConfigBackup → [[Configuration Backup Procedure]].

## CLI flavor (FortiGate)

```text
get system status                          # version, HA state
get system interface physical              # interfaces + IPs
diagnose sniffer packet any 'host 10.0.20.51' 4    # live packet capture — the killer feature
get router info routing-table all
diagnose sys session list                  # state table
execute backup config flash                # local config snapshot
```
pfSense equivalents live in **Diagnostics** (Packet Capture, States, Routes) — GUI-first.

## Use cases & conditionals

- **"Which box do I buy?"** → size by *licensed users + WAN speed with IPS on* (vendors publish NGFW-throughput, which is far below raw throughput). Gigabit fiber + IPS needs a mid-tier unit, not the smallest.
- **Dual WAN** → gateway groups (pfSense) / SD-WAN feature (FortiGate) → failover or load-balance → [[High Availability]]; heavier multi-site needs → [[SD-WAN Appliance]].
- **HA pair** → pfSense CARP / FortiGate FGCP: two units, sync'd config + states, virtual IPs — for sites where minutes of downtime hurt.
- **TLS inspection?** Only with a deployed internal CA and awareness that pinned apps break. Most SMBs skip it and rely on DNS/SNI filtering.

## Troubleshooting

| Symptom | First checks |
|---|---|
| "App can't reach X" | **Firewall log first.** Blocked? Which rule? Rule order/shadowing → [[Firewall Concepts]] |
| Rule seems ignored | States! Existing sessions survive rule changes — kill states (Diagnostics → States) and retest |
| Slow with IPS on | CPU pegged? Reduce rule categories or upsize hardware |
| VPN issues | → [[VPN Troubleshooting Playbook]] |
| Locked out | Console → option to reset LAN rules/webConfigurator password |

## Ops

Firmware on a schedule — firewall CVEs are actively exploited within days ([[Firmware Update Procedure]]) · config backup on every change · syslog + SNMP → [[SNMP and Monitoring]] · quarterly rule review: delete stale rules and port forwards · on the [[UPS]].

Related: [[Router]] · [[VPN Appliance]] · [[Network Segmentation]] · [[Security Incident Basics]]
