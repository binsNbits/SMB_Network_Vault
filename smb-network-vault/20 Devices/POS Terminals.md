---
tags: [device, endpoint, compliance]
aliases: [Point of Sale, Payment Terminals, Card Readers, PCI]
---

# POS Terminals

**Role:** transaction + payment processing — registers/tablets running POS software, and **payment terminals** (card readers) that talk to a payment processor. The one device class with a **compliance regime attached: PCI DSS.**

**Position:** dedicated POS [[VLANs|VLAN]], the most tightly firewalled segment after the [[Backup Appliance]] → [[Network Segmentation]].

## PCI DSS in one paragraph (SMB view)

If card data touches your network, PCI DSS applies. The SMB-practical strategy is **scope reduction**: use P2PE-validated payment terminals (card data is encrypted *in the reader* and tunnels opaquely to the processor) so your network never sees cleartext card data — then segmentation keeps the rest of the network out of scope, and your annual obligation is typically a self-assessment questionnaire (SAQ) + quarterly scans per your processor's program. **Never** let the POS design drift into card numbers traversing your LAN in the clear.

## Network design (the segmentation IS the compliance)

| Flow | Rule |
|---|---|
| Payment terminals → processor endpoints (TLS 443 / vendor-published hosts) | Allow — ideally the *only* thing they can reach |
| POS registers → POS backend (local server or cloud) | Allow explicitly |
| POS VLAN → everything else internal | **Deny** |
| Everything else → POS VLAN | **Deny** (admin from MGMT via defined ports only) |
| POS VLAN ↔ guest Wi-Fi | Deny loudly — sharing an SSID/VLAN between payments and guests is the classic small-retail audit failure |

Payment terminals on Wi-Fi: WPA2/3 on a dedicated SSID mapped to the POS VLAN → [[Wi-Fi Fundamentals]] — or better, wired/cellular models.

## Deployment — step-by-step

1. POS VLAN + [[DHCP]] scope (reservations for registers/printers-of-receipts); document → [[Documentation Standards]].
2. Firewall rules per the table; get the **processor's endpoint/IP-range doc** and pin egress to it → [[Firewall Concepts]].
3. Terminals: vendor pairing/activation flow; verify they function with the deny-all egress (if they don't, the vendor doc lists what to open — open *that*, not "any").
4. Registers: POS software per vendor; **no browsing/email on registers** (policy + egress rules enforce it).
5. Receipt printers/cash drawers: usually USB/serial to the register, or IP within the POS VLAN only.
6. **Test failure modes:** primary WAN down → do payments fail over (many terminals do 4G fallback; some processors support store-and-forward with risk caveats)? For retail, payments = revenue: dual WAN is justified here → [[High Availability]], [[SD-WAN Appliance]].

## Availability notes (downtime = lost sales)

- POS stack (register, payment terminal, local switch port, WAN) on the [[UPS]].
- Cloud-POS (Square/Toast/Shopify-style): offline mode capabilities documented and *practiced* by staff.
- Keep a spare configured terminal in the drawer — swap beats debugging at a queue of customers.

## Troubleshooting

| Symptom | Path |
|---|---|
| "Card declined" storm (all cards) | Terminal → processor connectivity: WAN up? egress rule changed? processor status page? |
| Terminal offline | VLAN/DHCP basics ([[Desktops and Laptops]] ladder), then vendor pairing state |
| Slow transactions | WAN latency/saturation → [[QoS]] priority for the POS VLAN is legitimate |
| Register can't reach backend, internet fine | The deny-all caught a changed vendor endpoint — update the pinned list from vendor docs, don't open the floodgates |

Ops: POS/terminal updates come **from the vendor pipeline** — don't block their update hosts · quarterly PCI scan cadence per processor · review firewall rules at each POS software major update · offboarding a location = deactivate terminals with the processor, wipe registers.

Related: [[Network Segmentation]] · [[Firewall Concepts]] · [[High Availability]] · [[VLANs]]
