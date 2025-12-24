# Architecture Critique: House / Cluster / Swarm-Grid

**Status:** Review document for future work
**Scope:** Inverter → House → Cluster → Swarm progression
**Next Steps:** Address issues as each level is implemented

---

## TLDR

### What Works Well
- **Self-reliant houses** - each 6-15kW inverter handles its own loads, sharing is backup only
- **Droop control** - physics-based load balancing, no complex coordination needed
- **Isolated DC** - 48V stays inside each house, only AC shared (simpler, cheaper)
- **Scalable** - same pattern repeats: house → cluster → swarm

### Critical Issues (must fix)
| Issue | Risk | Fix |
|-------|------|-----|
| Master ESP32 = single point of failure | Swarm loses sync if master dies | Add master election protocol |
| Linear topology (A─B─C) | Cable cut isolates downstream clusters | Consider ring or accept risk |
| No grounding scheme defined | Safety hazard, ground loops | Define N-G bond points |
| No metering | Social conflict (free riders) | Add kWh counters per house |
| No governance | Who decides maintenance, expansion? | Write governance document |

### Issues by Implementation Phase

**Phase 1 - Single Inverter:** No blocking issues. Build it.

**Phase 2 - House Scale:**
- Define grounding (where is N-G bond?)
- Test BMS-inverter communication

**Phase 3 - Cluster (3-4 houses):**
- Test droop stability with multiple inverters
- Add isolation breakers
- Solve master election

**Phase 4 - Swarm (12 houses):**
- CAN repeaters for 800m
- Metering for fairness
- Governance document

---

## Detailed Analysis

### 1. ASSUMPTIONS

These are beliefs embedded in the design. If any are wrong, the design breaks.

#### Electrical Assumptions

| ID | Assumption | Risk if Wrong |
|----|------------|---------------|
| A1 | All inverters can parallel via droop without oscillation | System instability, damage |
| A2 | EG8010 supports droop or can be modified | Need different chip |
| A3 | CAN bus works reliably at 800m | Communication failures |
| A4 | 4mm² cable handles 200-400m runs | Voltage drop, overheating |
| A5 | Single-phase 230V is sufficient | Can't run 3-phase equipment |
| A6 | 49.7-50.0 Hz range is acceptable | Sensitive equipment fails |
| A7 | Frequency deviation won't damage electronics | Clock drift, motor issues |

**Validation needed:** A1 and A3 are highest risk. Test with 3 inverters before scaling.

#### Battery/Solar Assumptions

| ID | Assumption | Risk if Wrong |
|----|------------|---------------|
| B1 | 5-10 kWh lasts overnight + cloudy day | Blackouts in bad weather |
| B2 | 6 kW solar matches local conditions | Under/over production |
| B3 | Houses have similar consumption | Imbalanced sharing |
| B4 | LiFePO4 is available/affordable | Higher costs |
| B5 | BMS correctly limits discharge | Battery damage |

**Validation needed:** B1 depends on your climate. Calculate for worst-case winter.

#### Social Assumptions

| ID | Assumption | Risk if Wrong |
|----|------------|---------------|
| S1 | Everyone agrees to share without metering | Resentment, conflict |
| S2 | Technical skills exist for maintenance | System degrades |
| S3 | No one abuses shared resource | Tragedy of commons |
| S4 | Master ESP32 owner is trusted | Power/control issues |
| S5 | No legal liability for faults | Lawsuits |

**Validation needed:** S1 and S3 cause most community energy projects to fail. Address early.

#### Physical Assumptions

| ID | Assumption | Risk if Wrong |
|----|------------|---------------|
| P1 | 800m linear site layout | Longer cable runs |
| P2 | Right-of-way for cabling exists | Legal/access issues |
| P3 | All roofs fit 6kW solar | Reduced capacity |
| P4 | Year-round solar production | Seasonal gaps |

---

### 2. ANALYSIS BY STANDPOINT

#### Electrical Engineering

**Droop Stability (HIGH risk)**
```
Problem: Multiple droop-controlled inverters can oscillate if:
- Droop slopes don't match
- Cable impedance creates phase shifts
- Response times differ

What happens:
  Inverter A increases output → frequency rises
  Inverter B sees high freq → reduces output
  Frequency drops → Inverter A increases again
  → Oscillation at 1-10 Hz, can damage equipment

Fix:
- Use identical droop coefficients on all inverters
- Test with 3 inverters before scaling
- Add damping (slow response) if oscillation occurs
```

