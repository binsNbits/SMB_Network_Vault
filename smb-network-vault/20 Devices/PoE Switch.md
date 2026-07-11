---
tags: [device, core, layer1, layer2]
aliases: [PoE Switch, Power over Ethernet Switch]
---

# PoE Switch

**Role:** a [[Managed Switch]] (usually) that also **powers** connected devices over the data cable → theory, standards (af/at/bt), and budgets in [[Power over Ethernet]].

**Feeds:** [[Wireless Access Point|APs]] · [[VoIP Phones and PBX|IP phones]] · [[IP Cameras and NVR|cameras]] · door controllers/[[IoT Devices]].

## Buying/planning decisions (conditionals)

1. **How many PoE ports?** Count powered devices + 25% growth. Partial-PoE models (8 of 24 ports) suit phone-light offices.
2. **Which standard?** Modern APs want 802.3at (PoE+) minimum; Wi-Fi 6E/7 APs and PTZ cameras may want 802.3bt → check datasheets against [[Power over Ethernet]] table.
3. **Total budget** — the spec that actually bites: 24-port "PoE+" with a 190 W budget ≠ 24×30 W. Sum realistic draws, add headroom.
4. **Fanless?** PoE switches run hot; office-placed units should be fanless or they'll be unplugged by annoyed humans. Racked units: mind airflow → [[Network Rack and Patch Panel]].
5. **[[UPS]] sizing:** the switch's *total* draw (PoE load + electronics) hits the UPS — a loaded PoE switch can be the biggest consumer in the rack. Phones staying up during outages = this pairing.

## Everything a managed switch does, plus power management

All of [[Managed Switch]] applies (VLANs, trunks, STP, security). PoE-specific CLI:

```text
show power inline                       ! per-port draw + remaining budget — check FIRST
show power inline gi0/5 detail          ! negotiation detail for one device
interface gi0/5
 power inline never                     ! disable PoE on a port (data continues)
 power inline static max 15400          ! reserve/limit wattage
 power inline port priority high        ! survives budget crunch (put APs/phones here)
```

**Remote power-cycle** — the everyday superpower; hung camera/AP, no ladder needed:
```text
interface gi0/17
 shutdown
 no shutdown
```
(Web-UI switches: port page → PoE toggle. Controller-managed: device page → Power Cycle.)

## PoE-specific troubleshooting

| Symptom | Cause → action |
|---|---|
| Device gets no power | Budget exhausted (`show power inline` — remaining ≈ 0) → raise priorities / upgrade; or passive-PoE device that can't negotiate → check specs |
| Device boots then dies under load | Negotiated af, needs at — class mismatch; force `static max`, verify cable |
| Far-end device flaky | Voltage drop: long/thin/coupled cable → shorten, re-terminate, Cat6 → [[Power over Ethernet]] |
| Newest camera dead, older ones fine | Classic silent budget exhaustion — the switch cut lowest priority |
| Port shows `faulty` in power inline | Short in cabling or PD — test cable, try known-good device |
| Everything rebooted at once | Switch PSU or brownout — check logs, verify [[UPS]] |

Ops: track budget utilization in monitoring ([[SNMP and Monitoring]] exposes PoE OIDs) · alert at >80% budget · otherwise identical care to [[Managed Switch]] (firmware, config backup, port map).

Related: [[Power over Ethernet]] · [[Managed Switch]] · [[Wireless Access Point]] · [[IP Cameras and NVR]]
