# Inverter Curriculum Part 3
## From SPWM Theory to Production-Quality OzInverter

**Output:** 24/48V DC → 220V AC (Pure Sine Wave)
**Power Range:** 1kW → 6-15kW
**Time:** 3-5 months

---

## Part 3 Overview

```
Phase 1              Phase 2              Phase 3              Phase 4
SPWM Theory          1kW Pure Sine        High-Power           OzInverter
& EG8010             Build                Design               Build
   │                     │                    │                    │
   ▼                     ▼                    ▼                    ▼
┌─────────┐         ┌─────────┐         ┌─────────┐         ┌─────────┐
│ Digital │────────►│ Working │────────►│ Scaling │────────►│ 6-15kW  │
│ SPWM    │         │ 1kW     │         │ Up      │         │ System  │
│ Filters │         │ Inverter│         │ Thermal │         │ Complete│
└─────────┘         └─────────┘         └─────────┘         └─────────┘
```

---

# PHASE 1: SPWM THEORY & EG8010

**Goal:** Understand digital SPWM and the EG8010 controller
**Time:** 1-2 weekends (theory + breadboard testing)
**Cost:** ~$15

---

## 1.1 What is SPWM?

### The Problem with Modified Sine

```
Modified Sine (from Part 2):
    ┌──┐    ┌──┐
┌───┘  └────┘  └───┐
│                  │
└───┐      ┌───┘
    └──┘    └──┘

Problems:
- ~15% Total Harmonic Distortion (THD)
- Causes humming in audio equipment
- Motors run hot
- Some electronics won't work properly
```

### Pure Sine is the Goal

```
Pure Sine Wave:
       ╱╲       ╱╲
      ╱  ╲     ╱  ╲
─────╱    ╲───╱    ╲─────
      ╲  ╱     ╲  ╱
       ╲╱       ╲╱

Benefits:
- <3% THD (same as grid power)
- All electronics work perfectly
- Motors run cool and quiet
- Professional-grade output
```

### How SPWM Creates Pure Sine

```
SPWM = Sinusoidal Pulse Width Modulation

High-frequency PWM with VARYING duty cycle:

Near zero-crossing (narrow pulses):
   ┌┐   ┌┐   ┌┐   ┌┐
───┘└───┘└───┘└───┘└───
   Low average voltage

Near peak (wide pulses):
┌────┐ ┌────┐ ┌────┐
│    │ │    │ │    │
┘    └─┘    └─┘    └
   High average voltage

After LC filter → smooth sine wave!
```

---

## 1.2 Understanding EG8010

### What is EG8010?

```
EG8010 = Digital SPWM Generator IC

Features:
┌────────────────────────────────────────┐
│ • Pure sine SPWM output                │
│ • <1.5% THD with proper filter         │
│ • Built-in dead-time (~300ns)          │
│ • 50Hz or 60Hz selectable              │
│ • Voltage feedback input               │
│ • Overcurrent protection               │
│ • Soft-start                           │
│ • LED status output                    │
│ • Single 5V supply                     │
└────────────────────────────────────────┘

This is the SAME IC used in OzInverter!
```

### EG8010 Pinout

```
        ┌──────────────────────┐
   SPWM │1    EG8010       24│ VCC (+5V)
  KAPM  │2                 23│ DT2 (dead-time adjust)
  KAPME │3                 22│ DT1 (dead-time adjust)
    FB  │4                 21│ FS (frequency select)
   REF  │5                 20│ NC
  SPWME │6                 19│ CON
   VFB  │7                 18│ NFAULT
   VR   │8                 17│ NC
   OT   │9                 16│ NC
   OCP  │10                15│ LED
   OLP  │11                14│ SS (soft-start)
   GND  │12                13│ COM
        └──────────────────────┘
```

### Key Pins Explained

| Pin | Name | Function |
|-----|------|----------|
| 1 | SPWM | PWM output for one H-bridge leg |
| 4 | FB | Feedback input (voltage regulation) |
| 7 | VFB | Voltage feedback sense |
| 10 | OCP | Overcurrent protection input |
| 14 | SS | Soft-start (add capacitor) |
| 15 | LED | Status LED output |
| 21 | FS | Frequency: GND=50Hz, VCC=60Hz |
| 22-23 | DT1/DT2 | Dead-time adjustment |

---

## 1.3 EG8010 vs EGS002/EGS003 Boards

### Option A: Bare EG8010 IC

```
Pros:
+ Cheapest ($3)
+ Full learning experience
+ Complete customization

Cons:
- Need 5V regulator
- Need crystal circuit
- More complex build
- More chances for errors
```

### Option B: EGS002/EGS003 Module

```
EGS002/EGS003 = Pre-built EG8010 + IR2110 driver board

┌─────────────────────────────────────┐
│  ┌─────┐  ┌─────┐  ┌─────┐        │
│  │EG8010│ │IR2110│ │IR2110│        │
│  └─────┘  └─────┘  └─────┘        │
│                                    │
│  [Crystal] [Regulators] [Caps]    │
│                                    │
│  ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○ ○         │ ← Connections
└─────────────────────────────────────┘

Pros:
+ Ready to use ($8-12)
+ Tested and working
+ Fewer components to buy
+ Good for first build

Cons:
- Less learning (black box)
- Limited customization
- Can't repair if IC fails

RECOMMENDATION: Use EGS002/003 for first build,
                bare EG8010 for OzInverter
```

---

## 1.4 LC Filter Design

### Why LC Filter is Critical

```
EG8010 output (before filter):
████ ██ █   █ ██ ████ ████ ██ █   (23.4kHz PWM)

Without filter: High-frequency noise, not usable!

After LC filter:
       ╱╲       ╱╲
      ╱  ╲     ╱  ╲     (50Hz sine)
─────╱    ╲───╱    ╲─────

LC filter removes PWM carrier, leaves only 50Hz
```

