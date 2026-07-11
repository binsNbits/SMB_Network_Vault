---
tags: [playbook, troubleshooting, security, email]
aliases: [Email Going to Spam, Deliverability Fix, SPF DKIM DMARC Rollout, Mail Delivery Problems]
---

# Email Deliverability Playbook

For **"our emails go to spam," "customers aren't getting our invoices," bounced mail, or reports of spoofed email "from us."** Concepts (read first if SPF/DKIM/DMARC or *alignment* is fuzzy): [[Email Authentication]] · syntax/commands: [[Email Authentication Cheat Sheet]] · record findings per client in the [[Email Deliverability Engagement Template]]. Runs fine as a fixed-scope engagement: Phases 0–3 in week one, Phase 4 is calendar time, Phase 5–6 close it out.

## The three symptom classes — identify yours first

| Symptom | Meaning | Urgency |
|---|---|---|
| Hard bounces (NDRs) | Receivers are **rejecting** — usually auth or blocklist | Now — mail is being lost |
| Silent spam-foldering | Auth missing/misaligned or reputation damage | Days — trust erosion |
| "You sent me a weird invoice" (spoofing) | No enforced DMARC — criminals send as the domain | Now — and check it isn't real compromise → [[Security Incident Basics]] |

## Phase 0 — Triage (30 minutes, before touching anything)

1. **Read a bounce verbatim.** The NDR quotes the receiving server's reason: `550 5.7.26` (Gmail: unauthenticated), `5.7.1` (policy/blocklist), `550 SPF fail` — the code table is in [[Email Authentication Cheat Sheet]]. No bounce = spam-foldering, not rejection.
2. **Capture a live specimen:** have the client email a Gmail account you control → *Show original* → the **Authentication-Results** header gives `spf=`, `dkim=`, `dmarc=` verdicts at a glance. This one header is 80% of the diagnosis.
3. **Pull current DNS state** (commands in the cheat sheet): SPF TXT, DKIM selectors for their platform, `_dmarc` TXT, MX. Screenshot/save — this is your *before* for the report.
4. **Classify:** auth failing → Phases 1–5. Auth *passing* but still spam-foldering → jump to Reputation, below. Wrong/missing MX → that's an inbound outage, fix it first → [[DNS]].

## Phase 1 — Sender inventory (the step everyone skips, and the reason fixes don't stick)

List **everything that sends mail as the domain** — the record you build in Phase 2 must cover all of it, and DMARC enforcement in Phase 4 will punish anything you missed:

