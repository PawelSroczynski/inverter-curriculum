# Inverter Building Curriculum

**From LED Blinker to Swarm-Networked Community Microgrid**

Build your own 6-15kW pure sine wave inverters that talk to each other, share power automatically, and create resilient off-grid communities—without utility grid dependence.

---

## The Goal: Swarm-Networked Microgrids

```
        ┌─────────────────────────────────────────────────────────────┐
        │                    COMMUNITY MICROGRID                      │
        │                                                             │
        │   Cluster A (48V DC)              Cluster B (48V DC)        │
        │   ┌─────────┐ ┌─────────┐         ┌─────────┐ ┌─────────┐   │
        │   │ House 1 │ │ House 2 │         │ House 3 │ │ House 4 │   │
        │   │  Solar  │ │  Solar  │         │  Solar  │ │  Solar  │   │
        │   │   BMS   │ │   BMS   │         │   BMS   │ │   BMS   │   │
        │   │ Inverter│ │ Inverter│         │ Inverter│ │ Inverter│   │
        │   └────┬────┘ └────┬────┘         └────┬────┘ └────┬────┘   │
        │        │           │                   │           │        │
        │        └─────┬─────┘                   └─────┬─────┘        │
        │              │                               │              │
        │         ┌────▼────┐    230V AC Tie     ┌────▼────┐          │
        │         │ TIE INV │◄──────────────────►│ TIE INV │          │
        │         │ (Master)│   Bi-directional   │ (Slave) │          │
        │         └─────────┘    Power Flow      └─────────┘          │
        │                                                             │
        │   ✓ No utility grid connection required                     │
        │   ✓ Automatic power balancing via droop control             │
        │   ✓ Any inverter can become master                          │
        │   ✓ Resilient: system works if units fail                   │
        └─────────────────────────────────────────────────────────────┘
```

**Swarm Architecture:** Inverters communicate via CAN bus or use droop control (no wires needed) to automatically balance power between clusters. When Cluster A has excess solar, power flows to Cluster B. At night, batteries share load. No central controller—each unit makes local decisions based on voltage/frequency.

---

## Learning Path

```
Part 1        Part 2         Part 3           Part 4           Part 5-7
────────────────────────────────────────────────────────────────────────────►

┌─────────┐   ┌─────────┐   ┌───────────┐   ┌───────────┐   ┌──────────────┐
│   LED   │   │ Safety  │   │   500W    │   │  1kW →    │   │   Smart +    │
│ Blinker │──►│   +     │──►│ Modified  │──►│ 2.5kW →   │──►│   Swarm      │
│ (26 proj)│  │MOSFET   │   │  + Winding│   │  6-15kW   │   │  Microgrid   │
└─────────┘   └─────────┘   └───────────┘   └───────────┘   └──────────────┘
   9V safe       12V/20W       220V output     Production      4 households
   Fundamentals  First MOSFET   + transformer   quality         droop control
                               practice         OzInverter      parallel 20kW
```

**Incremental Power Progression:**
```
9V → 12V/20W → 150W → 500W → 1kW → 2.5kW → 6-15kW → 20kW (parallel)
└Part 1─┘ └Part 2─┘ └──Part 3──┘   └────Part 4────┘   └Part 5-7┘
```

---

## What You'll Build

| Part | Phase | Project | Power | Key Skills |
|------|-------|---------|-------|------------|
| 1 | 1-5 | 26 fundamental projects | 9V | Soldering, Ohm's Law, transistors, 555, PWM |
| 2 | - | Safety + MOSFET Bridge | 12V/20W | HV safety, first power MOSFETs |
| 3 | 1-2 | CD4047 Push-Pull | 150W | MOSFETs, transformers |
| 3 | 3-4 | SG3525 Modified Sine | 500W | H-bridge, IR2110, dead-time |
| 3 | 5 | Transformer Winding | 50VA | Practice winding technique |
| 4 | 1-2 | EG8010 Pure Sine | 1kW | SPWM, LC filters |
| 4 | 3 | 2.5kW Intermediate | 2.5kW | MOSFET paralleling, thermal mgmt |
| 4 | 4-5 | OzInverter | 6-15kW | Large toroid, 16 MOSFETs |
| 5 | - | Smart Upgrade | — | ESP32, CAN bus, Home Assistant |
| 6 | - | Microgrid | 4 houses | Swarm communication, load sharing |
| 7 | - | Libre Solar Integration | — | Open MPPT, BMS, ThingSet |
| - | - | Parallel 2× OzInverter | 20kW | Droop control, master/slave |

