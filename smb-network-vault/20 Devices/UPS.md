---
tags: [device, infrastructure, power]
aliases: [Uninterruptible Power Supply, Battery Backup]
---

# UPS

**Role:** battery bridge + power conditioning for network gear. The **cheapest availability upgrade that exists** ([[High Availability]]): most outages are seconds-to-minutes, and a UPS turns them into non-events; longer ones become *graceful shutdowns* instead of corrupted [[NAS]] volumes and [[Server]] filesystems.

## Topologies — what you're buying

| Type | How | Use |
|---|---|---|
| Standby (offline) | Switches to battery on failure (~ms gap) | Desktops, small gear |
| **Line-interactive** | + voltage regulation (AVR) without battery hits | **The SMB rack default** |
| Online double-conversion | Always rebuilds the sine wave; zero transfer time | Servers/[[SAN]] in dirty-power environments |

## Sizing — step-by-step

1. **Sum the load (watts):** firewall ~20–60 W, [[Managed Switch|switch]] ~30 W, **[[PoE Switch]] = electronics + PoE budget draw** (often the rack's biggest consumer → [[Power over Ethernet]]), server 150–400 W, NAS ~60 W, [[Modem]]+misc ~20 W.
2. **Runtime target:** ride-through only (5–10 min + auto-shutdown) vs. work-through (laptops-on-Wi-Fi need [[Modem]]+firewall+switch+APs up: 30–60 min — note laptops self-power; only the network path needs the UPS).
3. Read the vendor's **runtime-at-load chart** (not the VA sticker); derate 20% for battery aging.
4. Watts vs VA: buy on watts; typical PF 0.6–0.9.

**What plugs in where:** battery outlets = network stack, servers, NAS, NVR. Surge-only outlets = monitors, printers ([[Network Printer|laser printers NEVER on battery]] — fuser inrush can trip/damage the UPS).

## The management card — a network device in its own right

Rack UPSes take an SNMP/network management card (APC NMC, Eaton gigabit card): its own IP on the MGMT [[VLANs|VLAN]], web UI + SSH, defaults per [[Vendor Default IPs and Logins]] (change them — an attacker who owns the UPS owns your power).

**Graceful-shutdown integration (the part people skip):**
- USB direct: NAS/server reads battery state (Synology: Control Panel → Hardware → UPS; one NAS can re-serve status to others as a "network UPS server").
- Network: `apcupsd`/NUT daemons or vendor agents (PowerChute) on each protected host, subscribed to the card: *on battery + X min → shut down VMs → hosts → NAS*. **Order matters:** compute down before storage; network gear last (it just dies harmlessly).
- Test the chain: pull the wall plug on a maintenance window and watch the cascade actually happen.

## Monitoring & alerts ([[SNMP and Monitoring]])

Alert on: **on-battery** (page immediately), battery-replace flag, runtime < threshold, self-test failure, temperature. UPS SNMP OIDs are well-supported in LibreNMS/PRTG out of the box.

## Troubleshooting

| Symptom | Meaning |
|---|---|
| Beeping steadily | On battery — is this a real outage or a tripped breaker? |
| Beeping + replace-battery LED | Batteries EOL (3–5 yr) — replaceable modules; order by UPS model |
| Load trips instantly on battery | Overloaded or dead batteries — recalc load, self-test |
| Random reboots of protected gear | UPS output relay/battery failing, or gear plugged into surge-only outlets by mistake |
| NMC unreachable | Card ≠ UPS: card can hang while power flows — reseat card |

Ops: **battery replacement on a 3–4 yr calendar**, not on failure · semiannual self-test + annual pull-the-plug drill · log battery dates on the unit → [[Documentation Standards]] · dispose of lead-acid properly.

Related: [[High Availability]] · [[PoE Switch]] · [[NAS]] · [[Network Rack and Patch Panel]]
