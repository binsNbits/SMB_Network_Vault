---
tags: [device, edge, security]
aliases: [VPN Concentrator, VPN Gateway, Remote Access Server]
---

# VPN Appliance

**Role:** terminates encrypted tunnels — remote workers in, branch offices linked. Theory: [[VPN Fundamentals]].

> [!note] Do you even need a separate box?
> In most SMBs, **the [[Firewall (NGFW)]] is the VPN appliance** — pfSense/FortiGate/UniFi all terminate WireGuard/IPsec/SSL-VPN natively. Deploy a *dedicated* appliance/VM when: the firewall is ISP-locked, VPN crypto load starves the firewall CPU, you want the VPN in a DMZ separated from the edge, or you adopt an overlay (Tailscale/ZeroTier headend, or a self-hosted WireGuard VM).

**Position:** on the edge firewall itself, **or** a DMZ host with UDP 51820/500/4500 forwarded to it → [[Common Ports]], [[NAT]].

## Deployment — remote-access WireGuard (step-by-step, works on pfSense or a Linux VM)

1. **Plan addressing:** a dedicated tunnel subnet, e.g. `10.99.0.0/24`, that overlaps nothing at the office or in employees' homes → [[Private IP Ranges]]. *Why: overlap = broken routing, the #1 VPN failure.*
2. **Generate the server keypair**, listen port UDP 51820; allow that port in on WAN → [[Firewall Concepts]].
3. **Per user, per device:** generate a keypair, assign a tunnel IP (`10.99.0.11/32`), add as a peer. *Why per-device: revoke a lost laptop without touching anyone else.*
4. **Client config — AllowedIPs decides split vs. full tunnel** ([[VPN Fundamentals]]):
   - Split: `AllowedIPs = 10.0.0.0/16` (office subnets only)
   - Full: `AllowedIPs = 0.0.0.0/0`
   Push internal [[DNS]] in the client config so names resolve.
5. **Firewall rules for the VPN zone:** VPN users → only the services they need (files on [[NAS]], RDP to specific hosts) — not the whole LAN → [[Network Segmentation]].
6. **Return routing:** office gateway must route `10.99.0.0/24` back to the VPN host (automatic when the firewall *is* the host) → [[Routing]].
7. Distribute configs via QR (mobile) or file; **MFA** where the platform supports it; document each peer → [[Documentation Standards]].

## Deployment — site-to-site IPsec (checklist)

Both peers must **mirror** each other:

| Parameter | Must match | Notes |
|---|---|---|
| IKE version | ✔ | Use IKEv2 |
| Phase 1: encryption/hash/DH group | ✔ | AES-256-GCM / SHA-256 / DH 14+ |
| Phase 2: local↔remote subnets | mirrored | HQ's "local" = branch's "remote" |
| PSK or certs | ✔ | Long random PSK minimum; certs better |
| NAT-T | on if either side is NAT'd | UDP 4500 |
| Dead Peer Detection | on | Auto-rebuild dropped tunnels |

Static WAN IP on at least one side (the other can initiate by FQDN + dynamic DNS).

## CLI spot-checks

```text
wg show                        # WireGuard: peers, last handshake, transfer counters
                               # handshake >3 min old = peer unreachable
ipsec statusall                # strongSwan: SA states
diagnose vpn ike gateway list  # FortiGate phase 1
diagnose vpn tunnel list       # FortiGate phase 2
```
pfSense: Status → WireGuard / IPsec; VPN logs under Status → System Logs → IPsec.

## Conditionals

- **No static IP / CGNAT?** → Overlay mesh (Tailscale) or rent a $5 VPS as the WireGuard hub — both sides dial *out* → [[NAT]].
- **>50 concurrent users?** → Watch appliance CPU (crypto-bound); move VPN to dedicated hardware/VM.
- **Vendor needs temporary access?** → Time-boxed peer, restricted to one host:port, delete on completion — never a shared standing account.

## Troubleshooting → [[VPN Troubleshooting Playbook]]

Ops: keys/PSK rotation on staff departure · firmware ([[Firmware Update Procedure]]) — SSL-VPN portals are the single most-exploited SMB attack surface; patch same-week · monitor tunnel state ([[SNMP and Monitoring]]).

Related: [[Firewall (NGFW)]] · [[SD-WAN Appliance]] · [[VPN Fundamentals]]
