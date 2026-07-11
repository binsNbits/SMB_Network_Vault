---
tags: [reference]
aliases: [Ports, Port Numbers, Well-Known Ports]
---

# Common Ports

Ports identify **which service** on a host traffic is destined for (Layer 4 of the [[OSI Model]]). You need these constantly: writing [[Firewall Concepts|firewall rules]], diagnosing blocked services, configuring port forwards on the [[Router]].

| Port | Proto | Service | Where you'll meet it |
|---|---|---|---|
| 20/21 | TCP | FTP (data/control) | Legacy file transfer — avoid; unencrypted |
| 22 | TCP | SSH / SFTP | CLI access to nearly every device → [[Device Access Methods]] |
| 23 | TCP | Telnet | Legacy CLI — **disable everywhere**, plaintext |
| 25 | TCP | SMTP | Mail relay; often blocked outbound by ISPs |
| 53 | TCP/UDP | DNS | [[DNS]] queries (UDP), zone transfers (TCP) |
| 67/68 | UDP | DHCP (server/client) | [[DHCP]] leases |
| 69 | UDP | TFTP | Firmware/config transfer to switches, phones |
| 80 | TCP | HTTP | Unencrypted web UIs — redirect to 443 |
| 88 | TCP/UDP | Kerberos | Active Directory auth → [[Server]] |
| 110/995 | TCP | POP3 / POP3S | Legacy mail retrieval |
| 123 | UDP | NTP | Time sync — critical for logs & certs |
| 137–139 | TCP/UDP | NetBIOS | Legacy Windows networking |
| 143/993 | TCP | IMAP / IMAPS | Mail retrieval |
| 161/162 | UDP | SNMP / SNMP traps | [[SNMP and Monitoring]] |
| 389/636 | TCP | LDAP / LDAPS | Directory lookups (AD) |
| 443 | TCP | HTTPS | Web UIs of almost every device |
| 445 | TCP | SMB | Windows/[[NAS]] file shares — **never expose to WAN** |
| 500/4500 | UDP | IKE / NAT-T | IPsec VPN → [[VPN Fundamentals]] |
| 514 | UDP | Syslog | Central logging |
| 587 | TCP | SMTP submission | Client mail sending (auth + TLS) |
| 631 | TCP/UDP | IPP | Network printing → [[Network Printer]] |
| 853 | TCP | DNS over TLS | Encrypted DNS |
| 1194 | UDP/TCP | OpenVPN | [[VPN Appliance]] |
| 1433 | TCP | MS SQL | Database — segment it → [[Network Segmentation]] |
| 3306 | TCP | MySQL/MariaDB | Database |
| 3389 | TCP | RDP | Windows remote desktop — **never expose to WAN**; use VPN |
| 5060/5061 | UDP/TCP | SIP / SIP-TLS | [[VoIP Phones and PBX]] signaling |
| 5000/5001 | TCP | Synology DSM | [[NAS]] web UI (HTTP/HTTPS) |
| 8080/8443 | TCP | Alt HTTP/HTTPS | Controllers ([[Wireless LAN Controller|UniFi]] inform = 8080) |
| 9100 | TCP | RAW/JetDirect | Direct-IP printing |
| 10000–20000 | UDP | RTP | VoIP audio streams (range varies by PBX) |
| 51820 | UDP | WireGuard | Modern VPN → [[VPN Fundamentals]] |

> [!warning] Rule of thumb
> If a port isn't needed **from the internet**, don't forward it. Expose services via VPN instead. See [[Firewall Concepts]].

Related: [[Firewall (NGFW)]] · [[Router]] · [[CLI Quick Reference]]
