---
tags: [business, ops, cloud]
aliases: [Platform Strategy, Migration Policy, M365 or Google]
---

# M365 vs Google Workspace Strategy

**Migration is not a requirement of the retainer.** The [[Ideal Client Profile]] deliberately targets businesses *already* on M365 or Google Workspace with nobody administering it; the retainer covers **administration** of whichever platform they have — users, licenses, groups, security defaults ([[Retainer Tiers]], Core).

Migration is a separate one-time **project** — $150/user, $2,000 minimum ([[Project Pricing]]) — for prospects who are:

- On legacy hosting: POP/IMAP from a web host or ISP, on-prem Exchange, or a "closet server" doing file/email duty → [[Server-to-Cloud Migration]]
- On consumer setups: personal Gmail/Outlook.com used for business
- Splitting the difference badly: domain email forwarding to personal Gmail, etc.

## Default recommendation for new migrations

Support both platforms (you'll inherit both), but default to **Microsoft 365** for:

- **Law, accounting, medical/dental** ([[Target Verticals]]) — their line-of-business software integrates with Outlook/Office desktop apps, and [[Cyber Insurance Questionnaire|insurance questionnaires]] map cleanly to Entra ID conditional access, Defender for Business, and Intune.

**Google Workspace stays right** for:

- **Nonprofits** — generous free/discounted nonprofit licensing is a day-one money-saver
- Younger, browser-native businesses

> [!warning] Never force-migrate a happy Google Workspace client to M365
> The "same stack for every client" rule ([[Launch Tool Stack]]) refers to **your tooling layer** — RMM, EDR, backup, password manager, documentation — which sits on top of either suite. That's where standardization pays; the productivity suite can vary.

Reselling M365 licenses (CSP via Pax8) is worth setting up around client #2–3 → [[Legal Insurance and Licensing]]. Tenant-level email authentication setup for either platform: [[Email Deliverability Playbook]] Phase 3.
