# IEC 62443 Zone & Conduit Mapping
**Applied to TCMS/CBTC Railway Systems — Mid-Life Upgrade Context**

> Zone and conduit analysis based on IEC 62443-3-3, aligned with CLC/TS 50701 and IEC 61375.
> Grounded in field maintenance experience (Alstom M2 Lausanne, 2007–2013) and academic research.

---

## 1. Standards Stack

Railway cybersecurity does not rely on a single standard. It operates as a layered stack:

```
┌─────────────────────────────────────────────────────┐
│  EN 50126 / EN 50128 / EN 50129                      │
│  Safety lifecycle — RAMS, SIL assessment             │
├─────────────────────────────────────────────────────┤
│  EN 50159                                            │
│  Safety-related communication (integrity, auth,      │
│  sequencing, timing — not full cybersecurity)        │
├─────────────────────────────────────────────────────┤
│  CLC/TS 50701                                        │
│  Railway-specific cybersecurity framework            │
│  (links EN 50159 safety comms to IEC 62443)          │
├─────────────────────────────────────────────────────┤
│  IEC 62443 (2-1, 3-2, 3-3)                          │
│  OT/IACS security — zones, conduits, SLs             │
├─────────────────────────────────────────────────────┤
│  IEC 61375                                           │
│  Train Communication Network — MVB/WTB/ETB/ECN       │
└─────────────────────────────────────────────────────┘
```

**Key relationship:** EN 50159 covers functional safety of communication (message errors, sequencing). It explicitly points to IEC 62443 for broader cybersecurity. CLC/TS 50701 bridges the two in railway-specific terms. *Neither EN 50159 alone nor IEC 61375 alone constitutes a cybersecurity framework.*

---

## 2. Zone Architecture

Based on IEC 62443-3-3 principles and railway-specific guidance (Procházka et al., 2020; Kour et al., 2022):

```
┌──────────────────────────────────────────────────────────────────┐
│  ZONE 0 — Corporate / Public Network                             │
│  Security Level: SL 0                                            │
│  Passenger Wi-Fi, public internet, corporate IT                  │
└──────────────┬───────────────────────────────────────────────────┘
               │ Conduit C1 (firewall + DMZ)
┌──────────────▼───────────────────────────────────────────────────┐
│  ZONE 1 — Operations & Supervision (ATS / OCC)                   │
│  Security Level: SL 1                                            │
│  ATS workstations, fleet monitoring dashboards, scheduling       │
│  ⚠ Often connected to both corporate IT and OT backbone          │
└──────────────┬───────────────────────────────────────────────────┘
               │ Conduit C2 (strict segmentation — critical boundary)
┌──────────────▼───────────────────────────────────────────────────┐
│  ZONE 2 — OT Control Backbone                                    │
│  Security Level: SL 2                                            │
│  Zone Controllers (ZC), interlocking, wayside control            │
│  ETB (Ethernet Train Backbone) where deployed                    │
└──────────────┬───────────────────────────────────────────────────┘
               │ Conduit C3 (radio interface — highest exposure)
┌──────────────▼───────────────────────────────────────────────────┐
│  ZONE 3 — Radio / Wayside Infrastructure                         │
│  Security Level: SL 2–3                                          │
│  802.11 / LTE access points, leaky feeder, balises, BTM          │
│  ⚠ Physically accessible from trackside                          │
└──────────────┬───────────────────────────────────────────────────┘
               │ Conduit C4 (onboard radio unit)
┌──────────────▼───────────────────────────────────────────────────┐
│  ZONE 4 — Onboard Safety-Critical Systems (TCMS / VOBC)          │
│  Security Level: SL 3                                            │
│  VOBC (ATP/ATO), TCMS (CCU, VCU, BCU, TCU, RIOMs)               │
│  MVB/ECN onboard network                                         │
│  ⚠ Legacy TCMS newly exposed via Ethernet/IP during mid-life     │
└──────────────┬───────────────────────────────────────────────────┘
               │ Conduit C5 (maintenance/diagnostic interface)
┌──────────────▼───────────────────────────────────────────────────┐
│  ZONE 5 — Maintenance & Engineering Access                       │
│  Security Level: SL 1 (access-controlled)                        │
│  DMI (Driver Machine Interface), TIU, diagnostic terminals       │
│  Engineering workstations (depot-only, physical access)          │
└──────────────────────────────────────────────────────────────────┘
```

---

## 3. Conduit Analysis

### C1 — Corporate/Public → ATS (Zone 0 → Zone 1)
| Property | Detail |
|---|---|
| Direction | Bidirectional (monitoring data up, schedule data down) |
| Protocol | TCP/IP over enterprise network |
| Current controls | Typically firewall + VPN |
| Risk | ATS workstations are often the weakest IT–OT boundary |
| IEC 62443 SR | SR 5.1 — Network Segmentation |

---

### C2 — ATS → OT Backbone (Zone 1 → Zone 2) ⚠ Critical
| Property | Detail |
|---|---|
| Direction | Bidirectional |
| Protocol | Proprietary TCMS/ATS protocols, increasingly IP-based |
| Current controls | Often inadequate — legacy designs assumed air-gap |
| Risk | Lateral movement from ATS workstation into Zone Controller |
| IEC 62443 SR | SR 5.1, SR 5.2 — Zone Boundary Protection |

