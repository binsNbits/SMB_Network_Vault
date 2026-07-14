---
tags: [business, security, sales]
aliases: [Insurance Questionnaire, WISP, Cyber Insurance Application]
---

# Cyber Insurance Questionnaire

The **cyber-insurance application/renewal questionnaire** issued by the client's insurance **carrier** (Coalition, Chubb, Travelers, At-Bay, Corvus, Hiscox, Beazley, …). Not a standardized government form, and not legally "required" — but businesses must complete it to get or renew a cyber policy, and increasingly to keep general liability/professional policies. It is the single best **buying trigger** in the [[Ideal Client Profile]].

> [!note] Related but distinct: the WISP
> CPA firms **are** legally required to have a **WISP** (written information security plan) under the IRS/FTC Safeguards Rule — the ready-made wedge for [[Target Verticals|accounting clients]]. The Secure tier's annual security summary doubles as one ([[Retainer Tiers]]).

## How to get a copy

1. **From the client** — they or their office manager have the last application/renewal. Ask for it during the [[Security Snapshot]]; "what you told the insurer vs. reality" is part of the snapshot scope.
2. **From their insurance broker** — brokers keep carriers' specimen (blank) applications and share them freely. This is why brokers are a top referral-partner target ([[Outreach Playbook]]): you make their clients insurable; they hand you the questionnaires.
3. **Publicly** — most carriers publish specimen applications online (search *"[carrier] cyber application specimen PDF"*). Coalition, Corvus, and At-Bay are commonly circulated.

## The controls every carrier asks about

Every form differs, but they converge on the same ~12–15 controls — which *are* the [[Security Snapshot]] checklist: MFA everywhere, EDR, tested/immutable [[Backup Strategy|backups]], patching and EOL software, [[Email Authentication|SPF/DKIM/DMARC]] + filtering, security training, privileged-account hygiene, and an incident-response plan ([[Security Incident Basics]]).

Note that `p=reject` DMARC is what questionnaires actually want — the staged rollout is in [[Email Deliverability Playbook]].
