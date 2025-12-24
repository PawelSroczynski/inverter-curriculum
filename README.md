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

**End Goal:** 4-household microgrid with 20kW+ capacity, automatic power sharing, no utility grid required.

---

## The Swarm Microgrid Vision

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                            COMMUNITY MICROGRID                                  │
│                                                                                 │
│   CLUSTER A                                      CLUSTER B                      │
│   ═════════════════════════════════              ═════════════════════════════  │
│                                                                                 │
│   ┌──── 48V DC BUS (shared) ────┐               ┌──── 48V DC BUS (shared) ────┐ │
│   │                             │               │                             │ │
│   │  ┌───────┐    ┌───────┐    │               │  ┌───────┐    ┌───────┐    │ │
│   │  │Solar 1│    │Solar 2│    │               │  │Solar 3│    │Solar 4│    │ │
│   │  │ MPPT  │    │ MPPT  │    │               │  │ MPPT  │    │ MPPT  │    │ │
│   │  └───┬───┘    └───┬───┘    │               │  └───┬───┘    └───┬───┘    │ │
│   │      │            │        │               │      │            │        │ │
│   │      ▼            ▼        │               │      ▼            ▼        │ │
│   │  ┌───────┐    ┌───────┐    │               │  ┌───────┐    ┌───────┐    │ │
│   │  │ BMS 1 │    │ BMS 2 │    │               │  │ BMS 3 │    │ BMS 4 │    │ │
│   │  │48V bat│    │48V bat│    │               │  │48V bat│    │48V bat│    │ │
│   │  └───┬───┘    └───┬───┘    │               │  └───┬───┘    └───┬───┘    │ │
│   │      │            │        │               │      │            │        │ │
│   │      └─────┬──────┘        │               │      └─────┬──────┘        │ │
│   │            │ 48V DC        │               │            │ 48V DC        │ │
│   └────────────┼───────────────┘               └────────────┼───────────────┘ │
│                │                                            │                 │
│        ┌───────┴───────┐                            ┌───────┴───────┐         │
│        │               │                            │               │         │
│   ┌────▼────┐    ┌────▼────┐                   ┌────▼────┐    ┌────▼────┐    │
│   │ INV 1   │    │ INV 2   │                   │ INV 3   │    │ INV 4   │    │
│   │  6kW    │    │  6kW    │                   │  6kW    │    │  6kW    │    │
│   │ EG8010  │    │ EG8010  │                   │ EG8010  │    │ EG8010  │    │
│   │ +ESP32  │    │ +ESP32  │                   │ +ESP32  │    │ +ESP32  │    │
│   └────┬────┘    └────┬────┘                   └────┬────┘    └────┬────┘    │
│        │  ▲      ▲    │                             │  ▲      ▲    │         │
│        │  └──────┘    │                             │  └──────┘    │         │
│        │   SYNC pin   │                             │   SYNC pin   │         │
│        │ (master/slave)                             │ (master/slave)         │
│        │              │                             │              │         │
│   ─────┴──────────────┴─────                   ─────┴──────────────┴─────    │
│   │  CLUSTER A: 230V AC BUS │                   │  CLUSTER B: 230V AC BUS │  │
│   ──────────┬───────────────                   ──────────┬────────────────   │
│             │                                            │                   │
│        ┌────┴────┐                                  ┌────┴────┐              │
│        │  LOADS  │                                  │  LOADS  │              │
│        │ House 1 │                                  │ House 3 │              │
│        │ House 2 │                                  │ House 4 │              │
│        └────┬────┘                                  └────┬────┘              │
│             │                                            │                   │
│             │         ┌─────────────────────┐            │                   │
│             └─────────┤  AC CONTACTOR/BRK   ├────────────┘                   │
│                       │  (inter-cluster)    │                                │
│                       │                     │                                │
│                       │  230V AC tie line   │                                │
│                       │  Droop control      │                                │
│                       │  handles sync       │                                │
│                       └─────────────────────┘                                │
│                                                                              │
│   ┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄  │
│   CAN BUS (communication layer - separate from power)                        │
│   ┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄  │
│        │              │                             │              │         │
│   ┌────┴────┐    ┌────┴────┐                   ┌────┴────┐    ┌────┴────┐    │
│   │ ESP32-1 │    │ ESP32-2 │                   │ ESP32-3 │    │ ESP32-4 │    │
│   └─────────┘    └─────────┘                   └─────────┘    └─────────┘    │
│                                                                              │
│   CAN bus carries: battery SOC, load %, faults, droop params, ThingSet msgs  │
│                                                                              │
└──────────────────────────────────────────────────────────────────────────────┘

POWER FLOW:    Solar → MPPT → 48V DC Bus → Inverter → 230V AC Bus → Loads
                                                            ↓
                                              Inter-cluster AC tie (droop)

COMMUNICATION: ESP32 ←──── CAN bus (ThingSet protocol) ────→ ESP32
               All inverters share status, coordinate via droop control
```

**Key points:**
- No special "TIE INV" needed - just an AC contactor between cluster AC buses
- Droop control in each inverter handles synchronization automatically
- CAN bus is separate communication layer, not power
- Within cluster: EG8010 SYNC pin for tight phase lock
- Between clusters: droop control allows slight freq/phase variation

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
