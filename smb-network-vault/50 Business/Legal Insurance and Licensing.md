---
tags: [business, legal, ops]
aliases: [MSA, E&O Insurance, LLC, Licenses, Contracts]
---

# Legal, Insurance, and Licensing

"Licenses" splits into three categories — legal, insurance, and software/reseller. Sequenced against revenue, same principle as [[Tool Stack Upgrade Path]].

## Before the first contract (blocking)

- **LLC** in your state (~$50–500 one-time). With it: EIN (free, IRS website), business bank account, ACH/auto-pay capability.
- **Local/county business license** if your municipality requires one (check the county clerk; usually <$100).
- **MSA + per-client SOW templates** ready to sign — no work before signature ([[Client Onboarding Runbook]] step 1).
- Software: only the [[Launch Tool Stack|free stack]] + your own domain and M365 Business Basic (~$15/mo). Nothing else.

## MSA non-negotiables

- Limitation of liability **capped at fees paid in the prior 12 months**
- Not liable for data loss where the client **declined backup coverage — get declines in writing** (this clause saves businesses like this every year; see [[Risks and Failure Modes]])
- 30-day termination
- Payment by **ACH auto-pay, net-10 — auto-pay is mandatory** for retainers; you are not in the receivables-chasing business
- Annual 4–6% price increase written in from day one ([[Retainer Tiers]])

## At the first paying engagement — "not before, not later"

- **E&O (professional liability) + general liability**, ~$600–1,200/yr (Hiscox, Next, Coalition). Earlier burns runway; later means working uninsured — and clients' own insurers will ask for a certificate of insurance (COI). Marketing claims are part of E&O exposure → [[Fractional IT vs Full MSP]].

## After the first client(s), as revenue arrives (per-seat, pass-through-billed)

- **EDR** per endpoint — with the first retainer
- **SaaS backup** per user — with the first Secure-tier client
- **Bitwarden org** per client (~$4–6/org/mo) — as each client onboards
- **Microsoft CSP reseller status** — enroll in the Microsoft AI Cloud Partner Program (free) + a distributor like **Pax8**, around client #2–3. Adds recurring license margin and makes you the licensing point of contact, but isn't required to *administer* a tenant (delegated/GDAP admin works from day one) → [[M365 vs Google Workspace Strategy]]
- **PSA/RMM** around client #6–8; **phishing platform** at 2–3 Secure clients → [[Tool Stack Upgrade Path]]

## What you do NOT need (over-preparation trap)

- No state or federal license is required to operate an IT services business in the U.S. **Exception:** some states/counties require a **low-voltage contractor license** for cabling — subcontract cabling out rather than licensing up in year 1 ([[New Site Deployment]]).
- No certifications are legally required. MS-102 or Security+ are cheap credibility signals for *slack time* — clients are won by [[Outreach Playbook|outreach reps]] and the [[Security Snapshot]], not certs. Don't let studying substitute for the 40 weekly touches ([[Weekly Quotas and Metrics]]).