### Filter Calculation

```
Cutoff frequency formula:
fc = 1 / (2π × √(L × C))

Design goals:
- Pass 50Hz (and harmonics up to ~500Hz)
- Block 23.4kHz PWM carrier
- Typical fc = 1-3 kHz

Example calculation:
L = 2mH, C = 4.7µF

fc = 1 / (2π × √(0.002 × 0.0000047))
fc = 1 / (2π × 0.00000306)
fc = 1 / 0.0000192
fc ≈ 1640 Hz ✓ (between 50Hz and 23kHz)
```

### Filter Circuit

```
From H-Bridge output:
      │
      │      L (2mH inductor)
      ├────(○○○○○○)────┬────────► To Load
      │                 │
      │               [C]
      │            4.7µF/400V
      │                 │
      └─────────────────┴────────► Neutral

Component requirements:
- Inductor: Must handle full load current!
  For 1kW at 220V: I = 1000/220 = 4.5A
  Use 2mH rated for 10A minimum

- Capacitor: Must handle AC voltage!
  Use film capacitor, NOT electrolytic
  Voltage rating: 400V or higher (for 220V output)
```

---

## 1.5 Breadboard Testing

### Test Circuit Components

| Qty | Component | Value | Purpose |
|-----|-----------|-------|---------|
| 1 | EGS002 or EGS003 | Module | SPWM + drivers |
| 1 | 7812 | Regulator | 12V for IR2110 |
| 1 | Capacitor | 100µF/25V | Power filter |
| 3 | Capacitor | 100nF | Bypass |
| 2 | LED + 1kΩ | Any color | Output indicators |
| 1 | Breadboard | - | Assembly |
| - | Wire | 22 AWG | Connections |

### Test Setup

```
+15-24V Input
    │
   [7812]─────────────► +12V to EGS module
    │
   ─┴─ 100µF
    │
   GND

EGS Module Connections:
┌─────────────────────────────────┐
│                                 │
│  +12V ───► VCC                 │
│  GND ────► GND                 │
│                                 │
│  GND ────► FS (for 50Hz)       │
│                                 │
│  HO1 ────► LED + 1kΩ ──► GND   │  Test outputs
│  LO1 ────► LED + 1kΩ ──► GND   │
│                                 │
└─────────────────────────────────┘

What to observe:
- LEDs should pulse at high frequency
- With oscilloscope: see SPWM pattern
- Both outputs should be complementary
```

---

## 1.6 Phase 1 Learning Outcomes

After completing Phase 1:
- [ ] Understand SPWM concept
- [ ] Know EG8010 pinout and function
- [ ] Can design LC filter
- [ ] Tested EGS module on breadboard
- [ ] Ready to build 1kW inverter!

---

# PHASE 2: 1kW PURE SINE BUILD

**Goal:** Build a complete 1kW pure sine wave inverter
**Time:** 3-4 weekends
**Cost:** ~$150

---

## 2.1 System Architecture

```
┌────────────────────────────────────────────────────────────────────┐
│                         1kW INVERTER                                │
│                                                                     │
│  24/48V ──► EG8010 ──► IR2110 ──► H-Bridge ──► XFMR ──► Filter ──► 220V
│  Battery    (SPWM)    (Drivers)   (4 FETs)   (Step-up)  (LC)       AC
│                                                                     │
└────────────────────────────────────────────────────────────────────┘
```

### Block Diagram

```
┌─────────────────────────────────────────────────────────────────────┐
│                        CONTROL SECTION                               │
│  ┌─────────┐    ┌─────────┐    ┌─────────┐                         │
│  │  5V     │    │ EG8010  │    │ IR2110  │                         │
│  │ Regulator│───►│  SPWM   │───►│ Drivers │                         │
│  └─────────┘    └─────────┘    └─────────┘                         │
│                       │              │                              │
│                  Feedback        Gate signals                       │
│                       │              │                              │
└───────────────────────┼──────────────┼──────────────────────────────┘
                        │              │
                        │              ▼
┌───────────────────────┼──────────────────────────────────────────────┐
│                       │      POWER SECTION                           │
│                       │                                              │
│     +24/48V ──────────┼───────────────────┐                         │
│         │             │                   │                         │
│         │    ┌────────┴────────┐          │                         │
│         │    │    H-BRIDGE     │          │                         │
│         │    │   Q1       Q3   │          │                         │
│         │    │    \       /    │          │                         │
│         │    │     ├─XFMR─┤    │──────────┤                         │
│         │    │    /       \    │          │                         │
│         │    │   Q2       Q4   │          │                         │
│         │    └─────────────────┘          │                         │
│         │                                 │                         │
│        GND ───────────────────────────────┘                         │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌──────────────────────────────────────────────────────────────────────┐
│                       OUTPUT SECTION                                  │
│                                                                       │
│   XFMR Secondary ───► [Inductor] ───┬───► 220V AC Output            │
│                                     │                                │
│                              [Capacitor]                             │
│                                     │                                │
│                                    GND                               │
│                                                                       │
└──────────────────────────────────────────────────────────────────────┘
```

---

## 2.2 Component Selection

### MOSFETs for 1kW

