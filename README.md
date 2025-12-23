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
Step 1          Step 2-3        Step 4-5         Step 6           Step 7
────────────────────────────────────────────────────────────────────────────►

┌─────────┐   ┌─────────┐   ┌───────────┐   ┌───────────┐   ┌──────────────┐
│   LED   │   │  150W   │   │   6-15kW  │   │   Smart   │   │    Swarm     │
│ Blinker │──►│Inverter │──►│ OzInverter│──►│  Monitor  │──►│  Microgrid   │
│ (basics)│   │(H-bridge│   │(pure sine)│   │  (ESP32)  │   │(bi-direction)│
└─────────┘   └─────────┘   └───────────┘   └───────────┘   └──────────────┘
   12 projects    500W          Toroid         Home Asst      4 households
   Ohm's Law    gate drivers    winding        MQTT/CAN       droop control
```

---

## What You'll Build

| Step | Project | Power | Key Skills |
|------|---------|-------|------------|
| 1 | 12 fundamental projects | — | Soldering, Ohm's Law, transistors, 555, PWM |
| 2 | CD4047 Square Wave | 150W | MOSFETs, transformers, push-pull |
| 3 | SG3525 Modified Sine | 500W | H-bridge, gate drivers, dead-time |
| 4 | EG8010 Pure Sine | 1kW | SPWM, LC filters, voltage feedback |
| 5 | OzInverter | 6-15kW | Toroid winding, thermal management |
| 6 | Smart Upgrade | — | ESP32, CAN bus, Home Assistant |
| **7** | **Swarm Microgrid** | **12kW+** | **Bi-directional tie, droop control** |

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
├── curriculum/                  # Learning materials (7 steps)
│   ├── index.md                 # Start here
│   ├── part1_fundamentals.md    # 12 basic projects
│   ├── part2_modified_sine.md   # 150W → 500W
│   ├── part3_pure_sine.md       # 1kW → 6-15kW OzInverter
│   ├── upgrade_smart.md         # ESP32 + CAN bus
│   ├── microgrid.md             # Swarm architecture
│   └── libre_solar.md           # Open source MPPT & BMS
│
└── hardware/                    # Production-ready designs
    ├── ozinverter/              # 6kW+ inverter (KiCad + Gerbers)
    └── libre_solar/             # MPPT, BMS, ESP32 gateway
```

---

## Investment

| Path | Time | Cost |
|------|------|------|
| Single Smart Inverter | 6-12 months | ~$775 |
| 4-Household Swarm Microgrid | 12-24 months | ~$14,300 (~$3,575/house) |

**Savings vs Commercial:** 60-75% less than Victron/SMA equivalent

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

1. **Read** → `curriculum/index.md`
2. **Build** → Complete Steps 1-5 (single inverter)
3. **Upgrade** → Add ESP32 smart monitoring (Step 6)
4. **Scale** → Connect multiple units (Step 7)

---

## Safety

⚠️ **High voltage warning.** 48V DC can cause fires. 230V AC can kill. Never work on live circuits.

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
