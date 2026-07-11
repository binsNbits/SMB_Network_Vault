---
tags: [device, edge]
aliases: [ONT, Cable Modem, DSL Modem, ISP Gateway]
---

# Modem

**Role:** terminates the ISP's medium (coax/DSL/fiber) and hands you Ethernet. Device #1 at the internet edge — everything downstream depends on it.

**Position:** `ISP plant → MODEM → [[Router]]/[[Firewall (NGFW)]] WAN port`

## Variants — know which you have

| Type | Medium | Notes |
|---|---|---|
| Cable modem (DOCSIS 3.0/3.1) | Coax | Diagnostics at `192.168.100.1` even when offline |
| DSL/VDSL modem | Phone line | May require ISP PPPoE credentials |
| **ONT** (fiber) | Fiber | Usually ISP-locked; you just take the Ethernet handoff |
| ISP **gateway** (combo modem+router+Wi-Fi) | any | The problem child — see bridge mode below |
| Fixed wireless / 5G | Radio | Often CGNAT ([[NAT]]) — inbound services need workarounds |

## The one big configuration decision: bridge mode

If the ISP supplied a **gateway** (modem+router combo) and you run your own [[Router]]/[[Firewall (NGFW)]]:

- **Bridge the gateway** (modem-only mode) so your firewall gets the **public IP** directly.
- **Why:** otherwise you have *double NAT* — broken port forwards, flaky [[VoIP Phones and PBX|VoIP]], confused [[VPN Fundamentals|VPNs]]. Detect it: your firewall's WAN IP is private → [[Private IP Ranges]].
- **How:** gateway's web UI → often "Bridge Mode" / "IP Passthrough" / "DMZ+" — or call the ISP (some models are carrier-bridged only). Disable its Wi-Fi regardless.

## Access

| Method | Detail |
|---|---|
| Browser | Cable: `192.168.100.1`; gateways: `192.168.0.1` / `192.168.1.1` → [[Vendor Default IPs and Logins]] |
| CLI | Generally none available to customers |
| Physical | Standby/reset buttons; label with account number → [[Documentation Standards]] |

## Health signals worth reading (cable example, `192.168.100.1`)

| Metric | Healthy | Bad sign |
|---|---|---|
| Downstream power | −7 to +7 dBmV | Beyond ±10: coax/splitter problem |
| SNR | > 33 dB | < 30: noise ingress |
| Upstream power | 35–50 dBmV | Near 51+: modem shouting = plant problem |
| Uncorrectable codewords | ~static | Climbing = intermittent drops — evidence for the ISP call |

Screenshot these when the line is *healthy* — your baseline for outage disputes.

## Troubleshooting sequence (feeds [[No Internet Playbook]])

1. Lights: power → downstream/DS → upstream/US → online. Stuck before "online" = ISP-side; call with your codeword/SNR evidence.
2. Power-cycle **modem first**, wait for online, *then* the router. Why: the modem must sync and (cable) register the router's MAC before DHCP on the WAN works.
3. Online light on but no WAN IP on the firewall → release/renew WAN, check PPPoE creds (DSL), check MAC-lock (some ISPs bind to first-seen MAC — clone it or power-cycle longer).
4. Bypass test: laptop directly into modem (mind: it's now unfirewalled — briefly!). Works → your router is the problem, not the ISP.

## Ops notes

- On the [[UPS]] — during power cuts, internet + Wi-Fi survive for laptops on battery.
- Record: account #, circuit ID, ISP support #, static IP details → [[Site Note Template]].
- No config backup needed (nothing to configure when bridged) — the exception on the device list.

Related: [[Router]] · [[NAT]] · [[SD-WAN Appliance]] (for WAN failover) · [[High Availability]]