```
For 1kW at 24V input:
Current = 1000W / 24V = 42A
Peak current = 42 × 1.4 = 59A

MOSFET requirements:
- Vds > 60V (2× input voltage minimum)
- Id > 60A (peak current with margin)
- Low Rds(on) (for efficiency)

Good choices:
┌────────────┬──────────┬───────────┬───────────┐
│ MOSFET     │ Vds      │ Id        │ Rds(on)   │
├────────────┼──────────┼───────────┼───────────┤
│ IRFP260N   │ 200V     │ 50A       │ 40mΩ      │
│ IRFP4110   │ 100V     │ 180A      │ 3.7mΩ     │ ← Good choice
│ IRFP4668   │ 200V     │ 130A      │ 8mΩ       │ ← Best for 48V
│ IRF3205    │ 55V      │ 110A      │ 8mΩ       │ ← Budget 24V
└────────────┴──────────┴───────────┴───────────┘
```

### Transformer

```
For 1kW inverter:
┌───────────────────────────────────────────┐
│ Rating: 1000VA minimum (1200VA better)    │
│                                           │
│ Primary (for 24V input):                  │
│   Voltage: 24V (no center tap needed)     │
│   Current: 42A minimum                    │
│   Wire: 6 AWG or thicker                  │
│                                           │
│ Primary (for 48V input):                  │
│   Voltage: 48V                            │
│   Current: 21A minimum                    │
│   Wire: 10 AWG or thicker                 │
│                                           │
│ Secondary:                                │
│   Voltage: 220-230V                       │
│   Current: 5A minimum                     │
│   Wire: 14 AWG                            │
│                                           │
│ Type: Toroidal preferred (more efficient) │
└───────────────────────────────────────────┘
```

---

## 2.3 Complete Schematic

```
                                      +24V (or 48V) from Battery Bank
                                              │
              ┌───────────────────────────────┼──────────────────────────┐
              │                               │                          │
              │                        ┌──────┴──────┐                   │
              │                        │ INPUT CAPS  │                   │
              │                        │ 4×2200µF    │                   │
              │                        └──────┬──────┘                   │
              │                               │                          │
              │  ┌─────────────────────────── │──────────────────────────┤
              │  │                            │                          │
              │  │  EGS002/003 MODULE         │                          │
              │  │  ┌─────────────────────┐   │                          │
              │  │  │                     │   │                          │
       +12V───┼──┼──┤ VCC           HO1 ─┼───┼──[10Ω]──► Q1 Gate        │
              │  │  │                     │   │                          │
              │  │  │              Vs1 ─┼───┼──────────► Q1 Source      │
              │  │  │                     │   │          │               │
              │  │  │              LO1 ─┼───┼──[10Ω]──► Q2 Gate        │
              │  │  │                     │   │                          │
              │  │  │              COM ─┼───┼──────────► GND            │
              │  │  │                     │   │                          │
              │  │  │              HO2 ─┼───┼──[10Ω]──► Q3 Gate        │
              │  │  │                     │   │                          │
              │  │  │              Vs2 ─┼───┼──────────► Q3 Source      │
              │  │  │                     │   │          │               │
              │  │  │              LO2 ─┼───┼──[10Ω]──► Q4 Gate        │
              │  │  │                     │   │                          │
        GND───┼──┼──┤ GND                 │   │                          │
              │  │  │                     │   │                          │
              │  │  │  FS ─── GND (50Hz)  │   │                          │
              │  │  │                     │   │                          │
              │  │  │  FB ─── Feedback    │   │                          │
              │  │  │                     │   │                          │
              │  │  └─────────────────────┘   │                          │
              │  │                            │                          │
              │  └────────────────────────────┘                          │
              │                                                          │
              │                    H-BRIDGE                              │
              │         ┌──────────────────────────────┐                 │
              │         │                              │                 │
              │         │    Q1          Q3            │                 │
              │         │     │           │            │                 │
              │         │     └────┬──────┘            │                 │
              │         │          │                   │                 │
              │         │      TRANSFORMER             │                 │
              │         │       PRIMARY                │                 │
              │         │          │                   │                 │
              │         │     ┌────┴──────┐            │                 │
              │         │     │           │            │                 │
              │         │    Q2          Q4            │                 │
              │         │     │           │            │                 │
              │         └─────┴───────────┴────────────┘                 │
              │                   │                                      │
              └───────────────────┴──────────────────────────────────────┘
                                  │
                                 GND


                           OUTPUT SECTION
           ┌──────────────────────────────────────────┐
           │                                          │
           │  TRANSFORMER    LC FILTER      OUTPUT    │
           │  SECONDARY                               │
           │      │                                   │
           │      │     L (2mH)                       │
           │      ├───(○○○○○○)────┬──────► 220V AC   │
           │      │                │        LIVE      │
           │      │               [C]                 │
           │      │            4.7µF/400V            │
           │      │                │                  │
           │      └────────────────┴──────► NEUTRAL   │
           │                                          │
           └──────────────────────────────────────────┘


                         FEEDBACK CIRCUIT
           ┌──────────────────────────────────────────┐
           │                                          │
           │  220V AC ───[470kΩ]───┬───► FB pin      │
           │                       │                  │
           │                    [10kΩ]                │
           │                       │                  │
           │                      GND                 │
           │                                          │
           │  This divides 220V down to ~5V          │
           │  for the EG8010 feedback input          │
           │                                          │
           └──────────────────────────────────────────┘
```

---

## 2.4 Bill of Materials

### Control Section

| Qty | Component | Value/Spec | Purpose | Est. Price |
|-----|-----------|------------|---------|------------|
| 1 | EGS002 or EGS003 | Module | SPWM + drivers | $10 |
| 1 | 7812 | Regulator | 12V supply | $0.50 |
| 1 | 7805 | Regulator | 5V supply | $0.30 |
| 4 | Capacitor | 100nF ceramic | Bypass | $0.20 |
| 2 | Capacitor | 100µF/25V | Power filter | $0.40 |
| 4 | Resistor | 10Ω 1W | Gate resistors | $0.30 |
| 1 | Resistor | 470kΩ 1W | Feedback divider | $0.10 |
| 1 | Resistor | 10kΩ | Feedback divider | $0.05 |