**Curriculum design:** Safety module, MOSFET bridge project, transformer winding practice, and 2.5kW intermediate build ensure gradual skill development with no dangerous gaps.

---

## Why Swarm Networking?

| Traditional Microgrid | Swarm Microgrid |
|-----------------------|-----------------|
| Central controller (single point of failure) | No central controller |
| Complex wiring between all units | Simple AC tie between clusters |
| Requires programming each unit | Self-organizing via droop control |
| Expensive commercial inverters | DIY inverters (~$775 each) |
| Proprietary protocols | Open CAN bus / ThingSet |

**Key Innovation:** Replace EG8010 with ESP32 for bi-directional control. Each inverter senses AC bus voltage/frequency and adjusts output automatically—like a flock of birds, no leader needed.

---

## Repository Structure

```
inverter-curriculum/
├── curriculum/
│   ├── part1_fundamentals.md            # 26 projects at safe 9V
│   ├── part2_safety_and_mosfet_bridge.md # Safety + first MOSFETs at 12V
│   ├── part3_modified_sine.md           # 150W → 500W + transformer practice
│   ├── part4_pure_sine.md               # 1kW → 2.5kW → 6-15kW OzInverter
│   ├── part5_smart_upgrade.md           # ESP32 + CAN bus
│   ├── part6_microgrid.md               # Swarm architecture
│   └── part7_libre_solar.md             # Open source MPPT & BMS
│
└── hardware/                            # Production-ready designs
    ├── ozinverter/                      # 6kW+ inverter (KiCad + Gerbers)
    └── libre_solar/                     # MPPT, BMS, ESP32 gateway
```

---

## Investment

| Path | Time | Cost |
|------|------|------|
| Part 1 (Fundamentals) | 6-8 weekends | ~$64 |
| Part 2 (Safety + Bridge) | 1-2 weekends | ~$25 |
| Part 3 (Modified Sine + Winding) | 6-8 weekends | ~$140 |
| Part 4 (1kW + 2.5kW + OzInverter) | 4-6 months | ~$1,000 |
| Parts 5-7 (Smart + Microgrid) | 3-6 months | ~$500 |
| **Complete: 20kW Parallel System** | **12-18 months** | **~$1,730** |
| **4-Household Microgrid** | **18-24 months** | **~$14,300 (~$3,575/house)** |

**Savings vs Commercial:** 60-75% less than Victron/SMA equivalent

**Note:** Intermediate builds (2.5kW) add ~$250 but prevent costly mistakes. Better to fail with a $30 transformer than a $70 one.

---

## Hardware Included

| Component | Source | Description |
|-----------|--------|-------------|
| OzInverter PCB | jdevine82 | 6kW+ main board, controller, output stage |
| BMS-C1 | Libre Solar | 16S/100A battery management |
| MPPT-2420-HC | Libre Solar | 20A solar charge controller |
| ESP32 Gateway | Libre Solar | CAN bus → WiFi → Home Assistant |

All designs are open source with KiCad files and Gerbers ready for fabrication.

---

## Getting Started

1. **Part 1** → Complete 26 fundamental projects at safe 9V
2. **Part 2** → Safety module + first MOSFET project (MANDATORY before 220V)
3. **Part 3** → Build modified sine inverters (150W → 500W) + winding practice
4. **Part 4** → Pure sine progression: 1kW → 2.5kW → 6-15kW OzInverter
5. **Parts 5-7** → Smart upgrade, microgrid, Libre Solar integration
6. **Scale** → Parallel two inverters for 20kW

---

## Safety

⚠️ **High voltage warning.** 48V DC can cause fires. 230V AC can kill. Never work on live circuits.

**Mandatory:** Complete Part 2 (Safety Module) before any work with 220V. This includes:
- Electrical safety fundamentals (30mA can kill)
- One-hand rule and safe work practices
- Capacitor discharge procedures
- Emergency response procedures
- Safety quiz (must pass before proceeding)

The curriculum is designed with incremental voltage progression: 9V → 12V → 220V to build skills and safety awareness before handling dangerous voltages.

---

## Resources

| Resource | URL |
|----------|-----|
| OzInverter | bryanhorology.com/ozinverter.php |
| Libre Solar | libre.solar |
| Home Assistant | home-assistant.io |
| ThingSet Protocol | thingset.io |

---

## License

Open source. Build for yourself, your community, or teach others.

---

<p align="center">
<b>From LED blinker to swarm-powered communities.</b><br>
<i>No utility grid required.</i>
</p>
