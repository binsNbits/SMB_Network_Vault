---
tags: [concept, wireless]
aliases: [Wi-Fi, 802.11, WLAN, SSID, Channels, Roaming]
---

# Wi-Fi Fundamentals

Radio is a **shared, half-duplex medium** — only one device per channel can transmit at a time. Every Wi-Fi problem traces back to this fact plus physics.

## Bands and channels

| Band | Range/penetration | Speed | Non-overlapping channels | Notes |
|---|---|---|---|---|
| 2.4 GHz | Best | Slowest | **Only 1, 6, 11** (20 MHz) | Crowded: microwaves, Bluetooth, neighbors. IoT lives here |
| 5 GHz | Medium | Fast | ~25 (varies; DFS channels shared with radar) | The workhorse band |
| 6 GHz (Wi-Fi 6E/7) | Shortest | Fastest | Many, clean spectrum | Wi-Fi 6E+ clients only; WPA3 mandatory |

**Channel plan rule:** adjacent [[Wireless Access Point|APs]] on different non-overlapping channels; 2.4 GHz strictly 1/6/11; keep channel width 20 MHz on 2.4, 40–80 MHz on 5 GHz (wider = faster but more overlap in dense areas).

## Standards quickly

Wi-Fi 4 (n) → 5 (ac, 5 GHz only) → **6/6E (ax — OFDMA: efficient with many clients, the SMB sweet spot)** → 7 (be). Buy Wi-Fi 6 minimum today; the AP generation matters less than **coverage design and channel hygiene**.

## Signal quality numbers

| RSSI | Quality | Use |
|---|---|---|
| > −50 dBm | Excellent | |
| −50 to −65 | Good | Target for work areas |
| −65 to −75 | Marginal | VoIP/video degrade |
| < −75 | Poor | Effectively unusable |

**SNR** (signal minus noise) ≥ 25 dB for good performance. Measure with a phone app (WiFiman, Aruba Utilities) or the [[Wireless LAN Controller|controller]]'s client view.

## Architecture decisions

- **SSIDs = 2–4 maximum.** Every SSID broadcasts beacons on every AP, consuming airtime even when idle. Map SSIDs to [[VLANs]]: Staff→VLAN 20, Guest→40, IoT→50 → [[Network Segmentation]].
- **Coverage:** more small cells at lower transmit power beat few APs at max power — clients are the weak transmitter, and max-power APs create sticky-client and hidden-node problems. Wire every AP ([[PoE Switch]]); mesh only where cable is impossible → [[Mesh Node and Range Extender]].
- **Roaming:** clients decide when to roam, badly. Help them: 802.11r (fast transition), overlapping cells at ~ −67 dBm edges, min-RSSI kick on the controller.
- **Band steering:** on — pushes capable clients to 5/6 GHz.

## Security (order of preference)

1. **WPA3-SAE** (or WPA2/WPA3 transition during migration)
2. **WPA2-Enterprise (802.1X/RADIUS)** — per-user credentials; revoke one departing employee without rekeying everyone → ties into [[Server|AD/NPS]]
3. WPA2-PSK — fine for small teams; rotate on staff departure
4. Never: WEP, WPA1, open SSIDs without client isolation

Guest SSID: client isolation ON, rate-limited, own VLAN, captive portal optional.

Related: [[Wireless Access Point]] · [[Wireless LAN Controller]] · [[Wi-Fi Problems Playbook]] · [[QoS]]
