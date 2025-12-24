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
┌────────────────────────────────────────────────────────────────────────────────────┐
│                              COMMUNITY MICROGRID                                   │
│                                                                                    │
│  ╔════════════════════════════════╗    ╔════════════════════════════════╗         │
│  ║  CLUSTER A                     ║    ║  CLUSTER B                     ║         │
│  ║  (Houses 1 & 2 share DC bus)   ║    ║  (Houses 3 & 4 share DC bus)   ║         │
│  ╚════════════════════════════════╝    ╚════════════════════════════════╝         │
│                                                                                    │
│  ┌─────────────────────────────────┐   ┌─────────────────────────────────┐        │
│  │         48V DC BUS              │   │         48V DC BUS              │        │
│  │  ┌─────────────┬─────────────┐  │   │  ┌─────────────┬─────────────┐  │        │
│  │  │             │             │  │   │  │             │             │  │        │
│  │  │  HOUSE 1    │  HOUSE 2    │  │   │  │  HOUSE 3    │  HOUSE 4    │  │        │
│  │  │  ┌───────┐  │  ┌───────┐  │  │   │  │  ┌───────┐  │  ┌───────┐  │  │        │
│  │  │  │ Solar │  │  │ Solar │  │  │   │  │  │ Solar │  │  │ Solar │  │  │        │
│  │  │  │Panels │  │  │Panels │  │  │   │  │  │Panels │  │  │Panels │  │  │        │
│  │  │  └───┬───┘  │  └───┬───┘  │  │   │  │  └───┬───┘  │  └───┬───┘  │  │        │
│  │  │      ▼      │      ▼      │  │   │  │      ▼      │      ▼      │  │        │
│  │  │  ┌───────┐  │  ┌───────┐  │  │   │  │  ┌───────┐  │  ┌───────┐  │  │        │
│  │  │  │ MPPT  │  │  │ MPPT  │  │  │   │  │  │ MPPT  │  │  │ MPPT  │  │  │        │
│  │  │  │Charger│  │  │Charger│  │  │   │  │  │Charger│  │  │Charger│  │  │        │
│  │  │  └───┬───┘  │  └───┬───┘  │  │   │  │  └───┬───┘  │  └───┬───┘  │  │        │
│  │  │      ▼      │      ▼      │  │   │  │      ▼      │      ▼      │  │        │
│  │  │  ┌───────┐  │  ┌───────┐  │  │   │  │  ┌───────┐  │  ┌───────┐  │  │        │
│  │  │  │  BMS  │  │  │  BMS  │  │  │   │  │  │  BMS  │  │  │  BMS  │  │  │        │
│  │  │  │48V/5kW│  │  │48V/5kW│  │  │   │  │  │48V/5kW│  │  │48V/5kW│  │  │        │
│  │  │  │Battery│  │  │Battery│  │  │   │  │  │Battery│  │  │Battery│  │  │        │
│  │  │  └───┬───┘  │  └───┬───┘  │  │   │  │  └───┬───┘  │  └───┬───┘  │  │        │
│  │  │      │      │      │      │  │   │  │      │      │      │      │  │        │
│  │  └──────┴──────┴──────┴──────┘  │   │  └──────┴──────┴──────┴──────┘  │        │
│  │              │                  │   │              │                  │        │
│  └──────────────┼──────────────────┘   └──────────────┼──────────────────┘        │
│                 │ 48V DC                              │ 48V DC                    │
│         ┌───────┴───────┐                     ┌───────┴───────┐                   │
│         │               │                     │               │                   │
│    ┌────▼────┐    ┌────▼────┐            ┌────▼────┐    ┌────▼────┐               │
│    │ INV 1   │    │ INV 2   │            │ INV 3   │    │ INV 4   │               │
│    │ 6kW     │    │ 6kW     │            │ 6kW     │    │ 6kW     │               │
│    │ EG8010  │    │ EG8010  │            │ EG8010  │    │ EG8010  │               │
│    │ +ESP32  │    │ +ESP32  │            │ +ESP32  │    │ +ESP32  │               │
│    └────┬────┘    └────┬────┘            └────┬────┘    └────┬────┘               │
│         │              │                      │              │                    │
│    ═════╧══════════════╧═════            ═════╧══════════════╧═════               │
│    ║ CLUSTER A: 230V AC BUS ║            ║ CLUSTER B: 230V AC BUS ║               │
│    ═════════╤════════╤══════             ══════╤════════╤═════════                │
│             │        │                         │        │                         │
│        ┌────▼────┐ ┌─▼──────┐             ┌────▼────┐ ┌─▼──────┐                  │
│        │ HOUSE 1 │ │ HOUSE 2│             │ HOUSE 3 │ │ HOUSE 4│                  │
│        │  LOADS  │ │ LOADS  │             │  LOADS  │ │  LOADS │                  │
│        │ fridge  │ │ lights │             │  oven   │ │ washer │                  │
│        │ lights  │ │ tools  │             │  pump   │ │  heat  │                  │
│        │ router  │ │ garage │             │  shop   │ │ office │                  │
│        └────┬────┘ └───┬────┘             └────┬────┘ └───┬────┘                  │
│             │          │                       │          │                       │
│             └────┬─────┘                       └────┬─────┘                       │
│                  │                                  │                             │
│                  │      ┌──────────────────┐        │                             │
│                  └──────┤  63A CONTACTOR   ├────────┘                             │
│                         │  (DIN rail box)  │                                      │
│                         │                  │                                      │
│                         │  ┌────┐ ┌────┐   │                                      │
│                         │  │ L  │ │ N  │   │  ← 230V AC wires                     │
│                         │  │────│ │────│   │    between clusters                  │
│                         │  │ ○  │ │ ○  │   │    (underground cable               │
│                         │  └────┘ └────┘   │     or overhead line)               │
│                         │  Coil: 230V      │                                      │
│                         │  Contacts: 63A   │                                      │
│                         └──────────────────┘                                      │
│                                                                                   │
│   ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─   │
│   CAN BUS (RJ45 cable or twisted pair, handles SYNC + STATUS)                     │
│   ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─   │
│         │              │                      │              │                    │
│    ┌────┴────┐    ┌────┴────┐            ┌────┴────┐    ┌────┴────┐               │
│    │ ESP32   │    │ ESP32   │            │ ESP32   │    │ ESP32   │               │
│    │ MASTER  │    │ SLAVE   │            │ SLAVE   │    │ SLAVE   │               │
│    │         │    │         │            │         │    │         │               │
│    │ MCP2515 │    │ MCP2515 │            │ MCP2515 │    │ MCP2515 │               │
│    └─────────┘    └─────────┘            └─────────┘    └─────────┘               │
│                                                                                   │
│   CAN messages: SYNC timing, SOC%, voltage, load watts, faults (ThingSet)         │
│                                                                                   │
│   SYNC METHOD: ESP32 master broadcasts timing @ 100Hz                             │
│                All slaves adjust EG8010 phase to match                            │
│                Works at any distance (same room or 500m)                          │
│                                                                                   │
└───────────────────────────────────────────────────────────────────────────────────┘

