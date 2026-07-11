---
tags: [device, endpoint]
aliases: [Workstations, Endpoints, Client Devices, PCs]
---

# Desktops and Laptops

**Role:** the network's *consumers* — and, from a security view, its most likely infection vector (phishing lands here first → [[Security Incident Basics]]). Network-wise your goals: correct auto-configuration, useful diagnostics, and containment.

**Position:** STAFF [[VLANs|VLAN]] (wired access ports / staff SSID), addresses via [[DHCP]] — endpoints are the one class that should *never* need static IPs.

## What a healthy endpoint gets automatically

| Item | From | Verify |
|---|---|---|
| IP + mask | [[DHCP]] pool | `ipconfig /all` / `ip a` — a 169.254.x.x means DHCP failed → [[Private IP Ranges]] |
| Gateway | DHCP option 3 | `ping <gateway>` |
| [[DNS]] | option 6 (the DC if domain-joined → [[Server]]) | `nslookup dc01` |
| Time | Domain/NTP | Kerberos breaks at >5 min skew |
| Certificates/policy | GPO/MDM | domain logins, Wi-Fi 802.1X profiles |

## The endpoint as your test instrument

Every network diagnosis starts from a client — full command table in [[CLI Quick Reference]]. The canonical ladder ([[OSI Model]] bottom-up):

```text
1. ipconfig /all         valid IP? right VLAN's subnet?
2. ping <gateway>        L2/L3 local OK?
3. ping 8.8.8.8          routing + NAT OK?
4. nslookup google.com   DNS OK?
5. Test-NetConnection app-server -Port 443    the actual service?
```
Where it first fails = which note to open: 1→[[DHCP]] · 2→[[VLANs]]/[[Switching and ARP]] · 3→[[Router]]/[[No Internet Playbook]] · 4→[[DNS]] · 5→[[Firewall Concepts]]/the [[Server]].

## Management at SMB scale

- **Identity:** AD domain-join (on-prem [[Server]]) or Entra/MDM (cloud-first) — pick one path; hybrid intentionally, not accidentally.
- **Config at scale:** GPO or Intune-style MDM for Wi-Fi profiles, cert deployment, firewall policy, mapped drives to the [[NAS]]. Hand-configuring endpoints stops at ~10 machines.
- **Wired vs Wi-Fi rule:** stationary desktops get cable (frees airtime for laptops → [[Wi-Fi Fundamentals]]); docked laptops too.
- **Remote workers:** VPN client (per-device keys → [[VPN Appliance]]), MFA, full-disk encryption — a stolen laptop must be a hardware loss only.
- **EDR/AV + OS patching** everywhere; endpoints are *assumed compromised* in the [[Network Segmentation]] design — that's why servers, cameras and management interfaces don't trust the staff VLAN.

## Endpoint-side issues that masquerade as network problems

| "Network problem" | Actual cause |
|---|---|
| One user slow, everyone else fine | Their Wi-Fi position/driver, their disk, background sync — check the [[Wireless LAN Controller|controller's]] client view first |
| Can't reach one internal app | Host firewall rule, stale DNS cache (`ipconfig /flushdns`), cached credentials |
| Wi-Fi drops when docking | NIC power-saving, dock firmware — not the AP |
| "Internet down" for one machine | Static IP someone set long ago, now conflicting → return to DHCP |
| VPN broken at home only | Home router subnet overlap → [[VPN Fundamentals]] |
| Randomly duplicate IPs on LAN | Sleep/resume holding stale leases + a shrunk pool → [[DHCP]] |

Ops: patch cycle (monthly OS + browser) · asset inventory with MACs → [[Documentation Standards]] · MAC randomization OFF on corporate Wi-Fi profiles (it defeats DHCP reservations and NAC) · offboarding checklist: disable account, revoke VPN peer, reclaim device.

Related: [[DHCP]] · [[Wi-Fi Fundamentals]] · [[CLI Quick Reference]] · [[Security Incident Basics]]
