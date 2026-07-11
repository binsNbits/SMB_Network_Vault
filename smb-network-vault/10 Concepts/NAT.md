---
tags: [concept, layer3]
aliases: [Network Address Translation, PAT, Port Forwarding, DNAT, SNAT]
---

# NAT

Network Address Translation lets many [[Private IP Ranges|private]] hosts share one public IP. Performed by the [[Router]] or [[Firewall (NGFW)]] at the internet edge.

## The three flavors

| Type | Direction | What it does | SMB use |
|---|---|---|---|
| **PAT / masquerade** (a.k.a. "NAT overload", SNAT) | Outbound | Rewrites source `10.0.20.51:52344 → 203.0.113.7:52344`, tracks the mapping, reverses it on replies | Default on every SMB edge — this is "the internet works" |
| **Port forward** (DNAT) | Inbound | `public:443 → 10.0.60.10:443` | Exposing a service; **prefer VPN instead** → [[VPN Fundamentals]] |
| **1:1 NAT** | Both | Maps one public IP fully to one internal host | Multiple public IPs, hosted servers |

## Consequences you must design around

- **Inbound is blocked by default.** Unsolicited traffic has no mapping → dropped. This accidental security is why home networks survive at all — but it is *not* a firewall; keep real [[Firewall Concepts|rules]] too.
- **Double NAT** — ISP [[Modem|gateway]] NATs *and* your router NATs. Breaks port forwards, some VoIP, some VPNs. **Fix:** bridge the ISP device or DMZ it to your router. Detect it: your router's WAN IP is itself private ([[Private IP Ranges]]).
- **CGNAT** — ISP-side NAT (WAN IP in 100.64.0.0/10). Port forwarding impossible; use outbound-initiated tunnels (WireGuard to a VPS, Tailscale, cloud relay).
- **Hairpin/loopback NAT** — internal clients hitting the *public* IP of an internal server need hairpin NAT enabled, or better, split-horizon [[DNS]].
- **SIP and NAT hate each other** — [[VoIP Phones and PBX|VoIP]] one-way-audio issues are usually NAT: disable SIP ALG (yes, *disable* — it mangles packets more than it helps), use STUN or a PBX-side SBC.

## Port-forward hygiene (when you truly must)

1. Forward the **exact port** to the **exact host** — never "DMZ host" a real machine.
2. Restrict the source IP on the rule if the far side is known (e.g., only the vendor's IP).
3. Put the exposed host in an isolated [[VLANs|VLAN]] → [[Network Segmentation]].
4. Log the rule and its business reason → [[Documentation Standards]].
5. Review quarterly; delete forwards nobody remembers.

## Verify NAT is working

- Inside host: `curl ifconfig.me` shows the router's public IP → outbound PAT fine.
- Inbound forward: test **from outside** (phone hotspot) — testing from inside tests hairpin, not the forward.
- State table full (rare, P2P-heavy nets): connections randomly fail → raise state limits on [[Firewall (NGFW)]].

Related: [[IP Addressing]] · [[Router]] · [[Common Ports]]
