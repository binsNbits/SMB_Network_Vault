---
tags: [device, compute]
aliases: [File Server, Domain Controller, Application Server, Hypervisor]
---

# Server

**Role:** hosts the services the business runs on — identity (Active Directory), files, applications, databases, print queues, and infrastructure services ([[DNS]], [[DHCP]], RADIUS, the [[Wireless LAN Controller]]). Network-wise it's the most-depended-upon endpoint you own.

**Position:** SERVERS [[VLANs|VLAN]] (e.g., VLAN 60), static [[IP Addressing|IPs]], tight inbound rules → [[Network Segmentation]].

## SMB server roles — what actually runs on it

| Role | Provides | Notes |
|---|---|---|
| **Domain Controller** (AD DS) | Central logins, group policy, Kerberos | Clients MUST use it for DNS → [[DNS]]; deploy **two** DCs if the business depends on logins → [[High Availability]] |
| File server | SMB shares (TCP 445 → [[Common Ports]]) | Or delegate to a [[NAS]] |
| **Hypervisor** (Proxmox/Hyper-V/ESXi) | Many VMs on one box | The modern default: virtualize everything, snapshot before changes |
| App/DB server | LOB apps, SQL | Follow vendor port lists; segment DBs |
| Infra services | DNS, DHCP, NPS/RADIUS, print | → [[DNS DHCP Print Servers]] |

**Cloud-first note (2026):** many SMBs replace on-prem AD/file/mail with Entra ID + SharePoint/Drive. The on-prem server then shrinks to: hypervisor for legacy LOB apps, [[Backup Appliance|backup target]], and network services. Decide deliberately; don't run both worlds by accident.

## Network configuration — step-by-step

1. **Static IP** (never DHCP for a server — services and firewall rules reference it): IP/mask/gateway/DNS per IPAM → [[Documentation Standards]].
2. **NIC redundancy** if dual-port: LACP team to the switch (both ends configured → [[Managed Switch]]) or active-passive failover.
3. **Firewall rules:** host firewall ON + network rules per the [[Network Segmentation]] grid: staff → 445/443 only; management (RDP 3389/SSH 22) → MGMT VLAN or [[VPN Fundamentals|VPN]] only. **RDP never faces the internet.**
4. **Out-of-band management:** iDRAC/iLO/IPMI port → MGMT VLAN, its own creds; this is console access when the OS is dead → [[Device Access Methods]].
5. Time: DC = authoritative NTP for the domain, itself synced upstream; everything else follows → [[SNMP and Monitoring]].

## Access & administration

| Method | Use |
|---|---|
| RDP (Windows) / SSH (Linux) | Daily admin — via MGMT VLAN/VPN only |
| iDRAC/iLO web+console | BIOS-level, power control, OS-dead recovery |
| PowerShell / WinRM | `Enter-PSSession srv01`, scripting at scale |
| Physical console | Last resort; racks → [[Network Rack and Patch Panel]] |

```powershell
# Windows Server network essentials
Get-NetIPConfiguration                        # what am I
New-NetIPAddress -InterfaceAlias "Ethernet" -IPAddress 10.0.60.5 -PrefixLength 24 -DefaultGateway 10.0.60.1
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses 10.0.60.5,10.0.60.6
Test-NetConnection nas01 -Port 445            # service reachability
Get-SmbSession                                # who's on my shares
dcdiag /v                                     # AD health (on a DC)
```
```bash
# Linux equivalents
ip a; ip route                               # identity/routing
ss -tulpn                                    # what's listening
systemctl status smbd                        # service state
journalctl -u smbd --since "1 hour ago"      # service logs
```

## Conditionals

- **Buy a server or a [[NAS]]?** Need AD/GPO/LOB apps → server. Just files+backups → NAS is cheaper and simpler.
- **One big host or two?** Two modest hypervisors + replication beat one giant box the day hardware fails → [[High Availability]].
- **Server room?** Needs: cooling, [[UPS]] with graceful-shutdown integration, physical lock, rack → [[Network Rack and Patch Panel]].

## Troubleshooting (network-facing)

| Symptom | Check |
|---|---|
| "Server is down" | Ping it; ping its gateway; iDRAC reachable? (isolates OS vs network vs hardware → [[OSI Model]] bottom-up) |
| Reachable by IP, not name | [[DNS]] record; client's DNS server |
| Shares slow | NIC errors (`Get-NetAdapterStatistics`), duplex → [[Switching and ARP]]; disk latency (it's not always the network) |
| Logins slow/failing | DC reachable on 88/389/445? Time skew >5 min breaks Kerberos |
| After VLAN move, half-broken | Old IP referenced in firewall rules, DNS, apps — grep your docs → [[Documentation Standards]] |

Ops: **patches monthly** (server CVEs = ransomware entry) · image-level [[Backup Strategy|backups]] + tested restores · monitor disk/CPU/RAID/services → [[SNMP and Monitoring]] · dual PSU on separate [[UPS]] feeds where possible.

Related: [[NAS]] · [[DNS DHCP Print Servers]] · [[Backup Appliance]] · [[Network Segmentation]]
