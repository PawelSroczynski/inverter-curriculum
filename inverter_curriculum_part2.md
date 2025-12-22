# Inverter Curriculum Part 2
## Step 2: CD4047 Square Wave Inverter (150W)
## Step 3: SG3525 Modified Sine Wave (500W)

---

# STEP 2: CD4047 SQUARE WAVE INVERTER

**Output:** 12V DC → 220V AC (square wave)
**Power:** 50-150W
**Time:** 2-3 weekends

---

## Step 2 Overview

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│  12V Battery → CD4047 → MOSFETs → Transformer → 220V AC    │
│                  │          │                               │
│              Oscillator  Push-Pull                          │
│               (50Hz)     Switching                          │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## Phase 2.1: Understanding the CD4047

### What is CD4047?

- CMOS astable/monostable multivibrator
- Generates 50% duty cycle square wave
- Has complementary outputs (Q and Q̄)
- Perfect for push-pull inverter

### CD4047 Pinout

```
        ┌────────────────┐
     C  │1   CD4047   14│ OSC
    RC  │2            13│ AST (astable enable)
   RET  │3            12│ RT
   AST  │4            11│ Q̄ (inverted output)
    NC  │5            10│ Q (output)
   -TR  │6             9│ NC
   GND  │7             8│ VCC
        └────────────────┘
```

### Frequency Formula

```
f = 1 / (4.4 × R × C)

For 50Hz:
R × C = 1 / (4.4 × 50) = 0.00454

Options:
- R = 68kΩ, C = 68nF → 49.6 Hz ✓
- R = 47kΩ, C = 100nF → 48.3 Hz ✓
- R = 100kΩ, C = 47nF → 48.5 Hz ✓
```

---

## Phase 2.2: MOSFET Selection

### Why MOSFETs?

| Parameter | BJT (TIP3055) | MOSFET (IRF3205) |
|-----------|---------------|------------------|
| Drive | Current (base) | Voltage (gate) |
| Speed | Slower | Faster |
| Efficiency | ~90% | ~97% |
| Heat | More | Less |
| Cost | Cheaper | Slightly more |

### IRF3205 Specifications

```
IRF3205:
- Vds: 55V (max drain-source voltage)
- Id: 110A (continuous drain current)
- Rds(on): 8mΩ (very low = less heat)
- Vgs(th): 2-4V (gate threshold)
- Package: TO-220
```

### MOSFET Pinout (TO-220)

```
    ┌─────────────┐
    │             │
    │   IRF3205   │
    │             │
    │   (metal    │
    │    tab =    │
    │    Drain)   │
    │             │
    └──┬──┬──┬────┘
       │  │  │
       G  D  S
      Gate Drain Source
       │  │  │
       1  2  3
```

---

## Phase 2.3: Transformer Selection

### Center-Tap Transformer

```
Primary (12V side):          Secondary (220V side):

     ┌────────┐              ┌────────┐
  ───┤        │              │        ├───  220V
     │        │              │        │     Hot
  CT─┤   ○○   ├──────────────┤   ○○   │
     │   ○○   │              │   ○○   │
  ───┤        │              │        ├───  220V
     └────────┘              └────────┘     Neutral

  End 1 ─┐
         ├─ Primary windings
    CT ──┤   (center-tap to +12V)
         ├─
  End 2 ─┘
```

### Transformer Specifications

For 100W inverter:
- Primary: 12-0-12V (center tap), 10A rating
- Secondary: 220V (or 230V)
- VA rating: ≥100VA (get 150VA for margin)
- Type: Toroidal preferred (more efficient)

---

## Phase 2.4: Complete Schematic

