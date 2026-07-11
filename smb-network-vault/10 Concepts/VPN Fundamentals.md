---
tags: [concept, security]
aliases: [VPN, IPsec, WireGuard, OpenVPN, Site-to-Site, Remote Access VPN]
---

# VPN Fundamentals

A VPN builds an **encrypted tunnel** over the internet so remote traffic behaves like LAN traffic. Terminated on the [[Firewall (NGFW)]], [[Router]], or a dedicated [[VPN Appliance]].

## The two shapes

| Type | Connects | Typical SMB use |
|---|---|---|
| **Remote access** ("road warrior") | One user's device ↔ office | Home workers reaching the [[NAS]], RDP, internal apps |
| **Site-to-site** | Office LAN ↔ office LAN, always-on | Branch ↔ HQ; both sides route to each other transparently |

## Protocol choice (2026 guidance)

| Protocol | Ports → [[Common Ports]] | Verdict |
|---|---|---|
| **WireGuard** | UDP 51820 | **Default choice.** Modern crypto, tiny config, fastest, near-instant reconnect on mobile |
| **IPsec (IKEv2)** | UDP 500/4500 | The site-to-site standard; universal vendor interop; native in Windows/iOS |
| OpenVPN | UDP/TCP 1194 | Mature, flexible, slower; TCP 443 mode traverses hostile networks |
| SSL-portal VPNs | TCP 443 | Vendor-specific (FortiClient etc.); fine when bundled with your firewall |
| PPTP / L2TP-PSK | — | **Broken/legacy — never deploy** |

Overlay meshes (Tailscale/ZeroTier = WireGuard + coordination) are excellent for SMBs without static IPs or behind CGNAT ([[NAT]]).

## Design rules (violate these and you'll meet [[VPN Troubleshooting Playbook]])

1. **No overlapping subnets.** Office on 192.168.1.0/24 + home user on 192.168.1.0/24 = routing chaos. Use uncommon office ranges → [[Private IP Ranges]].
2. **Split vs. full tunnel:** split (only office subnets via VPN) saves bandwidth; full (everything via office) enforces content filtering on remote staff. Choose per policy, document it.
3. Site-to-site needs **routes both ways** and matching "interesting traffic"/allowed-IPs definitions on both peers → [[Routing]].
4. VPN users land in a **VPN zone** with explicit firewall rules — not omnipotent LAN access → [[Firewall Concepts]], [[Network Segmentation]].
5. **MFA on remote access.** Stolen laptop ≠ network breach.
6. Prefer certificates/keys over PSKs for remote access; rotate what you can't replace.

## What VPN replaces

Every port-forwarded RDP (3389), SMB (445), NAS UI, camera NVR you're tempted to expose → put it behind the VPN instead. Inbound exposure is how SMB ransomware gets in → [[Security Incident Basics]].

## Quick health checks

- Tunnel up but no traffic → phase-2/allowed-IPs subnet mismatch or missing return route.
- Connects on some networks, not others → UDP blocked; enable NAT-T / TCP 443 fallback.
- Works by IP, not by name → push internal [[DNS]] to VPN clients.

Related: [[VPN Appliance]] · [[Firewall (NGFW)]] · [[SD-WAN Appliance]]
