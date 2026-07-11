---
tags: [playbook, troubleshooting, security]
aliases: [VPN Down, Tunnel Issues, VPN Debugging]
---

# VPN Troubleshooting Playbook

For tunnels that won't build, build-but-don't-pass-traffic, or flap. Concepts: [[VPN Fundamentals]] · Termination: [[VPN Appliance]] / [[Firewall (NGFW)]].

## The three failure stages — identify yours first

| Stage | Symptom | Meaning |
|---|---|---|
| 1. No handshake | Client "connecting…" forever; WireGuard handshake timestamp stale (`wg show`) | Packets not reaching the endpoint |
| 2. Tunnel up, no traffic | Connected, pings into the far side die | Routing/policy/overlap |
| 3. Partial/flaky | Some things work; drops periodically | DNS, MTU, or state timeouts |

## Stage 1 — No handshake

1. **Is the endpoint reachable at all?** From outside: `nc -zvu <public-ip> 51820` (or 500/4500) → [[Common Ports]]. Dead →
   - WAN down at the site → [[No Internet Playbook]]
   - Firewall WAN rule missing/changed → [[Firewall Concepts]]
   - Behind [[NAT]] without the forward; **CGNAT** (endpoint's WAN IP in 100.64/10 → inbound is impossible; redesign per [[VPN Appliance]] CGNAT options)
   - Dynamic IP changed and DNS name is stale → dynamic-DNS health
2. **Client network blocks UDP** (hotel/guest networks) → TCP/443 fallback (OpenVPN tcp, IKEv2→TLS) — diagnose by testing from a phone hotspot: works there = client-network problem.
3. Credentials/certs: expired cert, revoked key, wrong PSK — endpoint logs say which (`ipsec statusall`, VPN log on pfSense).

## Stage 2 — Tunnel up, no traffic

1. **Subnet overlap** — the classic: user's home LAN = office LAN range. `ipconfig` on the client: local subnet == remote subnet → renumber the office (long-term → [[Private IP Ranges]]) or NAT the tunnel (short-term).
2. **Routes:** client's AllowedIPs/split-tunnel list includes the office subnets? Office side has a **return route** to the tunnel pool? → [[Routing]] (two one-way trips!)
3. **IPsec phase-2 selectors mismatched** — HQ's local/remote must mirror the branch's exactly; a /24 vs /16 mismatch = SA builds, traffic dies → [[VPN Appliance]] site-to-site table.
4. **Firewall rules on the VPN zone** — tunnel ≠ permission: the VPN interface needs explicit allows → [[Firewall Concepts]].
5. One-way pings: capture on both ends (pfSense Diagnostics → Packet Capture; `diagnose sniffer packet` on Forti) — see where the packet vanishes; that side owns the bug.

## Stage 3 — Partial / flaky

1. **Works by IP, not name** → push internal [[DNS]] to VPN clients; full-tunnel clients may also need DNS-leak settings.
2. **Small pages load, big transfers/RDP stall = MTU.** Tunnel overhead shrinks usable MTU: set tunnel MTU 1400/MSS-clamp 1360 and retest; PPPoE WANs are prime suspects.
3. **Dies every N minutes exactly** → NAT state timeout on some middle router: keepalives (WireGuard `PersistentKeepalive = 25`; DPD on IPsec).
4. **Site-to-site flaps under load** → WAN saturation drowning the tunnel → [[QoS]] priority for tunnel traffic, or rekey-time collisions in logs.
5. Slow but stable → crypto CPU on the endpoint ([[VPN Appliance]] sizing), or expected: tunnel speed ≤ *slowest direction* of either side's WAN (usually someone's upload).

## Prevention

Monitor tunnel state + handshake age ([[SNMP and Monitoring]]) · certificate/PSK expiry calendar · document every peer: who, device, subnets, keys location → [[Documentation Standards]] · test after every firewall/firmware change on the endpoint ([[Firmware Update Procedure]]).