```
                              +12V (from battery)
                                │
         ┌──────────────────────┼──────────────────────────┐
         │                      │                          │
         │               ┌──────┴──────┐                   │
         │               │ TRANSFORMER │                   │
         │               │  CENTER-TAP │                   │
         │               └──────┬──────┘                   │
         │                      │                          │
         │         ┌────────────┴────────────┐             │
         │         │                         │             │
         │       Pri.1                     Pri.2           │
         │         │                         │             │
         │         D                         D             │
         │        ╱                         ╱              │
         │   Q1  ●   IRF3205          Q2   ●  IRF3205     │
         │        ╲                         ╲              │
         │         S                         S             │
         │         │                         │             │
         │         └───────────┬─────────────┘             │
         │                     │                           │
         │                    GND                          │
         │                                                 │
         │  ┌────────────────────────────────────┐         │
         │  │           CD4047                   │         │
         │  │                                    │         │
    +12V─┼──┤8 (VCC)                   10 (Q)───┼───[100Ω]─┴─► Q1 Gate
         │  │                                    │         │
         │  │          C1    R1                  │         │
         │  │        68nF   68kΩ                │         │
         │  │          │     │                   │         │
         │  │      1 ──┴──┬──┴── 2              │         │
         │  │             │                      │         │
         │  │            ─┴─                     │         │
         │  │           100nF                   │         │
         │  │             │                      │         │
         │  │                                    │         │
         │  │4 (AST)──────┴───── 13 (AST)      │         │
         │  │                                    │         │
         │  │                          11 (Q̄)───┼───[100Ω]─┴─► Q2 Gate
         │  │                                    │
    GND──┼──┤7 (GND)                            │
         │  │                                    │
         │  └────────────────────────────────────┘
         │
         └──────── Fuse (15A) ──────── Battery +
```

---

## Phase 2.5: Bill of Materials

### Main Components

| Qty | Component | Value/Spec | Purpose | Est. Price |
|-----|-----------|------------|---------|------------|
| 1 | CD4047 | IC | Oscillator | $0.50 |
| 1 | IC Socket | 14-pin DIP | Protection | $0.30 |
| 2 | IRF3205 | MOSFET 55V/110A | Power switching | $1.50 ea |
| 1 | Transformer | 12-0-12V CT / 220V, 100VA | Voltage step-up | $15 |
| 2 | Heatsink | TO-220 compatible | Cooling | $2 |
| 1 | Resistor | 68kΩ 1/4W | Frequency set | $0.05 |
| 2 | Resistor | 100Ω 1/4W | Gate current limit | $0.10 |
| 1 | Capacitor | 68nF ceramic/film | Frequency set | $0.20 |
| 1 | Capacitor | 100nF ceramic | Bypass | $0.10 |
| 2 | Capacitor | 100µF/25V electrolytic | Input filter | $0.40 |

### Protection & Connections

| Qty | Component | Value/Spec | Purpose | Est. Price |
|-----|-----------|------------|---------|------------|
| 1 | Fuse holder | Blade type | Safety | $1 |
| 3 | Fuse | 15A blade | Overcurrent | $1 |
| 1 | LED + resistor | Indicator | Power on | $0.20 |
| 1 | Switch | SPST 15A | Main power | $2 |
| - | Wire | 14 AWG red/black | Power | $3 |
| - | Wire | 22 AWG | Signal | $1 |
| 2 | Terminal block | 2-way, 15A | Connections | $2 |
| 1 | Perfboard | 10×15cm | Assembly | $2 |

### Optional but Recommended

| Qty | Component | Value/Spec | Purpose | Est. Price |
|-----|-----------|------------|---------|------------|
| 2 | TVS Diode | P6KE36A | Spike protection | $1 |
| 1 | Thermal paste | - | Heatsink contact | $2 |
| 4 | Screw + nut | M3 | Mounting | $0.50 |
| 1 | Enclosure | Ventilated | Housing | $10 |

---

## Step 2 BOM Summary

| Category | Est. Total |
|----------|------------|
| ICs & semiconductors | $5 |
| Transformer | $15 |
| Passive components | $3 |
| Protection & safety | $5 |
| Mechanical & wire | $8 |
| Enclosure (optional) | $10 |

**Step 2 Total: ~$36-46**

---

## Phase 2.6: Build Sequence

### Day 1: Oscillator Board
1. Test CD4047 on breadboard first
2. Verify 50Hz output with multimeter/scope
3. Solder oscillator section to perfboard

### Day 2: Power Stage
1. Mount MOSFETs on heatsink (with thermal paste)
2. Wire gates to oscillator outputs
3. Connect transformer primary

### Day 3: Testing
1. Test WITHOUT load first
2. Measure output voltage (should be ~220V AC)
3. Test with small load (25W bulb)
4. Gradually increase load

---

## Phase 2.7: Safety Checklist

- [ ] Fuse installed on battery positive
- [ ] Heatsinks properly mounted
- [ ] No exposed 220V connections
- [ ] Wire gauge adequate (14 AWG for 12V side)
- [ ] Keep fingers away from output!
- [ ] Test with incandescent bulb first (not electronics)

