---
tags: [business, ops, cloud, playbook]
aliases: [Kill the Closet Server, Cloud Migration Destinations]
---

# Server-to-Cloud Migration

The $3,000–$10,000 project ([[Project Pricing]]) that retires the "closet [[Server]]." **Default destination: whichever productivity cloud the client already pays for** — the destination follows the workload, not a blanket cloud choice ([[M365 vs Google Workspace Strategy]]).

| Workload on the closet server | M365 client → | Google Workspace client → |
|---|---|---|
| File shares | SharePoint Online / OneDrive (already in their license) | Google Shared Drives |
| On-prem Exchange email | Exchange Online | Gmail |
| Active Directory / domain login | Entra ID (+ Intune for device management) | Google cloud identity + endpoint management |
| Print server | Direct IP printing or Universal Print → [[Network Printer]] | Direct IP printing |
| QuickBooks / practice-management / LOB apps | **The vendor's own hosted/SaaS edition first** (QuickBooks Online, cloud PMS) | Same |
| LOB app with no SaaS edition | Small **Azure VM** as a last resort | Small Azure or AWS Lightsail VM as a last resort |

## Key principles

- **This is a SaaS migration, not an IaaS lift-and-shift.** The price band assumes moving data into services the client already licenses — not re-hosting a Windows server in Azure, which adds recurring compute cost and leaves you a pet server to babysit.
- Lift-and-shift to a VM is the escape hatch **only** when an LOB app has no cloud edition and can't be replaced. Price the ongoing VM cost into the retainer if you take it on ([[Retainer Tiers]]).
- For 5–30 employee offices, **killing the domain controller entirely** (Entra ID join + Intune) is almost always right — a cloud-hosted DC is the worst of both worlds.
- **Azure over AWS/GCP** when a VM is unavoidable for M365 clients: one vendor relationship, one admin portal family, and it aligns with the CSP licensing you'll resell ([[Legal Insurance and Licensing]]).

Backups for the migrated data land under [[Backup Strategy]] and the Secure tier's SaaS backup ([[Tool Stack Upgrade Path]]). What stays on-prem afterwards (NAS for local backup, network gear): [[NAS]], [[Backup Appliance]].
