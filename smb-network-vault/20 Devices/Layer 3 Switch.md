---
tags: [device, core, layer3]
aliases: [L3 Switch, Core Switch, Multilayer Switch, SVI]
---

# Layer 3 Switch

**Role:** a switch that also **routes between [[VLANs]] at wire speed** in hardware. Becomes the network **core** once inter-VLAN traffic (staff↔servers, PCs↔printers) outgrows the [[Firewall (NGFW)]]'s router-on-a-stick capacity.

**Position:** `[[Firewall (NGFW)]] ⇄ L3 CORE SWITCH ⇄ access [[Managed Switch|switches]]` — access switches uplink here; the firewall becomes "just" the internet/security edge.

## The architectural decision (read before deploying)

| Inter-VLAN routing at… | Pros | Cons |
|---|---|---|
| Firewall (router-on-a-stick) | Every inter-VLAN packet passes firewall rules — maximum control | All east-west traffic squeezes through one trunk + firewall CPU |
| **L3 switch (SVIs)** | Wire-speed east-west; scales | Inter-VLAN filtering now needs switch ACLs — the firewall no longer sees LAN↔LAN traffic |

**SMB middle path (common):** route *trusted* VLANs (staff/servers/print) on the L3 switch; keep *untrusted* VLANs (guest, IoT/cameras) gatewayed on the firewall so [[Network Segmentation]] policy still applies where it matters.

## Configuration — step-by-step

```text
! 0. Prereqs: everything from [[Managed Switch]] baseline (hostname, SSH, VLANs, trunks)

! 1. Turn on routing
ip routing

! 2. SVIs — one virtual interface per VLAN; its IP is that VLAN's gateway
interface vlan 20
 description GW-STAFF
 ip address 10.0.20.1 255.255.255.0
 no shutdown
interface vlan 60
 description GW-SERVERS
 ip address 10.0.60.1 255.255.255.0
 no shutdown

! 3. Routed point-to-point link to the firewall (cleaner than a VLAN)
interface gi0/48
 no switchport
 ip address 10.255.0.2 255.255.255.252     ! /30 → [[Subnet Cheat Sheet]]

! 4. Default route toward the firewall → [[Routing]]
ip route 0.0.0.0 0.0.0.0 10.255.0.1

! 5. DHCP relay per SVI (server-central model) → [[DHCP]]
interface vlan 20
 ip helper-address 10.0.60.5

! 6. Save
copy running-config startup-config
```

**⚠ The matching firewall config (forgotten in most first deployments):** the firewall needs **static routes back** — `10.0.20.0/24 via 10.255.0.2`, `10.0.60.0/24 via 10.255.0.2`, etc. — or internet replies never reach the VLANs (works ping-to-switch, dies beyond). Plus its [[NAT]] source list must include those subnets. → [[Routing]] (asymmetric-route section).

## Inter-VLAN filtering on the switch (when the firewall no longer sees it)

```text
ip access-list extended STAFF-IN
 permit tcp 10.0.20.0 0.0.0.255 host 10.0.60.10 eq 445    ! staff→NAS SMB
 deny   ip  10.0.20.0 0.0.0.255 10.0.60.0 0.0.0.255       ! staff→servers: nothing else
 permit ip any any
interface vlan 20
 ip access-group STAFF-IN in
```
ACLs are stateless (no connection tracking) — keep the truly security-critical boundaries on the [[Firewall (NGFW)]].

## Also make it the STP root

The core must be the [[Spanning Tree Protocol]] root bridge: `spanning-tree vlan 1-4094 priority 4096`. *Why: traffic follows the tree — a wrong root sends everything through a closet switch.*

## Verification & troubleshooting

| Check | Command |
|---|---|
| SVIs up? | `show ip interface brief` (SVI needs ≥1 active port in its VLAN to come up) |
| Routing on? | `show ip route` — connected subnets listed |
| Host can't leave its VLAN | Host's gateway = SVI IP? SVI up? |
| Inter-VLAN OK, internet dead | Default route on switch + **return statics on firewall** + NAT scope |
| One VLAN dead everywhere | VLAN missing from an uplink trunk allowed-list |
| DHCP dead on a VLAN | `ip helper-address` missing on that SVI |

Ops: identical to [[Managed Switch]] (firmware, [[Configuration Backup Procedure]], [[SNMP and Monitoring]], [[UPS]]) — plus this box is now a **single point of failure for everything**: stack it, or keep a config-loaded cold spare → [[High Availability]].

Related: [[VLANs]] · [[Routing]] · [[Managed Switch]] · [[Network Segmentation]]
