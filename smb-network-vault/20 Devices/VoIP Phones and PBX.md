---
tags: [device, endpoint, voice]
aliases: [IP Phones, VoIP, PBX, SIP Phones]
---

# VoIP Phones and PBX

**Role:** voice over the data network. Components: **IP phones** (SIP endpoints), a **PBX** (call brain — cloud-hosted like RingCentral/3CX-hosted, or on-prem VM/appliance), and a **SIP trunk** (the "phone line" from an ITSP). Protocols: SIP signaling 5060/5061, RTP audio ~10000–20000/udp → [[Common Ports]].

**Position:** VOIP [[VLANs|VLAN]] (30) — voice gets its own VLAN for [[QoS]] priority and isolation from PC malware → [[Network Segmentation]].

## The voice VLAN trick (why one jack serves phone + PC)

Phones have a built-in 2-port switch: wall → phone → PC. The switch port does double duty:

```text
interface gi0/5
 switchport mode access
 switchport access vlan 20      ! PC traffic: untagged, STAFF
 switchport voice vlan 30       ! phone traffic: tagged, VOIP
 spanning-tree portfast
```
The phone learns its VLAN via LLDP-MED/CDP from the [[Managed Switch]], tags its own traffic, passes the PC's through untagged. Phones are [[Power over Ethernet|PoE]]-powered (~4–7 W) → [[PoE Switch]] budget + [[UPS]] = phones survive power cuts.

## Deployment — step-by-step

1. **Choose the PBX model:**
   - **Cloud-hosted** (the 2026 SMB default): no PBX hardware; phones register outbound to the provider — survives your site dying; needs solid WAN + QoS
   - **On-prem** (3CX/FreePBX VM): control + cheaper trunks at scale; you own the security burden (exposed PBXs get toll-fraud'ed within days)
2. VOIP VLAN + [[DHCP]] scope, with **option 66/67 or vendor option pointing at the provisioning server** — phones boot, get the option, pull their config; zero-touch at scale.
3. Switch ports: voice-VLAN config above, fleet-wide via interface ranges.
4. **[[QoS]] end-to-end:** trust phone DSCP marks (EF) at the switch; priority queue + shaper on the [[Firewall (NGFW)]] WAN — the step that separates "works" from "works during the backup window."
5. Firewall for cloud PBX: allow outbound SIP/RTP to the provider's published ranges; **disable SIP ALG on every device in the path** (routers "helping" with SIP mangle it — the #1 VoIP misery source → [[NAT]]).
6. On-prem PBX extra: SIP trunk registration; **fail2ban + IP allowlists + strong SIP passwords** (toll fraud = your money); E911 configuration/location — a legal requirement.
7. Test: internal call, outbound, inbound, voicemail, transfer, *and* a call during a large file transfer (QoS proof).

## Phone access & management

Per-phone web UI (`http://<phone-ip>`, admin password from provisioning — change fleet defaults) for per-device debugging; **provisioning templates for everything else** — hand-configuring 30 phones is how config drift is born. PBX admin via its web console (on-prem: MGMT-VLAN-only access → [[Device Access Methods]]).

## Troubleshooting — voice has its own symptom language

| Symptom | Meaning | Fix path |
|---|---|---|
| **One-way audio** | Signaling OK, RTP blocked one direction | NAT/ALG (disable SIP ALG), asymmetric firewall rule, wrong RTP range → [[NAT]] |
| Choppy/robotic audio | Packet loss/jitter | [[QoS]] missing or shaper wrong; check WAN utilization; wired path errors → [[Switching and ARP]] |
| Phone won't register | Can't reach PBX/provider | VLAN/DHCP first (does it have the right subnet's IP?), then 5060 outbound rules, then credentials |
| Calls drop at exactly 15 min | SIP session-timer/NAT state expiry | Increase UDP timeout for 5060, enable keepalives |
| Phone boots, no config | Provisioning option 66 missing on the VOIP scope | Fix [[DHCP]] options |
| Echo | Analog interfaces/handset, rarely the network | Headset/gain settings |

Ops: phone firmware via provisioning (staged → [[Firmware Update Procedure]]) · monitor trunk registration + concurrent-call license headroom ([[SNMP and Monitoring]]) · document DIDs, trunk creds location, E911 addresses → [[Documentation Standards]] · review call logs monthly for fraud patterns (on-prem).

Related: [[QoS]] · [[VLANs]] · [[PoE Switch]] · [[NAT]]