**CAN Bus Distance (MEDIUM risk)**
```
Problem: Standard CAN rated for ~500m at 125kbps
         Your site is 800m

What happens:
  Signal reflection at cable ends
  Bit errors increase
  Sync messages lost
  → Inverters lose phase lock

Fix:
- Add CAN repeater at each cluster boundary
- Or reduce bitrate to 50kbps (still enough for 100Hz sync)
- Use proper termination (120Ω at each end)
```

**Voltage Drop on Long Cables (MEDIUM risk)**
```
Problem: 4mm² copper, 400m one-way, 20A load

Calculation:
  R = 0.00446 Ω/m × 800m (round trip) = 3.57Ω
  V_drop = 20A × 3.57Ω = 71V ← TOO HIGH

Wait, that's wrong. Let me recalculate:
  R = 4.61 mΩ/m for 4mm² copper
  R_total = 0.00461 × 800m = 3.69Ω
  At 20A: V_drop = 73.8V ← UNACCEPTABLE

Fix:
- Use 10mm² cable for inter-cluster runs (V_drop = 29V, ~13%)
- Or 16mm² for longer runs
- Or accept that inter-cluster sharing is limited to ~10A
```

**Grounding (HIGH risk)**
```
Problem: Where is the Neutral-Ground bond?
- If every house has N-G bond → ground loops, circulating current
- If no house has N-G bond → floating neutral, shock hazard

What happens with ground loops:
  House A ground is 2V different from House C ground
  Current flows through neutral: A → B → C → earth → A
  → Heats cables, trips RCDs randomly

Fix:
- ONE N-G bond per cluster (at "master" house)
- All other houses: neutral and ground separate
- Connect all house grounds to cluster ground bus
- Ground rod at each cluster
```

#### Reliability / Resilience

**Master ESP32 Single Point of Failure (HIGH risk)**
```
Current design:
  One ESP32 is "master" - broadcasts sync @ 100Hz
  All others are "slaves" - follow master timing

If master fails:
  - No sync signal
  - Inverters drift out of phase
  - When phase difference > 10°, circulating current flows
  - Breakers trip or inverters fight each other

Fix - Master Election Protocol:
  1. All ESP32s monitor CAN for sync messages
  2. If no sync received for 100ms, highest-ID node becomes master
  3. New master starts broadcasting
  4. If old master recovers, it becomes slave (lower priority)

  Implementation: ~50 lines of code, test thoroughly
```

**Linear Topology Vulnerability (HIGH risk)**
```
Current: A ─── B ─── C

If cable between A-B is cut:
  - Cluster C is completely isolated
  - Can only use its own 24kW
  - If C has low batteries and high load → blackout

If cable between B-C is cut:
  - Same problem for C

Fix options:
  1. Accept the risk (simplest)
  2. Ring topology: A ─── B ─── C ─── A (doubles cable cost)
  3. Backup wireless link (ESP-NOW for emergency sync)
```

#### Economics

**No Metering = Social Conflict (HIGH risk)**
```
Scenario:
  House A: 2 people, uses 8 kWh/day
  House B: 6 people, uses 25 kWh/day, has EV

  House B draws from shared bus constantly
  House A's battery never fully charges
  House A owner: "I'm subsidizing House B!"

This kills community energy projects.

Fix:
  Add simple kWh meter at each house AC output
  Options:
  - Display only (transparency, no billing)
  - Monthly "balancing" payments
  - Soft limits (ESP32 reduces output if house is net-negative)

  Even without billing, VISIBILITY reduces conflict.
```

#### Safety / Compliance

**Not Grid-Code Compliant (MEDIUM risk)**
```
Reality:
  - DIY inverters have no CE/UL certification
  - Home insurance may not cover fire/damage
  - If someone is injured, liability is unclear

Mitigations:
  - Document everything (design, testing, commissioning)
  - Have licensed electrician sign off on AC wiring
  - Keep inverters in fire-resistant enclosures
  - Consider forming legal entity (co-op) for liability shield
```

**Arc Fault Risk (MEDIUM risk)**
```
Problem: 800m of cable = many junction points
         Loose connection → arc → fire

Fix:
  - Use proper cable glands and junction boxes (IP65)
  - Torque all connections to spec
  - Annual thermographic inspection
  - Consider AFCI breakers at cluster entry
```

---

### 3. RECOMMENDATIONS BY PRIORITY

#### Critical (before connecting multiple houses)

