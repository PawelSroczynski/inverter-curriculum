# From Zero Electronics to Community Energy Independence

**Build droop-controlled inverters. Connect them into a tiered AC swarm grid.**

Learn electronics from scratch, build 6-15kW inverters, connect them into a self-organizing microgrid that powers your whole community—no utility required.

[![License: Open Source](https://img.shields.io/badge/License-Open%20Source-green.svg)](#license)
[![House: 6kW](https://img.shields.io/badge/House-6kW-blue.svg)](#level-3-house-detail)
[![Cluster: 24kW](https://img.shields.io/badge/Cluster-24kW-yellow.svg)](#level-2-cluster-internal-detail)
[![Swarm: 72kW](https://img.shields.io/badge/Swarm-72kW-orange.svg)](#level-1-tiered-bus-overview)

---

## Why Build This?

| Problem | This Project |
|---------|--------------|
| Commercial inverters cost $3,000+ each | Build for ~$775 per unit |
| Proprietary systems, no repair possible | Open source, fully repairable |
| Single inverter = single point of failure | Swarm shares load automatically |
| Grid dependency | Complete energy independence |
| Complex microgrid controllers | Self-organizing via droop control |

**End Goal:** 12-household swarm microgrid with 72kW capacity, tiered AC buses, automatic power sharing between clusters.

---

## System Architecture

The system uses a **tiered AC bus** design: each cluster has its own internal bus, and clusters connect through a separate swarm bus via gateways. This means only 3 gateways need to sync—not 12 inverters.

---

### Level 1: Tiered Bus Overview

```
┌──────────────────────────────────────────────────────────────────────────────────────┐
│                              SWARM BUS (1000m backbone)                              │
│                                                                                      │
│      ════════════════════════════════════════════════════════════════════════        │
│           │                         │                         │                      │
│      ┌────┴────┐               ┌────┴────┐               ┌────┴────┐                │
│      │GATEWAY A│               │GATEWAY B│               │GATEWAY C│                │
│      │Contactor│               │Contactor│               │Contactor│                │
│      │  ESP32  │               │  ESP32  │               │  ESP32  │                │
│      └────┬────┘               └────┬────┘               └────┬────┘                │
│           │                         │                         │                      │
│      ═════╪═════               ═════╪═════               ═════╪═════                │
│      CLUSTER A                 CLUSTER B                 CLUSTER C                  │
│      INTERNAL BUS              INTERNAL BUS              INTERNAL BUS               │
│      (80m)                     (80m)                     (80m)                       │
│           │                         │                         │                      │
│      ┌────┼────┐               ┌────┼────┐               ┌────┼────┐                │
│      │    │    │               │    │    │               │    │    │                │
│     H1   H2   H3  H4          H5   H6   H7  H8          H9  H10  H11 H12            │
│     INV  INV  INV INV         INV  INV  INV INV         INV  INV  INV INV           │
│                                                                                      │
│    ◄──────────────────────── 1000m total site ───────────────────────────►          │
│                                                                                      │
│    KEY BENEFIT: Only 3 gateways sync on swarm bus, not 12 inverters                 │
│                                                                                      │
│    TWO LEVELS OF DROOP:                                                             │
│    • Within cluster: 4 inverters share via droop on internal bus                    │
│    • Between clusters: 3 gateways share via droop on swarm bus                      │
│                                                                                      │
└──────────────────────────────────────────────────────────────────────────────────────┘
```

---

### Self-Reliance Levels

The system operates in 4 modes, from most isolated to most connected:

```
┌──────────────────────────────────────────────────────────────────────────────────────┐
│  LEVEL 1: HOUSE SELF-RELIANT (99% of time)                                          │
│                                                                                      │
│    House 1: [Solar] → [Battery] → [Inverter] → [Loads]                              │
│             Own production covers own consumption                                    │
│             No power flows to/from cluster bus                                       │
│                                                                                      │
├──────────────────────────────────────────────────────────────────────────────────────┤
│  LEVEL 2: CLUSTER SELF-RELIANT (swarm disconnected)                                 │
│                                                                                      │
│    Cluster A: H1 ←→ H2 ←→ H3 ←→ H4                                                  │
│               Sharing within cluster via droop                                       │
│               Gateway contactor: OPEN (isolated from swarm)                          │
│                                                                                      │
│    Use case: Swarm bus fault, or cluster has enough internal capacity               │
│                                                                                      │
├──────────────────────────────────────────────────────────────────────────────────────┤
│  LEVEL 3: CLUSTER NEEDS HELP (normal swarm operation)                               │
│                                                                                      │
│    Cluster A (depleted) ←── power ←── Cluster B (surplus)                           │
│                                             │                                        │
│    Gateway A closes contactor when:         │                                        │
│    • Cluster A frequency drops (heavy load) │                                        │
│    • Swarm frequency is higher (others OK)  │                                        │
│    • Frequencies match within ±0.5Hz        │                                        │
│                                                                                      │
├──────────────────────────────────────────────────────────────────────────────────────┤
│  LEVEL 4: HOUSE NEEDS HELP (within cluster)                                         │
│                                                                                      │
│    H1 (heavy load, e.g. EV) ←── power ←── H2, H3, H4 (light loads)                  │
│                                                                                      │
│    Via droop physics on cluster internal bus                                         │
│    No gateway involved - stays within cluster                                        │
│                                                                                      │
└──────────────────────────────────────────────────────────────────────────────────────┘
```

---

### Inter-Cluster Power Flow

How power moves from House 11 (Cluster C) to House 1 (Cluster A):

```
    H11 Inverter (50.0 Hz, light load)
            │
            ↓ power flows
    Cluster C Internal Bus
            │
            ↓
    Gateway C (contactor closed)
            │
            ↓
    ═══════ SWARM BUS ═════════════════════════════
            │
            ↓
    Gateway A (contactor closed)
            │
            ↓
    Cluster A Internal Bus (49.5 Hz, heavy load)
            │
            ↓ power flows
    H1 Inverter
            │
            ↓
    H1 Loads (kettle, EV, etc.)

    5 HOPS, PURE PHYSICS:
    No direct communication between H1 and H11.
    Power flows automatically from high frequency (surplus)
    to low frequency (deficit) via droop control.
```

---

### Gateway Sync Logic

Each gateway monitors both bus frequencies and decides when to connect:

```
    ┌─────────────────────────────────────────────────────────────────┐
    │  GATEWAY DECISION FLOWCHART                                     │
    │                                                                 │
    │  [Measure cluster frequency] → f_cluster                        │
    │  [Measure swarm frequency]   → f_swarm                          │
    │                                                                 │
    │  IF |f_cluster - f_swarm| < 0.5 Hz:                             │
    │      → CLOSE contactor (safe to connect)                        │
    │                                                                 │
    │  ELSE IF f_cluster < f_swarm - 0.5:                             │
    │      → Cluster needs power, wait for freq match                 │
    │                                                                 │
    │  ELSE IF f_cluster > f_swarm + 0.5:                             │
    │      → Cluster has surplus, wait for freq match                 │
    │                                                                 │
    │  IF f_cluster < 47.5 Hz:                                        │
    │      → EMERGENCY: cluster critically low                        │
    │      → Force close (accept brief surge)                         │
    │                                                                 │
    └─────────────────────────────────────────────────────────────────┘
```

---

### Level 2: Cluster Internal Detail

Each house is self-reliant with 6-15kW inverter. Load-balancing only kicks in for special scenarios.

```
┌──────────────────────────────────────────────────────────────────────────────────────┐
│                      CLUSTER - NORMAL OPERATION (99% of the time)                    │
│                                                                                      │
│   HOUSE 1                  HOUSE 2                  HOUSE 3                          │
│   ┌──────────────────┐     ┌──────────────────┐     ┌──────────────────┐            │
│   │ Solar 6kW        │     │ Solar 6kW        │     │ Solar 6kW        │            │
│   │ Bat 15kWh        │     │ Bat 15kWh        │     │ Bat 15kWh        │            │
│   │ Inv 6kW          │     │ Inv 6kW          │     │ Inv 6kW          │            │
│   │                  │     │                  │     │                  │            │
│   │ BATTERY 48V      │     │ BATTERY 48V      │     │ BATTERY 48V      │            │
│   │      │           │     │      │           │     │      │           │            │
│   │      ▼           │     │      ▼           │     │      ▼           │            │
│   │ INVERTER 50.0Hz  │     │ INVERTER 50.0Hz  │     │ INVERTER 50.0Hz  │            │
│   │      │           │     │      │           │     │      │           │            │
│   │      ▼           │     │      ▼           │     │      ▼           │            │
│   │ LOADS 2kW        │     │ LOADS 4kW        │     │ LOADS 1kW        │            │
│   │ (fridge,lights)  │     │ (oven,fridge)    │     │ (lights,TV)      │            │
│   └────────┼─────────┘     └────────┼─────────┘     └────────┼─────────┘            │
│            │                        │                        │                      │
│   ─────────┴────────────────────────┴────────────────────────┴───────────────►      │
│   230V AC BUS @ 50.0 Hz - NO POWER FLOWING BETWEEN HOUSES (each self-sufficient)    │
│   ──────────────────────────────────────────────────────────────────► to cluster    │
│                                                                                      │
│            │ RJ45                   │ RJ45                   │ RJ45                 │
│   ─────────○───────────────────────○───────────────────────○────────► to cluster   │
│   CAN BUS: sync timing + SOC% + status data                                         │
│                                                                                      │
│   NORMAL: Each 6kW inverter handles its own loads easily (oven=3kW, fridge=200W)    │
│                                                                                      │
└──────────────────────────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────────────────────────┐
│                  CLUSTER - SPECIAL SCENARIO: EV CHARGING EXCEEDS 6kW                 │
│                                                                                      │
│   HOUSE 1                  HOUSE 2 (needs help!)   HOUSE 3                          │
│   ┌──────────────────┐     ┌──────────────────┐     ┌──────────────────┐            │
│   │ Inv 6kW          │     │ Inv 6kW (MAXED)  │     │ Inv 6kW          │            │
│   │                  │     │                  │     │                  │            │
│   │ BATTERY 48V      │     │ BATTERY 48V      │     │ BATTERY 48V      │            │
│   │      │           │     │      │           │     │      │           │            │
│   │      ▼ 2kW       │     │      ▼ 6kW MAX   │     │      ▼ 1.5kW     │            │
│   │ INVERTER 50.0Hz  │     │ INVERTER 49.7Hz  │     │ INVERTER 49.9Hz  │            │
│   │      │           │     │      │           │     │      │           │            │
│   │      ▼           │     │      ▼           │     │      ▼           │            │
│   │ LOADS 1kW        │     │ LOADS 7kW !!!    │     │ LOADS 1kW        │            │
│   │                  │     │ (EV charger)     │     │                  │            │
│   └────────┼─────────┘     └────────┼─────────┘     └────────┼─────────┘            │
│            │                        │                        │                      │
│   ─────────┴────────────────────────┴────────────────────────┴───────────────►      │
│                                                                                      │
│            ════1.0kW═══════════════►║◄════════0.5kW══════════                       │
│                    House 1 pushes   ║   House 3 pushes                              │
│                                                                                      │
│   230V AC BUS @ 49.85 Hz - POWER FLOWS TO HOUSE 2 (droop triggered)                 │
│   ──────────────────────────────────────────────────────────────────► to cluster    │
│                                                                                      │
├──────────────────────────────────────────────────────────────────────────────────────┤
│   WHY LOAD-BALANCING KICKS IN:                                                      │
│                                                                                      │
│   House 2 plugs in EV charger (7kW) but inverter maxes at 6kW:                       │
│   • Inverter strains → frequency drops to 49.7 Hz                                   │
│   • House 1 & 3 see low freq → push power to bus                                    │
│   • House 2 gets: 6kW local + 1kW from H1 + 0.5kW from H3 = 7.5kW available         │
│                                                                                      │
│   OTHER SCENARIOS THAT TRIGGER SHARING:                                             │
│   • Low battery (BMS limits output) → neighbors supply deficit                      │
│   • Inverter fault/maintenance → house runs entirely from bus                       │
│   • Motor startup surge (9kW for 2 sec) → cluster absorbs spike                     │
│                                                                                      │
└──────────────────────────────────────────────────────────────────────────────────────┘
```

---

### Level 3: House Detail

Internal wiring of one house, showing connection to cluster AC bus.

```
┌────────────────────────────────────────────────────────────────────────────────────┐
│                              HOUSE (typical)                                       │
│                                                                                    │
│   ROOF                                                                             │
│   ┌──────────────────┐                                                             │
│   │   Solar Panels   │                                                             │
│   │      3kW         │                                                             │
│   └────────┬─────────┘                                                             │
│            │ DC (Vmp ~100V)                                                        │
│            ▼                                                                       │
│   ┌──────────────────┐                                                             │
│   │   MPPT Charger   │                                                             │
│   │  (Libre Solar)   │                                                             │
│   └────────┬─────────┘                                                             │
│            │ 48V DC                                                                │
│            ▼                                                                       │
│   ┌──────────────────┐                                                             │
│   │       BMS        │   48V DC stays inside this house                            │
│   │   48V / 15kWh    │   (no DC connection to other houses)                        │
│   │    LiFePO4       │                                                             │
│   └────────┬─────────┘                                                             │
│            │ 48V DC                                                                │
│            ▼                                                                       │
│   ┌──────────────────┐              ┌──────────────────────────────────────────┐   │
│   │    INVERTER      │     RJ45     │                                          │   │
│   │      6kW         │◄────────────►│  CAN BUS                                 │   │
│   │    EG8010        │              │  ◄───── to/from other houses + clusters  │   │
│   │    ESP32         │              └──────────────────────────────────────────┘   │
│   │    MCP2515       │                                                             │
│   │  (MASTER/SLAVE)  │                                                             │
│   └────────┬─────────┘                                                             │
│            │ 230V AC                                                               │
│            ▼                                                                       │
│   ┌──────────────────┐              ┌──────────────────────────────────────────┐   │
│   │   MAIN BREAKER   │    63A MCB   │                                          │   │
│   │   (house entry)  │◄────────────►│  230V AC CLUSTER BUS                     │   │
│   └────────┬─────────┘              │  ◄───── to/from other houses + clusters  │   │
│            │                        └──────────────────────────────────────────┘   │
│            │                                                                       │
│   ═════════╧═══════════════════════════════════════════════════════════            │
│   ║            230V AC HOUSE DISTRIBUTION BOARD                    ║               │
│   ══════╤══════════╤══════════╤══════════╤══════════╤══════════════                │
│         │          │          │          │          │                              │
│      ┌──┴──┐    ┌──┴──┐    ┌──┴──┐    ┌──┴──┐    ┌──┴──┐                          │
│      │ 6A  │    │ 10A │    │ 16A │    │ 16A │    │ 20A │                          │
│      │Light│    │Fridge│   │Plugs│    │Plugs│    │Oven │                          │
│      │ MCB │    │ MCB │    │ MCB │    │ MCB │    │ MCB │                          │
│      └──┬──┘    └──┬──┘    └──┬──┘    └──┬──┘    └──┬──┘                          │
│         │          │          │          │          │                              │
│         ▼          ▼          ▼          ▼          ▼                              │
│      Lights     Fridge     Kitchen    Living     Oven                              │
│                            plugs      plugs                                        │
│                                                                                    │
│   CONNECTION DETAIL: How House Connects to Cluster                                 │
│   ┌────────────────────────────────────────────────────────────────────────────┐   │
│   │                                                                            │   │
│   │    230V AC CLUSTER BUS (shared with other houses)                          │   │
│   │    ═══════════════╤═══════════════════════════════════════                 │   │
│   │                   │                                                        │   │
│   │              ┌────┴────┐                                                   │   │
│   │              │  63A    │  Main breaker: isolates house from cluster        │   │
│   │              │ BREAKER │  if fault or maintenance                          │   │
│   │              └────┬────┘                                                   │   │
│   │                   │                                                        │   │
│   │    ═══════════════╧═══════════════════════════════════════                 │   │
│   │    230V AC HOUSE DISTRIBUTION (internal to this house)                     │   │
│   │                                                                            │   │
│   │    Power flows BOTH directions:                                            │   │
│   │    • House produces excess → pushes to cluster bus → other houses use it   │   │
│   │    • House needs more → pulls from cluster bus → other houses supply it    │   │
│   │                                                                            │   │
│   │    ONLY TWO CONNECTIONS LEAVE THIS HOUSE:                                  │   │
│   │    1. 230V AC (L + N + PE) → to cluster AC bus                             │   │
│   │    2. CAN bus (RJ45) → to other houses for communication                   │   │
│   │                                                                            │   │
│   │    48V DC stays entirely inside the house (battery is isolated)            │   │
│   │                                                                            │   │
│   └────────────────────────────────────────────────────────────────────────────┘   │
│                                                                                    │
└────────────────────────────────────────────────────────────────────────────────────┘
```

---

### Hardware Summary

**Per House (isolated 48V DC system):**
| Component | Description |
|-----------|-------------|
| Solar Panels | 3kW array (size varies) |
| MPPT Charger | Libre Solar MPPT-2420-HC or similar |
| BMS + Battery | Libre Solar BMS-C1 + 48V/15kWh LiFePO4 (stays inside house) |
| Inverter | OzInverter 6kW with EG8010 + ESP32 + MCP2515 |
| Main Breaker | 63A MCB, isolates house from cluster AC bus |
| RJ45 Jack | CAN bus connection to other houses |

**Per Cluster (shared between houses):**
| Component | Description |
|-----------|-------------|
| 230V AC Bus | 4mm² cable connecting all house inverter outputs (10-30m) |
| CAN Bus | RJ45 cables daisy-chaining all houses (10-30m runs) |

**Per Gateway (connects cluster to swarm bus):**
| Component | Description |
|-----------|-------------|
| Gateway ESP32 | Monitors both bus frequencies, controls contactor |
| Contactor 63A | Connects/disconnects cluster from swarm bus |
| Voltage sensors | Measures cluster and swarm bus frequencies |

**Swarm Bus (between clusters):**
| Component | Description |
|-----------|-------------|
| 5-Wire Cable | L + N + PE (4mm²) + CAN-H + CAN-L (0.5mm²) |
| Distance | 200-400m between gateways |

---

### How It Works

**Power Flow (tiered):**
1. Sun → Solar panels → MPPT → Battery (48V DC, inside house)
2. Battery → Inverter → 230V AC → House loads
3. Excess → Cluster internal bus → Other houses in cluster (via droop)
4. Still excess → Gateway → Swarm bus → Other clusters (via droop)

**No DC Bus Between Houses:**
- Each house has isolated 48V battery (simpler, cheaper wiring)
- Power sharing happens via 230V AC (thinner cables for same power)
- Battery balancing is indirect: low battery house draws from AC, others supply

**Communication:**
1. ESP32 master broadcasts sync timing @ 100Hz on CAN bus
2. All slave ESP32s adjust their EG8010 phase to match
3. Status messages (SOC%, load, faults) shared via ThingSet protocol

**Droop Control (automatic power sharing):**
- Heavy load → inverter frequency drops slightly (e.g., 50.0 → 49.8 Hz)
- Other inverters sense lower frequency → push more power
- No central controller needed - physics handles balancing

```
How Droop Control Shares Power Between Houses (EV Charging Example):

  House 2 (charging EV at 7kW)           House 1 (light load - 1kW)
  ┌─────────────────────────────┐        ┌─────────────────────────────┐
  │  6kW inverter (maxed out)   │        │  6kW inverter (idle)        │
  │                             │        │                             │
  │  Battery ──► Inverter ──┬──►│ EV     │  Battery ──► Inverter ──┬──►│ load
  │    48V        6kW       │   │ 7kW    │    48V        2kW       │   │ 1kW
  │              49.7 Hz    │   │        │              50.0 Hz    │   │
  └─────────────────────────┼───┘        └─────────────────────────┼───┘
                            │                                      │
                ◄═══════════╧══════════════════════════════════════╧═══
                230V AC BUS - House 1 pushes 2kW → House 2 (freq settles at 49.85 Hz)

  WHY THIS TRIGGERS SHARING (with 6kW inverters):
  • House 2 needs 7kW but its inverter maxes out at 6kW
  • Inverter strains → frequency drops to 49.7 Hz
  • House 1 sees low freq → pushes power to help

  1. House 2 plugs in EV charger (7kW) → inverter hits 6kW limit → freq drops
  2. House 1 inverter sees 49.7 Hz → increases output → pushes power to bus
  3. Power flows: House 1 battery → inverter → AC bus → House 2 EV
  4. System settles at 49.85 Hz with 7kW delivered

  The math for House 2:
  • EV charger needs:         7.0 kW
  • Local 6kW inverter maxed: 6.0 kW (can't do more)
  • AC bus supplies:          1.0 kW (from House 1)
  • House 1 also powers own:  1.0 kW load
  • House 1 total output:     2.0 kW (1kW local + 1kW to bus)

  Key: Single inverter can't do 7kW, but cluster of two 6kW inverters can.
       Droop control automatically shares the load - no manual switching.
```

---

### When AC Bus Power Sharing Kicks In

With 6-15kW inverters, normal household loads won't trigger sharing. The real value is resilience:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  SCENARIO 1: Low Battery Protection                                        │
│                                                                             │
│  House 2: SOC at 15% → BMS limits discharge to 1kW                          │
│  House 2 load: 3kW (fridge + pump + lights)                                 │
│  ───────────────────────────────────────────────────────────────            │
│  House 1 & 3 push 2kW through AC bus → House 2 stays powered                │
│                                                                             │
├─────────────────────────────────────────────────────────────────────────────┤
│  SCENARIO 2: Inverter Failure / Maintenance                                 │
│                                                                             │
│  House 2: Inverter offline (fault or repair)                                │
│  House 2 loads: still connected to AC bus via breaker                       │
│  ───────────────────────────────────────────────────────────────            │
│  House 1 & 3 supply House 2 entirely → no blackout during repair            │
│                                                                             │
├─────────────────────────────────────────────────────────────────────────────┤
│  SCENARIO 3: Uneven Solar (shade, orientation, dust)                        │
│                                                                             │
│  Morning: House 1 (east panels) has full sun, House 3 (west) has none       │
│  Evening: reversed                                                          │
│  ───────────────────────────────────────────────────────────────            │
│  Power flows from sunny house → saves battery cycles across cluster         │
│                                                                             │
├─────────────────────────────────────────────────────────────────────────────┤
│  SCENARIO 4: Big Transient Loads (motor startup, welder)                    │
│                                                                             │
│  House 2: Starts 3kW water pump (startup surge 9kW for 2 seconds)           │
│  Single 6kW inverter might trip on overcurrent                              │
│  ───────────────────────────────────────────────────────────────            │
│  AC bus absorbs surge → all three inverters share the spike                 │
│                                                                             │
├─────────────────────────────────────────────────────────────────────────────┤
│  SCENARIO 5: EV Charging / Heavy Scheduled Loads                            │
│                                                                             │
│  House 2 wants to charge EV at 7kW                                          │
│  Their 6kW inverter can't do it alone                                       │
│  ───────────────────────────────────────────────────────────────            │
│  Neighbor's excess solar flows through AC bus → 7kW charging possible       │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

The AC bus is **community resilience** - nobody goes dark because of one failure.

---

### Swarm Capacity at Scale

What happens when you connect more houses and clusters:

**Per house baseline:**
- Inverter: 6-15 kW
- Solar: 6 kW
- Battery: 15 kWh

| Scale | Houses | Inverter Capacity | Solar | Battery Storage |
|-------|--------|-------------------|-------|-----------------|
| Single house | 1 | 6-15 kW | 6 kW | 15 kWh |
| **Cluster (4 houses)** | 4 | 24-60 kW | 24 kW | 60 kWh |
| **Swarm (12 houses)** | 12 | 72-180 kW | 72 kW | 180 kWh |

**What 72-180 kW powers simultaneously:**

```
72 kW (minimum swarm config):
├── 12× electric ovens (3kW each)
├── 3× EV chargers (7kW each)
├── 24× washing machines running at once
├── or one small workshop (welders + compressors)

180 kW (maximum swarm config):
├── Small factory level power
├── Community event (PA, lights, food trucks)
├── Emergency shelter for 50+ people
```

**Battery perspective (15 kWh per house):**

```
Typical day:
  Night consumption (16h): ~5 kWh
  15 kWh battery = 3 nights reserve

Cloudy winter day:
  Solar produces: ~2 kWh (vs 6 kWh normal)
  Consumption: 10 kWh
  Deficit: 8 kWh → covered by battery, still 7 kWh left

3 consecutive cloudy days:
  Deficit: 8 kWh × 3 = 24 kWh per house
  15 kWh battery alone: NOT ENOUGH
  But cluster sharing helps: sunny neighbor covers your deficit

180 kWh swarm total:
  Community event (5 hours × 20 kW): 100 kWh → still 80 kWh reserve
  Winter evening peak (all cooking): 40 kW for 2h = 80 kWh → covered
```

The swarm becomes a **small village grid** - completely independent from utility.

---

### Connecting High-Power Loads (>6kW)

Single house inverter maxes at 6kW. For larger loads, connect directly to cluster or swarm bus.

**Where to connect based on load size:**

| Load Power | Connect To | Why |
|------------|------------|-----|
| <6kW | House breaker | Single inverter handles it |
| 6-24kW | Cluster bus | 4 inverters share (24kW max) |
| 24-72kW | Swarm bus | All 12 inverters share |

**Connection point for high-power equipment:**

```
CLUSTER BUS ════════════════════════════════════════
     │           │           │           │
   House1      House2      House3      HIGH-POWER
   6kW inv     6kW inv     6kW inv     EQUIPMENT
                                       (connects here,
                                        not through house)
```

---

### Energy Accumulation Through Work

**Key insight:** On sunny days, use surplus power for work that stores energy in other forms.

Instead of filling batteries (limited capacity), convert solar abundance into:
- **Processed materials** (wood chips, grain, crushed stone)
- **Pumped water** (irrigation tanks, elevated storage)
- **Compressed air** (workshop tanks)
- **Heated mass** (water tanks, thermal storage)
- **Charged vehicles** (EVs, e-bikes, tractors)

**Seasonal high-power tasks (best done on sunny days):**

```
┌─────────────────────────────────────────────────────────────────────────┐
│  TASK                      │ POWER  │ CONNECT TO  │ ENERGY STORED AS   │
├────────────────────────────┼────────┼─────────────┼────────────────────┤
│  Wood chipper              │ 15-25kW│ Cluster bus │ Processed firewood │
│  Grain mill                │ 5-15kW │ Cluster bus │ Flour/feed         │
│  Water pump to hill tank   │ 3-10kW │ House/Clust │ Potential energy   │
│  Air compressor (big tank) │ 5-10kW │ Cluster bus │ Compressed air     │
│  EV fast charging          │ 7-22kW │ Cluster bus │ Vehicle range      │
│  Electric cement mixer     │ 2-5kW  │ House       │ Built structure    │
│  Welding (construction)    │ 5-15kW │ Cluster bus │ Built structure    │
│  Water heater (big tank)   │ 3-6kW  │ House       │ Hot water          │
│  Freezer loading (harvest) │ 1-3kW  │ House       │ Preserved food     │
└─────────────────────────────────────────────────────────────────────────┘
```

**Example: Wood chipping day**

```
Sunny summer day, 11:00-15:00 (peak solar)

Solar production:    ~50kW (not all panels at perfect angle)
Baseline house loads: -12kW
Wood chipper:        -20kW
─────────────────────────────────────
Net:                 +18kW → batteries charging

Result: 4 hours × 20kW = 80kWh of work
        = 4 cubic meters of wood chips
        = heating fuel for 2 weeks of winter

The sun's energy is now stored as processed firewood,
not limited by battery capacity.
```

**Scheduling principle:**
- Heavy processing → sunny midday
- Light tasks → morning/evening
- Storage tasks (freezing, heating water) → whenever surplus
- Construction/welding → plan around weather forecast

This turns the microgrid into a **productive community tool**, not just household power.

---

### Monitoring Layer (Home Assistant)

![Swarm Microgrid Dashboard](https://enklava.co/files/1963eB3rpV7vEEp1zKn49w/Generated%20Image%20December%2026,%202025%20-%2012_10AM.jpeg)
*Home Assistant dashboard concept: Swarm grid monitoring showing clusters, power flow, and cell health*

The microgrid has two control layers with different responsibilities:

| Layer | Tool | Response Time | Purpose |
|-------|------|---------------|---------|
| **Hard** | Inverter/BMS firmware | <1ms | Droop control, protection, contactor |
| **Soft** | Home Assistant | seconds | Dashboards, logging, alerts, analysis |

**Keep in firmware (hard layer):**
- Droop frequency control
- Protection disconnects
- Contactor decisions
- Cell balancing

**Good for Home Assistant (soft layer):**
- SOC monitoring across all 12 houses
- Energy production/consumption dashboards
- Cell imbalance alerts
- Historical data and trends
- AI-assisted failure analysis

**Integration points:**

| Device | HA Integration |
|--------|----------------|
| NEEY balancer | Bluetooth via ESPHome |
| JK BMS | [esphome-jk-bms](https://github.com/syssi/esphome-jk-bms) |
| ENNOID BMS | CAN → ESP32 → HA |
| Inverter ESP32 | MQTT / CAN → WiFi bridge |

**AI-assisted configuration:**
[HA Vibecode Agent](https://github.com/Coolver/home-assistant-vibecode-agent) enables natural language Home Assistant configuration via Claude Code MCP integration.

**Deep logging and debugging:**
See [LOGGING.md](LOGGING.md) for the 3-tier logging architecture with local house logs, cluster aggregation, and AI-powered failure analysis.

---

## Practical Droop Control Experiments

These hands-on experiments let you discover how droop control works. Each builds on the previous.

### Equipment Needed

| Item | Purpose | Cost |
|------|---------|------|
| Frequency counter or multimeter with Hz | Measure bus frequency | $20-50 |
| Clamp ammeter (AC) | Measure individual currents | $30-50 |
| Kill-A-Watt or power meter | Measure load power | $20-30 |
| Oscilloscope (optional) | See waveforms, phase | $50-300 |
| Variable load (heaters, bulbs) | Create controlled loads | $50-100 |
| 10Ω 100W power resistors | Current limiting for safety | $10-20 |

---

### Experiment 1: Single Inverter Frequency vs Load

**Goal:** Verify YOUR inverter drops frequency under load (droop characterization)

```
SETUP:
                    ┌─────────────┐
   BATTERY ────────►│  INVERTER   │────────► VARIABLE LOAD
   48V              │  (your DIY) │          (light bulbs/heater)
                    └──────┬──────┘
                           │
                    ┌──────┴──────┐
                    │ FREQUENCY   │
                    │ METER       │
                    └─────────────┘

PROCEDURE:
1. No load: Record frequency = _______ Hz (should be ~50.0)
2. Add 500W: Record frequency = _______ Hz
3. Add 1000W: Record frequency = _______ Hz
4. Continue to 3kW: Record frequency = _______ Hz

EXPECTED (with 3% droop, k=0.25 Hz/kW):
  Load        Frequency
  0W          50.00 Hz
  500W        49.88 Hz
  1000W       49.75 Hz
  2000W       49.50 Hz
  3000W       49.25 Hz

WHAT YOU LEARN:
  ✓ Your droop coefficient matches your code
  ✓ The relationship is linear
  ✓ Frequency = system load indicator
```

---

### Experiment 2: Two Inverters, First Parallel Connection

**Goal:** Watch two inverters find common frequency automatically

```
SETUP:
                         SWITCH (open initially)
                              ○
   INVERTER A ───────────────╱─────────────── INVERTER B
       │                                           │
       ▼                                           ▼
   FREQ METER A                               FREQ METER B

   LOAD A: 1kW                                LOAD B: 0kW

PROCEDURE:
1. Start both inverters separately (switch OPEN)
2. Measure: Freq A = _____ Hz, Freq B = _____ Hz
   (A should be ~49.75 Hz, B should be ~50.0 Hz)

3. CLOSE THE SWITCH (connect them together)

4. Watch frequencies converge!
   After 1 sec: Freq A = _____, Freq B = _____
   After 5 sec: Both should be ~49.87 Hz

5. Measure current in connecting wire: _____ A
   This current IS the power transfer!

EXPECTED:
  Before: A=49.75 Hz (loaded), B=50.0 Hz (unloaded)
  After:  Both≈49.87 Hz, each supplies ~0.5kW

WHAT YOU LEARN:
  ✓ Inverters find common frequency automatically
  ✓ Load balances without commands
  ✓ You SEE droop control working!
```

---

### Experiment 3: Unequal Loading, Equal Sharing

**Goal:** Prove droop shares load equally regardless of WHERE load connects

```
SETUP:
   INVERTER A ══════════════════════════ INVERTER B
       │                                      │
       ▼                                      ▼
   AMMETER A                              AMMETER B
       │                                      │
       └──────────────┬───────────────────────┘
                      │
                      ▼
                  LOAD (2kW)
                  (connected to A's side ONLY)

PROCEDURE:
1. Connect 2kW load to Inverter A's terminals only
2. Measure:
   - Current from A: _____ A
   - Current from B: _____ A
   - Bus frequency: _____ Hz

EXPECTED:
  Current A: ~4.3A (≈1kW)
  Current B: ~4.3A (≈1kW)
  Frequency: ~49.75 Hz

  Even though load is physically at A,
  BOTH inverters supply equal power!

WHAT YOU LEARN:
  ✓ Load location doesn't matter
  ✓ Droop balances automatically
  ✓ No complex load management needed
```

---

### Experiment 4: Droop Mismatch (What NOT to Do)

**Goal:** Experience instability when droop coefficients don't match

```
SETUP:
   INVERTER A ═══════════════════ INVERTER B
   k = 0.25 Hz/kW                 k = 0.5 Hz/kW (DIFFERENT!)

   Shared load: 2kW

PROCEDURE:
1. Set different droop coefficients
2. Connect together with shared load
3. Watch/listen for oscillation:
   - Frequency hunting (±0.2 Hz swings)
   - Audible hum changing pitch
   - Current swinging back and forth

4. Change B's droop to match A's (k = 0.25)
5. Oscillation should stop

EXPECTED:
  Mismatched: Unstable, hunting, buzzing
  Matched: Stable, quiet, steady

WHAT YOU LEARN:
  ✓ ALL inverters must have IDENTICAL droop
  ✓ Mismatch causes fighting
  ✓ Calibration is critical
```

---

### Experiment 5: Phase Mismatch Danger (LOW POWER ONLY!)

**Goal:** Understand why sync matters - observe circulating current

```
⚠️ WARNING: Use current-limiting resistor! Can damage inverters!

SETUP:
   INVERTER A ────[10Ω RESISTOR]──── INVERTER B
   (50.0 Hz)      (limits current!)   (50.0 Hz)
       │                                  │
   Manually set                       Keep at
   to 50.1 Hz                         50.0 Hz

PROCEDURE:
1. Set A to 50.1 Hz, B to 50.0 Hz (0.1 Hz difference)
2. Connect through 10Ω power resistor
3. Measure current through resistor: _____ A

4. Increase A to 50.2 Hz: Current = _____ A
5. Increase A to 50.5 Hz: Current = _____ A

EXPECTED:
  0.1 Hz diff: ~0.5A circulating
  0.2 Hz diff: ~1.0A circulating
  0.5 Hz diff: ~2-3A circulating

WHAT YOU LEARN:
  ✓ Frequency difference = circulating current
  ✓ Without resistor, current would be DESTRUCTIVE
  ✓ Why CAN sync matters for tight parallel
```

---

### Experiment 6: Four Inverter Cluster

**Goal:** Verify full cluster load sharing works

```
SETUP:
   INV1 ═══╤═══ INV2
          │
   BUS ═══╪═══════════════
          │
   INV3 ═══╧═══ INV4

   Variable loads (heaters, bulbs)

MEASUREMENT TABLE:

Test   Total    Bus      I₁     I₂     I₃     I₄     Equal?
       Load     Freq
────   ─────    ────     ──     ──     ──     ──     ──────
1      0kW      ____     __     __     __     __
2      4kW      ____     __     __     __     __     ~1kW each?
3      8kW      ____     __     __     __     __     ~2kW each?
4      12kW     ____     __     __     __     __     ~3kW each?
5*     8kW      ____     __     __     __     __     Still equal?

* Test 5: All 8kW load connected to INV1's terminals only

EXPECTED:
  All currents approximately equal in EVERY test
  Bus frequency drops as total load increases

WHAT YOU LEARN:
  ✓ 4-way sharing works
  ✓ Load location irrelevant
  ✓ System scales predictably
```

---

### Experiment 7: Master Failure & Recovery

**Goal:** Test that system survives master ESP32 failure

```
SETUP:
   INV1 (MASTER) ═══════ INV2 (SLAVE)
        │                    │
   CAN bus connecting all    │
        │                    │
   INV3 (SLAVE) ═══════ INV4 (SLAVE)

   Shared load: 8kW running

PROCEDURE:
1. System running, INV1 is master, 8kW load
2. Note: Each inverter supplies ~2kW
3. UNPLUG INV1's POWER (simulate failure)
4. Start timer - measure:
   - Lights flicker? How long? _____ ms
   - Do INV2/3/4 keep running? Y/N
   - Does new master emerge? Y/N
   - New steady state: ___ kW each (should be ~2.67kW)
5. Time to stable operation: _____ ms

EXPECTED:
  - Brief disturbance (<200ms)
  - One of INV2/3/4 becomes master
  - System continues at reduced capacity
  - Load still served (now 2.67kW each)

WHAT YOU LEARN:
  ✓ Master election works
  ✓ No single point of failure
  ✓ Graceful degradation
```

---

### Experiment 8: 80m Cable Power Transfer

**Goal:** Measure actual droop sharing over realistic distance

```
SETUP:
   INVERTER A ═══[80m cable]═══ INVERTER B
   House 1                      House 2
       │                            │
   No load                      3kW load

PROCEDURE:
1. INV B has 3kW load, INV A has no load
2. Start both, let system stabilize
3. Measure:
   - Bus frequency: _____ Hz
   - Current from A: _____ A (power to bus)
   - Current from B: _____ A (power to load)
   - Voltage at A: _____ V
   - Voltage at B: _____ V (should be slightly lower)

4. Calculate:
   - Power from A: _____ W
   - Power from B: _____ W
   - Voltage drop in cable: _____ V
   - Cable loss: _____ W

EXPECTED (4mm² cable, 80m):
  Frequency: ~49.63 Hz
  A supplies: ~1.5kW
  B supplies: ~1.5kW
  Voltage drop: ~5-10V (2-4%)
  Cable loss: ~50-100W (acceptable)

WHAT YOU LEARN:
  ✓ Real-world sharing works
  ✓ Cable loss is measurable but acceptable
  ✓ Distance doesn't prevent sharing
```

---

### Calibration Procedure (Required Before Parallel Operation)

**Every inverter must have matched droop coefficient:**

```
CALIBRATION STEPS:

1. REFERENCE INVERTER (INV1):
   - Apply exactly 3kW load
   - Measure frequency: _____ Hz
   - Calculate actual k: k = (50.0 - f) / 3.0 = _____ Hz/kW

2. TEST INVERTER (INV2):
   - Apply exactly 3kW load
   - Measure frequency: _____ Hz
   - If different from INV1, adjust code:

     // Adjust this value until frequencies match
     #define DROOP_K  0.25  // Change until INV2 matches INV1

3. VERIFICATION:
   - Both at 3kW: Freq should be within ±0.02 Hz
   - Connect together: Should be stable, no oscillation

4. REPEAT for INV3, INV4, etc.

RECORD YOUR CALIBRATION:
  INV1: k = _____ Hz/kW (reference)
  INV2: k = _____ Hz/kW (adjusted to match)
  INV3: k = _____ Hz/kW (adjusted to match)
  INV4: k = _____ Hz/kW (adjusted to match)
```

---

### The Droop Control Formula

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│   f = 50.0 - (k × P)                                            │
│                                                                 │
│   f = output frequency (Hz)                                     │
│   k = droop coefficient (recommend 0.25 Hz/kW for DIY)          │
│   P = power output (kW)                                         │
│                                                                 │
│   Example with k = 0.25:                                        │
│   ─────────────────────                                         │
│   0kW  load → 50.00 Hz                                          │
│   2kW  load → 49.50 Hz                                          │
│   4kW  load → 49.00 Hz                                          │
│   6kW  load → 48.50 Hz                                          │
│                                                                 │
│   Lower frequency = more loaded = receives power from others    │
│   Higher frequency = less loaded = sends power to others        │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Learning Path

```
Part 1       Part 2        Part 3         Part 4          Parts 5-7
─────────────────────────────────────────────────────────────────────►

┌────────┐  ┌────────┐  ┌──────────┐  ┌───────────┐  ┌─────────────┐
│  LED   │  │ Safety │  │  500W    │  │  1kW →    │  │  Smart +    │
│Blinker │─►│   +    │─►│ Modified │─►│  2.5kW →  │─►│   Swarm     │
│26 proj │  │ MOSFET │  │ + Winding│  │  6-15kW   │  │  Microgrid  │
└────────┘  └────────┘  └──────────┘  └───────────┘  └─────────────┘
   9V         12V/20W      220V        OzInverter      4 households
```

**Power Progression (no dangerous gaps):**
```
9V → 12V/20W → 150W → 500W → 1kW → 2.5kW → 6-15kW → 20kW
```

---

## What You'll Build

| Part | Project | Power | Key Skills |
|:----:|---------|:-----:|------------|
| 1 | 26 fundamental projects | 9V | Soldering, Ohm's Law, transistors, 555, PWM |
| 2 | Safety + MOSFET Bridge | 12V | HV safety, first power MOSFETs |
| 3 | Modified Sine Inverter | 500W | H-bridge, IR2110, dead-time, winding |
| 4 | Pure Sine + OzInverter | 6-15kW | EG8010, MOSFET paralleling, thermal |
| 5 | Smart Upgrade | — | ESP32, CAN bus, Home Assistant |
| 6 | Swarm Microgrid | 4 houses | Droop control, power sharing |
| 7 | Libre Solar Integration | — | Open MPPT, BMS, ThingSet |

**Key Innovation:** 2.5kW intermediate build lets you practice MOSFET paralleling with $30 mistakes instead of $150 mistakes.

---

## Investment

| Milestone | Time | Cost |
|-----------|:----:|-----:|
| Single 6kW Inverter | 6-9 months | ~$1,230 |
| 20kW Parallel System | 12-18 months | ~$1,730 |
| 4-Household Microgrid | 18-24 months | ~$14,300 |

**Per household:** ~$3,575 (vs $10,000+ commercial equivalent)

**Savings:** 60-75% compared to Victron/SMA systems

---

## Repository Structure

```
curriculum/
├── part1_fundamentals.md              # 26 projects at safe 9V
├── part2_safety_and_mosfet_bridge.md  # HV safety + first MOSFETs
├── part3_modified_sine.md             # 150W → 500W + winding practice
├── part4_pure_sine.md                 # 1kW → 2.5kW → 6-15kW OzInverter
├── part5_smart_upgrade.md             # ESP32 + Home Assistant
├── part6_microgrid.md                 # Swarm architecture
└── part7_libre_solar.md               # Open source MPPT & BMS
```

---

## Quick Start

1. **[Part 1](curriculum/part1_fundamentals.md)** — Build 26 fundamental projects at safe 9V
2. **[Part 2](curriculum/part2_safety_and_mosfet_bridge.md)** — Learn HV safety + build first MOSFET circuit
3. **[Part 3](curriculum/part3_modified_sine.md)** — Build modified sine inverters (150W → 500W)
4. **[Part 4](curriculum/part4_pure_sine.md)** — Pure sine: 1kW → 2.5kW → 6-15kW OzInverter
5. **[Parts 5-7](curriculum/part5_smart_upgrade.md)** — Smart monitoring, microgrid, Libre Solar

---

## Safety

> ⚠️ **230V AC can kill. 48V DC can cause fires. Never work on live circuits.**

**Part 2 is mandatory** before any 220V work. It covers:
- Electrical safety fundamentals (30mA through heart = death)
- One-hand rule and safe work practices
- Capacitor discharge procedures
- Emergency response
- Safety quiz (must pass)

The curriculum uses incremental voltage progression: **9V → 12V → 220V**

---

## Hardware References

| Component | Source | Description |
|-----------|--------|-------------|
| OzInverter | [bryanhorology.com](https://bryanhorology.com/ozinverter.php) | 6kW+ inverter design |
| BMS-C1 | [Libre Solar](https://libre.solar) | 16S/100A battery management |
| MPPT-2420-HC | [Libre Solar](https://libre.solar) | 20A solar charge controller |
| ThingSet | [thingset.io](https://thingset.io) | Open communication protocol |
| HA Vibecode Agent | [GitHub](https://github.com/Coolver/home-assistant-vibecode-agent) | AI-assisted HA configuration |
| ESPHome JK-BMS | [GitHub](https://github.com/syssi/esphome-jk-bms) | JK BMS integration for HA |

---

## Why Swarm Architecture?

| Traditional | Swarm (Tiered Bus) |
|-------------|-------------------|
| Central controller (SPOF) | No central controller |
| All inverters must sync | Only 3 gateways sync on backbone |
| Complex wiring | Cluster isolation via contactors |
| Manual programming | Self-organizing via droop control |
| Proprietary protocols | Open CAN bus / ThingSet |

Each cluster operates independently most of the time. Gateways connect clusters only when power sharing is needed—like a flock of birds, no leader needed.

---

## License

Open source. Build for yourself, your community, or teach others.

---

<p align="center">
<strong>From zero electronics to community energy independence.</strong><br>
<em>Build it. Connect it. Power your village.</em>
</p>