### Power Section

| Qty | Component | Value/Spec | Purpose | Est. Price |
|-----|-----------|------------|---------|------------|
| 4 | IRFP4668 | MOSFET 200V/130A | H-bridge | $4 ea |
| OR | IRFP260N | MOSFET 200V/50A | Budget option | $2 ea |
| 4 | Capacitor | 2200µF/63V | Input filter | $2 ea |
| 4 | Diode | MUR1560 | Freewheeling | $1 ea |
| 2 | Heatsink | 150×100mm aluminum | Cooling | $10 ea |
| 1 | Thermal paste | Large tube | Heatsink mounting | $3 |

### Transformer

| Qty | Component | Value/Spec | Purpose | Est. Price |
|-----|-----------|------------|---------|------------|
| 1 | Toroidal xfmr | 48V/220V, 1kVA | Step-up | $40-60 |
| OR | EI transformer | 48V/220V, 1kVA | Budget option | $30-40 |

### LC Filter

| Qty | Component | Value/Spec | Purpose | Est. Price |
|-----|-----------|------------|---------|------------|
| 1 | Inductor | 2mH / 10A | Filter L | $10 |
| 2 | Capacitor | 4.7µF/400V film | Filter C | $3 ea |

### Protection & Mechanical

| Qty | Component | Value/Spec | Purpose | Est. Price |
|-----|-----------|------------|---------|------------|
| 1 | Fuse holder | ANL type | Main fuse | $5 |
| 2 | Fuse | 60A ANL | Overcurrent | $3 ea |
| 1 | Varistor | 275V MOV | Surge protection | $1 |
| 1 | NTC | 10kΩ | Temperature sense | $0.50 |
| 2 | Fan | 80mm 12V | Cooling | $4 ea |
| 1 | PCB | 20×15cm | Main board | $10 |
| 1 | Enclosure | Metal, ventilated | Housing | $25 |
| - | Wire | 8 AWG (power), 18 AWG (signal) | Connections | $10 |
| - | Terminals | Anderson, ring | Connections | $8 |

---

## 2.5 BOM Summary

| Category | Est. Total |
|----------|------------|
| Control (EGS module, regulators) | $15 |
| Power MOSFETs | $8-16 |
| Input capacitors | $8 |
| Transformer | $40-60 |
| LC filter | $16 |
| Heatsinks & cooling | $28 |
| Protection | $12 |
| Mechanical & enclosure | $45 |

**Phase 2 Total: ~$145-175**

---

## 2.6 Build Sequence

### Week 1: Control Board

```
Day 1 - Power Supply Section:
□ Build 12V and 5V regulators
□ Add input/output capacitors
□ Test voltages with multimeter

Day 2 - EGS Module Integration:
□ Mount EGS002/003 module
□ Connect power rails
□ Set frequency (FS pin to GND for 50Hz)
□ Test: verify gate signals with scope or LEDs
□ Add feedback voltage divider
```

### Week 2: Power Stage

```
Day 1 - Heatsink Preparation:
□ Clean heatsink surfaces
□ Mark MOSFET positions
□ Apply thermal paste (thin, even layer)
□ Mount MOSFETs with insulating washers
□ Torque screws evenly (don't over-tighten!)

Day 2 - H-Bridge Wiring:
□ Wire drain connections (thick wire!)
□ Wire source connections
□ Add gate resistors
□ Connect to EGS module
□ Add freewheeling diodes
□ Add input capacitors
```

### Week 3: Transformer & Filter

```
Day 1 - Transformer Connection:
□ Connect transformer primary to H-bridge output
□ Secure transformer mounting
□ Test with LOW voltage (12V) first
□ Measure output voltage

Day 2 - LC Filter:
□ Wind or install inductor
□ Add film capacitors
□ Connect to transformer secondary
□ Measure output waveform
□ Check THD if possible (<5% good, <3% excellent)
```

### Week 4: Testing & Integration

```
Day 1 - No-Load Testing:
□ Full voltage test (48V) without load
□ Check output voltage (should be ~220-230V)
□ Verify waveform is clean sine
□ Monitor all temperatures
□ Run for 30 minutes

Day 2 - Load Testing:
□ 100W load (bulb) - 1 hour
□ 500W load - 30 minutes
□ 1000W load - 10 minutes
□ Check efficiency (input power vs output power)
□ Final temperature check
□ Enclosure assembly
```

---

## 2.7 Testing & Troubleshooting

### Expected Measurements

| Parameter | Expected Value | Action if Wrong |
|-----------|----------------|-----------------|
| Output voltage | 220-230V AC | Adjust feedback or transformer |
| Frequency | 50Hz ±0.5Hz | Check EGS module |
| THD | <5% | Check LC filter values |
| Efficiency | >90% | Check MOSFET temperatures |
| No-load current | <1A from battery | Check for oscillation |

### Common Problems

| Symptom | Likely Cause | Solution |
|---------|--------------|----------|
| No output | EGS not running | Check 12V/5V supply |
| Low voltage | Feedback wrong | Adjust divider resistors |
| High THD | Filter too small | Increase L or C |
| MOSFETs hot | Poor thermal | Improve heatsink mounting |
| Oscillation | Layout issue | Shorten gate leads |
| Transformer hum | DC offset | Check H-bridge symmetry |

---

## 2.8 Phase 2 Learning Outcomes

