---
tags: [device, core, layer2]
aliases: [Switch, Access Switch, Smart Switch]
---

# Managed Switch

**Role:** the wired LAN's backbone — forwards frames ([[Switching and ARP]]) with the management features that make a business network possible: [[VLANs]], [[Spanning Tree Protocol|STP]], [[QoS]], port security, LACP, and visibility ([[SNMP and Monitoring]]).

**Position:** `[[Firewall (NGFW)]] → MANAGED SWITCH → everything wired` — access switches per closet, uplinked to a core ([[Layer 3 Switch]] in bigger builds).

**Tiers:** "smart/websmart" (GUI-only, cheaper — fine for small sites) vs. fully managed (CLI+GUI, stacking). [[PoE Switch|PoE variants]] power APs/phones/cameras.

## Prerequisites

[[VLANs]] · [[Switching and ARP]] · [[Spanning Tree Protocol]] · [[Device Access Methods]]

## Access

| Method | Notes |
|---|---|
| Console | 9600/115200 8-N-1 — first config, recovery |
| SSH | After enabling; restrict to MGMT VLAN |
| Browser | HTTPS to management IP |
| Controller | UniFi/Omada/Meraki switches configure from the [[Wireless LAN Controller|controller]]/cloud |

## Initial configuration — step-by-step (Cisco-style; reasons inline)

```text
! 1. Identity — names appear in logs, LLDP, prompts → [[Documentation Standards]]
hostname HQ-SW-ACC-01

! 2. Management access on the MGMT VLAN — never manage over user VLANs
vlan 10
 name MGMT
interface vlan 10
 ip address 10.0.10.2 255.255.255.0
 no shutdown
ip default-gateway 10.0.10.1        ! L2 switch still needs a gateway to be reachable cross-subnet

! 3. Secure the box
username admin privilege 15 secret <strong>
ip domain-name corp.local
crypto key generate rsa modulus 2048
line vty 0 15
 transport input ssh
 login local
no ip http server                   ! or keep HTTPS only: ip http secure-server

! 4. VLANs + ports per plan → [[VLANs]]
vlan 20
 name STAFF
vlan 30
 name VOIP
interface range gi0/1-20            ! user ports
 switchport mode access
 switchport access vlan 20
 switchport voice vlan 30           ! daisy-chained IP phones → [[VoIP Phones and PBX]]
 spanning-tree portfast
 spanning-tree bpduguard enable     ! rogue-switch protection → [[Spanning Tree Protocol]]
interface gi0/24
 description UPLINK-to-FW
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30,40,50,60

! 5. Hygiene
interface range gi0/21-23           ! unused ports
 shutdown
 switchport access vlan 999         ! parking VLAN
ntp server 10.0.10.1
logging host 10.0.10.50             ! syslog → [[SNMP and Monitoring]]
snmp-server community <RO-string> ro

! 6. SAVE — unsaved config dies at reboot
copy running-config startup-config
```

Then export the config → [[Configuration Backup Procedure]].

## Daily-driver show commands ([[CLI Quick Reference]])

| Question | Command |
|---|---|
| Port up? errors? | `show interfaces gi0/5` / `show interfaces status` |
| What's on port 5? | `show mac address-table interface gi0/5`, `show lldp neighbors` |
| Which port has MAC/IP X? | `show mac address-table address <mac>` (get MAC from ARP → [[Switching and ARP]]) |
| VLAN membership | `show vlan brief` |
| Trunk carrying what? | `show interfaces trunk` |
| STP sane? | `show spanning-tree` (root = core?) |
| Config drift? | compare `show run` vs `show start` |

## Web-UI map (smart switches)

Status/dashboard (port states, traffic) · VLAN → 802.1Q (create IDs; per-port tagged/untagged = trunk/access) · Port config (speed/duplex/description) · Security (port security, DHCP snooping, 802.1X) · System (IP, users, firmware, backup).

## Use cases & conditionals

- **Uplink saturated?** → LACP bundle two+ ports (both ends!) or 10G uplinks → [[Slow Network Playbook]].
- **Need inter-VLAN routing at the switch?** → That's a [[Layer 3 Switch]].
- **Users plugging in rogue gear?** → `switchport port-security` (MAC limit 1–2, err-disable), BPDU guard, DHCP snooping — or full 802.1X → [[Network Segmentation]].
- **Two switches, redundancy?** → Dual uplinks + RSTP, or stack → [[High Availability]].
- **Cheap desk expansion?** → An [[Unmanaged Switch]] under a desk is acceptable *only* on an access VLAN with port security relaxed knowingly.

## Troubleshooting

| Symptom | Likely cause → fix |
|---|---|
| Device link-up but dead | Wrong access VLAN; or err-disabled (`show interfaces status err-disabled`; fix cause, `shut/no shut`) |
| Slow, errors climbing | Duplex mismatch / bad cable → [[Switching and ARP]]; re-terminate → [[Network Rack and Patch Panel]] |
| New VLAN dead across uplink | Trunk allowed-list missing the VLAN — the classic |
| Whole-LAN meltdown | Loop → [[Spanning Tree Protocol]] |
| Can't reach mgmt IP | Wrong gateway on switch, or you're testing from a VLAN without a route |

Ops: firmware ~semiannual ([[Firmware Update Procedure]]) · config backup on change · on the [[UPS]] · port map kept current ([[Documentation Standards]]).

Related: [[PoE Switch]] · [[Layer 3 Switch]] · [[Unmanaged Switch]] · [[VLANs]]
