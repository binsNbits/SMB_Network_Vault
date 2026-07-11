---
tags: [device, endpoint, security]
aliases: [Smart Devices, IoT, Smart Thermostats, Access Control]
---

# IoT Devices

**Role:** the long tail — thermostats, door/access controllers, smart TVs and conference displays, sensors, badge readers, vending telemetry, digital signage, HVAC controllers, smart plugs. Individually trivial; collectively the network's **largest unpatched attack surface** and its least documented inhabitants.

**Position:** IOT [[VLANs|VLAN]] (50), guilty-until-proven-innocent firewall posture → [[Network Segmentation]].

## The IoT security posture (assume compromise, contain it)

| Flow | Rule | Why |
|---|---|---|
| IoT → its vendor cloud (documented hosts/ports) | Allow explicitly | It's how most of them function at all |
| IoT → internet broadly | Deny + **log** | The logs reveal what it *actually* talks to — often surprising |
| IoT → LAN/servers/staff | **Deny** | A smart TV never needs your [[NAS]] |
| IoT ↔ IoT | Deny (switch port isolation) where workable | Worm containment |
| Staff → specific IoT control ports | Allow narrowly (app → thermostat :443) | Usability |
| IoT → your [[DNS]]/NTP | Allow; **block external DNS** | Many hardcode 8.8.8.8 — force them through your resolver to keep visibility → [[DNS]] |

Cross-VLAN discovery (casting, AirPlay, "app can't find the device"): **mDNS repeater** on the [[Firewall (NGFW)]] for the specific service — the sanctioned fix; moving the IoT device to the staff VLAN is the unsanctioned one that becomes permanent.

## Onboarding procedure (every device, no exceptions)

1. **Inventory first:** what is it, who owns it, why is it here, vendor, model, MAC → [[Documentation Standards]]. Undocumented IoT is how "mystery device on the network" tickets are born.
2. Change default credentials ([[Vendor Default IPs and Logins]]); disable UPnP/P2P/telnet where the UI allows.
3. Join to the **IoT SSID** (2.4 GHz-friendly, WPA2 — many IoT can't do WPA3/5 GHz → [[Wi-Fi Fundamentals]]) or wired to an IOT-VLAN access port.
4. [[DHCP]] reservation; firmware update; note the cloud endpoints it needs (vendor doc or the deny-log from posture table).
5. Verify containment: from the device's subnet, confirm LAN targets unreachable.

## Special cases

- **Access control/badge/alarm systems:** life-safety adjacent — vendor-managed maintenance contracts common; get *their* port/host requirements in writing, give them exactly that, and put vendor remote-access behind your [[VPN Fundamentals|VPN]] or a time-boxed rule — never a standing port-forward → [[NAT]].
- **Conference-room gear (casting, video bars):** the one IoT class staff must discover — mDNS repeat just those services; test from the staff VLAN before the all-hands, not during.
- **Building HVAC/OT:** if it predates 2015, assume it's fragile: never scan it aggressively, isolate hard, and prefer a vendor gateway in front of it.
- **"Shadow IoT"** (personal smart speakers, random plugs): periodic sweep of DHCP leases + wireless client lists against inventory; unknown MACs get investigated → OUI lookup identifies the vendor ([[Switching and ARP]]).

## Troubleshooting

| Symptom | Path |
|---|---|
| App can't find device | Cross-VLAN discovery (mDNS above) — or the app needs same-subnet for *setup only*: onboard from a phone temporarily on the IoT SSID |
| Device offline after working | Check deny-logs — vendor changed cloud endpoints; update the allow rule |
| Won't join Wi-Fi | 2.4 GHz-only client vs 5 GHz-steering SSID; WPA3-only SSID; captive-portal confusion → dedicated IoT SSID settings |
| Works, then dies nightly | DHCP lease + poor sleep behavior — longer lease/reservation |
| Suspicious traffic in logs | Isolate the port/SSID entry, investigate → [[Security Incident Basics]] |

Ops: quarterly inventory reconciliation (leases vs. records) · firmware on install + CVE-driven · replace devices whose vendor stopped patching — an unpatchable IoT device is a standing liability the [[VLANs|VLAN]] merely contains.

Related: [[Network Segmentation]] · [[IP Cameras and NVR]] · [[Wi-Fi Fundamentals]] · [[Firewall Concepts]]
