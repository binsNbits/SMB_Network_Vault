---
tags: [business, ops, playbook]
aliases: [Onboarding, First 30 Days]
---

# Client Onboarding Runbook

The first 30 days of every retainer. Same steps, every client — standardization is the margin ([[Solo Operations and Boundaries]]).

1. **Signed MSA + SOW + auto-pay on file — *before* work begins** ([[Legal Insurance and Licensing]]).
2. **Deploy the RMM agent** to every machine; inventory hardware/software/licenses ([[Launch Tool Stack]]).
3. **Take over admin credentials**; document in the vault per [[Documentation Standards]]; remove ex-employee and vendor stragglers — there are always some.
4. **Enforce MFA** on email/admin accounts; kill legacy auth ([[M365 vs Google Workspace Strategy]]).
5. **Verify or deploy backups; run one test restore and email the result** — this single email cements the relationship ([[Backup Strategy]]).
6. **Set up their ticket address** (e.g., help@yourcompany) and tell every employee: "email this or call this number, day or night, that's it."
7. **30-day report:** what you found, what you fixed, what you recommend next — this is the [[Project Pricing|project]]-upsell pipeline.

Network documentation captured during onboarding (IPs, [[VLANs]], credentials, ISP account info) goes in the per-client runbook — use the [[Site Note Template]]. Scope of ongoing network work: [[Retainer Scope vs Project Scope]].
