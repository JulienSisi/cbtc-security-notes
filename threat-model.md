# CBTC Threat Model
**STRIDE Analysis — Communication-Based Train Control Systems**

> Based on published research and field maintenance experience (Alstom M2 Lausanne, 2008–2013).

---

## 1. System Overview

A CBTC system replaces fixed-block signalling with continuous bidirectional train–wayside communication. Three core functional subsystems operate in real time:

| Subsystem | Function | Safety-critical |
|---|---|---|
| **ATP** — Automatic Train Protection | Enforces safe separation, triggers emergency braking | ✅ SIL 4 |
| **ATO** — Automatic Train Operation | Controls acceleration, braking, dwell time | ✅ SIL 2–3 |
| **ATS** — Automatic Train Supervision | Fleet monitoring, scheduling, traffic regulation | 🟡 SIL 1 |

The radio layer (802.11p / LTE / leaky feeder) carries all train–wayside data: position reports, movement authorities, control commands, and diagnostics.

---

## 2. Architecture Diagram (Logical)

```
┌─────────────────────────────────────────────────────┐
│                  CONTROL CENTER                      │
│  ┌──────────┐   ┌──────────┐   ┌──────────────────┐ │
│  │   ATS    │   │  ZC/CI   │   │  Interlocking    │ │
│  │Supervision│  │Zone Ctrl │   │  (wayside logic) │ │
│  └────┬─────┘   └────┬─────┘   └────────┬─────────┘ │
└───────┼──────────────┼──────────────────┼────────────┘
        │  Data network (OT backbone)      │
        ▼                                  ▼
┌──────────────────┐              ┌────────────────────┐
│  WAYSIDE RADIO   │◄────────────►│  WAYSIDE EQUIPMENT │
│  Access Points   │   RF link    │  Beacons, Balises  │
└────────┬─────────┘              └────────────────────┘
         │  802.11 / LTE / Leaky feeder
         ▼
┌─────────────────────────────────────────────────────┐
│                    ONBOARD                           │
│  ┌──────────┐   ┌──────────┐   ┌──────────────────┐ │
│  │   ATP    │   │   ATO    │   │  TCMS / Onboard  │ │
│  │ (VOBC)   │   │          │   │  Radio Unit      │ │
│  └──────────┘   └──────────┘   └──────────────────┘ │
└─────────────────────────────────────────────────────┘
```

**VOBC** = Vital Onboard Controller (handles ATP/ATO logic onboard)

---

## 3. Trust Boundaries

Four primary trust boundaries define where security controls are required:

```
[TB-1] Control Center ↔ OT Backbone
[TB-2] OT Backbone ↔ Wayside Radio Infrastructure
[TB-3] Radio Layer ↔ Onboard Equipment (VOBC)
[TB-4] Onboard ↔ TCMS / Engineering Workstation
```

**Field note:** During M2 Lausanne maintenance (2008–2013), engineering workstations connected directly to onboard systems for diagnostic log extraction. TB-4 was physically enforced by access control to the depot — not by network segmentation.

---

## 4. STRIDE Threat Analysis

### 4.1 Radio Layer (TB-3) — Highest exposure

The radio interface is the most exposed attack surface: it is physically accessible from trackside.

| STRIDE | Threat | Example | Impact |
|---|---|---|---|
| **S**poofing | Rogue wayside radio AP | Fake zone controller sends false movement authority | Train collision |
| **T**ampering | RF signal injection | Corrupt position reports | Wrong safe separation calculation |
| **R**epudiation | Log manipulation on wayside AP | Cover unauthorized access | Audit failure |
| **I**nfo Disclosure | Passive RF sniffing | Capture train position & scheduling data | Operational intelligence leak |
| **D**oS | RF jamming | Block ATP heartbeat | Emergency stop, service disruption |
| **E**levation | Compromised AP → OT backbone | Lateral movement to Zone Controller | Full line control loss |

**Relevant research:** Farooq & Soler (2017) identify the radio layer as the primary attack vector in CBTC. LTE deployment (Glickenstein, 2015) introduces new attack surface from cellular infrastructure.

---

