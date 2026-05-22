# Attack Surface Analysis
**TCMS/CBTC Subsystems — Component-Level Security Assessment**

> Component-by-component analysis of the attack surface in a TCMS/CBTC railway system,
> with focus on the mid-life upgrade scenario (legacy MVB/serial → Ethernet/IP/LTE).

---

## 1. Overview

A TCMS/CBTC system exposes attack surface at five distinct layers. Each layer has a different threat profile, exploitability, and safety impact:

```
Layer 5 ── Maintenance & Engineering Access (DMI, TIU, diagnostic tools)
Layer 4 ── Onboard Control Network (TCMS: CCU, VCU, BCU, TCU, RIOMs)
Layer 3 ── VOBC / CBTC Onboard Controller (ATP/ATO functions)
Layer 2 ── Train Communication Network (MVB → Ethernet/ECN migration)
Layer 1 ── External Interfaces (radio, wayside, ATS connectivity)
```

The mid-life upgrade simultaneously touches **all five layers**, creating a window where new and legacy components coexist without consistent security posture.

---

## 2. Layer-by-Layer Analysis

### Layer 1 — External Interfaces

#### 1a. Radio Interface (CBTC wayside ↔ VOBC)

| Attribute | Value |
|---|---|
| Protocol | 802.11p / LTE (Urbalis Fluence) |
| Physical access | Trackside — accessible without train access |
| Legacy state | Pre-upgrade: 802.11 often without mutual authentication |
| Post-upgrade state | LTE adds cellular attack surface |

**Threats:**

| Threat | Description | Safety impact |
|---|---|---|
| Rogue AP | Fake wayside access point sends false movement authority to VOBC | ✅ Critical — collision risk |
| RF jamming | Block CBTC heartbeat → ATP triggers emergency stop | 🟡 Service disruption |
| MITM on radio | Intercept/modify speed/position data in transit | ✅ Critical |
| Passive sniffing | Capture train positions, schedules, operational patterns | 🟡 Intelligence gathering |

**Field note:** The radio interface is the only attack surface physically accessible without entering a depot or control room. A threat actor with basic RF equipment and knowledge of the CBTC frequency band can operate from trackside.

---

#### 1b. ATS / OCC Connectivity (Zone 1 → Zone 2)

| Attribute | Value |
|---|---|
| Protocol | Proprietary + IP, increasingly web-based dashboards |
| Typical connection | ATS workstation connected to both corporate LAN and OT backbone |

**Threats:**

| Threat | Description | Safety impact |
|---|---|---|
| IT → OT pivot | Compromised ATS workstation as entry to Zone Controller | ✅ High |
| Schedule manipulation | Inject artificial delays, disrupt traffic regulation | 🟡 Operational |
| Intel gathering | Full fleet visibility from ATS dashboard | 🟡 Reconnaissance |

**Real incident reference:** WannaCry (2017) reached Deutsche Bahn's passenger information systems via IT–OT connection. The ATS–OT boundary is the most commonly exploited path in railway incidents.

---

### Layer 2 — Train Communication Network (TCN)

#### 2a. MVB (Multifunction Vehicle Bus) — Legacy

| Attribute | Value |
|---|---|
| Standard | IEC 61375-1 (WTB/MVB) |
| Security posture | Designed for robustness/real-time, not cybersecurity |
| Physical access required | Yes — bus connection in vehicle |

**Threats:**

| Threat | Description |
|---|---|
| Message interception | MVB traffic carries braking/traction commands — no encryption |
| Spoofing | Inject false sensor readings (speed, door state, brake availability) |
| Replay attack | Replay valid historical commands to trigger unsafe state |

**Research note (Jiang et al., 2013):** Formal verification of IEC 61375 MVB protocol implementation found safety-critical bugs in the original standard itself — corrected only after subway deployment.

---

#### 2b. Ethernet ECN/ETB — Post-Upgrade

| Attribute | Value |
|---|---|
| Standard | IEC 61375-2-x (ETB/ECN), moving toward TSN |
| Security posture | "Double-edged sword" — bandwidth gain, IT vulnerability import |
| Remote access | Possible via IP if not properly segmented |

**Threats (expanded vs. MVB):**

| Threat | Description | New vs. MVB |
|---|---|---|
| IP/port scanning | Enumerate onboard devices and services | ✅ New |
| DoS via packet flood | Saturate ECN, disrupt real-time control traffic | ✅ New |
| MITM on ECN | Intercept/modify braking commands on consist network | ✅ New |
| Legacy protocol over IP | MVB-like protocols encapsulated in IP lose their physical isolation | ✅ New |

**Key risk:** Gateways translating MVB → Ethernet are single points of failure. If misconfigured or compromised, they expose the formerly isolated MVB layer to IP-based attacks.

---

### Layer 3 — VOBC / CBTC Onboard Controller

The VOBC is the onboard safety-critical component that implements ATP/ATO. It interfaces with both the radio layer (external) and the TCMS (internal).

| Attribute | Value |
|---|---|
| Safety level | SIL 4 (ATP functions) |
| Interfaces | Radio (CBTC), TCMS/CCU, DMI, BTM (balise reader), SSI (speed sensor) |
| Mid-life change | New VOBC added — interface with existing TCMS must be validated |

**Threats:**

| Threat | Description | Safety impact |
|---|---|---|
| False movement authority | Radio-injected fake ZC command accepted by VOBC | ✅ Critical |
| VOBC impersonation | Attacker mimics valid VOBC to send false position to ZC | ✅ Critical |
| VOBC firmware tampering | Malicious firmware via maintenance interface (C5) | ✅ Critical — silent degradation |
| VOBC–TCMS interface spoofing | False brake/door status injected at VOBC→TCMS boundary | ✅ High |

