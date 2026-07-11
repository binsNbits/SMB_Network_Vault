---
tags: [device, wireless]
aliases: [AP, Access Point, WAP]
---

# Wireless Access Point

**Role:** bridges Wi-Fi clients onto the wired LAN. An AP is *not* a router — it's a wireless [[Managed Switch|switch port]] multiplier. Radio theory, bands, channels, security: [[Wi-Fi Fundamentals]].

**Position:** ceiling/wall-mounted, wired to a [[PoE Switch]] trunk port, managed standalone or via [[Wireless LAN Controller]].

## Physical deployment (where before how)

- **Ceiling-center of coverage areas**, not hallways (hall-mounted APs hear every room equally badly). One AP per ~1,500–2,500 sq ft of office as a starting density; walls (especially concrete/metal) shrink this.
- **Wire every AP.** Mesh backhaul halves throughput per hop → [[Mesh Node and Range Extender]] is the fallback, not the plan.
- Site survey: walk with a phone analyzer app pre/post install; target −65 dBm at work areas ([[Wi-Fi Fundamentals]] signal table).
- Power: 802.3at PoE+ for modern APs — verify against the [[PoE Switch]] budget → [[Power over Ethernet]].

## Switch port config (the step that breaks most installs)

Each SSID maps to a [[VLANs|VLAN]] — so the AP's port must be a **trunk**:

```text
interface gi0/17
 description HQ-AP-03
 switchport mode trunk
 switchport trunk allowed vlan 10,20,40,50   ! mgmt + every SSID VLAN
 spanning-tree portfast trunk
```
Symptom of getting this wrong: SSID broadcasts, clients associate, **no [[DHCP]]** → the SSID's VLAN isn't on the trunk.

## Configuration — standalone AP (small sites, 1–3 APs)

1. Power up, find its DHCP address (lease table / [[Vendor Default IPs and Logins]]), browse to it.
2. Set: static mgmt IP (or reservation) on the MGMT VLAN, admin password, hostname (`HQ-AP-01` → [[Documentation Standards]]).
3. Create SSIDs (≤4 → [[Wi-Fi Fundamentals]]): name, WPA2/WPA3 key or 802.1X, VLAN tag per SSID, guest = client isolation ON.
4. Radio: channels per plan (2.4: 1/6/11; 5 GHz spread), width 20/40–80, power *medium* not max.
5. Firmware → [[Firmware Update Procedure]]; export config → [[Configuration Backup Procedure]].

## Configuration — controller-managed (UniFi example; the normal SMB path ≥3 APs)

1. Controller runs first → [[Wireless LAN Controller]].
2. Cable the AP; it DHCPs and appears **pending adoption**; adopt in the UI. Cross-VLAN/site adoption: DHCP option 43, DNS record `unifi → controller-IP`, or SSH (`ubnt/ubnt`) → `set-inform http://<controller>:8080/inform`.
3. AP inherits site-wide SSID/radio config; per-AP overrides (channel/power) as needed.
4. All later management happens in the controller — per-AP UIs disappear.

## CLI (mostly diagnostic; APs are GUI/controller-first)

```text
ssh admin@<ap-ip>
info                        # UniFi: state, controller, firmware
mca-cli-op set-inform http://10.0.10.5:8080/inform
syswrapper.sh restore-default    # UniFi factory reset via SSH
```

## Conditionals

- **Coverage hole, cable impossible?** → mesh uplink to nearest wired AP → [[Mesh Node and Range Extender]].
- **High-density room (training/all-hands)?** → add an AP, *lower* everyone's power, 20 MHz channels — capacity comes from cells, not volume.
- **IoT gear won't join?** → 2.4 GHz-only SSID (or band-locked settings), WPA2 (not WPA3-only), on the IoT VLAN → [[IoT Devices]].
- **Outdoor?** → outdoor-rated APs, surge protection on the cable run.

## Troubleshooting (full tree → [[Wi-Fi Problems Playbook]])

| Symptom | Check |
|---|---|
| Associates, no IP | Trunk allowed-list (above); DHCP scope for that VLAN |
| AP offline in controller | PoE budget ([[PoE Switch]] `show power inline`), port err-disabled, adoption/inform address |
| Slow but strong signal | Channel overlap/interference — scan; co-channel APs? |
| Works near, dies far | Coverage — power was masking a layout problem; add AP |
| Random disconnects | Roaming settings, min-RSSI too aggressive, DFS radar events on 5 GHz |
| One device only | That client's driver/settings — test with a second device before touching the AP |

Ops: firmware via controller (staged: one AP → wait a day → fleet) · monitor AP up/down + client counts ([[SNMP and Monitoring]]) · PoE means the [[UPS]] question is the switch's, not the AP's.

Related: [[Wi-Fi Fundamentals]] · [[Wireless LAN Controller]] · [[PoE Switch]] · [[VLANs]]
