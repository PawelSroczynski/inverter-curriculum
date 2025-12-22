# Inverter Curriculum Part 3
## Step 4: EG8010 Pure Sine Wave Inverter (1kW)
## Step 5: OzInverter (6-15kW)

---

# STEP 4: EG8010 PURE SINE WAVE INVERTER

**Output:** 12V/24V/48V DC → 220V AC (pure sine)
**Power:** 500W - 1kW
**Time:** 1-2 months
**THD:** <3% (clean sine wave)

---

## Step 4 Overview

```
┌────────────────────────────────────────────────────────────────────────┐
│                                                                        │
│ Battery → EG8010 → IR2110 → H-Bridge → Transformer → LC Filter → AC  │
│             │                   │                         │            │
│          SPWM               4 MOSFETs              Pure Sine          │
│        Generator                                    Output             │
│                                                                        │
└────────────────────────────────────────────────────────────────────────┘
```

### What's Different from Step 3?

| Step 3 (Modified Sine) | Step 4 (Pure Sine) |
|------------------------|-------------------|
| SG3525 (analog PWM) | EG8010 (digital SPWM) |
| 3-step waveform | Sinusoidal PWM |
| ~15% THD | <3% THD |
| OK for motors, tools | Safe for all electronics |
| Simpler filter | Requires proper LC filter |

---

## Phase 4.1: Understanding EG8010

### What is EG8010?

- Digital SPWM (Sinusoidal PWM) generator IC
- Creates pure sine wave through PWM
- Built-in dead-time control
- Adjustable frequency (50/60Hz)
- Soft-start feature
- Feedback input for voltage regulation

### EG8010 Pinout

```
        ┌──────────────────────┐
   SPWM │1    EG8010       24│ VCC
  KAPM  │2                 23│ DT2
  KAPME │3                 22│ DT1
    FB  │4                 21│ FS
   REF  │5                 20│ NC
  SPWME │6                 19│ CON
   VFB  │7                 18│ NFAULT
   VR   │8                 17│ NC
   OT   │9                 16│ NC
   OCP  │10                15│ LED
   OLP  │11                14│ SS
   GND  │12                13│ COM
        └──────────────────────┘
```

### Key Pins

| Pin | Name | Function |
|-----|------|----------|
| 1 | SPWM | PWM output (to IR2110) |
| 4 | FB | Feedback input |
| 10 | OCP | Overcurrent protection |
| 14 | SS | Soft-start |
| 21 | FS | Frequency select (50/60Hz) |
| 22-23 | DT | Dead-time adjust |

---

## Phase 4.2: SPWM Explained

### What is SPWM?

```
Traditional PWM (Modified Sine):
    ┌────┐    ┌────┐    ┌────┐
    │    │    │    │    │    │
────┘    └────┘    └────┘    └────
  Fixed duty cycle = stepped output

SPWM (Sinusoidal PWM):
█ ██ ███ ████ ███ ██ █      █ ██ ███
Varying duty cycle follows sine shape!

After LC filter:
       ╱╲       ╱╲
      ╱  ╲     ╱  ╲
─────╱    ╲───╱    ╲─────
      ╲  ╱     ╲  ╱
       ╲╱       ╲╱
    Clean sine wave!
```

### Why SPWM Creates Sine Wave

```
High duty cycle = High average voltage
Low duty cycle = Low average voltage

By varying duty cycle in sine pattern:
→ Average voltage follows sine wave
→ LC filter removes PWM carrier frequency
→ Output = pure sine wave
```

---

## Phase 4.3: LC Filter Design

### Why LC Filter?

```
SPWM output (before filter):
████ ██ █   █ ██ ████  (high frequency PWM)

LC filter removes high frequency:
      ╱╲
     ╱  ╲    ← Only 50Hz remains
────╱    ╲───
```

### Filter Calculation

