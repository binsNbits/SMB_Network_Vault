---
tags: [device, storage]
aliases: [Storage Area Network, iSCSI, Block Storage]
---

# SAN

**Role:** **block-level** shared storage — the storage looks like a raw disk (LUN) to the servers that mount it, unlike a [[NAS]] which serves *files*. Exists in SMBs almost exclusively as **shared datastores for virtualization clusters**: two+ hypervisor hosts ([[Server]]) mounting the same LUNs so VMs can migrate/fail over → [[High Availability]].

## Honest scoping for SMBs

| Situation | Answer |
|---|---|
| Single hypervisor host | Local SSDs/RAID — no SAN needed |
| 2–3 hosts, want vMotion/HA | **iSCSI SAN** (or NAS with iSCSI target — Synology/TrueNAS do this respectably) |
| Modern alternative | Hyperconverged (vSAN/Ceph/StorageSpacesDirect): hosts' local disks pooled — no separate SAN box |
| Fibre Channel? | Enterprise territory: dedicated FC switches/HBAs — rarely justified at SMB scale; **iSCSI over Ethernet is the SMB SAN** |

## iSCSI vocabulary

| Term | Meaning |
|---|---|
| Target | The storage side (SAN/NAS exporting LUNs) |
| Initiator | The server side mounting them |
| LUN | A block device ("virtual disk") exported by the target |
| IQN | iSCSI name identifying each party (`iqn.2026-07.local.corp:esxi01`) |
| Multipath (MPIO) | Two+ network paths to the same LUN — redundancy AND the way iSCSI does bandwidth |

## Network design — where SANs are won or lost

1. **Dedicated storage network:** separate [[VLANs|VLAN]] minimum, ideally separate switches/NICs. *Why: storage traffic is latency-sensitive and voluminous; a busy LAN or a spanning-tree event mid-write = VM corruption risk.*
2. **No routing on the storage path** — initiators and targets in the same subnet ([[Subnetting and CIDR]]); nothing else lives there → [[Network Segmentation]].
3. **Two independent paths:** NIC-A → switch-A → controller-A, NIC-B → switch-B → controller-B; MPIO round-robins. Never LACP for iSCSI — MPIO is the correct multipathing.
4. Jumbo frames (MTU 9000) **end-to-end or not at all** — mixed MTU on the path causes silent, maddening failures. Verify: `ping -f -l 8972 <target>` (Windows) / `ping -M do -s 8972` (Linux).
5. 10 GbE minimum for the storage fabric in any new build.

## Configuration — step-by-step (generic iSCSI)

1. Target side: create LUN(s); restrict access by initiator IQN + storage-subnet IP; enable CHAP (mutual if supported). *Why: an open LUN is a raw disk anyone on that subnet can mount and corrupt.*
2. Initiator side (ESXi/Proxmox/Hyper-V): configure the storage NICs, add target portal IPs, discover, log in both paths.
3. Enable MPIO / round-robin path policy; verify both paths active.
4. Format with the **cluster filesystem** (VMFS/CSV) — never a normal filesystem mounted by two hosts (instant corruption).
5. Test failure modes *before production*: pull one path cable, reboot one storage controller → I/O must continue → [[High Availability]].

## Troubleshooting

| Symptom | Check |
|---|---|
| LUN not visible | IQN allowlist, CHAP creds, VLAN/subnet reachability (ping the portal) |
| Slow/latency spikes | Path flapping (`esxcli storage nmp path list`), MTU mismatch (ping test above), shared-switch congestion |
| All-paths-down events | Storage switch/controller reboot without redundancy — the design failed; fix the topology |
| Random VM corruption | Two hosts, non-cluster filesystem — restore from [[Backup Strategy|backup]], fix the filesystem choice |

Ops: firmware carefully & staged (storage firmware bugs eat data — vendor HCL compliance matters here more than anywhere → [[Firmware Update Procedure]]) · capacity + path + latency monitoring ([[SNMP and Monitoring]]) · SAN and its switches on [[UPS]] with the longest runtime in the building — storage must outlive the hosts during shutdown.

Related: [[NAS]] · [[Server]] · [[High Availability]] · [[Managed Switch]]
