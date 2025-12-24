# Inverter Building Curriculum

**From LED Blinker to Swarm-Networked Community Microgrid**

Build your own 6-15kW pure sine wave inverters that communicate, share power automatically, and create resilient off-grid communities.

[![License: Open Source](https://img.shields.io/badge/License-Open%20Source-green.svg)](#license)
[![Parts: 7](https://img.shields.io/badge/Parts-7-blue.svg)](#learning-path)
[![Power: 20kW](https://img.shields.io/badge/Max%20Power-20kW-orange.svg)](#what-youll-build)

---

## Why This Curriculum?

| Problem | Solution |
|---------|----------|
| Commercial inverters cost $3,000+ each | Build for ~$775 per unit |
| Proprietary systems, no repair possible | Open source, fully repairable |
| Single inverter = single point of failure | Swarm of inverters = resilient |
| Grid dependency | Complete energy independence |

**End Goal:** 9-household microgrid with 54kW capacity, automatic power sharing, no utility grid required.

---

## System Architecture

The system has three levels of detail. Start with the big picture, then zoom in.

---

### Level 1: Microgrid Overview

How clusters connect to form a community microgrid (linear topology for long sites).

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                           COMMUNITY MICROGRID                                       │
│                           (800m linear site example)                                │
│                                                                                     │
│   ┌───────────────┐   5-WIRE CABLE   ┌───────────────┐   5-WIRE CABLE   ┌─────────────────┐
│   │               │   (200-400m)     │               │   (200-400m)     │                 │
│   │   CLUSTER A   │◄════════════════►│   CLUSTER B   │◄════════════════►│   CLUSTER C     │
│   │               │  L+N+PE+CAN      │               │  L+N+PE+CAN      │                 │
│   │  4 houses     │                  │  4 houses     │                  │  4 houses       │
│   │  24kW inv     │                  │  24kW inv     │                  │  24kW inv       │
│   │  20kWh bat    │                  │  20kWh bat    │                  │  20kWh bat      │
│   │               │                  │               │                  │                 │
│   └───────────────┘                  └───────────────┘                  └─────────────────┘
│                                                                                     │
│   ◄──────────────────────────── 800m total site length ────────────────────────────►
│                                                                                     │
│   TOPOLOGY: LINEAR DAISY-CHAIN (not circular)                                       │
│   • Single 5-wire cable runs through entire site                                    │
│   • Each cluster taps into the cable                                               │
│   • No loop - just A ─── B ─── C                                                   │
│                                                                                     │
│   TOTALS: 12 houses │ 72kW solar │ 60kWh battery │ 72kW inverter capacity          │
│                                                                                     │
│   ═══════  230V AC power (L + N + PE, 4mm² each)                                   │
│   ┄┄┄┄┄┄┄  CAN bus data (CAN-H + CAN-L, 0.5mm² each, in same cable)                │
│                                                                                     │
│   DROOP CONTROL: Power flows automatically from surplus to deficit clusters        │
│   CAN SYNC: One master ESP32 broadcasts timing, all others follow                  │
│                                                                                     │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

---

### Level 2: Cluster Detail

Each house is self-reliant with 6-15kW inverter. Load-balancing only kicks in for special scenarios.

```
┌──────────────────────────────────────────────────────────────────────────────────────┐
│                      CLUSTER - NORMAL OPERATION (99% of the time)                    │
│                                                                                      │
│   HOUSE 1                  HOUSE 2                  HOUSE 3                          │
│   ┌──────────────────┐     ┌──────────────────┐     ┌──────────────────┐            │
│   │ Solar 6kW        │     │ Solar 6kW        │     │ Solar 6kW        │            │
│   │ Bat 5kWh         │     │ Bat 5kWh         │     │ Bat 5kWh         │            │
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
│   │   48V / 5kWh     │   (no DC connection to other houses)                        │
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
| BMS + Battery | Libre Solar BMS-C1 + 48V/5kWh LiFePO4 (stays inside house) |
| Inverter | OzInverter 6kW with EG8010 + ESP32 + MCP2515 |
| Main Breaker | 63A MCB, isolates house from cluster AC bus |
| RJ45 Jack | CAN bus connection to other houses |

**Per Cluster (shared between houses):**
| Component | Description |
|-----------|-------------|
| 230V AC Bus | 4mm² cable connecting all house inverter outputs (10-30m) |
| CAN Bus | RJ45 cables daisy-chaining all houses (10-30m runs) |

**Between Clusters:**
| Component | Description |
|-----------|-------------|
| 5-Wire Cable | L + N + PE (4mm²) + CAN-H + CAN-L (0.5mm²) |
| 63A Breakers | One on each cluster side (safety disconnect) |
| Distance | Typically 50-200m between clusters |

---

### How It Works

**Power Flow:**
1. Sun → Solar panels → MPPT → Battery (48V DC, inside house)
2. Battery → Inverter → 230V AC → House loads
3. Excess → Cluster AC bus → Other houses (via droop control)
4. Still excess → Inter-cluster cable → Other clusters (via droop control)

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
- Battery: 5-10 kWh

| Scale | Houses | Inverter Capacity | Solar | Battery Storage |
|-------|--------|-------------------|-------|-----------------|
| Single house | 1 | 6-15 kW | 6 kW | 5-10 kWh |
| **Cluster (4 houses)** | 4 | 24-60 kW | 24 kW | 20-40 kWh |
| **Swarm (12 houses)** | 12 | 72-180 kW | 72 kW | 60-120 kWh |

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

**Battery perspective (60-120 kWh):**

```
Average house uses ~10 kWh/day

60 kWh  = 6 houses for 1 full day (no sun)
120 kWh = 12 houses for 1 day
        = 6 houses for 2 days (cloudy spell)
```

The swarm becomes a **small village grid** - completely independent from utility.

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

---

## Why Swarm Architecture?

| Traditional | Swarm |
|-------------|-------|
| Central controller (SPOF) | No central controller |
| Complex wiring | Simple AC tie between clusters |
| Manual programming | Self-organizing via droop control |
| Proprietary protocols | Open CAN bus / ThingSet |

Each inverter senses AC bus voltage/frequency and adjusts output automatically—like a flock of birds, no leader needed.

---

## License

Open source. Build for yourself, your community, or teach others.

---

<p align="center">
<strong>From LED blinker to swarm-powered communities.</strong><br>
<em>No utility grid required.</em>
</p>