```
Cutoff frequency: fc = 1 / (2π × √(L × C))

For 50Hz output with 20kHz PWM:
- Want fc between 50Hz and 20kHz
- Typical: fc = 1-3kHz

Example:
L = 2mH, C = 4.7µF
fc = 1 / (2π × √(0.002 × 0.0000047))
fc = 1 / (2π × 0.00000306)
fc ≈ 1640 Hz ✓
```

### Filter Schematic

```
From H-Bridge                    To Load
      │                            │
      │      L (2mH)               │
      ├────(○○○○)────────┬─────────┤
      │                  │         │
      │                [C]         │
      │              4.7µF/400V    │
      │                  │         │
      └──────────────────┴─────────┘
```

---

## Phase 4.4: Complete Schematic (Block Diagram)

```
                                +24V (or 48V)
                                    │
                            ┌───────┴───────┐
                            │               │
                            │   H-BRIDGE    │
                            │   ┌───┬───┐   │
                     HO1 ───┤───┤Q1 │ Q3├───┤─── HO2
                            │   └─┬─┴─┬─┘   │
                            │     │   │     │
                            │  ───┴───┴───  │
                            │    XFMR PRI   │
                            │   ───┬───┬─── │
                            │     │   │     │
                     LO1 ───┤───┤Q2 │ Q4├───┤─── LO2
                            │   └───┴───┘   │
                            │               │
                            └───────┬───────┘
                                    │
                                   GND

┌─────────────────────────────────────────────────────────┐
│                        EG8010                           │
│                                                         │
│  VCC (24) ────── +5V                                   │
│  GND (12) ────── GND                                   │
│                                                         │
│  SPWM (1) ──────┬──────────────────► IR2110 #1 HIN    │
│                 │                                       │
│                 └──► Inverter ──────► IR2110 #1 LIN   │
│                                                         │
│                 ┌──► Same logic ────► IR2110 #2 HIN   │
│                 │                                       │
│  (inverted) ────┴──────────────────► IR2110 #2 LIN    │
│                                                         │
│  FS (21) ─────── GND (50Hz) or VCC (60Hz)             │
│                                                         │
│  DT1 (22) ──┬── 15kΩ to GND (dead-time adjust)        │
│  DT2 (23) ──┘                                          │
│                                                         │
│  FB (4) ──────── Voltage feedback from output          │
│                                                         │
│  OCP (10) ────── Current sense (shunt resistor)        │
│                                                         │
│  SS (14) ─────── 10µF to GND (soft-start)             │
│                                                         │
└─────────────────────────────────────────────────────────┘

                            ┌────────────────────────┐
                            │      IR2110 × 2        │
                            │                        │
         From EG8010 ──────►│ HIN ──► HO ──► Q1,Q3  │
                            │                        │
         From EG8010 ──────►│ LIN ──► LO ──► Q2,Q4  │
                            │                        │
                            │ Bootstrap circuit      │
                            └────────────────────────┘

                            ┌────────────────────────┐
                            │   OUTPUT SECTION       │
                            │                        │
    Transformer ───────────►│ Secondary (220V)       │
    Secondary               │        │               │
                            │    LC Filter           │
                            │        │               │
                            │    220V AC OUT         │
                            └────────────────────────┘
```

---

## Phase 4.5: Bill of Materials

### Control Section

| Qty | Component | Value/Spec | Purpose | Est. Price |
|-----|-----------|------------|---------|------------|
| 1 | EG8010 | SPWM generator | Main control | $3 |
| 1 | IC Socket | 24-pin DIP | Protection | $0.50 |
| 2 | IR2110 | Gate driver | MOSFET drive | $2 ea |
| 2 | IC Socket | 14-pin DIP | Protection | $0.30 ea |
| 1 | 7805 | Voltage regulator | 5V for EG8010 | $0.30 |
| 1 | Crystal | 12MHz | EG8010 clock | $0.50 |
| 2 | Capacitor | 22pF ceramic | Crystal load | $0.10 |
| 1 | Resistor | 15kΩ | Dead-time | $0.05 |
| 1 | Resistor | 1kΩ | Various | $0.05 |
| 1 | Capacitor | 10µF/16V | Soft-start | $0.20 |
| 5 | Capacitor | 100nF ceramic | Bypass | $0.25 |
| 2 | Capacitor | 10µF/25V | Bootstrap | $0.30 |
| 2 | Diode | UF4007 | Bootstrap | $0.30 |

