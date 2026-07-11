---
tags: [device, infrastructure, services]
aliases: [Infrastructure Services, Service Host, Pi-hole]
---

# DNS DHCP Print Servers

**Role:** the *placement decision* for core infrastructure services — [[DNS]], [[DHCP]], NTP, print queues, RADIUS. The services are documented in their concept notes; this note answers **"where should each service run?"** — a real architecture choice with failure-domain consequences.

## The placement matrix

| Service | On the [[Firewall (NGFW)]] | On a [[Server]]/DC | Dedicated box/VM (Pi-hole etc.) |
|---|---|---|---|
| [[DHCP]] | ✔ default for ≤3 VLANs, no AD | ✔ **required-ish with AD** (dynamic DNS updates); use relays → [[DHCP]] | rare |
| [[DNS]] resolver | ✔ (Unbound/pfBlocker filtering) | ✔ **mandatory for AD clients** (DC forwards upstream) | ✔ Pi-hole/AdGuard for network-wide filtering |
| NTP | ✔ natural fit (serves all VLANs) | DC serves the domain | — |
| Print queues | ✖ | ✔ classic print server → [[Network Printer]] | cloud print instead |
| RADIUS (802.1X, WPA-Enterprise) | some platforms | ✔ NPS on Windows / FreeRADIUS VM | ✔ small VM |

**Decision rules:**
- **Running Active Directory?** DNS lives on the DC(s), DHCP on Windows (or firewall pointing clients at DC DNS). Non-negotiable for AD health.
- **No AD, small site?** Firewall does DNS+DHCP+NTP — one box, one backup → [[Configuration Backup Procedure]].
- **Want ad/malware filtering?** Insert Pi-hole/AdGuard as the resolver clients get from DHCP; it forwards upstream. **Deploy two** or accept that its reboot = "internet down" for everyone.

## Failure-domain thinking (why placement matters)

Whatever hosts DHCP+DNS is a company-wide single point of failure with a *delayed* blast radius: DHCP dying hurts hours later (as leases expire, machines drop one by one — a confusing, creeping outage), DNS dying hurts instantly. Mitigations, in effort order:

1. Host them on the most reliable, [[UPS]]-backed, monitored box you have.
2. **Two DNS entries in every scope** (second = DC#2, the firewall, or 1.1.1.1 as last resort — noting a public fallback bypasses filtering/AD names when primary is down: acceptable for internet-only continuity, wrong for AD environments — prefer a second internal resolver).
3. DHCP redundancy: Windows DHCP failover pairs, or split scopes (server A: .50–.150, server B: .151–.239) → [[High Availability]].

## Migration cookbook (moving a service without an outage)

Example — DHCP from firewall to Windows server:
1. Build the new scope **identically** (pool, options 3/6/15/66, reservations — export/import; verify against IPAM → [[Documentation Standards]]).
2. Add `ip helper-address <new-server>` on VLANs' SVIs → [[Layer 3 Switch]].
3. Cut over in one motion: disable old server, enable new — **never run two authoritative DHCP servers on one subnet unconfigured for it** (rogue-DHCP chaos → [[DHCP]]).
4. Watch the lease table populate; keep the old config exported for instant rollback.

DNS moves: change option 6 in DHCP, then wait out lease renewals before decommissioning; grep configs for hardcoded references to the old resolver IP (printers! → [[Network Printer]]).

## Monitoring the invisible ([[SNMP and Monitoring]])

Synthetic checks beat metrics here: a probe that *actually does a DHCP handshake* and *resolves a name via each resolver* every few minutes. Alert on scope utilization >85% and DNS query latency — leading indicators of the creeping outages above.

Related: [[DHCP]] · [[DNS]] · [[Server]] · [[High Availability]]
