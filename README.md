# Inverter Building Curriculum

**From LED Blinker to 6-15kW Pure Sine Wave Inverter & Community Microgrid**

A complete, open-source electronics curriculum that takes you from zero knowledge to building production-quality power inverters and community-scale microgrids.

---

## Overview

```
You are here                                                    Your goal
     │                                                              │
     ▼                                                              ▼
┌─────────┐   ┌─────────┐   ┌─────────┐   ┌─────────┐   ┌─────────────────┐
│   LED   │──▶│  555    │──▶│ 150W    │──▶│  6kW    │──▶│   Community     │
│ Blinker │   │ Timer   │   │Inverter │   │OzInverter│  │   Microgrid     │
└─────────┘   └─────────┘   └─────────┘   └─────────┘   └─────────────────┘
   Step 1        Step 1       Step 2        Step 5          Step 7
  (basics)      (timing)    (first inv)   (full power)    (4 households)
```

---

## What You'll Build

| Step | Project | Power Output | Skills Learned |
|------|---------|--------------|----------------|
| 1 | 12 fundamental projects | — | Soldering, Ohm's Law, transistors, 555 timer, PWM |
| 2 | CD4047 Square Wave Inverter | 150W | MOSFETs, transformers, push-pull topology |
| 3 | SG3525 Modified Sine Wave | 500W | H-bridge, gate drivers, dead-time control |
| 4 | EG8010 Pure Sine Wave | 1kW | SPWM, LC filters, voltage feedback |
| 5 | OzInverter | 6-15kW | Toroid winding, thermal management, high power |
| 6 | Smart Upgrade | — | ESP32, Home Assistant, remote monitoring |
| 7 | Community Microgrid | 12kW | DC bus, AC tie, swarm architecture |

---

## Why This Curriculum?

### Problem
- Commercial inverters (Victron, etc.) cost $1,500-3,000+
- No path from beginner to advanced power electronics
- Existing tutorials skip fundamentals or lack progression

### Solution
- **Step-by-step progression** - each project builds on the last
- **Complete BOMs** - every component listed with prices
- **Real skills** - not just copying circuits, but understanding them
- **Cost effective** - build a 6kW inverter for ~$600 (vs $2,200 commercial)

---

## Curriculum Structure

```
inverter-curriculum/
│
├── inverter_curriculum_index.md           # Start here - master overview
│
├── inverter_curriculum_part1.md           # Step 1: Fundamentals
│   ├── Phase 1: No ICs (LED, switches, transistors)
│   ├── Phase 2: 555 Timer (monostable, astable, relay)
│   ├── Phase 3: Control (PWM, counters, power switching)
│   └── Phase 4: Measurement (build your own tools)
│
├── inverter_curriculum_part2.md           # Steps 2-3: First Inverters
│   ├── Step 2: CD4047 Square Wave (150W)
│   └── Step 3: SG3525 Modified Sine (500W)
│
├── inverter_curriculum_part3.md           # Steps 4-5: Pure Sine
│   ├── Step 4: EG8010 Pure Sine (1kW)
│   └── Step 5: OzInverter (6-15kW)
│
├── inverter_curriculum_upgrade_smart.md   # Smart Features
│   ├── ESP32/SmartBob integration
│   ├── Voltage, current, temperature monitoring
│   ├── Home Assistant dashboards
│   └── ATS (Automatic Transfer Switch)
│
└── inverter_curriculum_microgrid.md       # Step 7: Community Scale
    ├── 48V DC bus architecture
    ├── 230V AC tie between clusters
    ├── 4-household system design
    └── Swarm communication
```

---

## Investment Required

### Time
| Step | Duration |
|------|----------|
| Step 1 (Fundamentals) | 8-12 weekends |
| Step 2 (150W) | 2-3 weekends |
| Step 3 (500W) | 3-4 weekends |
| Step 4 (1kW) | 1-2 months |
| Step 5 (6kW) | 2-3 months |
| Step 6 (Smart) | 2-4 weeks |
| Step 7 (Microgrid) | 6-8 months |
| **Total** | **12-24 months** |

### Money
| Component | Cost |
|-----------|------|
| Step 1-4 components | ~$364 |
| Step 5 (OzInverter) | ~$600 |
| Step 6 (Smart upgrade) | ~$175 |
| Step 7 (4-unit microgrid) | ~$14,300 |
| Tools (if needed) | ~$300 |