- The mail platform itself (M365 / Google Workspace)
- Copier/scanner **scan-to-email** → [[Network Printer]]
- Website contact/order forms (often the *web host's* PHP mailer — a different server entirely)
- CRM, newsletter, e-sign, invoicing/accounting SaaS (Mailchimp, QuickBooks, DocuSign…)
- Monitoring/alert emails from the [[Firewall (NGFW)]], NAS, UPS → [[SNMP and Monitoring]]
- Any on-prem app or script relaying through a [[Server]]

Ask the office manager, then **trust the DMARC reports over the interview** — Phase 4's `p=none` reports reveal every sender they forgot. Also establish **who controls DNS** (registrar? web host? Cloudflare?) and get access *now* — DNS access is the schedule bottleneck of the whole job. Record all of it in the [[Email Deliverability Engagement Template]].

## Phase 2 — SPF

1. Drop the record's TTL to 300 first (cheap rollback), restore to 3600 when verified.
2. Build one record from the inventory — vendor `include:`s plus any direct-sending IPs:
   ```text
   v=spf1 include:spf.protection.outlook.com include:sendgrid.net ip4:203.0.113.25 ~all
   ```
3. Rules that break people: **exactly one** `v=spf1` record (a second = `permerror`); **≤ 10 DNS lookups counting nested includes** (validate — cheat sheet tools); every mechanism must earn its place (delete stale vendor includes — each is both a lookup spent and a standing authorization).
4. Ship with `~all` during the verification window, tighten to `-all` in Phase 5. Never leave `?all`/`+all` — they authorize the planet.

## Phase 3 — DKIM (per sending service — one selector each)

- **Microsoft 365:** Defender portal → Email authentication settings → DKIM → the portal shows two exact CNAMEs (`selector1._domainkey` / `selector2._domainkey` → the tenant's `onmicrosoft.com` targets). Publish both, wait for propagation, then **Enable** — enabling before DNS resolves throws an error.
- **Google Workspace:** Admin console → Apps → Gmail → *Authenticate email* → generate a **2048-bit** key (selector `google`) → publish the TXT → **Start authentication** only after DNS is live.
- **Third-party senders:** each provides its own CNAMEs/TXT in its domain-authentication settings. Verify the result signs with **`d=<client-domain>`** — a vendor signing with its *own* domain passes DKIM but can never DMARC-align for the client → [[Email Authentication]] alignment.
- **Devices that can't sign** (copiers, appliances): route them through the platform's authenticated SMTP submission ([[Common Ports|587]] with a mailbox/app credential, or Google's SMTP relay) so the *platform* signs — direct-send leaves them SPF-only, which dies the day the copier's mail gets forwarded.

## Phase 4 — DMARC rollout (deliberate; this phase is calendar time, not effort)

> [!warning] Never jump straight to `p=reject`
> Enforcement punishes every legitimate sender you missed in Phase 1 — payroll notifications silently spam-foldering is how a deliverability fix becomes an outage. The reports exist precisely to find those senders first.

| Step | Record state | Exit criterion |
|---|---|---|
| Week 0 | `v=DMARC1; p=none; rua=mailto:<report-addr>;` | Reports arriving |
| Weeks 1–3 | Still `p=none` — read aggregate reports weekly | Every legitimate source passes **aligned** SPF or DKIM (fix stragglers via Phases 2–3) |
| Enforce ① | `p=quarantine;` (optionally ramp `pct=25→100`, though not all receivers honor `pct`) | 1–2 clean weeks, no legit-mail complaints |
| Enforce ② | `p=reject; sp=reject;` | Steady state — the engagement's finish line |

Raw aggregate reports are XML — use a free-tier report digester (Postmark DMARC, dmarcian free tier, etc.) rather than reading them by hand. If `rua=` points at a domain other than the client's, the receiving domain must publish the external-authorization record (cheat sheet) — hosted digesters handle this.

## Phase 5 — Verify (test like a customer, not like an admin)

1. Send from the client's mailbox to Gmail **and** Outlook.com you control → Authentication-Results must show `spf=pass`, `dkim=pass`, `dmarc=pass` **with aligned domains**.
2. Trigger a send from **every inventoried source** — the copier, the website form, the CRM — not just the mail platform. Each must pass at least one aligned check.
3. Independent scoring: mail-tester.com / learndmarc.com (cheat sheet has the list).
4. Re-check any blocklists found in Phase 0 (MXToolbox) and request delisting — auth fixes stop *new* damage; listings need explicit removal.
5. Tighten SPF to `-all`, restore TTLs, confirm the original bouncing/spam scenario with the actual affected recipient.

## Phase 6 — Aftercare & handoff

- Record final records + *why each mechanism exists* in the [[Email Deliverability Engagement Template]] and the client's [[Site Note Template]] → [[Documentation Standards]].
- **New-sender rule** (tell the client, in writing): any new SaaS tool that sends email needs its include/DKIM added *before* go-live — this is the #1 way the fix decays.
- Calendar: monthly DMARC-report skim · re-verify after any DNS-host or mail-platform migration · rotate DKIM keys ~annually (publish new selector, switch, retire old).
- Inbound hardening while you're in the tenant: ensure the platform *enforces* DMARC on incoming mail and add an anti-spoof rule for the client's own domain — the same engagement protects them from receiving spoofs, not just sending reputation.
- Egress hygiene: block outbound port 25 from the LAN except the sanctioned relay ([[Firewall Concepts]]) — one malware-infected PC spamming directly will torch the IP reputation you just repaired.
- Lock every parked/secondary domain (null SPF + `p=reject`) → [[Email Authentication]] warning box.

## When auth passes but spam-foldering continues → reputation, not records

- **Blocklists:** check the sending IPs *and* domain; delist with cause fixed (open relay? infected host? → [[Security Incident Basics]]).
- **Shared-IP platforms** (M365/Google): reputation is mostly domain-level — fix engagement (stale lists, spam-trap addresses, no unsubscribe) and volume spikes.
- **Bulk senders (≥ ~5k/day to Gmail):** Google/Yahoo additionally require one-click unsubscribe and spam-complaint rate held under 0.3% — enroll in Google Postmaster Tools / Microsoft SNDS for the receiver's-eye view.
- **Content** matters least of the three, but link-shorteners, attachment-only mails, and all-image bodies still score badly.

Related: [[Email Authentication]] · [[Email Authentication Cheat Sheet]] · [[Email Deliverability Engagement Template]] · [[DNS]] · [[Security Incident Basics]]