### Power Section

| Qty | Component | Value/Spec | Purpose | Est. Price |
|-----|-----------|------------|---------|------------|
| 4 | IRFP460 | MOSFET 500V/20A | H-bridge (high voltage) | $3 ea |
| OR | | | | |
| 4 | IRF3205 | MOSFET 55V/110A | H-bridge (low voltage) | $1.50 ea |
| 2 | Heatsink | Large aluminum | Cooling | $8 ea |
| 1 | Transformer | 24-0-24V / 220V, 500VA | OR toroidal | $30 |
| 4 | Capacitor | 2200µF/63V | Input filter | $2 ea |
| 4 | Diode | MUR1560 | Freewheeling | $1 ea |
| 1 | Shunt resistor | 0.01Ω / 5W | Current sense | $1 |

### LC Filter (Output)

| Qty | Component | Value/Spec | Purpose | Est. Price |
|-----|-----------|------------|---------|------------|
| 1 | Inductor | 2mH / 10A | Filter L | $8 |
| 2 | Capacitor | 4.7µF/400V | Filter C (film type) | $2 ea |

### Feedback & Protection

| Qty | Component | Value/Spec | Purpose | Est. Price |
|-----|-----------|------------|---------|------------|
| 1 | Optocoupler | PC817 | Isolation | $0.30 |
| 1 | Resistor | 470kΩ 1W | Voltage sense | $0.20 |
| 1 | Resistor | 10kΩ | Voltage divider | $0.05 |
| 1 | Zener | 5.1V | Reference clamp | $0.10 |
| 1 | NTC Thermistor | 10kΩ | Temp sense | $0.50 |
| 1 | Comparator | LM393 | OVP/OCP | $0.30 |
| 2 | LED | Red, Green | Status | $0.20 |

### Mechanical & Wiring

| Qty | Component | Value/Spec | Purpose | Est. Price |
|-----|-----------|------------|---------|------------|
| - | Wire | 8 AWG | Power (high current) | $8 |
| - | Wire | 18 AWG | Signal | $3 |
| 2 | Fan | 80mm 12V | Cooling | $5 ea |
| 1 | PCB | Double-sided | Control board | $10 |
| 1 | Enclosure | Aluminum, ventilated | Housing | $25 |
| 1 | Terminal strip | High current | Connections | $5 |
| 1 | Thermal paste | - | Heatsink | $2 |
| - | Hardware | Screws, standoffs | Mounting | $5 |

---

## Step 4 BOM Summary

| Category | Est. Total |
|----------|------------|
| EG8010 + support | $8 |
| Gate drivers | $6 |
| MOSFETs | $6-12 |
| Transformer | $30 |
| Input capacitors | $8 |
| Output filter | $12 |
| Protection/feedback | $3 |
| Heatsinks & cooling | $26 |
| Mechanical | $45 |

**Step 4 Total: ~$144-150**

---

## Phase 4.6: Build Sequence

### Week 1-2: Control Board
1. Build 5V power supply section
2. Install EG8010, verify 50Hz output
3. Build IR2110 driver section
4. Test gate signals with scope

### Week 3: Power Stage
1. Mount MOSFETs on heatsink
2. Wire H-bridge carefully
3. Add freewheeling diodes
4. Test with LOW voltage first (12V)

### Week 4: Transformer & Filter
1. Connect transformer
2. Build LC output filter
3. Test output waveform

### Week 5-6: Integration
1. Add feedback circuit
2. Add protection circuits
3. Full load testing
4. Enclosure and final assembly

---

## Phase 4.7: Testing Procedure

### Stage 1: Control Board Only
```
1. Power with 5V (no power stage)
2. Measure EG8010 outputs with scope
3. Verify SPWM waveform
4. Check dead-time between pulses
```

