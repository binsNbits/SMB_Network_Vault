---
tags: [concept, services]
aliases: [Domain Name System, Name Resolution, DNS Records]
---

# DNS

DNS translates names → IPs. When DNS breaks, users say **"the internet is down"** even though connectivity is fine — always test both (`ping 8.8.8.8` vs `nslookup google.com`) → [[No Internet Playbook]]. Port 53 → [[Common Ports]].

## Resolution chain (what happens when a user opens a site)

```text
Browser → OS cache → hosts file → configured DNS server (from [[DHCP]])
  → that server's cache → root servers → TLD (.com) → authoritative server → answer, cached per TTL
```

## Record types you'll actually touch

| Type | Maps | SMB use |
|---|---|---|
| A / AAAA | name → IPv4/IPv6 | Internal hosts, your website |
| CNAME | name → another name | `mail.co.com → outlook.office365.com` |
| MX | domain → mail server | Email delivery |
| TXT | free text | **SPF/DKIM/DMARC** — email anti-spoofing; misconfigured = your mail lands in spam |
| PTR | IP → name | Reverse lookup; ISPs set these for mail servers |
| SRV | service → host:port | Active Directory *requires* these |

## Internal DNS — the SMB decision tree

- **Run Active Directory?** → Clients MUST use the domain controller as DNS ([[Server]]). AD breaks without its SRV records. The DC then *forwards* external queries upstream (1.1.1.1, 8.8.8.8, or ISP).
- **No AD, want internal names + filtering?** → Resolver on the [[Firewall (NGFW)]] (pfSense Unbound) or a Pi-hole/AdGuard box for network-wide ad/malware blocking.
- **Tiny office, no internal services?** → Point DHCP at a public resolver: 1.1.1.1 (Cloudflare), 8.8.8.8 (Google), 9.9.9.9 (Quad9, malware-filtering).

**Split-horizon DNS:** internal zone answers `nas.company.local → 10.0.60.10` while the public zone doesn't expose it — how users reach the [[NAS]] by name inside the LAN.

## Troubleshooting

| Test | Command ([[CLI Quick Reference]]) | Interpretation |
|---|---|---|
| Is it DNS? | `ping 8.8.8.8` OK but `ping google.com` fails | Yes, it's DNS |
| Who am I asking? | `ipconfig /all` (DNS servers line) | Wrong server = DHCP option 6 misconfig |
| Ask a specific server | `nslookup site.com 1.1.1.1` | Works ⇒ your normal server is the problem |
| Stale answer | `ipconfig /flushdns` | Caches hold old records for the TTL |
| Slow browsing, fast pings | Failing *primary* DNS timing out before secondary answers | Fix/replace primary |

> [!warning] Firewall note
> If you enforce a filtering DNS, **block outbound 53/853 from clients to anywhere else** ([[Firewall Concepts]]) — otherwise devices with hardcoded DNS (many [[IoT Devices]]) bypass your filtering.

Related: [[DHCP]] · [[Server]] · [[DNS DHCP Print Servers]]