After completing Phase 2:
- [ ] Built working 1kW pure sine inverter!
- [ ] Understand EGS module integration
- [ ] Can design LC output filters
- [ ] Know voltage feedback basics
- [ ] Have thermal management experience
- [ ] Ready for high-power design!

---

# PHASE 3: HIGH-POWER DESIGN

**Goal:** Learn techniques for scaling to 6kW+ power levels
**Time:** 2-3 weekends (theory + planning)
**Cost:** ~$50 (for test components and materials)

---

## 3.1 Scaling Challenges

### What Changes at High Power?

```
1kW Inverter:                    6kW+ Inverter:
- 4 MOSFETs                      - 16-32 MOSFETs (paralleled)
- Commercial transformer         - Hand-wound toroidal
- Small heatsinks                - Massive heatsinks
- Passive cooling OK             - Active cooling required
- Off-the-shelf inductor         - Custom wound inductor
- Simple wiring                  - Bus bar connections
- Minutes to build               - Weeks to build
```

---

## 3.2 Paralleling MOSFETs

### Why Parallel MOSFETs?

```
Single IRFP4668:
- Rds(on) = 8mΩ
- At 50A: Power loss = I²R = 50² × 0.008 = 20W heat per MOSFET

Four IRFP4668 in parallel:
- Combined Rds(on) = 8mΩ / 4 = 2mΩ
- At 50A: Power loss = 50² × 0.002 = 5W total!

More parallel = less heat = higher efficiency
```

### How to Parallel MOSFETs

```
CRITICAL: Equal current sharing!

Bad parallel (current hogging):
        ┌─────────┐
Gate ───┤         ├───► Drain
        │    ═════│═════════
        │ Q1 Q2 Q3│Q4       │
        │    │  │ │  │      │ One MOSFET takes most current!
        │    └──┴─┴──┘      │
        │         ├─────────┘
        └─────────┘

Good parallel (balanced):
        ┌───────────────────────────────┐
Gate ───┤                               │
        │ [Rg]   [Rg]   [Rg]   [Rg]    │ Gate resistors!
        │   │      │      │      │      │
        │  Q1     Q2     Q3     Q4     │
        │   │      │      │      │      │
        │   └──────┴──────┴──────┘      │
        │                ├──────────────┘
        └────────────────┘

Individual gate resistors (4.7-10Ω) ensure balanced switching!
```

### Layout for Paralleled MOSFETs

```
TOP VIEW OF HEATSINK:

┌────────────────────────────────────────────────┐
│                                                │
│    Q1A    Q1B    Q1C    Q1D                   │
│     │      │      │      │                     │
│     └──────┴──────┴──────┴── BUS BAR ─────────┼──► To XFMR
│                                                │
│    Q2A    Q2B    Q2C    Q2D                   │
│     │      │      │      │                     │
│     └──────┴──────┴──────┴── BUS BAR ─────────┼──► GND
│                                                │
└────────────────────────────────────────────────┘

Key points:
- MOSFETs mounted directly on heatsink
- Thick copper bus bars for low resistance
- Keep connections short and equal length
- Use thermal paste on every MOSFET
```

---

## 3.3 Transformer Winding

### Toroidal Core Selection

```
For 6kW inverter at 48V input:

Power rating: 6000VA minimum (get 7500VA for margin)

Core selection:
┌─────────────────────────────────────────────┐
│ Parameter        │ 5kVA Core  │ 7.5kVA Core │
├──────────────────┼────────────┼─────────────┤
│ Outer diameter   │ 180mm      │ 210mm       │
│ Inner diameter   │ 100mm      │ 120mm       │
│ Height           │ 70mm       │ 85mm        │
│ Weight           │ ~15kg      │ ~25kg       │
│ Cross-section    │ ~28cm²     │ ~38cm²      │
└─────────────────────────────────────────────┘
```

### Calculating Turns

```
Volts per turn formula:
V/turn = 4.44 × f × Ae × Bmax × 10⁻⁴

Where:
- f = 50 Hz
- Ae = Core cross-section in cm²
- Bmax = 1.0-1.2 Tesla (safe operating point)

Example for 38cm² core at Bmax = 1.1T:
V/turn = 4.44 × 50 × 38 × 1.1 × 0.0001
V/turn = 0.93V per turn ≈ 1V/turn

For 48V primary: 48 / 0.93 = 52 turns
For 220V secondary: 220 / 0.93 = 237 turns

Note: Always test and adjust!
```

### Winding Technique

```
Step 1: Core Preparation
┌─────────────────────────────────────┐
│                                     │
│  1. Clean core surface              │
│  2. Wrap with insulation tape       │
│     (2-3 layers Kapton or glass)    │
│  3. Mark winding start point        │
│                                     │
└─────────────────────────────────────┘

Step 2: Primary Winding
┌─────────────────────────────────────┐
│                                     │
│  • Use thick wire (6-8 AWG)         │
│  • For 6kW: can use 2 strands       │
│    of 10 AWG in parallel            │
│  • Wind evenly around core          │
│  • Secure start and end             │
│  • Apply insulation layer           │
│                                     │
└─────────────────────────────────────┘

Step 3: Secondary Winding
┌─────────────────────────────────────┐
│                                     │
│  • Use thinner wire (14-16 AWG)     │
│  • Count turns carefully!           │
│  • Layer evenly (not bunched)       │
│  • Insulate between layers          │
│  • Test ratio before varnishing     │
│                                     │
└─────────────────────────────────────┘

Step 4: Testing
┌─────────────────────────────────────┐
│                                     │
│  Apply 12V AC to primary            │
│  Measure secondary voltage          │
│  Calculate actual ratio             │
│  Adjust turns if needed             │
│  Varnish when satisfied             │
│                                     │
└─────────────────────────────────────┘
```

---

## 3.4 Thermal Management