### Stage 2: Low Voltage Test
```
1. Connect 12V battery (not full 24/48V)
2. Connect small transformer
3. Measure output voltage and waveform
4. Should see rough sine wave
```

### Stage 3: Full Voltage Test
```
1. Connect full battery bank
2. Test with 40W bulb first
3. Gradually increase load
4. Monitor temperature
5. Measure THD if possible (<5% is good)
```

---

## Phase 4.8: Learning Outcomes

After completing Step 4:
- [ ] Understand SPWM operation
- [ ] Can use EG8010 controller
- [ ] Know LC filter design
- [ ] Understand voltage feedback
- [ ] Can build 1kW pure sine inverter
- [ ] Ready for OzInverter!

---

# STEP 5: OZINVERTER (6-15kW)

**Output:** 48V DC → 220V AC (pure sine wave)
**Power:** 6kW continuous, up to 15kW peak
**Time:** 2-3 months
**Source:** https://www.bryanhorology.com/ozinverter.php

---

## Step 5 Overview

```
┌────────────────────────────────────────────────────────────────────────────┐
│                                                                            │
│  48V Battery Bank → Control Board → Power Board → Toroid → LC Filter → AC │
│         │              │               │            │                      │
│     800-1000Ah      EG8010          8×MOSFETs   Hand-wound               │
│                    + Drivers        (parallel)   26-30kg                   │
│                                                                            │
└────────────────────────────────────────────────────────────────────────────┘
```

---

## Phase 5.1: OzInverter Architecture

### System Blocks

```
┌─────────────────────────────────────────────────────────────────┐
│                      CONTROL BOARD                              │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐           │
│  │ EG8010  │  │ IR2110  │  │ IR2110  │  │ Voltage │           │
│  │  SPWM   │──│ Driver  │──│ Driver  │──│   Reg   │           │
│  │Generator│  │   #1    │  │   #2    │  │  5V/12V │           │
│  └─────────┘  └─────────┘  └─────────┘  └─────────┘           │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                       POWER BOARD                               │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │                      H-BRIDGE                             │  │
│  │   Q1A  Q1B  Q1C  Q1D        Q3A  Q3B  Q3C  Q3D          │  │
│  │   ═══  ═══  ═══  ═══        ═══  ═══  ═══  ═══          │  │
│  │     │    │    │    │          │    │    │    │          │  │
│  │     └────┴────┴────┴────┬─────┴────┴────┴────┘          │  │
│  │                         │                                │  │
│  │                    TRANSFORMER                           │  │
│  │                      PRIMARY                             │  │
│  │                         │                                │  │
│  │     ┌────┬────┬────┬────┴────┬────┬────┬────┐          │  │
│  │     │    │    │    │         │    │    │    │          │  │
│  │   Q2A  Q2B  Q2C  Q2D        Q4A  Q4B  Q4C  Q4D          │  │
│  │   ═══  ═══  ═══  ═══        ═══  ═══  ═══  ═══          │  │
│  └──────────────────────────────────────────────────────────┘  │
│         8 MOSFETs per leg = 32 total (or 16 minimum)          │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                    TOROIDAL TRANSFORMER                         │
│           ┌───────────────────────────────────┐                │
│           │         ╭─────────────╮           │                │
│           │        ╱               ╲          │                │
│           │       │    26-30 kg    │          │                │
│           │       │    TOROID      │          │                │
│           │        ╲    CORE      ╱           │                │
│           │         ╰─────────────╯           │                │
│           │                                   │                │
│           │   Primary: ~12 turns (48V side)   │                │
│           │   Secondary: ~180 turns (220V)    │                │
│           └───────────────────────────────────┘                │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                       LC FILTER                                 │
│           L = 1-2mH @ 50A        C = 10-20µF/400V              │
│           (custom wound)          (film capacitors)             │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
                        220V AC OUTPUT
                         (6-15 kW)
```

---

## Phase 5.2: Bill of Materials

### Control Board Components

