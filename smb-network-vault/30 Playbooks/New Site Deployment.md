---
tags: [playbook, deployment]
aliases: [Greenfield Build, New Office Network, Site Buildout]
---

# New Site Deployment

End-to-end build order for a new SMB site. Each step links to the device/concept note carrying the detail. **Design on paper before unboxing anything** — renumbering a live network costs 10× the planning time.

## Phase 1 — Design (before purchase)

1. **Requirements census:** user count (+3 yr growth), wired drops per desk, Wi-Fi coverage area, [[VoIP Phones and PBX|phones]]?, [[IP Cameras and NVR|cameras]]?, [[POS Terminals|POS]]?, on-prem [[Server|servers]] vs cloud, downtime cost → [[High Availability]] tier.
2. **IP & VLAN plan:** site block (e.g., 10.<site>.0.0/16), VLANs per the standard model → [[VLANs]], [[Private IP Ranges]]; allocation conventions → [[Documentation Standards]].
3. **Segmentation policy grid** written down → [[Network Segmentation]].
4. **Bill of materials:** [[Firewall (NGFW)]] sized to WAN+IPS · [[Managed Switch]]/[[PoE Switch]] (count ports + PoE budget → [[Power over Ethernet]]) · [[Wireless Access Point|APs]] (density → [[Wi-Fi Fundamentals]]) + [[Wireless LAN Controller|controller]] · [[UPS]] sized to the stack · rack/panels → [[Network Rack and Patch Panel]] · [[NAS]]/[[Backup Appliance]] per [[Backup Strategy]].
5. **ISP order early** — circuit delivery is the long pole (weeks). Static IP if hosting/VPN → [[Modem]].

## Phase 2 — Physical (can parallel the ISP wait)

6. Cabling contractor: drops per plan, certified test reports required → [[Network Rack and Patch Panel]] standards.
7. Rack layout, [[UPS]] installed first (bottom), label everything as it lands.

## Phase 3 — Core bring-up (bench-configure before mounting where possible)

8. **Firewall:** full setup per [[Firewall (NGFW)]] — WAN, VLANs, [[DHCP]] scopes, [[DNS]], rules from the policy grid, [[VPN Fundamentals|VPN]]. Export config → [[Configuration Backup Procedure]].
9. **Switches:** baseline per [[Managed Switch]] (hostname, MGMT SVI, SSH, VLANs, trunks, port roles, [[Spanning Tree Protocol|STP]] root on the core → [[Layer 3 Switch]] if used). Save + export.
10. **Verify the skeleton:** from a test laptop on each VLAN: correct DHCP subnet, gateway pings, internet works, *cross-VLAN denies actually deny*. Test now — with zero users, fixes are free.

## Phase 4 — Services & wireless

11. [[Wireless LAN Controller|Controller]] up → adopt [[Wireless Access Point|APs]] → SSIDs→VLANs → walk-test coverage.
12. [[Server]]/[[NAS]]: static IPs in SERVERS VLAN, shares/AD per need, [[UPS]] shutdown integration.
13. [[Backup Appliance]] + first full backup + **first test restore** before go-live (the only restore test with zero risk).
14. Peripherals: [[Network Printer|printers]] (reservations, hardening), [[VoIP Phones and PBX|phones]] (voice VLAN, [[QoS]], test calls under load), [[IP Cameras and NVR|cameras]] (bench-config → mount), [[IoT Devices]] (inventory + IOT VLAN).

## Phase 5 — Validation & handover

15. **Failure drills before users arrive:** pull WAN (failover?), pull UPS wall plug (graceful shutdown cascade?), pull one uplink (STP recovery?) → [[High Availability]].
16. [[SNMP and Monitoring]]: all devices polled, alert list configured, baselines captured.
17. **Documentation package** → [[Site Note Template]]: diagram, IPAM, port map, credentials (password manager), ISP/circuit info, vendor contacts, config exports.
18. Go-live + hypercare week: watch logs/graphs; the first week's tickets reveal every plan-vs-reality gap while memory is fresh.

> [!tip] Order of operations rationale
> Edge→core→services→endpoints mirrors dependency order: each layer is testable the moment it lands because everything it depends on already works. Skipping the phase-3 verification is the classic source of "opening-day chaos."
