---
tags: [concept, layer2]
aliases: [VLAN, 802.1Q, Virtual LAN, Trunk, Access Port]
---

# VLANs

A VLAN (802.1Q) splits one physical switch into multiple **logical** Layer-2 networks. Devices in different VLANs cannot talk without a [[Router]] or [[Layer 3 Switch]] — which is exactly the point: it creates enforcement chokepoints → [[Network Segmentation]].

## Core vocabulary

| Term | Meaning |
|---|---|
| **VLAN ID** | 1–4094. VLAN 1 = default — avoid using it for real traffic (see hardening below) |
| **Access port** | Belongs to ONE VLAN; frames untagged. For endpoints: PCs, [[Network Printer|printers]], [[IP Cameras and NVR|cameras]] |
| **Trunk port** | Carries MANY VLANs; frames carry an 802.1Q tag. For switch↔switch, switch↔router, switch↔[[Wireless Access Point|AP]] |
| **Native VLAN** | The one untagged VLAN on a trunk. **Must match on both ends** or traffic silently crosses VLANs |
| **Tagged/untagged** | Non-Cisco vendors' wording: tagged=trunk member, untagged=access |
| **Voice VLAN** | Access port carrying a second, tagged VLAN for a daisy-chained [[VoIP Phones and PBX|IP phone]] |

## Standard SMB VLAN plan (and *why* each exists)

| VLAN | Name | Why separate |
|---|---|---|
| 10 | MGMT | Device management UIs reachable only by admins |
| 20 | STAFF | Day-to-day users |
| 30 | VOIP | [[QoS]] priority + phones isolated from PC malware |
| 40 | GUEST | Visitors get internet ONLY — zero LAN access |
| 50 | IOT/CAM | [[IoT Devices]] and [[IP Cameras and NVR|cameras]] are the most-compromised device class — quarantine them |
| 60 | SERVERS | Tight inbound rules; crown jewels |

Subnet mapping for these: [[Private IP Ranges]].

## Configuration (Cisco-style — see [[CLI Quick Reference]])

```text
vlan 20
 name STAFF
interface gi0/5                      ! endpoint port
 switchport mode access
 switchport access vlan 20
 spanning-tree portfast
interface gi0/24                     ! uplink
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30,40,50,60
 switchport trunk native vlan 999    ! unused native — hardening
```

**Router-on-a-stick** (one router interface serves all VLANs via subinterfaces) vs. **[[Layer 3 Switch]] SVIs** (inter-VLAN routing in the switch — faster, standard once you exceed a few VLANs). Wi-Fi: each SSID maps to a VLAN; the AP's switch port must be a trunk → [[Wireless Access Point]].

## Conditionals & gotchas

- **Device gets no [[DHCP]] on a VLAN?** Either the port's VLAN is wrong, the trunk doesn't allow that VLAN, or the DHCP server/relay (`ip helper-address`) is missing for that subnet.
- **Everything works except one switch's devices?** Check the trunk `allowed vlan` list on the uplink — the most common VLAN mistake.
- **"VLAN hopping" hardening:** never leave native VLAN = 1 on trunks; shut down or blackhole-VLAN unused ports.
- **Do NOT create VLANs without firewall rules between them** — segmentation without enforcement is just labeling.

Related: [[Switching and ARP]] · [[Managed Switch]] · [[Firewall Concepts]] · [[Spanning Tree Protocol]]