| Qty | Component | Value/Spec | Purpose | Est. Price |
|-----|-----------|------------|---------|------------|
| 1 | EG8010 | SPWM IC | Main controller | $3 |
| 2 | IR2110 | Gate driver | MOSFET drive | $2 ea |
| 2 | TLP250 | Isolated driver | Alternative driver | $2 ea |
| 1 | 7812 | Regulator | 12V supply | $0.50 |
| 1 | 7805 | Regulator | 5V supply | $0.30 |
| 1 | Crystal | 12MHz | EG8010 clock | $0.50 |
| 10 | Capacitor | 100nF ceramic | Bypass | $0.50 |
| 5 | Capacitor | 10µF/25V | Filter | $0.50 |
| 2 | Capacitor | 22pF | Crystal | $0.10 |
| 10 | Resistor | Various | Timing, sense | $0.50 |
| 1 | PCB | OzInverter control | Ready-made or DIY | $15 |

**Control Board Subtotal: ~$30**

### Power Board Components

| Qty | Component | Value/Spec | Purpose | Est. Price |
|-----|-----------|------------|---------|------------|
| 16 | IRFP4668 | MOSFET 200V/130A | H-bridge (preferred) | $5 ea |
| OR | | | | |
| 16 | IRFP260N | MOSFET 200V/50A | H-bridge (budget) | $3 ea |
| 8 | Gate resistor | 4.7Ω 1W | Current limit | $1 |
| 16 | Zener | 18V | Gate protection | $1 |
| 8 | Capacitor | 4700µF/63V | Input filter | $4 ea |
| 8 | Diode | MUR3060 | Freewheeling | $2 ea |
| 4 | Bus bar | Copper, 5mm thick | High current | $10 |
| 2 | Heatsink | 300×100×40mm | Cooling | $20 ea |
| 1 | PCB | OzInverter power | Double copper | $25 |
| 1 | Thermal paste | Large tube | Heatsink contact | $5 |

**Power Board Subtotal: ~$170-200**

### Toroidal Transformer

| Qty | Component | Value/Spec | Purpose | Est. Price |
|-----|-----------|------------|---------|------------|
| 1 | Toroid core | OD 200mm, ID 120mm, H 80mm | Transformer base | $40-60 |
| 1 | Primary wire | 8 AWG (8mm²) ~5m | 48V winding | $15 |
| 1 | Secondary wire | 14 AWG (2.5mm²) ~50m | 220V winding | $20 |
| 1 | Insulation tape | Kapton or glass tape | Layer insulation | $10 |
| 1 | Varnish | Transformer varnish | Final coating | $10 |

**Transformer Subtotal: ~$95-115**

*Alternative: Buy pre-wound toroidal 48V/220V 5kVA = ~$150-200*

### LC Output Filter

| Qty | Component | Value/Spec | Purpose | Est. Price |
|-----|-----------|------------|---------|------------|
| 1 | Inductor core | Ferrite or powder | Filter L base | $15 |
| - | Inductor wire | 10 AWG ~10m | Winding | $10 |
| 4 | Capacitor | 4.7µF/400V film | Filter C | $3 ea |
| OR | | | | |
| 1 | Ready inductor | 1.5mH/50A | Pre-wound | $30 |

**Filter Subtotal: ~$40-50**

### Protection & Monitoring

| Qty | Component | Value/Spec | Purpose | Est. Price |
|-----|-----------|------------|---------|------------|
| 1 | Current sensor | ACS758 100A | Overcurrent | $8 |
| 1 | Voltage sensor | Resistor divider | Overvoltage | $1 |
| 2 | Relay | 48V/80A | Disconnect | $15 ea |
| 1 | Fuse holder | ANL | Main fuse | $5 |
| 2 | Fuse | 150A ANL | Protection | $5 ea |
| 1 | NTC | 10kΩ | Temperature | $1 |
| 1 | Fan controller | PWM based | Auto cooling | $5 |
| 2 | Thermistor | On heatsink | Temp monitor | $2 |

**Protection Subtotal: ~$60**

### Cooling System

