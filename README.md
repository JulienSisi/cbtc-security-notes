# CBTC Security Notes
**OT/ICS Security Analysis — Communication-Based Train Control Systems**

> Personal research notes at the intersection of railway engineering and OT cybersecurity.
> Written from both a technical and field maintenance perspective.

---

## Context

Between 2007 and 2013, I worked as a Test & Maintenance Technician on the Lausanne M2 metro fleet (Alstom, units 241–255) — static and dynamic testing, onboard diagnostics, embedded systems.

CBTC systems like Alstom's **Urbalis Fluence** are now being deployed on these same lines as part of mid-life upgrades. As I transition into OT/ICS cybersecurity, I use this background to analyze the attack surface of CBTC architectures from the inside out.

These notes are not a penetration test. They are a structured threat analysis and IEC 62443 zone mapping exercise, grounded in published research and field experience.

---

## Structure

```
cbtc-security-notes/
├── README.md              ← This file
├── threat-model.md        ← CBTC threat model (STRIDE methodology)
├── iec62443-zones.md      ← IEC 62443 zone & conduit mapping
└── attack-surface.md      ← Attack surface by subsystem (TCMS/VOBC — CCU/VCU/BCU/RIOMs)
```

---

## Scope

**In scope:**
- CBTC architecture: ATP, ATO, ATS subsystems
- Radio communication layer (802.11, LTE, leaky feeder)
- Wayside equipment and onboard computer interfaces
- IEC 62443 zone/conduit model applied to CBTC
- Threat modeling using STRIDE

**Out of scope:**
- Specific vulnerability exploitation
- Proprietary Alstom system internals
- Any live or operational system

---

## Methodology

| Layer | Approach |
|---|---|
| Architecture | Based on published CBTC research (IEEE, IET) |
| Threat modeling | STRIDE per subsystem |
| Security zoning | IEC 62443-3-3 zones & conduits |
| Field context | Personal maintenance experience, M2 Lausanne |

---

## References

Key academic sources used:

- Bin et al. (2006) — CBTC System and Development, WIT Transactions
- Farooq & Soler (2017) — Radio Communication for CBTC, IEEE Surveys & Tutorials
- Zhu et al. (2022) — Joint Security and Train Control in Blockchain-Empowered CBTC, IEEE IoT Journal
- Wang et al. (2019) — Train-to-Train Communications for CBTC, IEEE ITS
- Morar (2010) — Evolution of CBTC Worldwide, IET
- Glickenstein (2015) — 4G Pilot for CBTC, IEEE Vehicular Technology Magazine

---

## Author

**Julien Sisavath**
OT/ICS Security | Ex-Alstom Railway | BSc ISC @ HEIA-FR
eJPT (2026) → OSCP (2027)

[LinkedIn](https://www.linkedin.com/in/juliensisavath/) · [GitHub](https://github.com/JulienSisi) · [juliensisavath.com](https://juliensisavath.com)

---

*These notes are for educational and research purposes only.*
*No proprietary or operational data is referenced.*