HARDWARE LIST:
┌────────────────┬─────────────────────────────────────────────────────────────────┐
│ Component      │ Description                                                     │
├────────────────┼─────────────────────────────────────────────────────────────────┤
│ MPPT Charger   │ Solar charge controller (Libre Solar MPPT-2420-HC or similar)   │
│ BMS            │ Battery Management System (Libre Solar BMS-C1, 48V/100A)        │
│ INV (Inverter) │ OzInverter 6kW with EG8010 driver + ESP32 for sync & monitoring │
│ 63A Contactor  │ DIN-rail power relay, 230V coil, closes to connect clusters     │
│ ESP32 + CAN    │ ESP32 dev board + MCP2515 CAN transceiver (sync + telemetry)    │
│ RJ45 Cable     │ Standard Ethernet cable for CAN bus between all nodes           │
└────────────────┴─────────────────────────────────────────────────────────────────┘
```

**How it works:**
- CAN bus sync: Master ESP32 broadcasts timing, all inverters phase-lock via software
- One protocol for all: Same CAN bus handles sync (1m or 500m distance)
- AC loads: Each house connects to cluster's 230V AC bus
- Between clusters: 63A contactor connects the two AC buses + droop balances power
- Droop control: Each inverter adjusts frequency based on load, power flows automatically
- No special wiring: RJ45 Ethernet cables carry CAN signals everywhere

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