**Field note (M2 Lausanne, 2008–2013):** The VOBC diagnostic interface was proprietary serial, accessible only with specific Alstom tooling in the depot. Post-upgrade Ethernet connectivity to the VOBC fundamentally removes this physical-only protection.

---

### Layer 4 — TCMS Onboard Components

TCMS is the "nervous system" of the train. It connects all subsystems and is the layer Julien Sisavath worked on directly.

#### Component inventory:

| Component | Function | Attack relevance |
|---|---|---|
| **CCU** — Central Control Unit | TCMS master controller, coordinates all subsystems | High-value target — compromise affects all subsystems |
| **VCU** — Vehicle Control Unit | Interfaces wireless link to data center | Exposed to MITM via wireless (Purwanto et al., 2023) |
| **BCU** — Brake Control Unit | Executes braking commands from VOBC/CCU | Safety-critical — false commands = unsafe braking |
| **TCU** — Traction Control Unit | Executes traction commands | Safety-relevant — unexpected acceleration |
| **RIOMs** — Remote I/O Modules | Sensor/actuator interfaces (doors, HVAC, position) | Data integrity — spoofed sensor readings |
| **HMI/DMI** — Driver interface | Train status display, driver inputs | Insider vector, information disclosure |

**Threats (component-level):**

| Component | Primary threat | Impact |
|---|---|---|
| CCU | Firmware tampering via maintenance laptop | Silent safety logic modification |
| VCU | MITM on wireless link | False telemetry to data center |
| BCU | Spoofed brake commands via ECN | Emergency brake failure or unintended activation |
| RIOMs | Sensor data manipulation | Incorrect TCMS state (e.g., doors reported closed when open) |
| DMI/HMI | Physical access in cab | Unauthorized parameter read/write |

---

### Layer 5 — Maintenance & Engineering Access

The maintenance interface is the primary insider threat vector and the layer most directly affected by legacy-to-Ethernet migration.

| Attribute | Value |
|---|---|
| Historical access method | Proprietary serial tools (RS-422/485, MVB diagnostic cards) |
| Post-upgrade access | Ethernet-based diagnostic connections |
| Physical controls | Depot gate, cab key — no network-based access control |

**Threats:**

| Threat | Description | New with Ethernet? |
|---|---|---|
| Infected maintenance laptop | Malware propagates from laptop to TCMS/VOBC via diagnostic port | 🟡 Existed before — worse with Ethernet |
| Insider firmware modification | Technician with legitimate access modifies VOBC/CCU firmware | ✅ No change — existing risk |
| Credential theft | Diagnostic tool credentials used to authenticate remotely | ✅ New — remote access didn't exist with serial |
| Unauthorized parameter change | Modify braking curves, safety thresholds | Existed — easier post-Ethernet |

**Field note (M2 Lausanne, 2008–2013):** Diagnostic sessions on the M2 TCMS required physical presence in the cab or undercar equipment bay, specific Alstom diagnostic software, and hardware dongle authentication. This physical barrier was the primary (and often only) security control. Mid-life upgrades that introduce Ethernet remote diagnostic access remove this barrier entirely without equivalent compensating controls.

---

## 3. Attack Surface Comparison: Pre vs. Post Mid-Life Upgrade

| Attack surface | Pre-upgrade (2008–2013) | Post-upgrade (2026+) | Delta |
|---|---|---|---|
| Remote code execution on TCMS | ❌ Not possible (serial only) | ✅ Possible via Ethernet | 🔴 New |
| VOBC firmware modification | Depot physical access only | Potentially remote | 🔴 New |
| ECN traffic interception | Physical bus tap required | IP capture possible | 🔴 New |
| ATS→OT pivot | Limited (proprietary protocols) | IP-based lateral movement | 🔴 Worse |
| RF spoofing | 802.11 without auth | LTE + 802.11 — wider surface | 🟡 Wider |
| MVB spoofing | Physical tap required | MVB-over-IP accessible | 🔴 New |
| Insider threat | Physical access, proprietary tools | Remote access possible | 🔴 Worse |

---

## 4. Priority Mitigations by Layer

| Layer | Priority mitigation | Standard reference |
|---|---|---|
| Radio (L1a) | Mutual authentication on CBTC radio links | IEC 62443 SR 1.2, EN 50159 |
| ATS boundary (L1b) | Strict IT–OT segmentation, no dual-homed ATS workstations | IEC 62443 SR 5.1 |
| ECN/Ethernet (L2b) | Anomaly-based IDS on ECN traffic | IEC 62443 SR 6.1 |
| VOBC (L3) | Signed firmware, HSM-based key storage | IEC 62443 SR 3.2 |
| TCMS/VCU (L4) | Securebox-style HSM on VCU wireless interface | Purwanto et al. (2023) |
| Maintenance (L5) | Role-based access, session logging, MFA for remote access | IEC 62443 SR 2.1, SR 2.8 |

---

## 5. References

Key sources for this analysis:

- Ibadah et al. (2024) — Comprehensive cybersecurity strategy for onboard/trackside
- Purwanto et al. (2023) — Securebox architecture for TCMS security
- Yue et al. (2021) — IDS for Train Ethernet Consist Network
- Duo et al. (2021) — Anomaly detection for train real-time Ethernet
- Wang & Liu (2022) — Cyber security risk management for railway CPS
- Yang et al. (2025) — Hierarchical risk assessment for TCMS
- Kour et al. (2022) — Cybersecurity review in railways
- Jiang et al. (2013) — Formal verification of IEC 61375 MVB protocol

---

*See also: [threat-model.md](threat-model.md) — STRIDE analysis*
*See also: [iec62443-zones.md](iec62443-zones.md) — Zone & conduit mapping*

*Last updated: May 2026 — Julien Sisavath*
