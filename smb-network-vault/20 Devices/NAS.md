---
tags: [device, storage]
aliases: [Network Attached Storage, Synology, QNAP, File Share]
---

# NAS

**Role:** file-level shared storage over the network — team shares (SMB/AFP/NFS), [[Backup Strategy|backup]] target, snapshots, and (modern units) a light app platform (Docker, camera [[IP Cameras and NVR|surveillance]], sync clients). For many SMBs the NAS *is* the file server.

**Position:** SERVERS [[VLANs|VLAN]], static IP/reservation, LACP if >1 NIC. **Contrast:** [[SAN]] = block storage for hypervisors; a NAS serves *files*.

**Typical platforms:** Synology (DSM), QNAP (QTS), TrueNAS (ZFS, self-built).

## Deployment — step-by-step (Synology-flavored; concepts portable)

1. **Disks & RAID before data:** NAS-rated drives (WD Red Plus/IronWolf); RAID choice:
   - 2 bays → **RAID 1** (mirror)
   - 4+ bays → **RAID 5/SHR** (one-disk tolerance) or **RAID 6/SHR-2** (two-disk — right choice for >8 TB drives, since rebuilds take days and second failures during rebuild are the classic data-loss story)
   - **RAID ≠ backup** → the NAS itself must be backed up → [[Backup Strategy]]
2. Find it: `find.synology.com` / Qfinder → [[Vendor Default IPs and Logins]]; create admin (non-default name), **enable MFA**.
3. Network: static IP or [[DHCP]] reservation; bond NICs (LACP — switch side too → [[Managed Switch]]); DSM UI on HTTPS :5001 only.
4. Storage pool → volume (BTRFS where offered — snapshots need it) → **shared folders** per team/function.
5. **Permissions the maintainable way:** groups, not users → `GRP-Accounting` gets `Finance` share; users join groups. AD-join if you run a domain ([[Server]]) so one identity system rules → why: leaver-offboarding touches one place.
6. Enable **snapshots** (hourly/daily schedule + retention): user-restorable previous versions and the ransomware undo button.
7. Backup jobs off-box: Hyper Backup → cloud (immutable bucket) + optionally a second NAS off-site → 3-2-1 → [[Backup Strategy]].
8. Harden: disable SMBv1, default `admin`/`guest` accounts off, auto-block on failed logins, firewall app = allow LAN subnets only, **no QuickConnect/port-forward exposure** — remote access via [[VPN Fundamentals|VPN]].

## Access methods

| Surface | Where |
|---|---|
| Web UI | `https://<ip>:5001` (DSM) — everything lives here |
| SSH | Enable only when needed (Control Panel → Terminal), disable after — reduce surface |
| Shares | `\\nas01\finance` (Windows), `smb://nas01` (macOS) → TCP 445 ([[Common Ports]]) |
| Mobile/sync apps | Via VPN, not exposed ports |

## Conditionals

- **NAS or file-server VM?** NAS: simpler, appliance-patched, great snapshots. Server: needed when apps demand Windows ACL edge cases or run *on* the box.
- **10 GbE worth it?** Only if clients+switch have it and workloads are large-file (video/CAD). Office docs saturate disks before 1 GbE.
- **Running the NVR on the NAS?** Works (Surveillance Station) — mind camera licenses and disk contention → [[IP Cameras and NVR]].
- **Hosting the [[Wireless LAN Controller]] in Docker on it?** Common and fine — back up that container's volume too.

## Troubleshooting

| Symptom | Check |
|---|---|
| Share unreachable | Ping NAS → service up? → client credentials/SMB version (SMB1-only ancient clients vs hardened NAS) |
| Slow transfers | NIC negotiation (1 Gb ≈ 110 MB/s ceiling), bond misconfig, RAID rebuilding in background, disk health |
| Beeping / degraded volume | **Failed disk — replace promptly, same/larger size; rebuild before a second fails**; this is why RAID 6 for big arrays |
| "Volume crashed" | Stop. Don't reinitialize. Vendor support / recovery path — and this is the moment backups justify themselves |
| Users see files they shouldn't | Group/ACL audit — permissions drift; review quarterly |

Ops: DSM/QTS updates promptly (NAS ransomware campaigns target internet-visible, unpatched units — see hardening above) · disk health (SMART) + capacity alerts → [[SNMP and Monitoring]] · on the [[UPS]] with graceful-shutdown (USB/network UPS integration) · test-restore quarterly.

Related: [[Backup Strategy]] · [[Backup Appliance]] · [[SAN]] · [[Server]]