**Field note:** ATS systems have historically been connected to both OT and IT for reporting purposes — this dual connection is the most commonly exploited pivot point in railway incidents (e.g., Deutsche Bahn / WannaCry, 2017).

---

### C3 — OT Backbone → Radio Infrastructure (Zone 2 → Zone 3) ⚠ Highest exposure
| Property | Detail |
|---|---|
| Direction | Bidirectional |
| Protocol | 802.11p, LTE, leaky feeder RF |
| Current controls | Variable — legacy deployments often lack mutual authentication |
| Risk | Rogue access point, RF spoofing, MITM on movement authorities |
| IEC 62443 SR | SR 1.2 — Software/Device Identification, SR 4.1 — Confidentiality |

**Mid-life upgrade note:** Introduction of LTE/4G radio (Alstom Urbalis Fluence) expands attack surface from RF-local to cellular network perimeter.

---

### C4 — Radio → Onboard TCMS/VOBC (Zone 3 → Zone 4) ⚠ Safety-critical
| Property | Detail |
|---|---|
| Direction | Bidirectional |
| Protocol | CBTC radio protocol over 802.11/LTE |
| Current controls | EN 50159 message integrity (sequence numbers, CRC, timeouts) |
| Risk | False movement authority injection, VOBC spoofing |
| IEC 62443 SR | SR 3.1 — Communication Integrity |

**Key distinction:** EN 50159 protects against *accidental* message errors. It does not protect against *intentional* spoofing with correct message format. IEC 62443 SR 3.1 adds cryptographic authentication on top.

---

### C5 — TCMS → Maintenance Interface (Zone 4 → Zone 5)
| Property | Detail |
|---|---|
| Direction | Bidirectional (read diagnostics, write config) |
| Protocol | Legacy: proprietary serial (RS-422/485, MVB). Post-upgrade: Ethernet/IP |
| Current controls | Physical access control (depot gate) — not network-based |
| Risk | Malicious firmware via infected maintenance laptop, insider access |
| IEC 62443 SR | SR 3.2 — Malicious Code Protection, SR 2.6 — Remote Session Management |

**Field note (M2 Lausanne, 2008–2013):** VOBC and TCMS diagnostic access used proprietary serial interfaces with dedicated tools. No network-based remote access existed. Mid-life upgrade introducing Ethernet replaces physical-only access with network-reachable interfaces — this fundamentally changes the risk profile of Zone 5.

---

## 4. Security Level Assignment

Derived from Procházka et al. (2020) and IEC 62443-3-3:

| Zone | Network type | IEC 62443 SL | Rationale |
|---|---|---|---|
| Zone 0 | Public / corporate | SL 0 | No security requirements on TCMS side |
| Zone 1 | ATS / supervision | SL 1 | Operational disruption impact |
| Zone 2 | OT control backbone | SL 2 | Safety system adjacency |
| Zone 3 | Radio / wayside | SL 2–3 | Physical accessibility + safety impact |
| Zone 4 | Onboard TCMS/VOBC | SL 3 | Direct safety-critical functions |
| Zone 5 | Maintenance / DMI | SL 1–2 | Controlled physical access, insider risk |

---

## 5. Mid-Life Upgrade — Zone Impact Assessment

The M2 Lausanne modernization (Alstom Urbalis Fluence CBTC + fleet mid-life upgrade) modifies the zone architecture in three specific ways:

**Change 1: Zone 4 gains Ethernet/IP connectivity**
Legacy TCMS used MVB (closed fieldbus). Post-upgrade, Ethernet/ECN interfaces are introduced.
→ Zone 4 boundary (Conduit C4) expands attack surface from RF-local to IP-reachable.

**Change 2: New VOBC joins Zone 4**
VOBC (Urbalis Fluence onboard controller) is added as a new critical asset interfacing TCMS.
→ The VOBC–TCMS interface becomes a new internal conduit within Zone 4, requiring explicit segmentation.

**Change 3: Maintenance interface migrates from serial to IP (Zone 5)**
Diagnostic tools transition from proprietary serial to Ethernet-based connections.
→ Zone 5 physical-only access boundary no longer sufficient. Network-based access controls required.

---

## 6. Applicable Standards Summary

| Standard | Scope | Railway relevance |
|---|---|---|
| **IEC 62443-3-3** | System security requirements & SLs | Zone/conduit design, SR mapping |
| **CLC/TS 50701** | Railway-specific cybersecurity | Bridges IEC 62443 to EN 50159/50126 |
| **EN 50159** | Safety-related communication | Message integrity, not full cybersecurity |
| **IEC 61375** | Train Communication Network (TCN) | MVB/WTB/ETB/ECN architecture |
| **EN 50126** | RAMS / safety lifecycle | Cybersecurity at each lifecycle phase |
| **NIST CSF** | Identify/Protect/Detect/Respond/Recover | Complementary IT/OT governance |

---

*See also: [threat-model.md](threat-model.md) — STRIDE analysis per subsystem*
*See also: [attack-surface.md](attack-surface.md) — Component-level attack surface*

*Last updated: May 2026 — Julien Sisavath*
