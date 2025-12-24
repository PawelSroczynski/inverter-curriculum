# Inverter Curriculum - Master Index

## From Zero to OzInverter (6-15kW Pure Sine Wave)

---

## Documents

| File | Contents |
|------|----------|
| `part1_fundamentals.md` | Step 1: 37 projects (6 phases) - Includes SmartBob/ESP32 & Home Assistant |
| `part2_modified_sine.md` | Steps 2-3: CD4047 → SG3525 |
| `part3_pure_sine.md` | Steps 4-5: EG8010 → OzInverter |
| `part4_smart_upgrade.md` | Step 6: ESP32 monitoring |
| `part5_microgrid.md` | Step 7: 4-unit DC/AC microgrid |
| `part6_libre_solar.md` | Bonus: Open source MPPT & BMS |

**Hardware:** See `../hardware/` for KiCad designs & firmware

---

## Quick Reference

### Timeline
```
Step 1: 10-14 weekends (Fundamentals + SmartBob + Sync)
Step 2: 2-3 weekends   (First Inverter)
Step 3: 3-4 weekends   (H-Bridge)
Step 4: 1-2 months     (Pure Sine)
Step 5: 2-3 months     (OzInverter)
────────────────────────────────────
Total:  7-13 months
```

### Budget
```
Step 1:   ~400 PLN  (37 projects + ESP32)
Step 2:    $40      (150W inverter)
Step 3:   $110      (500W inverter)
Step 4:   $150      (1kW inverter)
Step 5:   $600      (6kW inverter)
Tools:   ~300 PLN   (soldering station)
────────────────────────────────────
Total: ~$1,000 + 700 PLN (~$1,175)
```

---

## Part 1: Fundamentals (37 Projects)

### Phase 1: DC Fundamentals (6 projects)
- 1.1 LED + Resistor (Ohm's Law)
- 1.2 Voltage Divider (Sensing foundation)
- 1.3 Capacitor Basics (Energy storage)
- 1.4 Diode - One Way Flow
- 1.5 Switch Circuits (Logic basics)
- 1.6 Transistor as Switch (BJT control)

### Phase 2: Power Flow & SmartBob (9 projects)
- 2.1 Two-Battery Power Flow
- 2.2 Diode Enforces Direction
- 2.3 Transistor Bi-Directional Control
- 2.4 Voltage Droop Control (CRITICAL!)
- 2.5 Current Sensing Introduction
- 2.6 SmartBob First Light (ESP32 intro)
- 2.7 SmartBob Current Monitor (ACS712)
- 2.8 SmartBob Power Calculator
- 2.9 SmartBob + Home Assistant

### Phase 3: Oscillation & AC (5 projects)
- 3.1 What IS AC? (Manual demo)
- 3.2 RC Oscillator (Simplest timing)
- 3.3 555 Astable (Controlled frequency)
- 3.4 Frequency = Sound (Hear it!)
- 3.5 PWM - Pulse Width Modulation

### Phase 4: Build the Inverter (5 projects)
- 4.1 Push-Pull (Two transistors alternating)
- 4.2 H-Bridge Manual Control
- 4.3 H-Bridge + Oscillator = INVERTER!
- 4.4 Dead-Time (CRITICAL safety)
- 4.5 Add Transformer (Step up!)

### Phase 5: Mini Swarm Microgrid (6 projects)
- 5.1 Two Inverters, One AC Bus
- 5.2 Power Sharing Observation
- 5.3 Master/Slave Synchronization
- 5.4 Voltage Droop Control Demo
- 5.5 SmartBob Swarm Monitor
- 5.6 SmartBob + Home Assistant Dashboard

### Phase 6: Advanced Synchronization (6 projects)
- 6.1 Why Phase Matters
- 6.2 Phase Visualization (Two 555s)
- 6.3 Zero-Cross Detection
- 6.4 Frequency Droop Control (P-f)
- 6.5 Soft Parallel Connection
- 6.6 Full Island Grid Demo

---

## Part 2: Oscillator to Modified Sine (4 Phases)

### Phase 1: Oscillator Mastery
- CD4047 theory & testing
- Frequency calculation
- Complementary outputs

### Phase 2: First Inverter (150W)
- Push-pull topology
- MOSFET fundamentals
- Center-tap transformer

### Phase 3: H-Bridge & Gate Drivers
- IR2110 bootstrap circuit
- Dead-time concepts
- Shoot-through prevention

### Phase 4: Modified Sine (500W)
- SG3525 PWM controller
- H-bridge power stage
- Output filtering

---

## Part 3: SPWM to OzInverter (4 Phases)

### Phase 1: SPWM Theory
- Digital SPWM concept
- EG8010/EGS002/003
- LC filter design

### Phase 2: 1kW Pure Sine
- Complete inverter build
- Voltage feedback
- THD <3%

### Phase 3: High-Power Design
- MOSFET paralleling
- Transformer winding
- Thermal management
- Bus bar design

### Phase 4: OzInverter (6-15kW)
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

## Key Components

| Component | Step 2 | Step 3 | Step 4 | Step 5 |
|-----------|--------|--------|--------|--------|
| Controller | CD4047 | SG3525 | EG8010 | EG8010 |
| Gate Driver | Direct | IR2110 | IR2110 | IR2110 |
| MOSFETs | 2× IRF3205 | 4× IRF3205 | 4× IRFP460 | 16× IRFP4668 |
| Topology | Push-pull | H-bridge | H-bridge | H-bridge |
| Transformer | 100VA | 300VA | 500VA | 5000VA |
| Output | Square | Modified | Pure Sine | Pure Sine |

---

## Extensions

### Step 6: Smart Upgrade
| Feature | Implementation |
|---------|----------------|
| Remote monitoring | Home Assistant app |
| Display | OLED + Web dashboard |
| Sensors | Voltage, current, temperature |
| Alerts | Telegram / Push |
| ATS | Grid backup switching |

**Cost:** +$130-175

### Step 7: Microgrid (4 Households)
```
Architecture: Hybrid DC/AC
├── Cluster A (Houses 1-2): 48V DC bus
├── Cluster B (Houses 3-4): 48V DC bus
└── AC Tie (230V): Connects clusters
```

| Metric | Value |
|--------|-------|
| Total capacity | 12kW |
| Solar | 8kW |
| Battery | 30kWh |
| Per household | ~$3,578 |

### Libre Solar Integration
| Component | Commercial | DIY | Savings |
|-----------|------------|-----|---------|
| MPPT | $150-300 | ~$43 | 70-85% |
| BMS | $100-200 | ~$52 | 50-75% |

**Hardware:** `../hardware/libre_solar/`

---

## Resources

- **OzInverter:** bryanhorology.com/ozinverter.php
- **Libre Solar:** libre.solar
- **YouTube:** GreatScott!, EEVBlog, ElectroBOOM
- **Textbook:** AllAboutCircuits.com
- **Simulator:** Falstad Circuit Simulator

---

*Created: December 2024*