| Qty | Component | Value/Spec | Purpose | Est. Price |
|-----|-----------|------------|---------|------------|
| 4 | Fan | 120mm 12V | Forced air | $5 ea |
| 1 | Fan guard | 120mm | Safety | $2 |
| 1 | Temp controller | Digital | Auto control | $10 |
| - | Air duct | DIY from sheet | Airflow | $5 |

**Cooling Subtotal: ~$40**

### Enclosure & Mechanical

| Qty | Component | Value/Spec | Purpose | Est. Price |
|-----|-----------|------------|---------|------------|
| 1 | Enclosure | 400×300×200mm metal | Housing | $50 |
| - | Terminal blocks | 100A rated | Connections | $15 |
| - | Cable glands | Various sizes | Wire entry | $10 |
| 1 | Display | Voltage/Current | Monitoring | $15 |
| - | Wire | 4 AWG, 8 AWG | Internal | $20 |
| - | Hardware | Bolts, nuts, standoffs | Assembly | $15 |
| 1 | AC outlet | 15A | Output | $5 |
| 1 | DC connector | Anderson 175A | Battery input | $10 |

**Mechanical Subtotal: ~$140**

---

## Step 5 Complete BOM Summary

| Category | Est. Total |
|----------|------------|
| Control board | $30 |
| Power board (MOSFETs, PCB) | $170-200 |
| Transformer (DIY) | $95-115 |
| LC filter | $40-50 |
| Protection | $60 |
| Cooling | $40 |
| Enclosure & mechanical | $140 |

**Step 5 Total: ~$575-635**

*Note: Buying some parts pre-assembled can cost more but saves time*

---

## Phase 5.3: Transformer Winding Guide

### Calculating Turns

```
For 48V input, 220V output, 50Hz:

Volts per turn = 4.44 × f × Ae × Bmax × 10⁻⁸

Where:
- f = 50 Hz
- Ae = Core cross-section (cm²)
- Bmax = 1.2-1.5 Tesla (for toroid)

For typical 5kVA toroid (Ae ≈ 30cm²):
V/turn ≈ 4.44 × 50 × 30 × 1.3 × 10⁻⁴ ≈ 4V/turn

Primary: 48V ÷ 4 = 12 turns
Secondary: 220V ÷ 4 = 55 turns

(Actual values depend on YOUR specific core!)
```

### Winding Steps

**Step 1: Prepare Core**
```
1. Clean toroid surface
2. Wrap with insulation tape (2 layers)
3. Mark start point
```

**Step 2: Wind Primary**
```
1. Use thick wire (8 AWG / 8mm²)
2. Wind 12 turns evenly spaced
3. For center-tap: wind 6 turns, tap, wind 6 more
4. Secure ends, add insulation layer
```

**Step 3: Wind Secondary**
```
1. Use thinner wire (14 AWG / 2.5mm²)
2. Wind ~55 turns (adjust based on test)
3. Keep turns tight and even
4. Add insulation between layers
```

**Step 4: Test & Adjust**
```
1. Apply LOW voltage to primary (12V)
2. Measure secondary voltage
3. Calculate actual turns ratio
4. Add/remove turns if needed
```

---

## Phase 5.4: Build Sequence

### Month 1: Preparation & Control

**Week 1-2:**
- Order all components
- Study OzInverter documentation thoroughly
- Join OzInverter forum for support

**Week 3-4:**
- Build control board
- Test EG8010 output
- Verify gate driver signals

### Month 2: Power Stage & Transformer

**Week 1:**
- Wind transformer (or receive pre-wound)
- Test transformer ratio

**Week 2-3:**
- Assemble power board
- Mount MOSFETs (careful with static!)
- Wire bus bars

**Week 4:**
- Build LC filter
- Connect all boards
- Initial power-up tests

### Month 3: Integration & Testing

**Week 1:**
- Low voltage testing (24V input)
- Verify waveforms
- Check for heating issues

**Week 2:**
- Full voltage testing (48V)
- Light load tests (500W)
- Thermal monitoring