| # | Action | Why | Effort |
|---|--------|-----|--------|
| R1 | Define grounding scheme | Safety, prevents ground loops | 1 day design |
| R2 | Test droop with 3 inverters | Proves stability before scaling | 1 week |
| R3 | Implement master election | Eliminates single point of failure | 2 days code |
| R4 | Calculate actual cable sizing | 4mm² may be too small | 1 day |

#### High Priority (before 12-house swarm)

| # | Action | Why | Effort |
|---|--------|-----|--------|
| R5 | Add CAN repeaters | 800m exceeds reliable range | $50 + 1 day |
| R6 | Add kWh metering | Prevents social conflict | $30/house |
| R7 | Write governance doc | Defines decision-making | 1 day |
| R8 | Add cluster isolation breakers | Safe maintenance | $100/cluster |

#### Medium Priority (for robustness)

| # | Action | Why | Effort |
|---|--------|-----|--------|
| R9 | Add surge protection (SPD) | Lightning protection | $50/cluster |
| R10 | Document commissioning procedure | Repeatable deployment | 1 day |
| R11 | Add per-inverter status display | Easier debugging | $20/inverter |
| R12 | Create fault diagnosis guide | Faster repair | 1 day |

#### Nice to Have

| # | Action | Why | Effort |
|---|--------|-----|--------|
| R13 | Home Assistant dashboard | Visibility for all users | 2 days |
| R14 | Automatic load shedding | Graceful degradation | 1 week |
| R15 | Ring topology | Redundant path | 2x cable cost |

---

### 4. IMPLEMENTATION PHASES

```
PHASE 1: Single Inverter (current work)
├── Build OzInverter 6kW
├── Test with resistive load
├── Test with motor (inductive)
├── Measure output: voltage, frequency, THD
└── NO CLUSTER ISSUES YET

PHASE 2: House Scale
├── Add solar + MPPT + battery
├── Define grounding (N-G bond location)
├── Test BMS communication with inverter
├── Verify inverter reduces output when BMS signals low SOC
└── Run house on inverter for 1 month

PHASE 3: First Cluster (3-4 houses)
├── Connect 2 inverters, test droop stability
├── Add 3rd inverter, verify stability holds
├── Implement master election protocol
├── Test failure scenarios:
│   ├── Master power off → slave takes over?
│   ├── One inverter overloaded → others help?
│   └── One inverter fault → others isolate it?
├── Add isolation breakers
└── Run cluster for 1 month

PHASE 4: Full Swarm (12 houses)
├── Add CAN repeaters between clusters
├── Install kWh meters
├── Create governance document
├── Commission remaining houses one by one
├── Test inter-cluster power sharing
└── Document everything
```

---

### 5. QUESTIONS TO RESOLVE

These need answers before scaling:

1. **Grounding:** Where exactly is N-G bond? One per cluster or one per swarm?

2. **Master election:** What's the tiebreaker if two ESP32s have same ID?

3. **Droop coefficients:** What frequency deviation per kW? 0.1Hz/kW? 0.2Hz/kW?

4. **Cable sizing:** Is 4mm² actually enough? Recalculate for your actual distances.

5. **Metering:** Display-only or actual billing? Who owns the data?

6. **Governance:** Who can veto adding a new house? Majority vote? Unanimous?

7. **Exit:** If one house leaves, who pays to reconfigure?

8. **Insurance:** Does homeowner insurance cover DIY inverter damage?

---

### 6. REFERENCE: FAILURE MODE TABLE

| Failure | Detection | Impact | Recovery |
|---------|-----------|--------|----------|
| Master ESP32 dies | No sync on CAN for 100ms | Phase drift → circulating current | Master election kicks in |
| Inverter MOSFET short | Overcurrent trip | House offline | Breaker isolates, neighbors supply via bus |
| Battery BMS lockout | CAN message "BMS fault" | House draws from bus only | Check BMS, reset |
| Cable cut (intra-cluster) | Loss of CAN + no power flow | Houses downstream isolated | Manual inspection, repair |
| Cable cut (inter-cluster) | Cluster B sees no traffic from A | Clusters operate independently | Repair cable |
| Droop oscillation | Frequency swings ±0.5Hz | Equipment stress | Reduce droop slope, add damping |
| Ground fault | RCD trips | House/cluster offline | Find fault, repair insulation |
| Lightning strike | SPD clamps, may sacrifice | SPD needs replacement | Replace SPD, check equipment |

---

*This document is a living critique. Update as issues are resolved or new ones discovered.*
