---
tags: [reference, security]
aliases: [Default Credentials, Factory Defaults, Default Gateway IPs]
---

# Vendor Default IPs and Logins

Factory defaults for first-time access ([[Device Access Methods]]). **Verify against the label on the device or vendor docs — vendors change defaults between hardware revisions.**

| Vendor / product | Default IP | Default login | Notes |
|---|---|---|---|
| pfSense/Netgate | 192.168.1.1 | admin / pfsense | Setup wizard on first web login |
| Ubiquiti UniFi AP | DHCP client | ubnt / ubnt (SSH, pre-adoption) | Managed via controller → [[Wireless LAN Controller]] |
| Ubiquiti EdgeRouter | 192.168.1.1 | ubnt / ubnt | eth0 only |
| MikroTik RouterOS | 192.168.88.1 | admin / (blank or on label) | WinBox/WebFig/SSH |
| Cisco (many SMB switches) | 192.168.1.254 or DHCP | cisco / cisco | Forced password change on login |
| Netgear | 192.168.1.1 | admin / password | routerlogin.net |
| TP-Link | 192.168.0.1 | admin / admin | tplinkwifi.net |
| D-Link | 192.168.0.1 | admin / (blank) | |
| Synology NAS | DHCP | created at setup | Discover via find.synology.com; DSM on :5000/:5001 |
| QNAP NAS | DHCP | admin / MAC-derived (newer) | Qfinder utility |
| HP/Aruba switches | DHCP or 192.168.2.10 | admin / (blank) | |
| Zyxel | 192.168.1.1 | admin / 1234 | |
| Common ISP gateways | 192.168.0.1 / 192.168.1.1 / 192.168.100.1 | on device label | 192.168.100.1 = cable modem diagnostics ([[Modem]]) |
| APC UPS NMC | DHCP/BOOTP | apc / apc | [[UPS]] network card |

## Can't reach the default IP?

1. Your PC must be **in the same subnet** — set a static IP (e.g., device is 192.168.1.1 → set PC to 192.168.1.50/24). Why: no [[Routing|router]] exists between you and a factory-fresh device.
2. Connect **directly** with one cable — bypass the production LAN so DHCP conflicts and VLAN tags don't interfere.
3. Still nothing → factory-reset (hold reset 10–30 s while powered — check vendor doc) or use the console port → [[Device Access Methods]].

> [!danger] Security obligations
> - **Change every default password on day one.** Default creds are the #1 way SMBs get breached — automated scanners try them constantly.
> - Store new credentials in a **password manager**, never a spreadsheet → [[Documentation Standards]].
> - Disable WAN-side management everywhere → [[Firewall Concepts]].

Related: [[New Site Deployment]] · [[Security Incident Basics]]
