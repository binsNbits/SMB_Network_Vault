---
tags: [reference, security, email]
aliases: [SPF Syntax, DMARC Tags, DKIM Selectors, Email Record Lookup]
---

# Email Authentication Cheat Sheet

Fast lookup for the records and commands behind [[Email Authentication]], used throughout the [[Email Deliverability Playbook]]. Diagnostics run from any OS → [[CLI Quick Reference]].

## Look up the records

| Record | dig | nslookup (Windows) |
|---|---|---|
| SPF | `dig TXT company.com +short` | `nslookup -type=TXT company.com` |
| DKIM | `dig TXT selector1._domainkey.company.com +short` | `nslookup -type=TXT selector1._domainkey.company.com` |
| DMARC | `dig TXT _dmarc.company.com +short` | `nslookup -type=TXT _dmarc.company.com` |
| MX | `dig MX company.com +short` | `nslookup -type=MX company.com` |
| PTR (rDNS) | `dig -x 203.0.113.25 +short` | `nslookup 203.0.113.25` |

DKIM is often a CNAME to the vendor — `dig` follows it; an empty answer usually means you guessed the wrong **selector** (table below).

**Read a received message's verdicts:** Gmail → ⋮ → *Show original* · Outlook → open message → ⋯ → View source, find the header:
```text
Authentication-Results: mx.google.com;
  spf=pass  smtp.mailfrom=company.com;      ← envelope domain SPF checked
  dkim=pass header.d=company.com;           ← signing domain (must align with From)
  dmarc=pass header.from=company.com        ← the visible From domain
```

## SPF syntax

```text
v=spf1 include:spf.protection.outlook.com ip4:203.0.113.25 -all
```

| Mechanism | Authorizes | Lookup cost |
|---|---|---|
| `ip4:` / `ip6:` | Literal IP/CIDR | 0 — prefer when possible |
| `include:` | Another domain's SPF (vendors) | 1 + whatever it nests |
| `a` / `mx` | The domain's A/MX hosts | 1 each |
| `redirect=` | Replaces the whole record with another domain's | 1 |

| Qualifier on `all` | Verdict for unlisted senders | Use |
|---|---|---|
| `-all` | fail | The goal |
| `~all` | softfail | Transition/verification window only |
| `?all` / `+all` | neutral / pass | Never — authorizes anyone |

**Hard limits:** one `v=spf1` record per domain · **≤ 10 DNS lookups total** (nested includes count) · 255 chars per quoted string (split longer records into multiple strings). Violations = `permerror` — treated as no SPF at all.

## DMARC tags

```text
v=DMARC1; p=quarantine; sp=reject; rua=mailto:dmarc@company.com; adkim=r; aspf=r
```

| Tag | Meaning | Values / default |
|---|---|---|
| `p=` | Policy for the domain | `none` → `quarantine` → `reject` (rollout order → [[Email Deliverability Playbook]]) |
| `sp=` | Policy for **subdomains** | Defaults to `p=` — set explicitly when locking down |
| `rua=` | Aggregate (daily XML) report destination | `mailto:` — use a digest service, not a human mailbox |
| `ruf=` | Per-failure forensic reports | Rarely honored; usually omit |
| `adkim=` / `aspf=` | Alignment strictness | `r` relaxed (org domain, default) / `s` strict (exact) |
| `pct=` | % of failing mail policy applies to | Ramp tool; not honored by all receivers |
| `fo=` | Failure-report trigger detail | Only matters with `ruf=` |

**External `rua=` domain?** The *report-receiving* domain must publish `company.com._report._dmarc.reportservice.com TXT "v=DMARC1"` — hosted digesters do this for you.

## Common provider values (verify against vendor docs — these change)

| Sender | SPF include | DKIM location / selectors |
|---|---|---|
| Microsoft 365 | `include:spf.protection.outlook.com` | 2 CNAMEs `selector1`/`selector2._domainkey` → tenant `onmicrosoft.com`; enable in Defender portal |
| Google Workspace | `include:_spf.google.com` | TXT `google._domainkey` (2048-bit; generate in Admin console → Gmail → Authenticate email) |
| SendGrid | `include:sendgrid.net` | CNAMEs `s1`/`s2._domainkey` (domain authentication) |
| Mailchimp | (CNAME-based; no include needed for DMARC) | CNAMEs `k2`/`k3._domainkey` (`k1` legacy) |
| Zoho | `include:zohomail.com` | TXT, selector shown at setup (often `zmail`) |
| Web-host PHP mail | The host's own include — or better, re-route via SMTP 587 | Usually can't sign → re-route → [[Email Deliverability Playbook]] Phase 3 |

## Non-sending / parked domain lockdown (per domain, 5 minutes)

```text
company-parked.com    TXT  "v=spf1 -all"
_dmarc.company-parked.com  TXT  "v=DMARC1; p=reject;"
```

## Rejection codes you'll meet in bounces

| Code | Typical meaning |
|---|---|
| `550 5.7.26` | Gmail: sender unauthenticated (no aligned SPF/DKIM) |
| `550 5.7.1` | Policy rejection — DMARC reject, blocklist, or content filter (read the text) |
| `554 / 5.7.509+` | M365: DMARC/reputation rejection variants |
| `421 4.7.x` | Temporary throttling — reputation/volume, retry happens automatically |

## Testing & monitoring tools

| Tool | For |
|---|---|
| MXToolbox (SuperTool + blocklist check) | Record lookups, SPF validation incl. lookup count, blocklists |
| learndmarc.com | Send a test mail → live SPF/DKIM/DMARC + alignment walkthrough |
| mail-tester.com | One-shot spam score incl. content factors |
| Google Admin Toolbox *Messageheader* | Paste headers → parsed Authentication-Results |
| Google Postmaster Tools / Microsoft SNDS | Ongoing domain/IP reputation as the big receivers see it |
| Postmark DMARC / dmarcian (free tiers) | Weekly human-readable DMARC aggregate digests |

Related: [[Email Authentication]] · [[Email Deliverability Playbook]] · [[DNS]] · [[Common Ports]] · [[CLI Quick Reference]]