### Heat Calculation

```
For 6kW inverter at 95% efficiency:

Power loss = 6000 × (1 - 0.95) = 300W of heat!

This 300W must be dissipated:
- MOSFETs: ~150W (at full load)
- Transformer: ~100W (core and copper losses)
- Other: ~50W (diodes, inductors, wiring)
```

### Cooling Design

```
Passive cooling (no fans):
- Need ~0.1°C/W thermal resistance
- Heatsink size: ~1000cm² surface area
- Weight: 5-10kg of aluminum
- Only OK for <50% continuous load

Active cooling (with fans):
- Can use smaller heatsinks
- Need 2-4 × 120mm fans
- Control based on temperature
- Required for full rated power

Recommended setup:
┌─────────────────────────────────────┐
│                                     │
│     [FAN] → [HEATSINK] → [FAN]     │
│              ↓                      │
│         [MOSFETS]                   │
│                                     │
│   Push-pull configuration           │
│   for best airflow                  │
│                                     │
└─────────────────────────────────────┘
```

### Temperature Monitoring

```
Add thermistors to critical points:
□ MOSFET heatsink (shutdown at 80°C)
□ Transformer core (warning at 70°C)
□ Ambient air (for derating)

Simple thermal shutdown circuit:
Thermistor → Comparator → Shutdown pin

Or use ESP32 for smart monitoring (Step 6)
```

---

## 3.5 High-Current Wiring

### Bus Bars vs Wire

```
At 150A, wire resistance matters!

10 AWG wire (2m length):
- Resistance: ~6.6mΩ
- Power loss: 150² × 0.0066 = 149W!
- Wire gets very hot!

Copper bus bar (5mm × 30mm × 200mm):
- Resistance: ~0.2mΩ
- Power loss: 150² × 0.0002 = 4.5W
- Stays cool!

ALWAYS use bus bars for high current paths!
```

### Bus Bar Design

```
Bus bar sizing for 150A:

Material: Copper (electrolytic grade)
Cross-section: 5mm × 30mm = 150mm²
Current density: 150A / 150mm² = 1A/mm²

              ┌────────────────────────┐
              │                        │
              │   30mm wide × 5mm thick│
              │   ═══════════════════  │
              │                        │
              │   Holes for connections│
              │   ○        ○        ○  │
              │                        │
              └────────────────────────┘
```

---

## 3.6 Phase 3 Learning Outcomes

After completing Phase 3:
- [ ] Understand MOSFET paralleling
- [ ] Know transformer winding basics
- [ ] Can calculate thermal requirements
- [ ] Understand bus bar design
- [ ] Ready for OzInverter build!

---

# PHASE 4: OZINVERTER BUILD (6-15kW)

**Goal:** Build a production-quality 6-15kW pure sine wave inverter
**Time:** 2-3 months
**Cost:** ~$600
**Source:** https://www.bryanhorology.com/ozinverter.php

---

## 4.1 OzInverter System Overview

```
┌────────────────────────────────────────────────────────────────────────────┐
│                           OZINVERTER (6-15kW)                              │
│                                                                            │
│  48V Battery ──► Control Board ──► Power Board ──► Toroid ──► Filter ──►  │
│  Bank              │                 │               │                     │
│  800-1000Ah     EG8010            16× MOSFET      26-30kg     220V AC     │
│                 IR2110            (paralleled)    hand-wound   6-15kW     │
│                                                                            │
└────────────────────────────────────────────────────────────────────────────┘
```

---

## 4.2 Bill of Materials

### Control Board

| Qty | Component | Value/Spec | Purpose | Est. Price |
|-----|-----------|------------|---------|------------|
| 1 | EG8010 | SPWM IC | Main controller | $3 |
| 2 | IR2110 | Gate driver | High/low drive | $2 ea |
| 2 | TLP250 | Isolated driver | Alternative | $2 ea |
| 1 | 7812 | Regulator | 12V supply | $0.50 |
| 1 | 7805 | Regulator | 5V supply | $0.30 |
| 1 | Crystal | 12MHz | EG8010 clock | $0.50 |
| 10 | Capacitor | 100nF ceramic | Bypass | $0.50 |
| 5 | Capacitor | 10µF/25V | Filter | $0.50 |
| 2 | Capacitor | 22pF | Crystal | $0.10 |
| 10 | Resistor | Various | Timing, sense | $0.50 |
| 1 | PCB | OzInverter control | Main board | $15 |

**Control Board: ~$30**

### Power Board

| Qty | Component | Value/Spec | Purpose | Est. Price |
|-----|-----------|------------|---------|------------|
| 16 | IRFP4668 | 200V/130A MOSFET | H-bridge | $5 ea |
| 16 | Gate resistor | 4.7Ω 1W | Current balance | $1 |
| 16 | Zener | 18V | Gate protection | $1 |
| 8 | Capacitor | 4700µF/63V | Input filter | $4 ea |
| 8 | Diode | MUR3060 | Freewheeling | $2 ea |
| 4 | Bus bar | Copper 5×30mm | High current | $10 |
| 2 | Heatsink | 300×150×40mm | Cooling | $25 ea |
| 1 | PCB | OzInverter power | Heavy copper | $30 |
| 1 | Thermal paste | Large tube | Mounting | $5 |

**Power Board: ~$200**

### Toroidal Transformer

| Qty | Component | Value/Spec | Purpose | Est. Price |
|-----|-----------|------------|---------|------------|
| 1 | Toroid core | OD 220mm, 7.5kVA | Transformer base | $50-70 |
| 1 | Primary wire | 6 AWG (16mm²) ~6m | 48V winding | $20 |
| 1 | Secondary wire | 14 AWG (2.5mm²) ~70m | 220V winding | $25 |
| 1 | Insulation tape | Kapton 25mm | Layer insulation | $10 |
| 1 | Varnish | Transformer grade | Protection | $10 |

