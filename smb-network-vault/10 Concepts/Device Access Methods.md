---
tags: [concept, operations]
aliases: [Console Access, SSH Access, Web UI, Out-of-Band Management]
---

# Device Access Methods

Every managed device offers some mix of these. Know all four — when the network is down, only the console works.

## 1. Console (serial) — the lifeline

Works with **no network and no config** — first-time setup, password recovery, bricked-device rescue.

- **Hardware:** device's RJ45/USB-C console port ↔ your laptop via USB-serial adapter (FTDI-based recommended) or Cisco rollover cable.
- **Settings:** usually **9600 baud, 8-N-1, no flow control** (newer gear often 115200 — try both).
- **Clients:** PuTTY (Windows), `screen /dev/ttyUSB0 9600` or `minicom` (Linux/macOS). Find the port: Device Manager → COMx / `ls /dev/ttyUSB*`.
- Blank screen? Press Enter a few times; verify baud; check adapter driver.

## 2. SSH — daily CLI driver

Encrypted CLI over the network (TCP 22 → [[Common Ports]]). The replacement for telnet, which is plaintext and must be disabled everywhere.

```text
ssh admin@10.0.10.2                 # from any OS
ssh -o KexAlgorithms=+diffie-hellman-group14-sha1 admin@old-switch   # crusty old gear
```

Enabling on Cisco ([[CLI Quick Reference]]):
```text
hostname SW01
ip domain-name corp.local
crypto key generate rsa modulus 2048   ! host key — required for SSH
username admin privilege 15 secret <strong-password>
line vty 0 15
 transport input ssh                   ! ssh only, no telnet
 login local
```
Prefer **key-based auth** where supported. Restrict SSH sources to the MGMT [[VLANs|VLAN]] (`access-class`).

## 3. Web UI (HTTPS) — daily GUI driver

Most SMB gear is primarily web-managed. Rules:
- **HTTPS only**; accept/replace the self-signed cert (internal CA or firewall-issued cert removes the browser warning).
- Reach it via the device's **management IP** — recorded in IPAM → [[Documentation Standards]]; factory defaults → [[Vendor Default IPs and Logins]].
- Restrict UI access to the MGMT VLAN; **never expose management on WAN** → [[Firewall Concepts]].

## 4. Cloud / controller management

UniFi, Meraki, Aruba Central, TP-Link Omada: devices phone home to a controller; you configure fleet-wide from one dashboard → [[Wireless LAN Controller]]. Trade-offs: superb multi-site ops and zero-touch provisioning vs. vendor dependency, subscriptions (Meraki gear *stops working* unlicensed), internet-dependent management plane. Keep local fallback credentials documented.

## Access hierarchy when things are broken

1. Web/SSH via network → 2. from a host *on the same VLAN* (maybe routing broke) → 3. direct cable + static IP in the device's subnet → 4. **console** → 5. factory reset + restore from [[Configuration Backup Procedure]].

> [!tip] AAA hygiene
> Unique admin accounts per person where supported (or RADIUS/TACACS+ centrally), password manager for secrets, log out idle sessions, and audit who has access quarterly.

Related: [[Managed Switch]] · [[Router]] · [[Firewall (NGFW)]] · [[Security Incident Basics]]