---

## Phase 2.8: Testing & Troubleshooting

| Symptom | Likely Cause | Solution |
|---------|--------------|----------|
| No output | CD4047 not oscillating | Check power, timing RC |
| Low voltage | Weak battery | Charge/replace battery |
| MOSFETs hot | High load / poor heatsink | Reduce load, improve cooling |
| Buzzing sound | Loose transformer | Tighten mounting |
| Fuse blows | Short circuit | Check wiring, MOSFET shorts |

---

## Phase 2.9: Learning Outcomes

After completing Step 2:
- [ ] Understand push-pull topology
- [ ] Can calculate oscillator frequency
- [ ] Know MOSFET basics (gate, drain, source)
- [ ] Understand transformer operation
- [ ] Safety with high voltage

---

# STEP 3: SG3525 MODIFIED SINE WAVE INVERTER

**Output:** 12V/24V DC → 220V AC (modified sine)
**Power:** 200-500W
**Time:** 3-4 weekends

---

## Step 3 Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│                                                                     │
│  Battery → SG3525 → IR2110 → H-Bridge → Transformer → Filter → AC │
│              │         │        │                                   │
│            PWM     Gate      4 MOSFETs                              │
│           Control  Driver                                           │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### What's New in Step 3?

| Step 2 | Step 3 |
|--------|--------|
| Push-pull (2 MOSFETs) | H-bridge (4 MOSFETs) |
| CD4047 (simple) | SG3525 (PWM capable) |
| No gate driver | IR2110 gate driver |
| Square wave | Modified sine wave |
| 150W | 500W |

---

## Phase 3.1: Understanding SG3525

### What is SG3525?

- PWM controller IC
- Adjustable frequency and duty cycle
- Soft-start capability
- Dual complementary outputs
- Dead-time control (prevents shoot-through)

### SG3525 Pinout

```
        ┌────────────────────┐
   INV  │1    SG3525     16│ Vref (5V out)
   N.I. │2               15│ Vcc (supply)
   SYNC │3               14│ OUT A
  OSC   │4               13│ Vc (output stage supply)
    Ct  │5               12│ GND
    Rt  │6               11│ OUT B
  DISCH │7               10│ SHUTDOWN
  SOFTST│8                9│ COMP
        └────────────────────┘
```

### Frequency Calculation

```
f = 1 / (Ct × (0.7×Rt + 3×Rd))

Simplified (Rd = 0):
f = 1 / (Ct × 0.7 × Rt)

For 50Hz (output is half of oscillator):
Oscillator = 100Hz
Rt = 10kΩ, Ct = 1.4µF → ~100Hz osc → 50Hz output
```

---

## Phase 3.2: Understanding IR2110 Gate Driver

### Why Gate Driver?

```
Problem:
- High-side MOSFET needs gate voltage ABOVE drain
- If drain is at +12V, gate needs +22V to turn on!
- SG3525 can't provide this

Solution: IR2110
- Bootstrap circuit creates high voltage
- Drives both high and low side MOSFETs
- Fast switching (nanoseconds)
```

### IR2110 Pinout

```
        ┌────────────────────┐
    LO  │1    IR2110     14│ Vcc
   COM  │2               13│ HIN
   Vss  │3               12│ LIN
   NC   │4               11│ SD
   Vs   │5               10│ Vb
   HO   │6                9│ NC
   NC   │7                8│ NC
        └────────────────────┘

HIN = High side input       HO = High side output
LIN = Low side input        LO = Low side output
Vb  = Bootstrap supply      Vs = High side return
```

### Bootstrap Circuit

```
     +12V
       │
    [D1]  Fast diode (UF4007)
       │
       ├──────── Vb (pin 10)
       │
     [C] 10µF bootstrap capacitor
       │
       └──────── Vs (pin 5) ── High side MOSFET source
```

---

## Phase 3.3: H-Bridge Topology

### What is H-Bridge?

```
         +Vdc
           │
     ┌─────┴─────┐
     │           │
    Q1          Q3
     │           │
     ├─── Load ──┤
     │           │
    Q2          Q4
     │           │
     └─────┬─────┘
           │
          GND

Switching pattern:
- Q1+Q4 ON: Current flows LEFT→RIGHT
- Q3+Q2 ON: Current flows RIGHT→LEFT
- NEVER Q1+Q2 or Q3+Q4 (shoot-through = explosion!)
```