**Transformer: ~$115** (DIY) or **~$200** (pre-wound)

### LC Output Filter

| Qty | Component | Value/Spec | Purpose | Est. Price |
|-----|-----------|------------|---------|------------|
| 1 | Inductor core | Ferrite/powder | Filter L base | $20 |
| - | Inductor wire | 8 AWG ~15m | Winding | $15 |
| 4 | Capacitor | 4.7µF/400V film | Filter C | $4 ea |

**LC Filter: ~$50**

### Protection & Monitoring

| Qty | Component | Value/Spec | Purpose | Est. Price |
|-----|-----------|------------|---------|------------|
| 1 | Current sensor | ACS758-150A | Overcurrent | $10 |
| 2 | Relay | 48V/80A | Disconnect | $15 ea |
| 1 | Fuse holder | ANL bolt-on | Main fuse | $8 |
| 2 | Fuse | 200A ANL | Protection | $8 ea |
| 3 | NTC thermistor | 10kΩ | Temperature | $2 ea |
| 1 | Voltage display | 0-100V DC | Battery | $10 |

**Protection: ~$80**

### Cooling System

| Qty | Component | Value/Spec | Purpose | Est. Price |
|-----|-----------|------------|---------|------------|
| 4 | Fan | 120mm 12V high-flow | Forced air | $8 ea |
| 2 | Fan guard | 120mm | Safety | $2 ea |
| 1 | PWM controller | 12V | Speed control | $10 |
| 1 | Duct material | Aluminum sheet | Airflow guide | $10 |

**Cooling: ~$50**

### Enclosure & Mechanical

| Qty | Component | Value/Spec | Purpose | Est. Price |
|-----|-----------|------------|---------|------------|
| 1 | Enclosure | 500×400×250mm metal | Housing | $60 |
| 2 | Terminal block | 150A | Battery connection | $15 ea |
| 1 | Anderson connector | 350A | Quick disconnect | $20 |
| 1 | AC outlet | 30A 220V | Output | $10 |
| - | Cable glands | Assorted | Wire entry | $10 |
| - | Wire | 2 AWG (battery), 8 AWG | Internal | $25 |
| - | Hardware | Bolts, nuts, standoffs | Assembly | $20 |

**Mechanical: ~$175**

---

## 4.3 BOM Summary

| Category | Est. Total |
|----------|------------|
| Control board | $30 |
| Power board (MOSFETs, PCB) | $200 |
| Transformer (DIY) | $115 |
| LC filter | $50 |
| Protection | $80 |
| Cooling | $50 |
| Enclosure & mechanical | $175 |

**Total: ~$600-700**

*Note: Pre-wound transformer adds ~$100. Quality enclosure may cost more.*

---

## 4.4 Build Sequence

### Month 1: Preparation & Control

```
Week 1-2: Ordering and Study
□ Order all components
□ Download OzInverter documentation
□ Join OzInverter forum
□ Study schematics thoroughly
□ Plan workspace and tools

Week 3: Control Board Build
□ Populate control PCB
□ Install EG8010, IR2110
□ Build power supply section
□ Test 5V/12V rails
□ Verify EG8010 output with scope

Week 4: Control Board Testing
□ Connect LEDs to gate outputs
□ Verify SPWM waveform
□ Check dead-time (~300ns)
□ Test feedback circuit
□ Verify all protection inputs
```

### Month 2: Power Stage & Transformer

```
Week 1: Transformer Winding
□ Prepare toroid core
□ Apply insulation layers
□ Wind primary (count turns!)
□ Insulate primary
□ Wind secondary
□ Test ratio (apply 12V, measure output)
□ Adjust turns if needed
□ Apply varnish (cure 24h)

Week 2: Power Board MOSFETs
□ Clean heatsink mounting surfaces
□ Apply thermal paste
□ Mount all 16 MOSFETs
□ Check insulation between MOSFETs
□ Verify torque on all screws

Week 3: Power Board Wiring
□ Install bus bars
□ Wire MOSFET drains to bus bars
□ Wire MOSFET sources to bus bars
□ Install gate resistors
□ Connect gate wiring from control
□ Add freewheeling diodes
□ Install input capacitors

Week 4: LC Filter & Integration
□ Wind output inductor
□ Mount filter capacitors
□ Connect to transformer secondary
□ Wire control to power board
□ Install current sensor
□ Complete protection wiring
```

### Month 3: Testing & Assembly

```
Week 1: Initial Testing (Low Power)
□ Connect 24V battery (not 48V!)
□ Use small test transformer
□ Verify output with 40W load
□ Check waveform quality
□ Monitor all temperatures
□ Test protection circuits

Week 2: Full Voltage Testing
□ Connect 48V battery bank
□ Connect main transformer
□ Test with 500W load
□ Run for 1 hour
□ Check temperatures
□ Measure efficiency

Week 3: Load Testing
□ 1kW load - 1 hour
□ 2kW load - 30 minutes
□ 4kW load - 15 minutes
□ 6kW load - 5 minutes
□ Record all temperatures
□ Verify output voltage stability
□ Test overcurrent protection

Week 4: Final Assembly
□ Mount in enclosure
□ Wire ventilation
□ Install fan control
□ Final safety checks
□ Label all connections
□ Create documentation
□ Test complete system
```

---

## 4.5 Safety Checklist

### Electrical Safety

