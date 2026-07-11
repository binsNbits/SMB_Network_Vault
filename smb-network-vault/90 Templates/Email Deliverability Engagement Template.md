---
tags: [template, email]
aliases: []
---

# Email Deliverability Engagement Template

> Copy per engagement. This is both the working document and the raw material for the client-facing before/after report. Procedure → [[Email Deliverability Playbook]] · concepts → [[Email Authentication]] · syntax → [[Email Authentication Cheat Sheet]]. On completion, fold the final records into the client's [[Site Note Template|site note]].

---

# Deliverability Fix: <CLIENT>

**Domain(s):** · **Mail platform:** M365 / Google Workspace / other
**DNS hosted at:** · **DNS access obtained (date/how):**
**Engagement dates:** start · target enforce-date

## Symptom & baseline (Phase 0 — capture *before* changing anything)

- Reported symptom: bounces / spam-foldering / spoofing reports — details:
- Sample NDR code + text:
- Authentication-Results of test message (paste): `spf=` `dkim=` `dmarc=`
- Blocklist status (MXToolbox, date):

**Records as found:**

| Record | Value found | Verdict |
|---|---|---|
| SPF (TXT @ apex) | | one record? lookup count? `all` qualifier? |
| DKIM (per selector) | | enabled? key size? |
| DMARC (`_dmarc`) | | present? policy? `rua`? |
| MX | | correct for platform? |

## Sender inventory (Phase 1 — DMARC reports will grade its completeness)

| # | Source | Sends via (platform / own IP / vendor) | SPF covered | DKIM signs as client domain | Aligned & verified (date) |
|---|---|---|---|---|---|
| 1 | Mail platform | | ☐ | ☐ | |
| 2 | Copier scan-to-email → [[Network Printer]] | | ☐ | ☐ | |
| 3 | Website forms (web host mailer!) | | ☐ | ☐ | |
| 4 | CRM / newsletter / e-sign / invoicing | | ☐ | ☐ | |
| 5 | Device/monitoring alerts → [[SNMP and Monitoring]] | | ☐ | ☐ | |

## Change log (every DNS edit, timestamped — the paper trail)

| Date | Record | Old → New | Why | TTL |
|---|---|---|---|---|
| | | | | |

## DMARC rollout schedule (Phase 4)

| Step | Planned date | Done | Notes / report findings |
|---|---|---|---|
| `p=none` + `rua` live | | ☐ | |
| Report review week 1 / 2 / 3 | | ☐ ☐ ☐ | senders discovered: |
| `p=quarantine` | | ☐ | |
| `p=reject` (+ `sp=reject`) | | ☐ | |
| SPF tightened to `-all`, TTLs restored | | ☐ | |

## Verification checklist (Phase 5)

- [ ] Gmail test: `spf=pass dkim=pass dmarc=pass`, aligned
- [ ] Outlook.com test: same
- [ ] Every inventory row test-fired and passing
- [ ] mail-tester / learndmarc score recorded:
- [ ] Blocklists clear / delisting requested:
- [ ] Original complainant scenario re-tested with the affected recipient
- [ ] Parked/secondary domains locked (null SPF + `p=reject`):

## Handoff (Phase 6)

- Report digest service + who reads it monthly:
- **New-sender rule communicated in writing (date):**
- DKIM rotation reminder set (≈ annual):
- Inbound anti-spoof rule + DMARC enforcement on:  ☐
- Outbound 25 blocked at [[Firewall (NGFW)|firewall]] except relay: ☐
- Final records documented in [[Site Note Template|site note]] → [[Documentation Standards]]
