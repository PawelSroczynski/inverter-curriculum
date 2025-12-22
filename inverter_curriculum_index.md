# Inverter Building Curriculum - Master Index

## From Zero to OzInverter (6-15kW Pure Sine Wave)

---

## Document Structure

| File | Contents |
|------|----------|
| `inverter_curriculum_part1.md` | Step 1: Fundamentals (12 projects, 4 phases) |
| `inverter_curriculum_part2.md` | Step 2: CD4047 (150W) + Step 3: SG3525 (500W) |
| `inverter_curriculum_part3.md` | Step 4: EG8010 (1kW) + Step 5: OzInverter (6-15kW) |
| `inverter_curriculum_upgrade_smart.md` | **UPGRADE:** Smart monitoring (ESP32/SmartBob) |
| `inverter_curriculum_microgrid.md` | **Step 7:** DC Microgrid with AC Tie (4 units) |
| `inverter_curriculum_libre_solar.md` | **BONUS:** Open source MPPT & BMS (Libre Solar) |

---

## Quick Reference

### Timeline
```
Step 1: 8-12 weekends  (Fundamentals)
Step 2: 2-3 weekends   (First Inverter)
Step 3: 3-4 weekends   (H-Bridge)
Step 4: 1-2 months     (Pure Sine)
Step 5: 2-3 months     (OzInverter)
────────────────────────────────────
Total:  6-12 months
```

### Budget
```
Step 1:    $64   (12 projects)
Step 2:    $40   (150W inverter)
Step 3:   $110   (500W inverter)
Step 4:   $150   (1kW inverter)
Step 5:   $600   (6kW inverter)
Tools:    $300   (one-time)
────────────────────────────────────
Total: ~$1,264
```

---

## Part 1: Fundamentals (12 Projects)

### Phase 1: No ICs
- 1.1 LED + Resistor (Ohm's Law)
- 1.2 Switch Circuits (AND/OR logic)
- 1.3 Transistor as Switch

### Phase 2: 555 Timer
- 1.4 Monostable (One-Shot)
- 1.5 Astable (Blinker)
- 1.6 Relay Control

### Phase 3: Control Concepts
- 1.7 PWM Dimmer
- 1.8 LED Chaser (4017)
- 1.9 Power Switching (TIP120)

### Phase 4: Measurement
- 1.10 Continuity Tester
- 1.11 Voltage Indicator
- 1.12 Audio Oscillator

---

## Part 2: From Oscillator to Modified Sine (4 Phases)

### Phase 1: Oscillator Mastery
- CD4047 theory & testing
- Frequency calculation
- Complementary outputs

### Phase 2: First Inverter Build (150W)
- Push-pull topology
- MOSFET fundamentals
- Center-tap transformer

### Phase 3: H-Bridge & Gate Drivers
- IR2110 bootstrap circuit
- Dead-time concepts
- Shoot-through prevention

### Phase 4: Modified Sine Build (500W)
- SG3525 PWM controller
- H-bridge power stage
- Output filtering

---

## Part 3: From SPWM to OzInverter (4 Phases)

### Phase 1: SPWM Theory & EG8010
- Digital SPWM concept
- EG8010/EGS002/003
- LC filter design

### Phase 2: 1kW Pure Sine Build
- Complete inverter build
- Voltage feedback
- THD <3%

### Phase 3: High-Power Design
- MOSFET paralleling
- Transformer winding
- Thermal management
- Bus bar design

### Phase 4: OzInverter Build (6-15kW)
- Full production build
- Hand-wound toroidal
- 16+ paralleled MOSFETs
- Comprehensive testing

---

## Skills Progression

| After Step | You Can Build |
|------------|---------------|
| 1 | Any basic electronics project |
| 2 | 150W backup inverter |
| 3 | 500W tool/workshop inverter |
| 4 | 1kW home electronics inverter |
| 5 | 6-15kW whole-house inverter |

---

## Key Components Across All Steps

| Component | Step 2 | Step 3 | Step 4 | Step 5 |
|-----------|--------|--------|--------|--------|
| Controller | CD4047 | SG3525 | EG8010 | EG8010 |
| Gate Driver | Direct | IR2110 | IR2110 | IR2110 |
| MOSFETs | 2× IRF3205 | 4× IRF3205 | 4× IRFP460 | 16× IRFP4668 |
| Topology | Push-pull | H-bridge | H-bridge | H-bridge |
| Transformer | 100VA | 300VA | 500VA | 5000VA |
| Output | Square | Modified | Pure Sine | Pure Sine |

---

## How to Use This Curriculum

1. **Start with Part 1** - Complete all 12 fundamentals projects
2. **Build Part 2 projects** - Steps 2 and 3 in sequence
3. **Progress to Part 3** - Steps 4 and 5
4. **Don't skip steps** - Each builds on previous knowledge
5. **Ask for help** - Use forums, this assistant, YouTube

---

---

## Step 7: Microgrid Expansion (Community Scale)

After completing smart upgrade, scale to 4-unit community:

```
Architecture: Hybrid DC/AC
├── Cluster A (Houses 1-2): 48V DC bus
├── Cluster B (Houses 3-4): 48V DC bus
└── AC Tie (230V): Connects clusters, grid-ready
```

| Metric | Value |
|--------|-------|
| Total capacity | 12kW (4× 3kW inverters) |
| Solar | 8kW (4× 2kW per house) |
| Battery | 30kWh (2× 15kWh clusters) |
| Households | 4 |
| Total cost | ~$14,300 |
| Per household | ~$3,578 |

**Savings vs commercial: 60-75%**

---

## Smart Upgrade Module (Victron-Like Features)

After completing Step 5, add smart monitoring:

| Feature | With ESP32/SmartBob |
|---------|---------------------|
| Remote monitoring | Home Assistant app |
| Voltage/Current display | OLED + Web dashboard |
| Temperature monitoring | Multi-point sensors |
| Data logging | InfluxDB / Grafana |
| Alerts | Telegram / Push notifications |
| Remote ON/OFF | WiFi control |
| ATS (Auto Transfer) | Grid backup switching |

**Cost:** +$130-175 (vs $1,500+ for Victron equivalent features)

---

## Libre Solar Integration (100% Open Source Stack)

Replace commercial charge controllers and BMS with open source alternatives:

| Component | Commercial | Libre Solar DIY | Savings |
|-----------|------------|-----------------|---------|
| MPPT Controller | $150-300 | ~$43 | 70-85% |
| BMS (8S LiFePO4) | $100-200 | ~$52 | 50-75% |
| **Total** | $250-500 | **~$95** | **60-80%** |

**Key Features:**
- CAN bus networking with ThingSet protocol
- ESP32 gateway for Home Assistant integration
- Full schematics and firmware (open source)
- Compatible with 48V microgrid architecture

**Designs:**
- MPPT 2420 HC (20A, 12V/24V - use with DC-DC boost for 48V)
- BMS C1 (supports 3-16S, LiFePO4/Li-ion)

---

## Resources

- **OzInverter Official:** https://www.bryanhorology.com/ozinverter.php
- **Libre Solar Hardware:** https://libre.solar/hardware/
- **SmartBob Controllers:** https://smartbob.pl
- **ESPHome:** https://esphome.io
- **Home Assistant:** https://home-assistant.io
- **YouTube:** GreatScott!, EEVBlog, ElectroBOOM
- **Free Textbook:** AllAboutCircuits.com
- **Simulator:** Falstad Circuit Simulator

---

*Created: December 2024*
*Goal: Build Smart OzInverter 6-15kW Pure Sine Wave Inverter with Victron-like monitoring*