### 4.2 Zone Controller / Interlocking (TB-1, TB-2)

The Zone Controller (ZC) calculates movement authorities based on train positions. Compromise = direct safety impact.

| STRIDE | Threat | Example | Impact |
|---|---|---|---|
| **S**poofing | Impersonation of VOBC | Send false position to ZC | Incorrect movement authority |
| **T**ampering | Modify ZC configuration | Change safe braking parameters | Loss of ATP function |
| **D**oS | Flood ZC with position messages | Resource exhaustion | ZC fallback / degraded mode |
| **E**levation | OT network pivot via ATS workstation | ATS → ZC lateral movement | Safety system compromise |

**Field note:** From maintenance experience, ZC fallback mode (degraded signalling) requires manual intervention and significantly reduces line capacity — this alone is a viable disruption target.

---

### 4.3 ATS / Supervision Layer (TB-1)

Lower safety criticality but high operational impact and likely softer security posture.

| STRIDE | Threat | Example | Impact |
|---|---|---|---|
| **I**nfo Disclosure | ATS dashboard access | Full fleet position, schedule, headways | Operational intelligence |
| **T**ampering | Schedule manipulation | Inject artificial delays | Service disruption |
| **E**levation | ATS workstation → OT backbone | Engineering workstation pivot | Deeper network access |

**Note:** ATS systems are often connected to both the OT network and corporate IT for reporting purposes — a common weak point in segmentation.

---

### 4.4 Onboard Systems / TCMS (TB-4)

Physical access required (depot or maintenance). Lower remote threat, but significant during maintenance windows.

| STRIDE | Threat | Example | Impact |
|---|---|---|---|
| **T**ampering | Malicious firmware on VOBC | Modified ATP logic | Silent safety degradation |
| **E**levation | Maintenance laptop → VOBC | Infected diagnostic tool | Persistent onboard compromise |
| **I**nfo Disclosure | Log extraction | Operational patterns, system config | Reconnaissance |

**Field note:** At Alstom M2 (2008–2013), VOBC diagnostic access was via proprietary serial interfaces. No network-based remote access existed at the time — but mid-life upgrades introducing Ethernet/IP connectivity change this threat model significantly.

---

## 5. Top 5 Risk Priorities

Ranked by impact × likelihood for a modernized CBTC deployment (post Urbalis Fluence upgrade):

| # | Threat | Subsystem | Why now |
|---|---|---|---|
| 1 | Rogue wayside AP / RF spoofing | Radio layer | LTE introduction widens attack surface |
| 2 | ATS → OT backbone lateral movement | ATS / IT-OT boundary | ATS often IT-connected |
| 3 | Engineering workstation compromise | TB-4 | Mid-life upgrades introduce new Ethernet interfaces |
| 4 | ZC DoS via position message flooding | Zone Controller | Protocol rarely authenticated |
| 5 | Passive RF intelligence gathering | Radio layer | No encryption in legacy 802.11 CBTC deployments |

---

## 6. Mitigations (IEC 62443 aligned)

| Risk | Mitigation | IEC 62443 Reference |
|---|---|---|
| RF spoofing | Mutual authentication on radio interfaces | SR 1.2 — Software Process and Device Identification |
| Lateral movement | Strict zone/conduit segmentation between ATS and ZC | SR 5.1 — Network Segmentation |
| Workstation compromise | Endpoint hardening, signed firmware updates | SR 3.2 — Malicious Code Protection |
| ZC DoS | Rate limiting, protocol anomaly detection | SR 7.1 — DoS Protection |
| RF sniffing | Encrypted radio link (LTE with proper PKI) | SR 4.1 — Information Confidentiality |

*Full IEC 62443 zone & conduit mapping: see [iec62443-zones.md](iec62443-zones.md)*

---

## 7. Limitations & Scope Reminder

- This analysis is based on **published academic research** and general CBTC architecture knowledge.
- No proprietary Alstom system documentation was used or referenced.
- No operational or live system data is included.
- Field notes reflect publicly known maintenance practices — not confidential information.

---

*Last updated: May 2026*
*Author: Julien Sisavath — [juliensisavath.com](https://juliensisavath.com)*