### Modified Sine Wave

```
Square wave:          Modified sine:
    ┌────┐               ┌──┐    ┌──┐
    │    │            ┌──┘  └────┘  └──┐
────┘    └────    ────┘                └────
                      Zero-crossing gaps
                      (less harsh on equipment)
```

---

## Phase 3.4: Complete Schematic (Simplified)

```
                                    +12V
                                      │
        ┌─────────────────────────────┼──────────────────────┐
        │                             │                      │
        │            ┌────────────────┼───────────┐          │
        │            │   IR2110 (×2)  │           │          │
        │            │                │           │          │
        │      Vb ───┤10             │           │          │
        │       │    │          Vcc ─┤14        │          │
        │      [C]   │               │           │          │
        │       │    │    ┌─────────┐│           │          │
        │      Vs ───┤5   │         ││           │          │
        │            │    │ HIGH    ││    ┌──────┴──────┐   │
        │      HO ───┤6───┼─SIDE────┼┼────┤     Q1      │   │
        │            │    │ MOSFET  ││    │   IRF3205   │   │
        │            │    └─────────┘│    └──────┬──────┘   │
        │            │               │           │          │
        │            │    ┌─────────┐│           │ LOAD     │
        │      LO ───┤1───┼─LOW─────┼┼──────────┬┤(XFMR)    │
        │            │    │ SIDE    ││          ││          │
        │            │    │ MOSFET  ││    ┌─────┴┴─────┐    │
        │            │    └─────────┘│    │     Q2     │    │
        │     COM ───┤2              │    │   IRF3205  │    │
        │            │               │    └──────┬─────┘    │
        │     HIN ───┤13             │           │          │
        │            │               │          GND         │
        │     LIN ───┤12             │                      │
        │            │               │    (Same for Q3, Q4  │
        │      SD ───┤11             │     with 2nd IR2110) │
        │            │               │                      │
        │            └───────────────┘                      │
        │                                                   │
        │  ┌─────────────────────────────────────────────┐  │
        │  │              SG3525                         │  │
        │  │                                             │  │
        │  │  OUT A (14) ────────────────► IR2110 #1    │  │
        │  │                                             │  │
        │  │  OUT B (11) ────────────────► IR2110 #2    │  │
        │  │                                             │  │
   +12V─┼──┤  Vcc (15)                                  │  │
        │  │                                             │  │
        │  │  Vref (16) ───[voltage divider]──► Feedback│  │
        │  │                                             │  │
        │  │  Ct (5) ────── 1µF                         │  │
        │  │                                             │  │
        │  │  Rt (6) ────── 15kΩ                        │  │
        │  │                                             │  │
   GND──┼──┤  GND (12)                                  │  │
        │  │                                             │  │
        │  └─────────────────────────────────────────────┘  │
        │                                                   │
        └───────────────────────────────────────────────────┘
```

---

## Phase 3.5: Bill of Materials

### Control Section

| Qty | Component | Value/Spec | Purpose | Est. Price |
|-----|-----------|------------|---------|------------|
| 1 | SG3525 | PWM controller | Main control | $1.50 |
| 1 | IC Socket | 16-pin DIP | Protection | $0.30 |
| 2 | IR2110 | Gate driver | MOSFET drive | $2 ea |
| 2 | IC Socket | 14-pin DIP | Protection | $0.30 ea |
| 1 | Resistor | 15kΩ 1/4W | Rt (timing) | $0.05 |
| 1 | Capacitor | 1µF film | Ct (timing) | $0.30 |
| 2 | Capacitor | 10µF/25V | Bootstrap | $0.30 |
| 2 | Diode | UF4007 (fast) | Bootstrap | $0.30 |
| 4 | Resistor | 10Ω 1/2W | Gate resistors | $0.20 |
| 2 | Capacitor | 100nF ceramic | Bypass | $0.10 |

### Power Section

| Qty | Component | Value/Spec | Purpose | Est. Price |
|-----|-----------|------------|---------|------------|
| 4 | IRF3205 | MOSFET 55V/110A | H-bridge | $1.50 ea |
| 2 | Heatsink | Large, finned | Cooling | $5 ea |
| 1 | Transformer | 12-0-12V / 220V, 300VA | Power | $25 |
| 4 | Capacitor | 1000µF/25V | Input filter | $1 ea |
| 4 | Diode | MUR860 (fast recovery) | Freewheeling | $0.50 ea |

