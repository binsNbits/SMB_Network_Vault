---
tags: [concept, security]
aliases: [Segmentation, Zero Trust, Microsegmentation, Guest Network]
---

# Network Segmentation

Segmentation limits **blast radius**: when (not if) one device is compromised, walls stop lateral movement. Mechanism = [[VLANs]] for separation + [[Firewall Concepts|firewall rules]] for enforcement. One without the other is decoration.

## Why SMBs specifically need it

Ransomware playbook: phish one PC → scan the flat LAN → hit every share, camera, and server via SMB/RDP → encrypt everything including backups. On a flat 192.168.1.0/24 network this takes minutes. Segmented, the attacker meets a deny rule at every turn and the [[Backup Appliance]] on its own VLAN survives → [[Security Incident Basics]].

## The standard SMB segmentation model

Zones and VLAN/subnet mapping: see [[VLANs]] and [[Private IP Ranges]]. The **policy grid** is the design artifact:

| From ↓ To → | MGMT | STAFF | VOIP | GUEST | IOT/CAM | SERVERS | WAN |
|---|---|---|---|---|---|---|---|
| MGMT (admins) | — | ✔ | ✔ | ✔ | ✔ | ✔ | ✔ |
| STAFF | ✖ | — | ✖ | ✖ | ✖ | selected ports | ✔ |
| VOIP | ✖ | ✖ | — | ✖ | ✖ | PBX only | SIP provider |
| GUEST | ✖ | ✖ | ✖ | — | ✖ | ✖ | ✔ (rate-limited) |
| IOT/CAM | ✖ | ✖ | ✖ | ✖ | — | NVR only | vendor cloud only |
| SERVERS | ✖ | replies only | ✖ | ✖ | ✖ | — | selected |
| WAN | ✖ | ✖ | ✖ | ✖ | ✖ | ✖ (VPN only) | — |

"Selected ports" example: STAFF → SERVERS allow TCP 445 ([[NAS]] shares), 443 (intranet), 3389 to specific admin hosts only → [[Common Ports]].

## Implementation order (brownfield — migrating a flat network)

1. Document what exists: IPAM, port maps → [[Documentation Standards]].
2. Create VLANs + subnets on [[Managed Switch]]/[[Layer 3 Switch]] and firewall; add [[DHCP]] scopes per VLAN.
3. Move the **easy wins first**: GUEST, then IOT/CAM (nothing on the LAN should miss them).
4. Start each new inter-VLAN policy as *allow-any + log*, watch logs a week, convert observed-legit flows to explicit allows, then flip default to deny. **Why:** discovering dependencies by outage is worse.
5. VOIP, then SERVERS, then STAFF last (most dependencies).

## Beyond VLANs

- **Wireless:** client isolation on guest SSIDs ([[Wi-Fi Fundamentals]]).
- **Switch-level:** private VLANs / port isolation stop *intra*-VLAN device-to-device traffic (cameras never need to talk to each other).
- **802.1X/NAC:** ports authenticate the device before granting a VLAN — the endgame; RADIUS on a [[Server]].

Related: [[Firewall (NGFW)]] · [[IoT Devices]] · [[IP Cameras and NVR]] · [[New Site Deployment]]
