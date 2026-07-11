---
tags: [device, storage, security]
aliases: [Backup Server, Backup Target, BDR Appliance]
---

# Backup Appliance

**Role:** dedicated hardware/VM whose only job is running [[Backup Strategy]]: pulling images of [[Server|servers]], syncing [[NAS]] data, replicating off-site, and enabling fast restores (including booting failed servers *as VMs on itself* — the "BDR" pitch of Datto/Axcient-style units).

**Forms:** turnkey BDR appliance (vendor-managed, subscription) · self-built (Veeam/Synology Active Backup on dedicated hardware) · a hardened NAS dedicated to backup only.

## The prime directive: isolation

This box is the ransomware last line — design it to survive a *full domain compromise* ([[Security Incident Basics]]):

1. **Own [[VLANs|VLAN]]**, firewall allowing only: appliance → protected hosts (agent/hypervisor ports) and appliance → cloud replication. **Nothing initiates connections *to* it** except admin from MGMT → [[Network Segmentation]], [[Firewall Concepts]].
2. **No domain join. Unique local credentials** existing nowhere else. *Why: attackers with AD admin must not inherit backup access.*
3. **Pull architecture:** appliance reaches into hosts; hosts hold no repository credentials.
4. **Immutable tier:** local snapshots + object-locked cloud copy — even the appliance's own admin can't purge inside the lock window.
5. MFA on the console; alert on any retention-policy change.

## Deployment — step-by-step

1. Size: total protected data × retention ≈ 2–3× primary capacity minimum; RAID 6 for the repository.
2. Network per isolation rules above; static IP; document → [[Documentation Standards]].
3. Add protection sources:
   - Hypervisor-level (agentless, via ESXi/Hyper-V/Proxmox APIs) — preferred for VMs
   - Agents on physical boxes
   - SMB/NFS pulls for [[NAS]] shares
4. **Schedule around business + bandwidth:** nightly fulls/incrementals locally; cloud replication overnight — seed the first copy locally/by disk if WAN is thin. Set RPO per system: finance DB hourly, file shares nightly → [[Backup Strategy]].
5. Retention: e.g., hourly×24, daily×14, weekly×8, monthly×12 — balancing restore granularity vs. capacity.
6. Alerts: job failure AND anomalies (job size collapse/spike = deletion or encryption upstream) → [[SNMP and Monitoring]].
7. **Verification automation:** nightly boot-check screenshots (appliance boots each image, screenshots the login screen) where supported; quarterly manual restore drills regardless.

## Restore scenarios this box must handle (test each)

| Scenario | Method | Target time |
|---|---|---|
| One deleted file | Granular restore from image | Minutes |
| Dead server hardware | **Instant virtualization on the appliance**, migrate later | < 1 hour |
| Ransomware, whole environment | Isolate ([[Security Incident Basics]]) → restore from pre-infection immutable points | Hours–day |
| Building loss | Cloud copies → cloud virtualization / new hardware | Per DR plan |

## Troubleshooting

| Symptom | Check |
|---|---|
| Job fails: unreachable | Firewall rules changed? Host agent service? VSS errors on Windows source (`vssadmin list writers`) |
| Jobs slow / overrun window | Incremental chain too long (synthetic fulls), repository fragmentation, uplink saturation → schedule/QoS → [[QoS]] |
| Cloud replication lagging | Bandwidth math: changed-data/day vs upload speed — reseed or upgrade WAN |
| Restore test fails | THE alarm that matters — treat as P1 until a clean restore passes |

Ops: patch the appliance itself promptly (backup software CVEs are actively hunted) · capacity forecasting monthly · on the [[UPS]] · keep the vendor's DR runbook printed — you'll need it when everything else is down → [[Site Note Template]].

Related: [[Backup Strategy]] · [[NAS]] · [[Server]] · [[Security Incident Basics]]
