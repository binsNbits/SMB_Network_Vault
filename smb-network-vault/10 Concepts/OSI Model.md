---
tags: [concept, fundamentals]
aliases: [OSI, Seven Layers, Layer Model]
---

# OSI Model

The mental map for **where** a problem lives. Every troubleshooting playbook in this vault implicitly walks these layers bottom-up.

| # | Layer | Unit | What it does | SMB examples | "Broken here" looks like |
|---|---|---|---|---|---|
| 7 | Application | data | User-facing protocols | HTTP(S), [[DNS]], SMB shares, SIP | App error, site loads but login fails |
| 6 | Presentation | data | Encoding, encryption | TLS certificates | Certificate warnings |
| 5 | Session | data | Session management | SIP dialogs, API sessions | Calls drop mid-conversation |
| 4 | Transport | segment | End-to-end delivery, **ports** | TCP, UDP → [[Common Ports]] | "Connection refused/timed out" |
| 3 | Network | packet | Logical addressing, **routing** | IP, ICMP → [[IP Addressing]], [[Routing]] | Can ping gateway but not beyond |
| 2 | Data Link | frame | Local delivery via **MAC** | Ethernet, [[VLANs]], [[Switching and ARP]], [[Spanning Tree Protocol]] | Link up but no traffic; wrong VLAN |
| 1 | Physical | bit | Cables, radio, signals | Cat6, fiber, [[Wi-Fi Fundamentals|RF]], [[Power over Ethernet|PoE]] | No link light, flapping port |

## Why it matters in practice

**Troubleshoot bottom-up.** ~50% of field issues are Layer 1 (cable, port, power). Checking layers in order prevents wasted hours:

1. **L1** — Link light? Cable seated? Port errors (`show interfaces` → [[CLI Quick Reference]])?
2. **L2** — Right [[VLANs|VLAN]]? MAC learned (`show mac address-table`)? STP blocking?
3. **L3** — Valid IP (not 169.254.x.x → [[Private IP Ranges]])? Can ping gateway? Route exists?
4. **L4** — Port open? Firewall rule permitting it → [[Firewall Concepts]]?
5. **L7** — Service actually running? DNS resolving?

## Where devices operate

| Device | Primary layer |
|---|---|
| [[Unmanaged Switch]], cabling, [[Network Rack and Patch Panel]] | 1–2 |
| [[Managed Switch]], [[Wireless Access Point]] | 2 |
| [[Layer 3 Switch]], [[Router]] | 3 |
| [[Firewall (NGFW)]] | 3–7 |
| [[Server]], [[NAS]] | 7 |

Related: [[No Internet Playbook]] · [[Slow Network Playbook]]
