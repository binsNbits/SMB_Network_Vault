---
tags: [concept, operations, security]
aliases: [3-2-1 Backup, Backups, RPO, RTO, Immutable Backup]
---

# Backup Strategy

Backups are the last line against ransomware, fat-fingers, theft, fire, and dead disks. Ransomware crews **hunt backups first** — strategy matters more than software.

## The 3-2-1 rule (minimum standard)

- **3** copies of data (production + 2 backups)
- **2** different media/systems ([[NAS]] + cloud, or [[Backup Appliance]] + tape/disk rotation)
- **1** copy **off-site** (cloud, or rotated drive that physically leaves)

Modern extension **3-2-1-1-0**: +1 copy *offline or immutable* (object-lock cloud storage, or a NAS snapshot the backup credentials cannot delete), and **0** errors on the last **test restore**.

## Two numbers that define the design

| Metric | Question | Drives |
|---|---|---|
| **RPO** (recovery point) | How much work can we lose? | Backup frequency: nightly = up to 24 h lost; hourly snapshots shrink it |
| **RTO** (recovery time) | How long can restore take? | Method: bare-metal image & VM replicas restore in minutes; cloud-only full restores can take days over SMB bandwidth |

## What to back up (people forget rows 3–5)

1. [[Server]] data + system state (AD!) — image-based
2. [[NAS]] shares — snapshots + replication off-box
3. **Device configurations** — firewall, switches, APs → [[Configuration Backup Procedure]]
4. SaaS data — Microsoft 365/Google Workspace are NOT backed up by Microsoft/Google beyond short retention; use a SaaS backup service
5. Application-level exports — databases, [[VoIP Phones and PBX|PBX]] config, [[IP Cameras and NVR|NVR]] settings

## Anti-ransomware backup rules

- **Separate credentials**: backup repository accounts exist nowhere in Active Directory. Domain admin compromise must not reach backups.
- **Pull, don't push** where possible: the backup server reaches into clients; clients hold no credentials to the repository.
- **Immutability/snapshots**: BTRFS/ZFS snapshots on the NAS, object-lock (S3/B2/Wasabi) in cloud — encrypting the share doesn't touch snapshots.
- **Isolate the backup VLAN** → [[Network Segmentation]]; only the backup system initiates connections.
- **Alert on job failure AND on job *success anomalies*** (job suddenly 10× smaller = something's wrong) → [[SNMP and Monitoring]].

## Test restores — the step everyone skips

Quarterly minimum: restore a random file, a whole VM, and (annually) a bare-metal server to spare hardware. Time it against RTO. **An untested backup is a hope, not a plan.**

Related: [[Backup Appliance]] · [[NAS]] · [[Security Incident Basics]] · [[High Availability]]
