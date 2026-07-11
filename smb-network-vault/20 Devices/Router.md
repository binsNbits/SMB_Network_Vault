---
tags: [device, edge, layer3]
aliases: [Gateway, Edge Router, Default Gateway]
---

# Router

**Role:** connects networks to networks — forwards packets between your LAN(s) and the internet using [[Routing]], translates addresses via [[NAT]], and usually serves [[DHCP]]/[[DNS]] for small sites. In most modern SMBs the router function lives **inside the [[Firewall (NGFW)]]**; a standalone router appears when the ISP requires one, at the branch end of an [[SD-WAN Appliance|SD-WAN]], or in Cisco-style separations.

**Position:** `[[Modem]] → ROUTER → [[Managed Switch]] → everything`

## Prerequisites

[[IP Addressing]] · [[Subnetting and CIDR]] · [[NAT]] · [[Routing]] · [[DHCP]]

## Access ([[Device Access Methods]])

| Method | How |
|---|---|
| Browser | `https://<LAN-IP>` — defaults per [[Vendor Default IPs and Logins]] |
| SSH | `ssh admin@<LAN-IP>` after enabling |
| Console | RJ45/USB serial, 9600 or 115200 8-N-1 — for recovery and first boot |

## Initial configuration — step-by-step (with reasons)

1. **Connect laptop directly to a LAN port**, get DHCP or set a static in the router's default subnet. *Why: no dependencies on the production network during setup.*
2. **Change the admin password + create your admin account.** *Why: default creds are scanned for constantly.*
3. **Configure WAN:**
   - DHCP from ISP (cable/fiber) — usually zero-config
   - PPPoE (DSL) — ISP username/password
   - Static — IP/mask/gateway/DNS from the ISP letter
   *Verify: WAN status shows a public IP (private = double NAT → [[Modem]]).*
4. **Configure LAN:** pick your scheme from [[Private IP Ranges]] (avoid 192.168.0/1.x → VPN conflicts). Set the LAN IP (convention: .1) *before* enabling DHCP so leases hand out the right gateway.
5. **DHCP scope:** pool sized per [[Documentation Standards]] allocation plan; DNS option → your resolver choice ([[DNS]]).
6. **Disable remote (WAN) management + UPnP.** *Why: WAN-side admin = brute-force target; UPnP lets any LAN malware open inbound holes silently.*
7. **Update firmware** → [[Firmware Update Procedure]].
8. **Set NTP + timezone.** *Why: logs are useless with wrong clocks → [[SNMP and Monitoring]].*
9. **Back up the config** → [[Configuration Backup Procedure]]. Do this before AND after every future change.

## CLI reference (Cisco IOS-style; full crib → [[CLI Quick Reference]])

```text
show ip interface brief             ! triage: interfaces + IPs + status
show ip route                       ! routing table
show ip nat translations            ! active NAT sessions
show ip dhcp binding                ! who has which lease
interface gi0/0
 description WAN-to-ISP
 ip address dhcp                    ! or: ip address 203.0.113.7 255.255.255.248
 ip nat outside
interface gi0/1
 description LAN
 ip address 10.0.20.1 255.255.255.0
 ip nat inside
ip nat inside source list 1 interface gi0/0 overload    ! PAT
access-list 1 permit 10.0.20.0 0.0.0.255
ip route 0.0.0.0 0.0.0.0 203.0.113.1                    ! default route
copy running-config startup-config                       ! SAVE
```

## Browser configuration map (consumer/SMB web UIs)

| Menu (typical) | What lives there |
|---|---|
| Status / Dashboard | WAN IP, uptime, client list — first stop when triaging |
| Network → WAN | Connection type, MAC clone, MTU |
| Network → LAN | LAN IP, DHCP scope, reservations |
| NAT / Port Forwarding | DNAT rules → hygiene rules in [[NAT]] |
| Firewall / Security | Basic rules, UPnP (off), DoS settings |
| System | Password, firmware, backup/restore, NTP, logs |

## Use cases & conditionals

- **Just internet for a tiny office** → router alone suffices; add rules later.
- **Need VLANs, IDS, VPN, content filtering** → you want a [[Firewall (NGFW)]] as the edge instead.
- **Multiple VLANs on one router port** → router-on-a-stick subinterfaces, or push inter-VLAN routing to a [[Layer 3 Switch]].
- **Second ISP** → dual-WAN model or [[SD-WAN Appliance]] → [[High Availability]].
- **Hosting anything inbound** → prefer [[VPN Fundamentals|VPN]] over port forwards.

## Troubleshooting (→ [[No Internet Playbook]] for full tree)

| Symptom | Check |
|---|---|
| No WAN IP | Modem online? Renew WAN; PPPoE creds; MAC binding |
| WAN IP is 192.168.x/100.64.x | Double NAT / CGNAT → [[NAT]] |
| LAN works, no internet | Default route present? NAT enabled on correct interfaces? |
| Internet works, names don't | [[DNS]] settings in DHCP scope |
| Random disconnects | MTU (PPPoE needs 1492), overheating, ISP line → [[Modem]] stats |

## Ops

Firmware quarterly ([[Firmware Update Procedure]]) · config backup on change · syslog+SNMP to the monitor ([[SNMP and Monitoring]]) · on the [[UPS]] · replace consumer-grade units ~5 years or at EoL of security updates.

Related: [[Firewall (NGFW)]] · [[Modem]] · [[Managed Switch]] · [[VPN Appliance]]