### Single Inverter Path
**~$1,139** for complete 6kW smart inverter (Steps 1-6)

### Community Microgrid Path
**~$14,300** for 4-household system (~$3,575 per household)

---

## Comparison with Commercial

| Feature | Victron MultiPlus 5kW | Your OzInverter + Smart |
|---------|----------------------|------------------------|
| Pure sine wave | ✅ | ✅ |
| Power output | 5kW | 6-15kW |
| Remote monitoring | ✅ VRM app | ✅ Home Assistant |
| Mobile app | ✅ | ✅ |
| Data logging | ✅ | ✅ |
| **Price** | **$2,200** | **$775** |
| **Savings** | — | **65%** |
| Repairable by you | ❌ | ✅ |
| Expandable | Limited | ✅ Fully |
| Learning value | None | Immense |

---

## Who Is This For?

### Ideal For
- 🔧 **Makers** who want to understand power electronics
- 🏠 **Off-grid enthusiasts** building their own power systems
- 🌍 **Development workers** bringing power to rural communities
- 📚 **Students** learning practical electronics
- 💰 **Budget-conscious** builders who want quality without premium prices

### Prerequisites
- Basic math (algebra)
- Ability to follow instructions carefully
- Patience and willingness to learn from mistakes
- **No prior electronics experience required**

---

## Safety Warning

⚠️ **This curriculum involves dangerous voltages and currents.**

- 48V DC at high current can cause fires
- 220V AC can kill you
- Always follow safety guidelines in each module
- Never work on live circuits
- Keep fire extinguisher nearby
- If unsure, ask before proceeding

---

## Getting Started

1. **Read the index** - `inverter_curriculum_index.md`
2. **Start with Step 1** - Don't skip fundamentals!
3. **Order parts** - BOMs included in each section
4. **Build progressively** - Each step prepares you for the next
5. **Ask questions** - Use Issues or Discussions

---

## Key Technologies

| Technology | Used In | Purpose |
|------------|---------|---------|
| **555 Timer** | Step 1 | Learn timing circuits |
| **CD4047** | Step 2 | Simple oscillator |
| **SG3525** | Step 3 | PWM control |
| **EG8010** | Steps 4-5 | SPWM pure sine generation |
| **IR2110** | Steps 3-5 | MOSFET gate driving |
| **IRF3205/IRFP460** | Steps 2-5 | Power switching |
| **ESP32** | Steps 6-7 | Smart monitoring |
| **ESPHome** | Steps 6-7 | IoT firmware |
| **Home Assistant** | Steps 6-7 | Dashboard & automation |

---

## Based On

- **OzInverter** - Open source 6-15kW inverter design
  - https://www.bryanhorology.com/ozinverter.php
- **EG8010** - Digital SPWM controller IC
- **SmartBob** - ESP32 home automation controllers
- **Research** from UC Berkeley, MDPI on DC microgrids

---

## Project Status

| Module | Status |
|--------|--------|
| Step 1: Fundamentals | ✅ Complete |
| Step 2: CD4047 | ✅ Complete |
| Step 3: SG3525 | ✅ Complete |
| Step 4: EG8010 | ✅ Complete |
| Step 5: OzInverter | ✅ Complete |
| Step 6: Smart Upgrade | ✅ Complete |
| Step 7: Microgrid | ✅ Complete |

---

## Contributing

Contributions welcome! Areas that could use help:

- [ ] Translate to other languages
- [ ] Add photos/videos of builds
- [ ] Test and verify BOMs
- [ ] Add troubleshooting tips
- [ ] Create PCB designs (KiCad/EasyEDA)
- [ ] Improve safety documentation

---

## License

This curriculum is open source. Use it to:
- ✅ Learn and build for personal use
- ✅ Teach others
- ✅ Modify and improve
- ✅ Build systems for communities

Please credit this project if you share derivatives.

---

## Acknowledgments

- **Oztules** and the OzInverter community
- **GreatScott!**, **EEVBlog**, **ElectroBOOM** - YouTube educators
- **AllAboutCircuits.com** - Free electronics education
- UC Berkeley researchers on DC microgrids
- The open source hardware community

---

## Support

- **Issues** - Report problems or suggest improvements
- **Discussions** - Ask questions, share your builds
- **Pull Requests** - Contribute improvements

---

<p align="center">
  <b>From LED blinker to powering communities.</b><br>
  <i>One step at a time.</i>
</p>
