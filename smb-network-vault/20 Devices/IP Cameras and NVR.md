---
tags: [device, endpoint, security]
aliases: [Security Cameras, NVR, CCTV, Surveillance]
---

# IP Cameras and NVR

**Role:** video surveillance. **Cameras** stream (RTSP 554, ONVIF discovery) to an **NVR** (Network Video Recorder — appliance, [[NAS]] app, or VMS on a [[Server]]) which records, retains, and serves playback.

**Position:** cameras on an isolated CAM/IoT [[VLANs|VLAN]] (50), [[Power over Ethernet|PoE]]-fed → [[PoE Switch]]; the NVR straddles: one leg (or allowed flows) in the camera VLAN, management/viewing access from MGMT/staff per policy → [[Network Segmentation]].

> [!danger] Threat reality
> IP cameras are the most-botnetted device class on earth (Mirai et al.): hardcoded/default creds, ancient firmware, cloud P2P backdoors. The segmentation below is not optional hardening — it's the baseline.

## The non-negotiable firewall posture

| Flow | Verdict |
|---|---|
| Cameras → NVR (554/ONVIF) | Allow |
| Cameras → **internet** | **DENY** (breaks P2P cloud misfeatures — good) |
| Cameras → other VLANs, each other | Deny (port isolation on the switch if available) |
| Staff/admin → NVR web/app ports | Allow per policy |
| Remote viewing | **Via [[VPN Fundamentals|VPN]] only — never port-forward an NVR or camera** |
| NVR → NTP, vendor update host | Allow explicitly (cameras get time **from the NVR**, not the internet) |

## Deployment — step-by-step

1. **Plan coverage & retention:** entrances, POS, stock areas (mind jurisdiction privacy law — no audio in many places, signage requirements). Retention drives disk: `cameras × bitrate × days` — e.g., 8 cams × 4 Mbps ≈ 4 MB/s ≈ 345 GB/day → 30 days ≈ 10 TB. Surveillance-rated drives (WD Purple/Skyhawk).
2. Bench-configure each camera *before* the ladder: connect to a setup switch, set per-camera strong password, static IP or [[DHCP]] reservation in the CAM subnet, disable cloud/P2P/UPnP, update firmware, set the stream profile (H.265 saves ~40% disk over H.264).
3. Mount + cable home-runs to the [[PoE Switch]] (verify budget → [[Power over Ethernet]]); label everything → [[Documentation Standards]].
4. NVR: add cameras (ONVIF or vendor-native), **continuous + motion-event hybrid recording** (motion-only misses pre-event context; continuous at lower FPS + motion-boost is the usual compromise), configure retention/overwrite.
5. Firewall rules per the table above; test that a camera literally cannot ping 8.8.8.8.
6. Viewing: NVR mobile app via VPN; wall-monitor spot views via a display leg; export procedure documented (incident evidence needs correct timestamps — NTP matters legally).

## Access

Cameras: per-device web UI (`http://<ip>`; you set the creds at bench stage — no defaults survive → [[Vendor Default IPs and Logins]]). NVR: local console (HDMI), web UI, mobile app (VPN). RTSP check from any PC: `vlc rtsp://user:pass@<cam-ip>:554/stream1` — the universal "is the camera itself OK" test.

## Troubleshooting

| Symptom | Path |
|---|---|
| Camera offline | `show power inline` (PoE budget — the silent killer → [[PoE Switch]]) → link → remote power-cycle the port → cable (outdoor runs: water/surge — inspect) |
| Video loss at night | IR power draw pushes marginal PoE/cable over the edge — same checks |
| Choppy/artifacted streams | Uplink saturation (cams × bitrate vs switch uplink), or NVR disk maxed — check both |
| NVR "no signal" but RTSP-in-VLC works | NVR-side: re-add camera, creds, ONVIF port |
| Gaps in recordings | Disk full/failing (surveillance writes kill desktop drives), motion sensitivity, camera reboots — read camera uptime |
| Wrong timestamps | NTP chain broken — fix before you ever need evidence |

Ops: firmware for cameras is realistically "on install + when a CVE drops" (fleet-update via NVR where supported) · monthly: verify recording + retention actually met · monitor NVR disk SMART + camera up/down ([[SNMP and Monitoring]]) · NVR on the [[UPS]] · decommission = wipe recordings per policy.

Related: [[PoE Switch]] · [[Network Segmentation]] · [[NAS]] · [[IoT Devices]]
