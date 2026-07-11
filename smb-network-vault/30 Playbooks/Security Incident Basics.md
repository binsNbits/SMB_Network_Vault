---
tags: [playbook, security]
aliases: [Incident Response, Compromise Response, Ransomware Response]
---

# Security Incident Basics

First-response for suspected compromise — malware alert, ransomware note, "weird traffic," phished credentials. **Goal at SMB scale: contain fast, preserve evidence, escalate honestly.** Most SMBs should retain an IR firm / cyber-insurance hotline for anything beyond a single cleaned endpoint — know that number *before* you need it → [[Site Note Template]].

## Guiding rules

1. **Isolate ≠ power off.** Disconnect from the network (kill the switch port / Wi-Fi) but leave the machine running — RAM holds evidence, and ransomware mid-encryption sometimes recovers keys from memory. Powering off destroys both.
2. **Contain before you investigate.** Lateral movement is measured in minutes on flat networks — [[Network Segmentation]] bought you time; spend it wisely.
3. **Assume credentials are burned.** Anything the compromised context could see gets rotated.
4. **Write everything down with timestamps** as you act — insurance, law enforcement, and the postmortem all need it.

## Immediate containment (first hour)

1. **Isolate affected machines:** `shutdown` on the switch port ([[CLI Quick Reference]]) or block the client in the [[Wireless LAN Controller|controller]] — faster and cleaner than chasing cables.
2. **Protect the backups NOW:** verify the [[Backup Appliance]] is unreached (its isolation design → its note), snapshot states, disconnect replication *into* it if attacker scope is unknown → [[Backup Strategy]].
3. Scope quickly: [[Firewall (NGFW)]] logs — what did the affected host talk to (internal spread? exfil destinations?); [[DHCP]] lease + [[Switching and ARP|MAC tables]] to enumerate what/where it is.
4. Widening blast radius → **cut wider**: shut the affected [[VLANs|VLAN]]'s SVI, or in a ransomware-active scenario, the WAN — revenue interruption beats total encryption.
5. Rotate in order: domain admin creds → the affected users' → service accounts the host used → [[VPN Fundamentals|VPN]] keys/[[Vendor Default IPs and Logins|device admins]] if management-plane exposure is possible.

## Evidence & escalation

- Preserve: firewall/syslog exports ([[SNMP and Monitoring]] server has centralized copies — this is a big reason it exists), affected-host disk images if capability exists, your timeline notes.
- Escalate when: ransomware beyond one machine, any server/AD compromise, data exfiltration signs, or you're guessing — the insurance hotline/IR retainer call is cheap; a botched self-remediation isn't.
- Legal/notification duties may exist (jurisdiction/industry) — management + counsel decide, but flag it early.

## Recovery (after containment, typically with IR guidance)

1. Rebuild — don't "clean" — compromised systems: reimage endpoints ([[Desktops and Laptops]]), restore servers from **pre-infection immutable points** → [[Backup Appliance]] restore scenarios.
2. Close the entry point *before* reconnecting: the unpatched [[VPN Appliance|VPN portal]], the exposed RDP, the phished account without MFA — else you're rehearsing.
3. Staged reconnection: restored systems into a quarantine VLAN, watch their traffic, then release.
4. Postmortem → convert every lesson into a change: rules, [[Firmware Update Procedure|patching]], segmentation, monitoring alerts → [[Documentation Standards]] change log.

## Signals worth investigating *before* they're incidents

Odd egress in deny-logs ([[IoT Devices]] posture pays off here) · impossible-travel logins · a host suddenly uploading at 3 AM ([[Slow Network Playbook]] finding) · new admin accounts nobody made · [[Backup Strategy|backup job]] size anomalies · antivirus mass-disabling. Cheap monthly habit: skim these → [[SNMP and Monitoring]].
