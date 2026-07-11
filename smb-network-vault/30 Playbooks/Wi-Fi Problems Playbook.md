---
tags: [playbook, troubleshooting, wireless]
aliases: [WiFi Issues, Wireless Troubleshooting, Bad WiFi]
---

# Wi-Fi Problems Playbook

Wireless triage. Physics and vocabulary: [[Wi-Fi Fundamentals]]. Fleet views: [[Wireless LAN Controller]].

## Step 0 — Classify the complaint

| Complaint | Jump to |
|---|---|
| Can't connect at all | A |
| Connects, no internet/no IP | B |
| Slow / drops in one place | C |
| Slow / drops everywhere | D |
| One device only | E |
| Roaming problems (dies walking between rooms) | F |

## A. Can't connect (association/auth failing)

1. Wrong password vs auth failure: controller client-log shows the reason (PSK mismatch, RADIUS reject).
2. **WPA3-only SSID + older client** → enable WPA2/WPA3 transition mode.
3. 802.1X: RADIUS server reachable? cert expired? (NPS/[[Server]] event log) — expired RADIUS cert = *everyone* fails at once.
4. Client-count limit on the AP/SSID hit (events/all-hands) → raise limits, add [[Wireless Access Point|APs]].
5. MAC filtering/NAC denying (check policy before blaming radio).

## B. Connects but no IP / no traffic

**This is a wire problem wearing a Wi-Fi costume.** The SSID's [[VLANs|VLAN]] isn't delivering:
1. AP's switch port trunk missing the SSID's VLAN → the #1 cause → [[Wireless Access Point]] port config
2. No [[DHCP]] scope/relay for that VLAN
3. Client got an IP but 169.254.x.x → same, DHCP never answered → [[Private IP Ranges]]
4. Guest SSID + client isolation working as designed (user expects LAN access guests don't get → policy conversation, not a fault)

## C. Bad in one place

1. **Measure there:** RSSI < −70 dBm ([[Wi-Fi Fundamentals]] table) = coverage hole → add/move an AP; mesh only if unwireable → [[Mesh Node and Range Extender]]
2. Good RSSI, bad SNR = interference: scan — neighbor on the same channel, microwave near the kitchen AP, non-Wi-Fi noise → change channel, 20 MHz width in dense areas
3. Many clients on one AP (controller shows load) → capacity, not coverage → add AP, lower both APs' power

## D. Bad everywhere

1. Controller RF view: channel overlap after auto-RF ran or a neighbor changed → re-plan channels (2.4: 1/6/11 only)
2. All APs on max power (sticky clients, hidden nodes) → reduce to medium, let cells shrink
3. Uplink/WAN bottleneck masquerading as Wi-Fi → wired speedtest first → [[Slow Network Playbook]]
4. After a firmware update → check vendor forums, roll back per [[Firmware Update Procedure]]
5. DFS events (5 GHz APs near radar/airport randomly channel-hop) → pin non-DFS channels

## E. One device

Driver update, forget/rejoin network, MAC randomization fighting reservations/NAC, band preference forced wrong, power-saving NIC settings ([[Desktops and Laptops]]). Test the same spot with a second device — if it's fine, stop touching the APs.

## F. Roaming

1. Sticky client (shows −78 dBm on far AP while under a near one): enable min-RSSI kick (~ −75 dBm) + 802.11r
2. Coverage gaps between cells (walk-test: hard drop mid-hallway) → cell overlap should hit ~ −67 dBm at edges
3. VoWiFi call drops on roam: 802.11r/k/v on, same VLAN across APs (L3 roams break sessions)

## Evidence kit

Phone analyzer app (channel + RSSI walk), controller client history (per-client RSSI/rate/roam log), `ping -t` while walking. **Never tune blind — measure, change one thing, re-measure** → log changes ([[Documentation Standards]]).
