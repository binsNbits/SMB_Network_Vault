---
tags: [concept, services, security]
aliases: [SPF, DKIM, DMARC, Email Anti-Spoofing, Email Security Records, Sender Authentication]
---

# Email Authentication

SPF, DKIM, and DMARC are three [[DNS]]-published standards that let receiving mail servers verify a message claiming to come from your domain **actually did**. Raw SMTP ([[Common Ports|25/587]]) verifies *nothing* — anyone can put any address in the From field. Unauthenticated mail today means **spam-foldering or outright rejection** (Google and Yahoo enforce authentication for all senders since Feb 2024), and an unprotected domain means criminals can send invoices "from" you. Fixing a broken setup → [[Email Deliverability Playbook]] · record syntax and commands → [[Email Authentication Cheat Sheet]].

## The two "From" addresses (read this first — alignment depends on it)

Every message has **two** sender identities, and they routinely differ:

| Identity | Also called | Who sees it | Checked by |
|---|---|---|---|
| **Envelope From** (RFC 5321.MailFrom) | Return-Path, bounce address | Servers only — where bounces go | **SPF** |
| **Header From** (RFC 5322.From) | The From: line | **The human reading the mail** | **DMARC** (via alignment) |

Spoofing works because SMTP never cross-checks these. A phish can pass SPF for `bulkmailer.com` (its real envelope) while displaying `From: owner@yourcompany.com`. Only DMARC closes that gap — which is why SPF+DKIM without DMARC is an unfinished job.

## SPF — *which servers* may send for the domain

A single TXT record at the domain apex listing authorized sending sources. The receiver checks the **connecting server's IP** against the **envelope-from** domain's record.

```text
v=spf1 include:spf.protection.outlook.com ip4:203.0.113.25 -all
```

- **Mechanisms** (`ip4:`/`ip6:`, `include:`, `a`, `mx`) build the allow-list; the final `all` qualifier sets the default verdict: `-all` = fail everything else (goal), `~all` = softfail (transition only) → full syntax in [[Email Authentication Cheat Sheet]].
- **Exactly one `v=spf1` record per domain.** Two records = `permerror` = worse than none.
- **10 DNS-lookup limit**, counting *nested* includes — exceeding it is also `permerror`. This is the classic silent breakage as SaaS tools accumulate.
- **Weaknesses:** breaks on forwarding (the forwarder's IP isn't in your record), and it never looks at the visible From line — SPF alone does **not** stop display-From spoofing.

## DKIM — a *cryptographic signature* on the message itself

The sending platform signs selected headers + a body hash with a private key; the public key is published in DNS at `<selector>._domainkey.<domain>`. The signature's `d=` tag declares which domain signed.

- **Survives forwarding** (the signature travels with the message) — DKIM covers SPF's blind spot.
- **Selectors** let many services sign for one domain independently (`selector1` for M365, `google` for Workspace, `s1` for SendGrid…), and let you rotate keys without downtime.
- 2048-bit keys are the current standard (1024 minimum). A 2048-bit key exceeds DNS's 255-char string limit — published as two quoted strings; most DNS UIs handle this automatically.
- **Weakness:** mailing lists and some gateways modify subject/body in transit, invalidating the signature (ARC exists to relay original verdicts through such intermediaries).

## DMARC — the *policy* that ties it all to the visible From

A TXT record at `_dmarc.<domain>` telling receivers what to do with mail that fails, and where to send reports:

```text
v=DMARC1; p=quarantine; rua=mailto:dmarc@yourcompany.com; sp=quarantine
```

**DMARC passes if SPF passes *or* DKIM passes — AND the passing identity *aligns* with the header From domain.** Alignment is the whole point:

| Check | Aligns when… | Mode |
|---|---|---|
| SPF alignment | envelope-from domain matches header-From domain | `aspf=r` relaxed (same org domain, default) / `s` strict (exact) |
| DKIM alignment | signature `d=` domain matches header-From domain | `adkim=r` / `s` likewise |

This is why a third-party sender that signs with *its own* domain (`d=bulkmailer.com`) never helps your DMARC — it must sign as **your** domain.

- **Policies:** `p=none` (monitor only) → `p=quarantine` (spam-folder failures) → `p=reject` (refuse) — always rolled out in that order using the aggregate reports (`rua=`) to find legitimate senders you forgot → [[Email Deliverability Playbook]] Phase 4.
- `sp=` sets a separate policy for subdomains — attackers spoof `billing.yourcompany.com` when the apex is locked.

## Who checks what — one table

| Standard | DNS location | Proves | Defeated by |
|---|---|---|---|
| SPF | TXT @ apex | Sending IP is authorized for the *envelope* domain | Forwarding; display-From spoofing |
| DKIM | TXT/CNAME @ `<selector>._domainkey` | Message unmodified since a `d=` domain signed it | Content-modifying relays (lists) |
| DMARC | TXT @ `_dmarc` | An *aligned* SPF or DKIM pass backs the **visible** From | Nothing — but only as strong as its policy (`p=none` proves without enforcing) |

## Supporting players

- **MX** records handle *inbound* routing ([[DNS]]) — unrelated to outbound auth, but a wrong MX is its own delivery outage.
- **PTR / reverse DNS:** any server sending *directly* to the internet (on-prem relay on a [[Server]], appliances) must have a PTR matching its HELO name — big receivers reject IPs without one. Set by whoever owns the IP (ISP → [[Modem]] static-IP service).
- **MTA-STS / TLS-RPT:** enforce TLS on inbound delivery — nice hardening after the big three are done.
- **BIMI:** your logo in inboxes; requires DMARC at `p=quarantine`/`reject` first — a natural "we finished the job" flourish.

> [!warning] Every domain you own — including the ones that never send
> Parked and secondary domains are spoofed *because* they're unwatched. Lock each one with a null SPF (`v=spf1 -all`), no DKIM, and `_dmarc` → `v=DMARC1; p=reject;`. Takes five minutes per domain.

> [!tip] Authentication is the entry ticket, not the whole game
> Mail can pass all three and still spam-folder on **reputation** (shared-IP neighbors, volume spikes, blocklists) or **content/engagement**. Auth failures are fixed in DNS; reputation is fixed with time and sending hygiene → [[Email Deliverability Playbook]] reputation section.

Related: [[DNS]] · [[Common Ports]] · [[Email Deliverability Playbook]] · [[Email Authentication Cheat Sheet]] · [[Security Incident Basics]]
