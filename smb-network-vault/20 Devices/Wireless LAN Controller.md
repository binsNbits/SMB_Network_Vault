---
tags: [device, wireless, operations]
aliases: [WLC, UniFi Controller, Wireless Controller, Cloud Controller]
---

# Wireless LAN Controller

**Role:** single pane of glass for the whole Wi-Fi fleet — config templates, coordinated channels/power (RRM), roaming assistance, firmware orchestration, client analytics. At ≥3 [[Wireless Access Point|APs]], per-AP management stops scaling; the controller is how business Wi-Fi is actually run.

## Form factors — pick one

| Form | Examples | Trade-offs |
|---|---|---|
| **Software, self-hosted** | UniFi Network Server on a VM/[[Server]]/Docker | Free, yours; you patch and back it up |
| **Hardware appliance** | UniFi Cloud Key/UDM, Omada OC200 | Plug-in simplicity; one more box on the [[UPS]] |
| **Cloud-hosted** | Meraki Dashboard, Aruba Central, Omada Cloud | Zero infra, multi-site native; subscription — Meraki APs **stop serving** when license lapses |
| Controller-on-AP | Some EnGenius/Omada modes | Tiny sites only |

**Key fact (UniFi/Omada):** APs keep serving Wi-Fi if the controller dies — you lose management/stats, not the network. (Meraki-style cloud-managed similarly data-planes locally.) So: controller availability is an ops concern, not an outage concern — but **its backup is critical** (it holds the entire wireless config).

## Deployment — self-hosted UniFi walkthrough

1. Install (Docker/apt/Windows) on a host with a **static IP** on the MGMT [[VLANs|VLAN]] → [[IP Addressing]].
2. Browse `https://<host>:8443` → setup wizard: admin account (+ MFA), site name.
3. Define the wireless estate *before* adopting: **Networks** (VLANs 20/40/50 with tags), **Wi-Fi** (SSID→VLAN, WPA2/3, guest isolation) — mirroring the plan in [[Wi-Fi Fundamentals]] / [[Network Segmentation]].
4. Adopt APs (they appear when L2-adjacent; cross-VLAN → inform via DNS/option 43 → [[Wireless Access Point]]).
5. Let RRM/"nightly channel optimization" set channels, then review — auto is a starting point, not gospel.
6. **Settings → System → Backup**: schedule auto-backups AND export off-box (the controller backing up to itself is not a backup) → [[Configuration Backup Procedure]], [[Backup Strategy]].
7. Firmware: stage per [[Firmware Update Procedure]] — controller first, then one canary AP, then fleet.

## What you manage here (surface map)

| Area | Contents |
|---|---|
| Dashboard | Client count, AP health, alerts — the morning glance |
| Devices | Per-AP: channel, power, overrides, port config, restart |
| Clients | Per-client: signal, rate, roam history — **start every "Wi-Fi is slow for me" ticket here** |
| Wi-Fi/Networks | SSIDs, VLANs, band steering, min-RSSI, 802.11r |
| Insights/Logs | Rogue APs, interference, retry rates |
| Hotspot | Guest portal, vouchers → guest [[VLANs|VLAN]] |

## Ports the controller needs open ([[Common Ports]], [[Firewall Concepts]])

UniFi: 8443 (UI), **8080 (device inform — adoption fails without it)**, 3478/udp (STUN), 10001/udp (discovery). If APs and controller sit on different VLANs, allow these across.

## Conditionals

- **Multi-site MSP-style management?** → cloud controller or one self-hosted with site separation; per-site local fallback creds documented → [[Documentation Standards]].
- **Controller VM died, need an AP change NOW?** → restore backup to a fresh install (same version!), re-adopts automatically. This is why the off-box backup exists.
- **Migrating controllers?** → export site/backup, restore on new host, update inform address (DNS `unifi` record makes this painless — set it up on day one → [[DNS]]).
- **AP stuck "adopting"/"disconnected"?** → SSH to AP, `info` shows the inform URL it's trying; reset + re-inform → [[Wireless Access Point]].

Ops: patch the controller host like any [[Server]] · MFA on admin accounts · monitor the controller service itself ([[SNMP and Monitoring]]) · verify backups restorable quarterly.

Related: [[Wireless Access Point]] · [[Wi-Fi Fundamentals]] · [[Wi-Fi Problems Playbook]]