### Protection & Output

| Qty | Component | Value/Spec | Purpose | Est. Price |
|-----|-----------|------------|---------|------------|
| 1 | Fuse holder | ANL type | High current | $3 |
| 2 | Fuse | 40A ANL | Protection | $2 ea |
| 1 | Varistor | 275V MOV | Surge protection | $1 |
| 1 | Inductor | 1mH / 5A | Output filter | $3 |
| 1 | Capacitor | 2.2µF/400V | Output filter | $2 |
| 1 | Relay | 12V / 30A | Output disconnect | $5 |

### Mechanical & Wiring

| Qty | Component | Value/Spec | Purpose | Est. Price |
|-----|-----------|------------|---------|------------|
| - | Wire | 10 AWG red/black | Power (2m) | $5 |
| - | Wire | 18 AWG | Signal | $2 |
| 1 | Fan | 80mm 12V | Cooling | $5 |
| 1 | PCB/Perfboard | 15×20cm | Assembly | $5 |
| 1 | Enclosure | Metal, ventilated | Housing | $20 |
| - | Terminals | Ring, spade, etc. | Connections | $3 |
| 1 | Thermal paste | - | Heatsink | $2 |

---

## Step 3 BOM Summary

| Category | Est. Total |
|----------|------------|
| Control ICs | $7 |
| Gate drivers | $5 |
| MOSFETs | $6 |
| Transformer | $25 |
| Capacitors | $8 |
| Diodes | $3 |
| Protection | $10 |
| Heatsinks & cooling | $15 |
| Mechanical | $30 |

**Step 3 Total: ~$109**

---

## Phase 3.6: Build Sequence

### Weekend 1: Control Board
1. Build SG3525 oscillator section
2. Test output frequency (should be 50Hz)
3. Test dead-time (both outputs never HIGH together)

### Weekend 2: Driver Board
1. Build IR2110 circuits (one at a time)
2. Test bootstrap operation
3. Verify gate signals with oscilloscope

### Weekend 3: Power Stage
1. Mount MOSFETs on heatsink
2. Wire H-bridge (carefully!)
3. Connect transformer
4. Add freewheeling diodes

### Weekend 4: Integration & Testing
1. Connect all boards
2. Test with light load (40W bulb)
3. Add output filter
4. Test with higher loads gradually

---

## Phase 3.7: Critical Safety Notes

### Shoot-Through Prevention
```
NEVER allow Q1+Q2 or Q3+Q4 to be ON simultaneously!
This creates direct short: +Vdc → Q1 → Q2 → GND = BOOM!

SG3525 has built-in dead-time, but verify with scope!
```

### Dead-Time Check
```
Using oscilloscope:
1. Probe OUT A and OUT B
2. Both should go LOW before either goes HIGH
3. Gap should be ~1-2µs minimum
```

---

## Phase 3.8: Learning Outcomes

After completing Step 3:
- [ ] Understand H-bridge operation
- [ ] Know PWM control basics
- [ ] Can use gate drivers (IR2110)
- [ ] Understand dead-time importance
- [ ] Know modified sine vs square wave
- [ ] Can build 500W power stage

---

## Step 2 & 3 Combined Shopping Strategy

### Buy Together (Saves Shipping)

**From one supplier (AliExpress/LCSC/Mouser):**

| Component | Step 2 | Step 3 | Total |
|-----------|--------|--------|-------|
| IRF3205 | 2 | 4 | 6 + spares = 10 |
| IR2110 | 0 | 2 | 2 + spares = 4 |
| CD4047 | 1 | 0 | 1 + spare = 2 |
| SG3525 | 0 | 1 | 1 + spare = 2 |
| 1N4007 | 5 | 0 | 5 |
| UF4007 | 0 | 4 | 4 + spares = 6 |
| MUR860 | 0 | 4 | 4 + spares = 6 |
| Heatsinks | 2 small | 2 large | 4 total |

---

## Progression Check

After Steps 2 & 3, you can:

| Skill | Step 2 | Step 3 |
|-------|--------|--------|
| MOSFET switching | ✓ | ✓✓ |
| Oscillator design | Basic | PWM |
| Topology | Push-pull | H-bridge |
| Gate driving | Direct | IR2110 |
| Power level | 150W | 500W |
| Waveform | Square | Modified sine |

**Ready for Step 4: Pure Sine Wave!**