**Week 3:**
- Increase load gradually
- Full power test (6kW)
- Run for extended periods

**Week 4:**
- Final assembly in enclosure
- Safety checks
- Documentation

---

## Phase 5.5: Safety Requirements

### Electrical Safety

```
⚠️ DANGER - 48V DC at 150A = 7200W potential!
⚠️ DANGER - 220V AC output is LETHAL!

REQUIRED:
□ Isolate battery before ANY work
□ Use insulated tools
□ Discharge capacitors before touching
□ Never work alone
□ Keep fire extinguisher nearby
□ Use proper fusing
□ Ground enclosure properly
```

### Thermal Safety

```
MOSFETs can reach 150°C failure point!

REQUIRED:
□ Thermal paste on ALL MOSFETs
□ Torque heatsink screws properly
□ Verify fan operation before load
□ Thermal shutdown protection
□ Monitor temperature during testing
```

---

## Phase 5.6: Testing Protocol

### Stage 1: No Power
```
□ Visual inspection of all solder joints
□ Continuity test all connections
□ Check for shorts (power rails)
□ Verify MOSFET orientation
```

### Stage 2: Control Board Only (5V/12V)
```
□ EG8010 outputs SPWM
□ Gate drivers produce proper signals
□ Dead-time visible on scope
□ No abnormal heating
```

### Stage 3: Low Voltage (12V Battery)
```
□ Connect small test transformer
□ Verify AC output
□ Check waveform shape
□ Note any oscillation or noise
```

### Stage 4: Medium Voltage (24V)
```
□ Connect proper transformer
□ Test with 100W load
□ Monitor all temperatures
□ Run for 30 minutes
```

### Stage 5: Full Voltage (48V)
```
□ Light load (500W) - 1 hour
□ Medium load (2kW) - 30 min
□ High load (4kW) - 15 min
□ Full load (6kW) - 5 min
□ Monitor THD if possible
```

---

## Phase 5.7: Learning Outcomes

After completing Step 5:
- [ ] Can build production-quality power electronics
- [ ] Understand transformer design and winding
- [ ] Know thermal management for high power
- [ ] Can troubleshoot complex power systems
- [ ] Achieved 6kW+ pure sine wave inverter!

---

# COMPLETE CURRICULUM SUMMARY

## Total Investment

| Step | Components | Time |
|------|------------|------|
| Step 1 (Fundamentals) | ~$64 | 8-12 weekends |
| Step 2 (CD4047 150W) | ~$40 | 2-3 weekends |
| Step 3 (SG3525 500W) | ~$110 | 3-4 weekends |
| Step 4 (EG8010 1kW) | ~$150 | 1-2 months |
| Step 5 (OzInverter 6kW) | ~$600 | 2-3 months |
| **Tools** (if needed) | ~$300 | — |

**Grand Total: ~$1,264**
**Time: 6-12 months**

---

## Skills Progression Map

```
START                                                          FINISH
  │                                                              │
  ▼                                                              ▼
┌─────┐   ┌─────┐   ┌─────┐   ┌─────┐   ┌─────┐   ┌──────────┐
│Ohm's│──►│Trans│──►│555  │──►│Push-│──►│H-   │──►│Pure Sine │
│Law  │   │istor│   │Timer│   │Pull │   │Bridge│   │6-15kW    │
└─────┘   └─────┘   └─────┘   └─────┘   └─────┘   └──────────┘
  │         │         │         │         │           │
  │         │         │         │         │           │
Step 1    Step 1    Step 1    Step 2    Step 3     Step 4-5
Phase 1   Phase 1   Phase 2
```

---

## Final Notes

### You WILL Succeed If You:
- Follow the steps in order
- Don't skip projects
- Ask for help when stuck
- Accept some failures as learning
- Take your time

### Resources:
- OzInverter Forum: Active community support
- YouTube: GreatScott!, EEVBlog, ElectroBOOM
- This curriculum: Your roadmap

**Good luck on your journey from LED blinker to 6kW inverter!**
