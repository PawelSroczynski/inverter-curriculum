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

How clusters connect to form a community microgrid.

```
┌────────────────────────────────────────────────────────────────────────────────┐
│                           COMMUNITY MICROGRID                                  │
│                                                                                │
│                              50-200m between clusters                          │
│                                                                                │
│         ┌───────────────┐       5-WIRE        ┌───────────────┐               │
│         │               │◄═════ CABLE ═══════►│               │               │
│         │   CLUSTER A   │    L+N+PE+CAN       │   CLUSTER B   │               │
│         │               │                      │               │               │
│         │  3 houses     │                      │  3 houses     │               │
│         │  18kW gen     │                      │  18kW gen     │               │
│         │  15kWh store  │                      │  15kWh store  │               │
│         │               │                      │               │               │
│         └───────┬───────┘                      └───────┬───────┘               │
│                 │                                      │                       │
│                 │            5-WIRE CABLE              │                       │
│                 │     ┌───────────────────────┐        │                       │
│                 └─────┤                       ├────────┘                       │
│                       │      CLUSTER C        │                                │
│                       │                       │                                │
│                       │      3 houses         │                                │
│                       │      18kW gen         │                                │
│                       │      15kWh store      │                                │
│                       │                       │                                │
│                       └───────────────────────┘                                │
│                                                                                │
│  TOTALS: 9 houses │ 54kW solar │ 45kWh battery │ 54kW inverter capacity       │
│                                                                                │
│  ═══════  230V AC power (L + N + PE, 4mm² each)                               │
│  ┄┄┄┄┄┄┄  CAN bus data (CAN-H + CAN-L, 0.5mm² each, in same cable)            │
│                                                                                │
│  DROOP CONTROL: Power flows automatically from surplus to deficit clusters    │
│  CAN SYNC: One master ESP32 broadcasts timing, all others follow              │
│                                                                                │
└────────────────────────────────────────────────────────────────────────────────┘
```

---

### Level 2: Cluster Detail

How houses within a cluster share power and communicate.

```
┌────────────────────────────────────────────────────────────────────────────────────┐
│                              CLUSTER (typical)                                     │
│                                                                                    │
│   ┌─────────────┐         ┌─────────────┐         ┌─────────────┐                 │
│   │   HOUSE 1   │         │   HOUSE 2   │         │   HOUSE 3   │                 │
│   │             │         │             │         │             │                 │
│   │  Solar 3kW  │         │  Solar 3kW  │         │  Solar 3kW  │                 │
│   │  Bat 5kWh   │         │  Bat 5kWh   │         │  Bat 5kWh   │                 │
│   │  Inv 6kW    │         │  Inv 6kW    │         │  Inv 6kW    │                 │
│   │  ESP32 MSTR │         │  ESP32 SLV  │         │  ESP32 SLV  │                 │
│   │             │         │             │         │             │                 │
│   └──────┬──────┘         └──────┬──────┘         └──────┬──────┘                 │
│          │                       │                       │                        │
│          │ 48V DC                │ 48V DC                │ 48V DC                 │
│          │                       │                       │                        │
│   ┄┄┄┄┄┄┄┴┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┴┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┴┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄    │
│   48V DC BUS (shared within cluster only - batteries help each other)             │
│   ┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄    │
│                                                                                    │
│                    ┌──────────────────────────────────────┐                        │
│                    │  INVERTERS CONVERT: 48V DC → 230V AC │                        │
│                    └──────────────────────────────────────┘                        │
│                                                                                    │
│          │ 230V AC               │ 230V AC               │ 230V AC                │
│          │                       │                       │                        │
│   ═══════╧═══════════════════════╧═══════════════════════╧══════════════════►     │
│   230V AC BUS (shared within cluster, synced via CAN)               to other      │
│   ══════════════════════════════════════════════════════════════════ cluster      │
│                                                                                    │
│          │ RJ45                  │ RJ45                  │ RJ45                   │
│          │                       │                       │                        │
│   ───────○───────────────────────○───────────────────────○──────────────────►     │
│   CAN BUS (daisy-chain via RJ45, 10-30m between houses)             to other      │
│   ──────────────────────────────────────────────────────────────────  cluster     │
│                                                                                    │
│   ○ = RJ45 jack on inverter                                                       │
│                                                                                    │
│   WITHIN CLUSTER:                                                                 │
│   • 48V DC bus: batteries share charge (House 1 low → House 2 helps)              │
│   • 230V AC bus: inverters share load (synced via CAN, no phase conflict)         │
│   • CAN bus: ESP32s exchange SOC%, load, faults, sync timing                      │
│                                                                                    │
│   TO OTHER CLUSTERS:                                                              │
│   • 230V AC + CAN travel together in 5-wire cable                                 │
│   • 48V DC stays local (never crosses to other cluster)                           │
│                                                                                    │
└────────────────────────────────────────────────────────────────────────────────────┘
```

---

### Level 3: House Detail

Internal wiring of one house, showing connection to cluster buses.

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
│   ┌──────────────────┐              ┌──────────────────────────────────────────┐   │
│   │       BMS        │    30A fuse  │                                          │   │
│   │   48V / 5kWh     │◄────────────►│  48V DC CLUSTER BUS                      │   │
│   │    LiFePO4       │              │  ◄───── to/from other houses in cluster  │   │
│   └────────┬─────────┘              └──────────────────────────────────────────┘   │
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
│   CONNECTION DETAIL: Cluster Bus ↔ House                                           │
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
│   └────────────────────────────────────────────────────────────────────────────┘   │
│                                                                                    │
└────────────────────────────────────────────────────────────────────────────────────┘
```

---

### Hardware Summary

**Per House:**
| Component | Description |
|-----------|-------------|
| Solar Panels | 3kW array (size varies) |
| MPPT Charger | Libre Solar MPPT-2420-HC or similar |
| BMS + Battery | Libre Solar BMS-C1 + 48V/5kWh LiFePO4 |
| Inverter | OzInverter 6kW with EG8010 + ESP32 + MCP2515 |
| Main Breaker | 63A MCB, isolates house from cluster |
| DC Fuse | 30A, protects battery connection to cluster DC bus |
| RJ45 Jack | CAN bus connection to other houses |

**Per Cluster (shared):**
| Component | Description |
|-----------|-------------|
| 48V DC Bus | Heavy cable (10mm²+) connecting all house batteries |
| 230V AC Bus | Standard wiring connecting all house inverter outputs |
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
1. Sun → Solar panels → MPPT → Battery (48V DC)
2. Battery → Inverter → 230V AC → House loads
3. Excess → Cluster AC bus → Other houses (via droop control)
4. Still excess → Inter-cluster cable → Other clusters (via droop control)

**Communication:**
1. ESP32 master broadcasts sync timing @ 100Hz on CAN bus
2. All slave ESP32s adjust their EG8010 phase to match
3. Status messages (SOC%, load, faults) shared via ThingSet protocol

**Droop Control (automatic power sharing):**
- Heavy load → inverter frequency drops slightly (e.g., 50.0 → 49.8 Hz)
- Other inverters sense lower frequency → push more power
- No central controller needed - physics handles balancing

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
