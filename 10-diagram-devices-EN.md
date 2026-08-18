# 10 · Lab Diagram — Device Functions (Reference)

Companion to [`assets/lab-diagram-tabloid-EN.pdf`](assets/lab-diagram-tabloid-EN.pdf). It lists **what each device in the diagram does**. This document does **not** modify the diagram; it only explains it.

> Convention: one site in the sprint (Ring 0 = Site A); three sites + central control in the expansion (Rings 1–2). All base hardware is identical between sites; only the **profile** (native / bonding / candidate) rotates between units.

---

## Central Control — cloud + lab host (Proxmox) · AS65000

| Device | Function |
|---|---|
| **Digi Remote Manager Premier** (cloud) | Single management plane: golden config + per-site settings, alerts (Device Offline / noncompliance), Push Monitor webhook → SIEM, Digi Containers deployment, and CCCS OTA for the embedded sensor. |
| **Core / route collector** | Public overlay endpoint (IKEv2/IPsec) and eBGP peer for every site (AS65000). Collects the network each site originates. |
| **SIEM** | Receives Push Monitor webhooks and the MP255 agent stream; event / security correlation. |
| **Impairment platform** | Injects controlled loss / jitter / latency to reproduce Starlink conditions during tests. |
| **WAN Bonding server** | External Tier 3 (1 Gb/s) bonding server required by the WAN-bonding experiment (A/B vs SureLink). |
| **Lighthouse Enhance VM** *(Phase 2)* | Opengear OOB management plane. **Only needed if the OOB device is the Opengear OM1304**; with a **Digi Connect IT 4** the OOB is managed by Digi Remote Manager instead (no separate VM). |

## Site A — AS65001 · NATIVE profile · Ring 0 (2-week sprint)

| Device | Function |
|---|---|
| **Starlink Performance Kit (Gen 3)** | Primary WAN. Copper to IX40 `WAN/ETH1`; delivers CGNAT `100.64.0.0/10` (public IPv4 optional). |
| **Carrier A — 5G** | In-band backup WAN through the IX40 5G modem; own SIM/APN. |
| **Digi IX40-05** | The resilient gateway. Runs SureLink failover, the IKEv2/IPsec overlay, eBGP (AS65001) and the Digi Container. |
| **Digi Container** | Metrics collector running **on** the IX40 under resource limits (proves Containers without degrading forwarding). |
| **Service fabric (switch)** | Service network behind `SFP/ETH5` (1 Gb/s SFP, never SFP+): IPv4 `/24` + IPv6 `/64` advertised via eBGP. |
| **Digi XBee Hive** | RF gateway; isolates the sensor application from packet forwarding. |
| **Digi XBee 3 (Zigbee)** | Zigbee radio linking the Hive to the physical sensor. |
| **Physical sensor** | Measures temperature / door state / a digital input; not part of critical forwarding. |
| **Digi ConnectCore MP255** | Embedded SoM. DEY (Yocto) image; agent → SIEM (offline buffer + replay); OTA via CCCS. |
| **OOB console server** *(Phase 2)* — **Opengear OM1304-4E-C-LSP** *and/or* **Digi Connect IT 4** | Out-of-band rescue: serial console (DB9, `Login` mode) to the IX40, independent **5G Carrier B**, management switch and independent power. Two interchangeable options — see the note below. Candidate to deploy at **Site B**. |
| **Carrier B — 5G** | Independent carrier feeding the OOB path (different operator and power from Carrier A / Starlink). |
| **Primary PDU/UPS** | Powers Starlink + IX40 + load. |
| **Secondary PDU/UPS** | Powers the OOB device + modem B (independent power domain). |

## Sites B & C (expansion · Rings 1–2)

| Device | Function |
|---|---|
| **Site B — AS65002** | Identical hardware to Site A. **BONDING** profile: Starlink + 5G inside the bonded tunnel. Planned **OOB via Digi Connect IT 4**. |
| **Site C — AS65003** | Identical to Site A. **CANDIDATE / CHAOS** profile: candidate containers, config and firmware; A/B/C profile rotation. |

---

## OOB console server — two interchangeable options

Both do the same job in the diagram (serial console to the IX40 + independent 5G + own power). They differ mainly in the management plane:

| | **Opengear OM1304-4E-C-LSP** | **Digi Connect IT 4** |
|---|---|---|
| Serial to IX40 | DB9 ↔ RJ45, adapter **319015** (Opengear X2 pinout) | RJ45 RS-232, Digi/Cisco-pinout cable |
| Cellular | 5G Carrier B | LTE Cat 4 Carrier B (Digi CORE modem) |
| Mgmt ports | 4-port mgmt switch | 2× Ethernet 10/100 (add a small switch for 4 ports) |
| Managed by | Opengear **Lighthouse** (needs the Phase-2 VM) | **Digi Remote Manager** (same tenant as the routers → one plane) |

> **Under consideration: Connect IT 4 at Site B.** Choosing it unifies management under DRM and removes the Lighthouse VM; keeping the OM1304 preserves vendor-diverse OOB. Both are listed on purpose — the diagram is left unchanged for now.

## Link legend (as drawn)

- **Starlink primary WAN / overlay + eBGP** — thick blue
- **5G in-band backup / inter-site links** — blue
- **Service fabric (1 Gb/s SFP fiber)** — green
- **Management / LAN (HTTPS · API · DRM)** — dotted grey
- **Out-of-band / Phase 2 elements** — dashed orange
- **Zigbee / cellular RF** — violet wave
- **Dashed box = Phase 2**

---

*Source: [`assets/lab-diagram-tabloid-EN.tex`](assets/lab-diagram-tabloid-EN.tex) and [`03-arquitectura.md`](03-arquitectura.md). Product logos are property of their respective owners.*