```
⚠️ CRITICAL SAFETY REQUIREMENTS ⚠️

Before ANY work:
□ Disconnect battery bank
□ Wait 2 minutes for capacitors to discharge
□ Verify with multimeter (DC bus should be 0V)
□ Use insulated tools

During testing:
□ Keep hands away from power stage
□ Use isolated oscilloscope probe
□ Never touch output terminals!
□ Have fire extinguisher ready
□ Work with supervision

Fuse ratings:
□ Battery fuse: 200A
□ Output fuse (optional): 30A

Ground:
□ Enclosure must be grounded
□ Use ground fault outlet if testing with grid backup
```

### Thermal Safety

```
Temperature limits:
□ MOSFETs: 100°C max (shutdown at 80°C)
□ Transformer: 80°C max (warning at 70°C)
□ Enclosure: 50°C max (increase cooling)

Thermal protection:
□ NTC sensors on heatsink
□ Thermal switch on transformer
□ Fan failure detection
```

---

## 4.6 Testing Protocol

### Stage 1: Control Board Only

```
Power: 12V supply only
Duration: 30 minutes

Checks:
□ 5V regulator output: 5.0V ±0.1V
□ 12V regulator output: 12.0V ±0.5V
□ EG8010 SPWM output: 23.4kHz PWM visible
□ Dead-time: ~300ns between outputs
□ LED indicator: blinking
□ Current draw: <100mA
```

### Stage 2: Low Voltage (24V)

```
Power: 24V battery
Load: 100W bulb
Duration: 1 hour

Checks:
□ Output voltage: ~110V AC (half voltage)
□ Frequency: 50.0Hz ±0.2Hz
□ Waveform: clean sine (check with scope)
□ MOSFET temperature: <50°C
□ No unusual sounds or smells
```

### Stage 3: Full Voltage (48V)

```
Power: 48V battery bank
Load: Various
Duration: As specified

Test sequence:
□ No load - 30 minutes
  - Output: 220-230V AC
  - Idle current: <2A from 48V

□ 500W load - 1 hour
  - Voltage drop: <5V
  - Temperature: <60°C

□ 2kW load - 30 minutes
  - Temperature: <70°C
  - Waveform quality: maintained

□ 6kW load - 5 minutes
  - Temperature: <80°C (fans at full speed)
  - Voltage regulation: <10% drop
```

### Stage 4: Protection Testing

```
□ Overcurrent: Apply 150% load, verify shutdown
□ Over-temperature: Block airflow, verify warning
□ Short circuit: Apply momentary short, verify fast trip
□ Low battery: Reduce input to 42V, verify warning
```

---

## 4.7 Performance Expectations

| Parameter | Target | Acceptable |
|-----------|--------|------------|
| Output power | 6kW continuous | 5kW |
| Peak power | 12-15kW (5 sec) | 10kW |
| Output voltage | 230V ±5% | 220-240V |
| Frequency | 50Hz ±0.1% | ±0.5% |
| THD | <3% | <5% |
| Efficiency | >94% | >90% |
| Idle consumption | <50W | <80W |
| Transfer time | <10ms (if ATS) | <20ms |

---

## 4.8 Phase 4 Learning Outcomes

After completing Phase 4:
- [ ] Built production-quality 6kW+ inverter!
- [ ] Can wind transformers
- [ ] Mastered high-power thermal management
- [ ] Know how to parallel MOSFETs
- [ ] Understand comprehensive testing
- [ ] Have skills for maintenance and repair

---

# PART 3 SUMMARY

## What You've Achieved

```
┌─────────────────────────────────────────────────────────────┐
│                    YOUR PROGRESSION                          │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Phase 1: SPWM theory, EG8010, LC filter design             │
│     ↓                                                        │
│  Phase 2: 1kW pure sine inverter (working!)                 │
│     ↓                                                        │
│  Phase 3: High-power design techniques                       │
│     ↓                                                        │
│  Phase 4: 6-15kW OzInverter (production quality!)           │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

## Complete Curriculum Summary

| Part | Phase | Project | Power |
|------|-------|---------|-------|
| 1 | 1-4 | Fundamentals (12 projects) | - |
| 2 | 1 | CD4047 Oscillator | - |
| 2 | 2 | Push-pull inverter | 150W |
| 2 | 3 | H-bridge & drivers | - |
| 2 | 4 | Modified sine inverter | 500W |
| 3 | 1 | SPWM & EG8010 | - |
| 3 | 2 | Pure sine inverter | 1kW |
| 3 | 3 | High-power design | - |
| 3 | 4 | OzInverter | 6-15kW |

## Total Investment

| Category | Cost |
|----------|------|
| Part 1 (Fundamentals) | ~$64 |
| Part 2 (150W + 500W) | ~$115 |
| Part 3 (1kW + 6kW) | ~$750 |
| Tools (if needed) | ~$300 |
| **Grand Total** | **~$1,230** |

## Skills Acquired

```
Electronics Fundamentals ──► Oscillators ──► MOSFETs
        │                        │              │
        ▼                        ▼              ▼
   PWM Control ──► Gate Drivers ──► H-Bridge
        │                              │
        ▼                              ▼
   SPWM/EG8010 ──► LC Filters ──► Transformers
        │                              │
        ▼                              ▼
   Thermal Mgmt ──► High Power ──► Production Quality
```

## Next Steps

After completing Part 3, you can:

1. **Add Smart Features (Part 6)**
   - ESP32 monitoring
   - Home Assistant integration
   - Remote control & alerts

2. **Build Community Microgrid (Part 7)**
   - 4-household DC/AC hybrid system
   - Swarm communication
   - Load sharing

3. **Integrate Libre Solar (Bonus)**
   - Open source MPPT
   - Open source BMS
   - CAN bus networking

**Congratulations on completing the inverter curriculum!**
