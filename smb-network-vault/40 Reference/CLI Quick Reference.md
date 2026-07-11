---
tags: [reference, cli]
aliases: [Cisco Commands, Command Crib Sheet, IOS Commands]
---

# CLI Quick Reference

Cisco IOS syntax shown — Aruba/HPE, Juniper, and others differ in syntax but share the same *concepts*. Access the CLI per [[Device Access Methods]].

## Mode ladder (Cisco IOS)

```text
Router>                 ← user EXEC (read-only basics)
Router> enable
Router#                 ← privileged EXEC ("enable mode": show, copy, debug)
Router# configure terminal
Router(config)#         ← global config
Router(config)# interface gi0/1
Router(config-if)#      ← interface config
  exit → up one level    end (or Ctrl+Z) → straight to privileged EXEC
```

## Show commands (safe — read-only; run these first)

| Command | Tells you |
|---|---|
| `show running-config` | Active config (RAM) |
| `show startup-config` | Saved config (NVRAM) — differs from running? Someone didn't save |
| `show ip interface brief` | Every interface: IP, up/down — **the #1 triage command** |
| `show interfaces gi0/1` | Errors, duplex, speed, drops — cabling problems live here |
| `show vlan brief` | [[VLANs]] and port membership |
| `show mac address-table` | Which MAC learned on which port → [[Switching and ARP]] |
| `show ip route` | Routing table → [[Routing]] |
| `show ip arp` | ARP cache (IP↔MAC) |
| `show spanning-tree` | [[Spanning Tree Protocol]] state, root bridge |
| `show power inline` | [[Power over Ethernet]] draw per port |
| `show cdp neighbors` / `show lldp neighbors` | What's plugged into each port |
| `show version` | Firmware, uptime, model, serial |
| `show logging` | Local log buffer |

## Change commands (require config mode — back up first: [[Configuration Backup Procedure]])

```text
hostname SW-CORE-01                      ! name the device (labeling standard → [[Documentation Standards]])
ip default-gateway 10.0.10.1             ! L2 switch management gateway
interface vlan 10
 ip address 10.0.10.2 255.255.255.0      ! management SVI
vlan 20
 name STAFF
interface gi0/5
 description Front-desk PC               ! ALWAYS describe ports
 switchport mode access
 switchport access vlan 20
 spanning-tree portfast                  ! edge port: skip STP delay
interface gi0/24
 description Uplink-to-router
 switchport mode trunk                   ! carries multiple VLANs tagged
 switchport trunk allowed vlan 10,20,30,40
```

## Save / restore / recover

```text
copy running-config startup-config       ! SAVE — changes are lost on reboot otherwise
write memory                             ! same thing, old habit
copy running-config tftp:                ! export config to TFTP server
reload                                   ! reboot (confirm prompt)
reload in 10                             ! safety net: auto-reboot in 10 min unless cancelled —
                                         ! use before risky remote changes; cancel with: reload cancel
```

## Diagnostics from any OS ([[Desktops and Laptops]])

| Task | Windows | Linux/macOS |
|---|---|---|
| My IP/mask/gateway | `ipconfig /all` | `ip a` / `ifconfig` |
| Reach a host | `ping 10.0.10.1` | `ping 10.0.10.1` |
| Path to host | `tracert 8.8.8.8` | `traceroute 8.8.8.8` |
| DNS lookup | `nslookup example.com` | `dig example.com` |
| Renew DHCP | `ipconfig /release && ipconfig /renew` | `sudo dhclient -r && sudo dhclient` |
| Flush DNS cache | `ipconfig /flushdns` | `sudo resolvectl flush-caches` |
| Open ports/conns | `netstat -ano` | `ss -tulpn` |
| ARP cache | `arp -a` | `ip neigh` |
| Test a TCP port | `Test-NetConnection host -Port 443` | `nc -zv host 443` |

Related: [[Managed Switch]] · [[Router]] · [[Layer 3 Switch]] · [[No Internet Playbook]]
