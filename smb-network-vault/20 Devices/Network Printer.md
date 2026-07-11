---
tags: [device, endpoint]
aliases: [Printer, MFP, Multifunction Printer, Print Server]
---

# Network Printer

**Role:** shared printing/scanning/copying. Deceptively simple, historically the top helpdesk-ticket generator — and an underrated security hole (printers are full computers with disks, firmware, and rarely-changed default passwords).

**Position:** printers VLAN or SERVERS [[VLANs|VLAN]] with reachability from staff; **[[DHCP]] reservation** (never pure dynamic — every client references its address; never device-side static without IPAM entry → [[Documentation Standards]]).

## Connection architectures — pick deliberately

| Model | How | Verdict |
|---|---|---|
| **Direct IP** | Each client → printer:9100 | Simple, no server dependency; drivers managed per-client — fine ≤15 users |
| **Print server** | Clients → [[Server]] queue → printer | Central drivers/GPO deployment, visible queues, accounting → [[DNS DHCP Print Servers]] |
| Cloud print | Universal Print/PaperCut-style via cloud | Right answer for cloud-first orgs and remote staff |

Protocols you'll see: RAW/JetDirect 9100 (the direct-IP standard), IPP/IPPS 631 (modern, driverless), LPD 515 (legacy) → [[Common Ports]]. mDNS/AirPrint discovery is **L2 broadcast — doesn't cross VLANs** without an mDNS repeater/Avahi reflector on the [[Firewall (NGFW)]] — the classic "iPhones can't see the printer" cause.

## Setup — step-by-step

1. Unbox → connect **wired** (Wi-Fi printers wander when signal blips — cable if at all possible), power.
2. Print the config page from the panel (it shows the DHCP address), or check the lease table.
3. **Web UI** (`http://<ip>`): set admin password (defaults are public knowledge → [[Vendor Default IPs and Logins]]), HTTPS-only, hostname (`HQ-PRN-01`).
4. Create the DHCP reservation; update IPAM.
5. **Disable what you don't use** — printers ship promiscuous: Telnet, FTP, SNMP-write with `public`/`private` communities, IPv6 if unmanaged, cloud-print agents, open SMTP relay. Each is an attack surface → [[Security Incident Basics]].
6. Scan-to-folder: a dedicated low-privilege account writing to one [[NAS]] share (SMBv2+, never a user's own creds baked into the printer). Scan-to-email: authenticated submission on 587 → [[Common Ports]].
7. Deploy to clients per your architecture (GPO/Intune for fleet; IPP URL for driverless).

## Firewall posture

Staff → printer: 9100/631 only. **Printer → anywhere: deny except** its scan-to targets (NAS:445, mail:587) and vendor-update hosts if desired. *Why the egress rules: compromised printers exfiltrate; also stops the "printer spontaneously ordering toner" cloud chatter you never audited.* → [[Firewall Concepts]], [[Network Segmentation]]

## Troubleshooting

| Symptom | Path |
|---|---|
| "Printer offline" (client-side) | Ping it: dead → power/link/[[Switching and ARP]]; alive → client spooler (`Restart-Service Spooler`), stale port IP vs reservation |
| Everyone can't print | Printer hung (power-cycle — [[Power over Ethernet|PoE]]-style remote cycle rarely available; smart-PDU helps), or print server queue jammed |
| One user can't print | Driver/permissions — compare with a working client |
| Prints garbage pages | Wrong driver language (PCL vs PS) — reinstall correct driver |
| Scan-to-folder stopped | Password rotated/expired on the service account; SMB version hardening on the NAS side |
| AirPrint invisible | mDNS across VLANs (above) |
| Intermittent offline, Wi-Fi model | Move to cable; it was never the queue |

Ops: firmware ~semiannual (real CVEs → [[Firmware Update Procedure]]) · toner/consumable alerts via [[SNMP and Monitoring]] · **wipe/remove the disk on decommission** (MFP disks cache scanned documents) · lease-return printers get factory-reset + disk-sanitize.

Related: [[DNS DHCP Print Servers]] · [[NAS]] · [[Common Ports]] · [[Network Segmentation]]
