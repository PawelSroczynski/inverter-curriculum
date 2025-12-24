# Inverter Curriculum Part 3
## From First Oscillator to Modified Sine Wave Inverter

**Prerequisites:** Part 1 (Fundamentals) + Part 2 (Safety & MOSFET Bridge)
**Output:** 12V DC → 220V AC
**Power Range:** 150W → 500W
**Time:** 6-8 weekends

---

## Before You Begin

```
┌─────────────────────────────────────────────────────────────────┐
│                     PREREQUISITES CHECK                          │
│                                                                  │
│  □ Completed Part 1 Fundamentals (all 5 phases)                 │
│  □ Completed Part 2 Safety Module                               │
│    □ Passed safety quiz with 100%                               │
│    □ Understand capacitor discharge procedure                   │
│    □ Have fire extinguisher in workspace                        │
│  □ Completed Part 2 Baby MOSFET Project                         │
│    □ Built working 12V push-pull with IRF540                    │
│    □ Comfortable with MOSFET switching                          │
│  □ Have appropriate workspace and tools                         │
│                                                                  │
│  ⚠️  Part 3 involves 220V LETHAL VOLTAGE                        │
│  ⚠️  Do NOT proceed without completing Part 2!                 │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Part 2 Overview

```
Phase 1         Phase 2         Phase 3         Phase 4         Phase 5
Oscillator      First Inverter  H-Bridge &      Modified Sine   Transformer
Mastery         Build (150W)    Gate Drivers    Build (500W)    Winding
   │                │               │               │               │
   ▼                ▼               ▼               ▼               ▼
┌───────┐      ┌───────┐      ┌───────┐      ┌───────┐      ┌───────┐
│CD4047 │─────►│2 FETs │─────►│IR2110 │─────►│SG3525 │─────►│Practice│
│Theory │      │Push-  │      │4 FETs │      │500W   │      │Winding │
│Testing│      │Pull   │      │H-Bridge│     │Output │      │50VA    │
└───────┘      └───────┘      └───────┘      └───────┘      └───────┘
```

---

# PHASE 1: OSCILLATOR MASTERY

**Goal:** Understand and build the timing circuit that creates AC from DC
**Time:** 1 weekend
**Cost:** ~$5

---

## 1.1 Why Oscillators Matter

```
DC (Battery):        AC (What we need):
───────────          ┌──┐  ┌──┐  ┌──┐
   +12V              │  │  │  │  │  │
───────────      ────┘  └──┘  └──┘  └──
   constant          alternating!

Oscillator = The "heartbeat" that switches DC on/off to create AC
```

---

## 1.2 Understanding CD4047

### What is CD4047?

- CMOS astable/monostable multivibrator IC
- Generates 50% duty cycle square wave
- Has complementary outputs (Q and Q̄)
- Perfect for push-pull inverter
- Low power consumption (~1mA)

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

Key pins:
- Pin 1 (C): Timing capacitor
- Pin 2 (RC): Timing resistor
- Pin 10 (Q): Output - goes HIGH, then LOW
- Pin 11 (Q̄): Inverted output - opposite of Q
- Pin 13 (AST): Connect to VCC for astable mode
```

### Why Complementary Outputs?

```
Time:    0   T/2   T   3T/2   2T
         │    │    │    │     │
Q:       ┌────┐    ┌────┐    ┌──
         │    │    │    │    │
      ───┘    └────┘    └────┘

Q̄:    ──┐    ┌────┐    ┌────┐
         │    │    │    │    │
         └────┘    └────┘    └──

Q and Q̄ are NEVER high at the same time!
This is critical for push-pull inverter safety.
```

---

## 1.3 Frequency Calculation

### The Formula

```
f = 1 / (4.4 × R × C)

Where:
- f = frequency in Hz
- R = resistance in Ohms
- C = capacitance in Farads
```

### Calculating for 50Hz

```
Need: f = 50Hz

Rearranging: R × C = 1 / (4.4 × 50) = 0.00454

Options that work:
┌─────────┬─────────┬──────────┐
│ R       │ C       │ Actual f │
├─────────┼─────────┼──────────┤
│ 68kΩ    │ 68nF    │ 49.6 Hz  │ ← Recommended
│ 47kΩ    │ 100nF   │ 48.3 Hz  │
│ 100kΩ   │ 47nF    │ 48.5 Hz  │
│ 56kΩ    │ 82nF    │ 49.5 Hz  │
└─────────┴─────────┴──────────┘
```

### Fine-Tuning Frequency

```
Use a potentiometer in series with fixed resistor:

     ┌───[47kΩ]───┬───[20kΩ pot]───┐
     │            │                 │
  Pin 2          adjust            GND
  (RC)           here

This allows 45-55Hz adjustment range
```

---

## 1.4 Breadboard Test Circuit

### Components Needed

| Qty | Component | Value | Purpose |
|-----|-----------|-------|---------|
| 1 | CD4047 | - | Oscillator IC |
| 1 | Resistor | 68kΩ | Timing |
| 1 | Capacitor | 68nF | Timing |
| 1 | Capacitor | 100nF | Power bypass |
| 2 | LED | Red, Green | Output indicators |
| 2 | Resistor | 1kΩ | LED current limit |
| 1 | Breadboard | - | Assembly |
| - | Wire | 22 AWG | Connections |

### Test Circuit Schematic

```
                        +12V
                          │
            ┌─────────────┼─────────────┐
            │             │             │
            │     ┌───────┴───────┐     │
            │     │    CD4047     │     │
            │     │               │     │
            │     │ 8(VCC)   13(AST)────┘
            │     │               │
      68nF ═╧═ 1(C)          10(Q)────[1kΩ]───► LED1 (Green)
            │     │               │                 │
            │     │          11(Q̄)────[1kΩ]───► LED2 (Red)
            │     │               │                 │
       68kΩ ─┬─ 2(RC)        7(GND)──┬──────────────┘
            │     │               │  │
            │     └───────────────┘  │
            │                        │
            └────────────────────────┴──── GND
                                     │
                               [100nF]
                                     │
                                    GND
```

### What You Should See

```
LEDs alternate on/off at ~1 second each (visible blinking)
Wait... 50Hz would be too fast to see!

For TESTING, use slower values first:
- R = 680kΩ, C = 1µF → ~0.3Hz (visible!)
- Then switch to 68kΩ + 68nF for real 50Hz

At 50Hz, both LEDs appear continuously on
(switching too fast for eyes to see)
```

---

## 1.5 Measuring with Multimeter

### Frequency Check (if meter has Hz function)

```
1. Set multimeter to Hz (frequency)
2. Connect probe to Pin 10 (Q)
3. Connect ground to circuit GND
4. Should read ~50Hz (±2Hz is OK)
```

### Duty Cycle Check

```
If you have oscilloscope:
1. Set to 50ms/div timebase
2. Probe Pin 10
3. Should see perfect 50% duty cycle square wave

      ┌─────┐     ┌─────┐     ┌─────┐
      │     │     │     │     │     │
  ────┘     └─────┘     └─────┘     └────
      |←10ms→|←10ms→|

      50% on, 50% off = perfect for push-pull!
```

---

## 1.6 Phase 1 Learning Outcomes

After completing Phase 1:
- [ ] Understand RC timing circuits
- [ ] Can calculate frequency from R and C values
- [ ] Know CD4047 pinout and operation
- [ ] Understand complementary outputs
- [ ] Have working 50Hz oscillator

---

# PHASE 2: FIRST INVERTER BUILD (150W)

**Goal:** Build your first working DC→AC inverter
**Time:** 2 weekends
**Cost:** ~$35

---

## 2.1 MOSFET Fundamentals

### Why MOSFETs?

| Parameter | BJT (TIP3055) | MOSFET (IRF3205) |
|-----------|---------------|------------------|
| Drive | Current (base) | Voltage (gate) |
| Speed | Slower | Faster |
| Efficiency | ~90% | ~97% |
| Heat | More | Less |
| On resistance | Higher | 8mΩ (very low) |

### IRF3205 Specifications

```
IRF3205 - "The workhorse MOSFET":
┌────────────────────────────────┐
│ Vds (max):     55V             │
│ Id (max):      110A            │
│ Rds(on):       8mΩ             │ ← Very low = less heat
│ Vgs(th):       2-4V            │ ← Easy to drive
│ Package:       TO-220          │
└────────────────────────────────┘
```

### MOSFET Pinout (TO-220)

```
    ┌─────────────┐
    │             │
    │   IRF3205   │
    │             │
    │   (metal    │
    │    tab =    │
    │    Drain)   │  ← Tab is connected to Drain!
    │             │
    └──┬──┬──┬────┘
       │  │  │
       G  D  S
      Gate Drain Source
       │  │  │
       1  2  3

IMPORTANT: Metal tab is electrically connected to Drain!
Use insulator when mounting multiple MOSFETs on same heatsink.
```

### Gate Resistor - Why 100Ω?

```
CD4047 output → [100Ω] → MOSFET Gate

Purpose:
1. Limits current spike when gate charges
2. Reduces ringing/oscillation
3. Slows switching slightly (reduces EMI)
4. Protects CD4047 output

Without gate resistor: Risk of CD4047 damage!
```

---

## 2.2 Transformer Basics

### Center-Tap Transformer

```
Primary (12V side):           Secondary (220V side):

     ┌────────┐               ┌────────┐
  A ─┤        │               │        ├─── Live (220V)
     │   ○○   │               │   ○○   │
 CT ─┤   ○○   ├───────────────┤   ○○   │
     │   ○○   │   magnetic    │   ○○   │
  B ─┤        │    coupling   │        ├─── Neutral
     └────────┘               └────────┘

CT = Center Tap (connects to +12V)
A and B = ends (connect to MOSFET drains)
```

### How Push-Pull Works

```
Half-cycle 1 (Q1 ON, Q2 OFF):
    +12V
      │
    ──┼── CT
   ┌──┴──┐
   │     │
   ▼
  Q1    Q2
  ON    OFF
   │
  GND

Current flows: +12V → CT → A → Q1 → GND

Half-cycle 2 (Q1 OFF, Q2 ON):
    +12V
      │
    ──┼── CT
   ┌──┴──┐
        │
        ▼
  Q1    Q2
  OFF   ON
         │
        GND

Current flows: +12V → CT → B → Q2 → GND

Result: Alternating current in transformer = AC output!
```

### Transformer Selection

```
For 100-150W inverter:
┌────────────────────────────────────┐
│ Primary:   12-0-12V (center tap)   │
│ Current:   10A minimum             │
│ Secondary: 220V (or 230V)          │
│ VA rating: 150VA or more           │
│ Type:      EI core or toroidal     │
└────────────────────────────────────┘

Note: "12-0-12V" means 12V from center to each end
Total primary = 24V end-to-end
```

---

## 2.3 Complete 150W Inverter Schematic

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
         │       Pri.A                     Pri.B           │
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


SECONDARY SIDE (220V OUTPUT):

         ┌──────────────────────┐
         │    TRANSFORMER       │
         │    SECONDARY         │
         └──────┬───────┬───────┘
                │       │
               Live   Neutral
                │       │
            AC OUTPUT 220V
            (DANGEROUS!)
```

---

## 2.4 Bill of Materials - 150W Inverter

### Main Components

| Qty | Component | Value/Spec | Purpose | Est. Price |
|-----|-----------|------------|---------|------------|
| 1 | CD4047 | IC | Oscillator | $0.50 |
| 1 | IC Socket | 14-pin DIP | Protection | $0.30 |
| 2 | IRF3205 | MOSFET 55V/110A | Power switching | $1.50 ea |
| 1 | Transformer | 12-0-12V CT / 220V, 150VA | Voltage step-up | $15 |
| 2 | Heatsink | TO-220 compatible | Cooling | $2 ea |
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
| 1 | LED + resistor | Green | Power on | $0.20 |
| 1 | Switch | SPST 15A | Main power | $2 |
| - | Wire | 14 AWG red/black | Power | $3 |
| - | Wire | 22 AWG | Signal | $1 |
| 2 | Terminal block | 2-way, 15A | Connections | $2 |
| 1 | Perfboard | 10×15cm | Assembly | $2 |

**Phase 2 Total: ~$35-40**

---

## 2.5 Build Sequence

### Day 1: Oscillator Board

```
Step 1: Test on breadboard
   - Build CD4047 circuit from Phase 1
   - Verify 50Hz output
   - Confirm both outputs working

Step 2: Transfer to perfboard
   - Solder IC socket first (never solder IC directly!)
   - Add timing components (R, C)
   - Add bypass capacitor
   - Add gate resistors (100Ω × 2)
```

### Day 2: Power Stage

```
Step 1: Prepare heatsinks
   - Clean mounting surface
   - Apply thermal paste (thin layer)
   - Mount MOSFETs with insulating washers (if sharing heatsink)
   - Torque screws evenly

Step 2: Wire MOSFETs
   - Gate ← oscillator through 100Ω
   - Drain ← transformer primary ends
   - Source ← common GND
   - Keep high-current wires short!

Step 3: Connect transformer
   - Center tap ← +12V through fuse
   - Ends ← MOSFET drains
   - Secondary ← output terminals (CAREFUL - 220V!)
```

### Day 3: Testing

```
⚠️ SAFETY FIRST:
□ Double-check all connections
□ Ensure no shorts on secondary
□ Have fire extinguisher ready
□ DO NOT touch output!

Test sequence:
1. Power on (no load) - should hear faint hum
2. Measure output voltage: expect 200-240V AC
3. Test with 25W incandescent bulb
4. Check MOSFET temperature (should be warm, not hot)
5. Test with 60W bulb
6. Run for 10 minutes, monitor temperature
```

---

## 2.6 Troubleshooting

| Symptom | Likely Cause | Solution |
|---------|--------------|----------|
| No output at all | CD4047 not oscillating | Check power, timing R/C |
| Low voltage (<180V) | Weak battery / wrong transformer | Charge battery, check ratio |
| MOSFETs very hot | Load too high / poor heatsink | Reduce load, improve cooling |
| Buzzing/humming | Normal at light load | Add small load if annoying |
| Fuse blows | Short circuit / MOSFET failure | Check wiring, replace MOSFET |
| Output unstable | Battery voltage dropping | Use bigger battery |

---

## 2.7 Phase 2 Learning Outcomes

After completing Phase 2:
- [ ] Built working 150W inverter!
- [ ] Understand push-pull topology
- [ ] Know MOSFET operation and mounting
- [ ] Can work with transformers safely
- [ ] Understand basic inverter troubleshooting

---

# PHASE 3: H-BRIDGE & GATE DRIVERS

**Goal:** Master the 4-MOSFET H-bridge with proper gate driving
**Time:** 1-2 weekends
**Cost:** ~$25 (components for experimentation)

---

## 3.1 Why H-Bridge?

### Push-Pull vs H-Bridge

```
Push-Pull (Phase 2):              H-Bridge (Phase 3):
- 2 MOSFETs                       - 4 MOSFETs
- Needs center-tap transformer    - Works with ANY transformer
- Simpler                         - More flexible
- Less efficient at high power    - Better for high power
- Limited to ~500W practically    - Scales to 15kW+
```

### H-Bridge Topology

```
              +Vdc
                │
     ┌──────────┼──────────┐
     │          │          │
    Q1          │         Q3     ← High-side MOSFETs
     │          │          │
     ├────── LOAD ─────────┤     ← Transformer primary
     │                     │
    Q2                    Q4     ← Low-side MOSFETs
     │                     │
     └──────────┬──────────┘
                │
               GND

Switching patterns:
- Q1+Q4 ON, Q2+Q3 OFF: Current flows LEFT → RIGHT
- Q3+Q2 ON, Q1+Q4 OFF: Current flows RIGHT → LEFT
- NEVER Q1+Q2 or Q3+Q4 simultaneously! (shoot-through = explosion)
```

---

## 3.2 The Shoot-Through Problem

### What is Shoot-Through?

```
DANGER - If Q1 and Q2 turn on together:

     +Vdc
       │
      Q1 (ON)
       │
       ├──── (load bypassed!)
       │
      Q2 (ON)
       │
      GND

Current path: +Vdc → Q1 → Q2 → GND
= DIRECT SHORT CIRCUIT!
= Hundreds of amps!
= Explosion, fire, destroyed MOSFETs!
```

### Solution: Dead-Time

```
Dead-time = small gap where BOTH MOSFETs are OFF

Q1 gate:  ┌────────┐          ┌────────┐
          │        │          │        │
      ────┘        └──────────┘        └────

Q2 gate:            ┌────────┐          ┌────
                    │        │          │
      ──────────────┘        └──────────┘

              │←─►│ Dead-time (1-2µs)

During dead-time: Both OFF = safe!
```

---

## 3.3 Understanding IR2110

### Why Gate Driver?

```
Problem with high-side MOSFETs:

     +48V ──────┬──────
                │
            Q1 (high-side)
          G ──?── Need Vgs = +10V above Source
                │
             Source (floating at ~48V!)
                │
              LOAD

To turn on Q1: Gate needs 48V + 10V = 58V!
CD4047 output is only 0-12V... won't work!

Solution: IR2110 with bootstrap circuit
```

### IR2110 Pinout and Function

```
        ┌────────────────────┐
    LO  │1    IR2110     14│ Vcc (12-20V)
   COM  │2               13│ HIN (high-side input)
   Vss  │3               12│ LIN (low-side input)
   NC   │4               11│ SD (shutdown)
   Vs   │5               10│ Vb (bootstrap supply)
   HO   │6                9│ NC
   NC   │7                8│ NC
        └────────────────────┘

Inputs (from controller):
- HIN: Logic signal for high-side MOSFET
- LIN: Logic signal for low-side MOSFET
- SD: Shutdown (active high) - tie to GND normally

Outputs (to MOSFETs):
- HO: High-side gate drive (referenced to Vs)
- LO: Low-side gate drive (referenced to COM)

Power:
- Vcc: 12-20V supply
- Vb: Bootstrap voltage (created by bootstrap circuit)
- Vs: High-side return (connects to MOSFET source)
```

### Bootstrap Circuit Explained

```
How bootstrap creates high voltage:

Phase 1: Low-side ON (Q2 conducting)

    +12V ──[D1]──┬── Vb            Vs is at ~0V (Q2 on)
                 │
              [Cboot]              Cboot charges to 12V!
                 │
    Vs ──────────┴── (near GND)

Phase 2: High-side ON (Q1 conducting)

    Vb ────┬── now at ~60V!        Vs rises to ~48V
           │
        [Cboot] (still 12V)        Vb = Vs + 12V = 60V!
           │
    Vs ────┴── (at ~48V)           Enough to drive Q1 gate!

Bootstrap components:
- D1: Fast diode (UF4007) - charges Cboot when low-side on
- Cboot: 10µF/25V - stores charge for high-side drive
```

---

## 3.4 IR2110 Test Circuit

### Build This on Breadboard First!

```
                              +12V
                                │
              ┌─────────────────┼─────────────────┐
              │                 │                 │
              │    ┌────[D1]────┤ UF4007         │
              │    │            │                 │
              │   ═╧═ 10µF     ─┴─ 100nF         │
              │    │ Cboot      │                 │
              │    │            │                 │
              │    └─── Vb(10) ─┤                 │
              │                 │                 │
              │    ┌─── Vs(5) ──┤                 │
              │    │            │                 │
              │   ═╧═ Test      │                 │
              │    │  LED       │   IR2110        │
              │    │  +1kΩ      │                 │
              │    │            │                 │
         ┌────┴────┴── HO(6) ───┤              ───┤ Vcc(14)
         │                      │                 │
         │         ┌── LO(1) ───┤              ───┤ HIN(13) ← Test signal
         │         │            │                 │
         │        ═╧═ Test      │              ───┤ LIN(12) ← Test signal
         │         │  LED       │                 │
         │         │  +1kΩ      │              ───┤ SD(11) → GND
         │         │            │                 │
         │         └── COM(2) ──┴──────┬──────────┘
         │                             │
         └─────────────────────────────┴─── GND

Test procedure:
1. Apply 12V power
2. Toggle HIN high → HO LED lights
3. Toggle LIN high → LO LED lights
4. Never both at same time!
```

---

## 3.5 Complete H-Bridge Test Setup

### Schematic (Low Power Test)

```
                                    +12V
                                      │
                    ┌─────────────────┼─────────────────┐
                    │                 │                 │
         IR2110 #1  │      ┌─────────┴─────────┐      │ IR2110 #2
         (Left leg) │      │                   │      │ (Right leg)
              ┌─────┴──────┤                   ├──────┴─────┐
              │            │                   │            │
           HO1 ─[10Ω]─ G   │                   │   G ─[10Ω]─ HO2
              │        │   │                   │   │        │
              │       Q1   │      12V BULB     │  Q3        │
              │        │   │    (test load)    │   │        │
              │        └───┤         │         ├───┘        │
              │            │        ═══        │            │
           LO1 ─[10Ω]─ G   │         │         │   G ─[10Ω]─ LO2
              │        │   │                   │   │        │
              │       Q2   │                   │  Q4        │
              │        │   │                   │   │        │
              └────────┴───┤                   ├───┴────────┘
                           │                   │
                           └─────────┬─────────┘
                                     │
                                    GND

Control signals from CD4047:
- Q output → HIN1 and LIN2 (diagonal pair)
- Q̄ output → LIN1 and HIN2 (other diagonal)

This creates alternating current through bulb!
```

---

## 3.6 Phase 3 Bill of Materials

| Qty | Component | Value | Purpose | Price |
|-----|-----------|-------|---------|-------|
| 2 | IR2110 | Gate driver IC | MOSFET drive | $2 ea |
| 2 | IC Socket | 14-pin DIP | Protection | $0.30 ea |
| 4 | IRF3205 | MOSFET | H-bridge | $1.50 ea |
| 2 | Diode | UF4007 | Bootstrap | $0.20 ea |
| 2 | Capacitor | 10µF/25V | Bootstrap | $0.20 ea |
| 4 | Capacitor | 100nF | Bypass | $0.20 |
| 4 | Resistor | 10Ω 1W | Gate resistors | $0.20 |
| 1 | Perfboard | 15×10cm | Assembly | $3 |
| - | Wire | 18 AWG | Signal | $2 |
| 1 | 12V bulb | 10-20W | Test load | $2 |

**Phase 3 Total: ~$25**

---

## 3.7 Phase 3 Learning Outcomes

After completing Phase 3:
- [ ] Understand H-bridge topology
- [ ] Know why shoot-through is dangerous
- [ ] Understand dead-time concept
- [ ] Can use IR2110 gate driver
- [ ] Understand bootstrap circuit
- [ ] Have working H-bridge test circuit

---

# PHASE 4: MODIFIED SINE INVERTER (500W)

**Goal:** Build a 500W modified sine wave inverter with SG3525
**Time:** 2 weekends
**Cost:** ~$50

---

## 4.1 Understanding SG3525

### What is SG3525?

- PWM (Pulse Width Modulation) controller IC
- Adjustable frequency and duty cycle
- Built-in dead-time control
- Soft-start capability
- Dual complementary outputs
- Professional-grade inverter control

### SG3525 Pinout

```
        ┌────────────────────┐
   INV  │1    SG3525     16│ Vref (5V internal reference)
   N.I. │2               15│ Vcc (supply 8-35V)
   SYNC │3               14│ OUT A
  OSC   │4               13│ Vc (output stage supply)
    Ct  │5               12│ GND
    Rt  │6               11│ OUT B
  DISCH │7               10│ SHUTDOWN
  SOFTST│8                9│ COMP (error amp output)
        └────────────────────┘

Key pins:
- Rt, Ct (6,5): Set oscillator frequency
- OUT A, OUT B (14,11): Complementary PWM outputs
- DISCH (7): Dead-time adjust
- SOFTST (8): Soft-start (add capacitor)
```

### Frequency Calculation

```
Oscillator frequency:
fosc = 1 / (Ct × (0.7×Rt + 3×Rd))

Where Rd = dead-time resistor (pin 7)

For simplified calculation (Rd small):
fosc ≈ 1 / (0.7 × Rt × Ct)

Output frequency = fosc / 2 (outputs toggle alternately)

Example for 50Hz output:
- Need fosc = 100Hz
- Rt = 10kΩ, Ct = 1.4µF
- fosc = 1 / (0.7 × 10000 × 0.0000014) = 102Hz ✓
```

### Dead-Time Adjustment

```
Dead-time pin (7) sets gap between OUT A and OUT B

        OUTA: ┌────┐      ┌────┐
              │    │      │    │
          ────┘    └──────┘    └──

        OUTB:       ┌────┐      ┌────┐
                    │    │      │    │
          ──────────┘    └──────┘    └──

              │←→│ Dead-time (set by Rd resistor)

Typical: 1-2µs dead-time
Rd ≈ 100Ω gives ~1µs dead-time
```

---

## 4.2 Modified Sine Wave

### What is Modified Sine Wave?

```
Pure Sine:              Modified Sine:
       ╱╲                    ┌──┐    ┌──┐
      ╱  ╲               ┌───┘  └────┘  └───┐
─────╱    ╲─────     ────┘                  └────
      ╲  ╱               └───┐      ┌───┘
       ╲╱                    └──┘    └──┘

Modified sine = "staircase" approximation
- Better than square wave
- OK for most loads (motors, tools, lights)
- NOT ideal for sensitive electronics
```

### How SG3525 Creates Modified Sine

```
SG3525 can vary pulse width based on feedback:

Narrow pulses near zero-crossing:
    ┌┐    ┌┐
    ││    ││
────┘└────┘└────

Wide pulses at peak:
  ┌────┐  ┌────┐
  │    │  │    │
──┘    └──┘    └──

Result after filter: stepped approximation of sine
```

---

## 4.3 Complete 500W Inverter Schematic

```
                                    +12V (or 24V)
                                        │
            ┌───────────────────────────┼───────────────────────────┐
            │                           │                           │
            │            ┌──────────────┼──────────────┐            │
            │            │   IR2110 #1  │  IR2110 #2   │            │
            │            │              │              │            │
            │      Vb ───┤10           │         10├─── Vb         │
            │       │    │          Vcc│Vcc          │    │        │
            │      [C]   │           ──┼──           │   [C]       │
            │       │    │             │             │    │        │
            │      Vs ───┤5            │          5├─── Vs        │
            │            │             │             │            │
            │      HO ───┤6    ┌───────┼───────┐  6├─── HO        │
            │            │     │       │       │     │            │
            │            │    Q1       │      Q3     │            │
            │            │     │       │       │     │            │
            │            │     └───[TRANSFORMER]───┘     │            │
            │            │             │ PRIMARY         │            │
            │            │     ┌───────┴───────┐     │            │
            │      LO ───┤1    │               │  1├─── LO        │
            │            │    Q2               Q4    │            │
            │            │     │               │     │            │
            │     COM ───┤2    │               │  2├─── COM       │
            │            │     │               │     │            │
            │     HIN ───┤13   │               │ 13├─── HIN       │
            │            │     │               │     │            │
            │     LIN ───┤12   │               │ 12├─── LIN       │
            │            │     │               │     │            │
            │      SD ───┤11   │               │ 11├─── SD        │
            │            │     │               │     │            │
            │            └─────┴───────────────┴─────┘            │
            │                          │                           │
            │                         GND                          │
            │                                                      │
            │  ┌───────────────────────────────────────────────┐   │
            │  │                   SG3525                       │   │
            │  │                                                │   │
            │  │  OUT A (14) ─────────────► IR2110 #1 HIN/LIN  │   │
            │  │                                                │   │
            │  │  OUT B (11) ─────────────► IR2110 #2 HIN/LIN  │   │
            │  │                                                │   │
       +12V─┼──┤  Vcc (15)                                     │   │
            │  │                                                │   │
            │  │  Vref (16) ───[voltage divider]──► Feedback   │   │
            │  │                                                │   │
            │  │  Ct (5) ────── 1µF                            │   │
            │  │                                                │   │
            │  │  Rt (6) ────── 15kΩ                           │   │
            │  │                                                │   │
            │  │  DISCH (7) ─── 100Ω ─── GND (dead-time)      │   │
            │  │                                                │   │
            │  │  SOFTST (8) ── 10µF ── GND (soft start)      │   │
            │  │                                                │   │
       GND──┼──┤  GND (12)                                     │   │
            │  │                                                │   │
            │  └───────────────────────────────────────────────┘   │
            │                                                      │
            └───────────── Fuse (40A) ─────────── Battery +        │


OUTPUT SECTION:

TRANSFORMER SECONDARY ─────┬───[L = 1mH]───┬─── 220V AC Live
                           │               │
                          ═╧═ 2.2µF       ═╧═ Load
                           │  400V         │
                           │               │
                           └───────────────┴─── 220V AC Neutral
```

---

## 4.4 Bill of Materials - 500W Inverter

### Control Section

| Qty | Component | Value/Spec | Purpose | Est. Price |
|-----|-----------|------------|---------|------------|
| 1 | SG3525 | PWM controller | Main control | $1.50 |
| 1 | IC Socket | 16-pin DIP | Protection | $0.30 |
| 2 | IR2110 | Gate driver | MOSFET drive | $2 ea |
| 2 | IC Socket | 14-pin DIP | Protection | $0.30 ea |
| 1 | Resistor | 15kΩ 1/4W | Rt (timing) | $0.05 |
| 1 | Capacitor | 1µF film | Ct (timing) | $0.30 |
| 1 | Resistor | 100Ω | Dead-time | $0.05 |
| 2 | Capacitor | 10µF/25V | Bootstrap | $0.30 ea |
| 2 | Diode | UF4007 (fast) | Bootstrap | $0.20 ea |
| 4 | Resistor | 10Ω 1W | Gate resistors | $0.20 |
| 4 | Capacitor | 100nF ceramic | Bypass | $0.20 |
| 1 | Capacitor | 10µF | Soft-start | $0.20 |

### Power Section

| Qty | Component | Value/Spec | Purpose | Est. Price |
|-----|-----------|------------|---------|------------|
| 4 | IRF3205 | MOSFET 55V/110A | H-bridge | $1.50 ea |
| 2 | Heatsink | Large, finned | Cooling | $5 ea |
| 1 | Transformer | 12-0-12V / 220V, 300VA | Power | $25 |
| 4 | Capacitor | 1000µF/25V | Input filter | $1 ea |
| 4 | Diode | MUR860 (fast recovery) | Freewheeling | $0.50 ea |

### Output Filter

| Qty | Component | Value/Spec | Purpose | Est. Price |
|-----|-----------|------------|---------|------------|
| 1 | Inductor | 1mH / 5A | Output filter | $3 |
| 1 | Capacitor | 2.2µF/400V film | Output filter | $2 |
| 1 | Varistor | 275V MOV | Surge protection | $1 |

### Mechanical

| Qty | Component | Value/Spec | Purpose | Est. Price |
|-----|-----------|------------|---------|------------|
| - | Wire | 10 AWG red/black | Power (2m) | $5 |
| - | Wire | 18 AWG | Signal | $2 |
| 1 | Fan | 80mm 12V | Cooling | $5 |
| 1 | Perfboard/PCB | 15×20cm | Assembly | $5 |
| - | Terminals | Ring, spade | Connections | $3 |
| 1 | Fuse holder | ANL type | Protection | $3 |
| 2 | Fuse | 40A ANL | Overcurrent | $2 ea |

---

## 4.5 BOM Summary

| Category | Est. Total |
|----------|------------|
| Control ICs (SG3525, IR2110) | $8 |
| Passive components | $5 |
| MOSFETs | $6 |
| Transformer | $25 |
| Input capacitors | $4 |
| Output filter | $6 |
| Heatsinks & cooling | $15 |
| Protection | $8 |
| Wiring & mechanical | $15 |

**Phase 4 Total: ~$92**

---

## 4.6 Build Sequence

### Weekend 1: Control Board

```
Day 1 - SG3525 Section:
□ Build oscillator (Rt, Ct)
□ Add dead-time resistor
□ Add soft-start capacitor
□ Test: verify output frequency (~100Hz oscillator)
□ Test: verify dead-time on scope

Day 2 - Driver Section:
□ Build IR2110 circuits
□ Add bootstrap diodes and capacitors
□ Connect SG3525 outputs to drivers
□ Test: verify gate drive signals
```

### Weekend 2: Power Stage

```
Day 1 - H-Bridge Assembly:
□ Mount MOSFETs on heatsinks
□ Wire H-bridge configuration
□ Add freewheeling diodes
□ Connect transformer primary
□ Triple-check wiring!

Day 2 - Integration & Testing:
□ Connect control board to power board
□ Add input capacitors
□ Test with LOW voltage first (12V)
□ Measure output with light load
□ Build output filter
□ Full voltage test with 100W load
□ Gradually increase load
```

---

## 4.7 Testing & Safety

### Critical Checks Before Power-On

```
□ No shorts between +V and GND
□ All MOSFETs oriented correctly (check pinout!)
□ Gate resistors installed
□ Fuse installed
□ Heatsinks properly mounted
□ No exposed 220V connections
```

### Test Sequence

```
Stage 1: Control only (no power stage connected)
- Verify SG3525 oscillating
- Check dead-time between outputs
- Verify IR2110 outputs

Stage 2: Low voltage (12V input)
- Connect small transformer
- Light bulb load (40W)
- Check output voltage
- Monitor temperatures

Stage 3: Full voltage (24V input)
- Use proper transformer
- Start with 100W load
- Check waveform if possible
- Run for 30 minutes
- Monitor all temperatures

Stage 4: Load testing
- 200W for 30 minutes
- 400W for 15 minutes
- 500W for 5 minutes
- Note maximum temperature
```

---

## 4.8 Phase 4 Learning Outcomes

After completing Phase 4:
- [ ] Understand SG3525 PWM controller
- [ ] Can set frequency and dead-time
- [ ] Know soft-start operation
- [ ] Built 500W modified sine inverter!
- [ ] Understand output filtering
- [ ] Ready for pure sine wave (Part 3)!

---

# PHASE 5: TRANSFORMER WINDING PRACTICE

**Goal:** Learn to wind transformers before attempting large power cores
**Time:** 1 weekend
**Cost:** ~$25

---

## 5.1 Why Practice Winding?

```
THE PROBLEM:
────────────
In Part 3, you'll wind a 6-15kW toroidal transformer.
That core costs $50-70.
One mistake = expensive paperweight.

THE SOLUTION:
─────────────
Practice on small, cheap cores first!
Learn technique on $10 core, not $70 core.
```

### What You'll Learn

| Skill | Importance for Part 3 |
|-------|----------------------|
| Counting turns accurately | Critical - wrong ratio = wrong voltage |
| Even winding distribution | Affects efficiency and heat |
| Insulation between layers | Safety - prevents arc-through |
| Terminating wire ends | Reliability - bad joints = failure |
| Testing before varnishing | Saves time and money |

---

## 5.2 Practice Core Selection

### Option A: Small Toroid (Recommended)

```
┌─────────────────────────────────────────┐
│         SMALL TOROIDAL CORE             │
│                                         │
│   Specifications:                       │
│   ├── Outer diameter: 50-70mm          │
│   ├── Inner diameter: 30-40mm          │
│   ├── Height: 20-30mm                  │
│   ├── Power rating: 30-50VA            │
│   ├── Cost: $8-12                      │
│   └── Source: Salvage from old PSU     │
│              or buy new                 │
│                                         │
│   Perfect for learning technique!       │
│                                         │
└─────────────────────────────────────────┘
```

### Option B: EI Core (Budget Alternative)

```
┌─────────────────────────────────────────┐
│           EI LAMINATED CORE             │
│                                         │
│   Salvage from old transformer          │
│   (wall wart, old TV, etc.)            │
│                                         │
│   Pros:                                 │
│   + Free (salvaged)                     │
│   + Easier to wind (rectangular)        │
│                                         │
│   Cons:                                 │
│   - Different technique than toroid     │
│   - Less relevant for Part 3            │
│                                         │
└─────────────────────────────────────────┘
```

---

## 5.3 Project: 12V to 12V Isolation Transformer

### Why 1:1 Ratio?

```
Primary: 12V AC (from your Part 1B inverter!)
Secondary: 12V AC (isolated)

Benefits:
├── Easy to test (same voltage in and out)
├── If ratio is wrong, obvious immediately
├── Safe - only 12V, can't hurt you
└── Actually useful for isolating equipment
```

### Calculating Turns

```
For small toroidal core (~50VA, ~10cm²):

Volts per turn ≈ 0.2-0.3V (depends on core)

Conservative calculation:
V/turn = 0.25V

For 12V: 12V / 0.25 = 48 turns

Both primary and secondary: 48 turns each

Wire gauge (for 2A test):
├── Primary: 22 AWG (0.64mm²)
└── Secondary: 22 AWG (0.64mm²)
```

---

## 5.4 Winding Step-by-Step

### Materials Needed

| Qty | Item | Purpose | Cost |
|-----|------|---------|------|
| 1 | Toroidal core | 30-50VA | $10 |
| 10m | Enameled wire | 22 AWG primary | $3 |
| 10m | Enameled wire | 22 AWG secondary | $3 |
| 1 roll | Kapton tape | Insulation | $5 |
| 1 roll | Electrical tape | Layer insulation | $1 |
| 1 | Sandpaper | Wire stripping | $1 |

**Total: ~$23**

### Step 1: Core Preparation

```
1. Clean the core
   └── Remove dust, oil, debris

2. Wrap with insulation tape (2 layers)
   ┌───────────────────┐
   │   ┌───────────┐   │
   │   │   CORE    │   │  ← Tape wrapped around
   │   │           │   │
   │   └───────────┘   │
   └───────────────────┘

3. Mark starting point
   └── Use permanent marker or tape flag
```

### Step 2: Wind Primary

```
WINDING TECHNIQUE:
──────────────────

Pass wire through center hole, around outside, repeat.

               Pass wire through
                     │
                     ▼
                ┌─────────┐
   Wind around→ │    ○    │ ←Wind around
     outside    │   │ │   │   outside
                │   │ │   │
                └─────────┘
                    ↑
              Core center hole

TIPS:
├── Keep wire TIGHT (no loose loops)
├── Distribute evenly around core
├── Count every pass = 1 turn
├── Use tally marks on paper every 10 turns
└── Don't overlap excessively
```

### Step 3: Insulate Primary

```
After primary is complete:

1. Secure end temporarily (tape)
2. Wrap entire primary with tape layer
   ┌───────────────────┐
   │  █████████████    │
   │  █ PRIMARY █████  │  ← Tape over primary
   │  █████████████    │
   └───────────────────┘
3. This prevents secondary touching primary
```

### Step 4: Wind Secondary

```
Same technique as primary:

1. Start at OPPOSITE SIDE from primary start
   (distributes magnetic coupling evenly)

2. Wind 48 turns (same as primary)

3. Count carefully!

4. Keep even tension
```

### Step 5: Terminate Wires

```
Wire ends need proper termination:

STRIPPING ENAMEL:
─────────────────
1. Sand last 15mm of wire
2. Use fine sandpaper (400 grit)
3. Sand all around until copper shines
4. Test with multimeter (continuity)

TERMINATION OPTIONS:
────────────────────
Option A: Solder to short solid wire
Option B: Crimp ring terminal
Option C: Solder directly to test leads
```

---

## 5.5 Testing Your Transformer

### Test 1: Continuity

```
With multimeter in continuity mode:

□ Primary: Both ends should beep (continuous)
□ Secondary: Both ends should beep (continuous)
□ Primary to Secondary: Should NOT beep (isolated!)

If primary-secondary shows continuity = INSULATION FAILURE
    → Unwrap, add more insulation, rewind secondary
```

### Test 2: Voltage Ratio

```
SAFETY: Use low voltage AC source!

Connect 12V AC to primary (from your Part 1B inverter)
Measure secondary with multimeter (AC mode)

Expected: 11-13V AC (close to 12V)

If voltage is way off:
├── Too high: Secondary has too few turns
├── Too low: Secondary has too many turns
└── Recalculate and adjust
```

### Test 3: Load Test

```
Connect 12V bulb (5-10W) to secondary
Power primary with 12V AC

Check:
□ Bulb lights at normal brightness
□ Transformer doesn't get hot quickly
□ No buzzing or vibration
□ Output voltage stays stable

Run for 10 minutes and feel temperature.
Warm = OK
Hot = Too much load or winding problem
```

---

## 5.6 Common Mistakes and Fixes

| Mistake | Symptom | Fix |
|---------|---------|-----|
| Wrong turn count | Voltage too high/low | Recount, add/remove turns |
| Loose winding | Buzzing, inefficient | Rewind tighter |
| Poor insulation | Sparking, shorts | Add more tape between layers |
| Wire too thin | Overheating | Use thicker wire |
| Uneven distribution | One side hot | Spread turns evenly |
| Bad termination | Intermittent | Re-strip and resolder |

---

## 5.7 Advanced Practice (Optional)

### Build a Step-Up Transformer

```
After mastering 1:1, try 12V to 24V (1:2 ratio):

Primary: 48 turns
Secondary: 96 turns (use thinner wire - less current)

This teaches:
├── Different wire gauges for different currents
├── Higher turn counts (more counting discipline!)
└── Voltage doubling effect
```

### Practice with Toroidal Core from AliExpress

```
You can buy toroidal cores specifically for practice:

Search: "toroidal transformer core 50VA"
Cost: $8-15 shipped

Sizes to look for:
├── 50VA (small, perfect for practice)
├── 100VA (medium, more wire but still cheap)
└── Avoid >200VA for practice (expensive if ruined)
```

---

## 5.8 Phase 5 Learning Outcomes

After completing Phase 5:
- [ ] Wound a working isolation transformer
- [ ] Know how to calculate turns for voltage
- [ ] Understand insulation requirements
- [ ] Can terminate enameled wire properly
- [ ] Tested transformer under load
- [ ] Ready for larger transformer in Part 3!

---

## 5.9 Preparing for Part 3

```
YOUR PART 3 TRANSFORMER WILL BE:
────────────────────────────────
• 6-15kW rating (vs 50W practice)
• 180-220mm diameter (vs 50-70mm)
• 15-25kg weight (vs 200g)
• 6 AWG primary (vs 22 AWG)
• $50-70 core (vs $10)

But the TECHNIQUE is identical!
What you learned on the small core scales up.

The big difference: you'll use a shuttle or bobbin
to pass thick wire through the center hole.

     ┌──────────────────────────────────┐
     │   WINDING SHUTTLE (for thick wire)│
     │                                   │
     │   ╔═══════════════════╗          │
     │   ║   Pre-cut wire    ║ ←Wrapped │
     │   ║   wound on spool  ║  on form │
     │   ╚═══════════════════╝          │
     │                                   │
     │   Pass shuttle through center,    │
     │   unwind as you go.               │
     │                                   │
     └──────────────────────────────────┘
```

---

# PART 3 SUMMARY

## What You've Achieved

```
┌─────────────────────────────────────────────────────────────┐
│                    YOUR PROGRESSION                          │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Phase 1: CD4047 oscillator theory & testing                │
│     ↓                                                        │
│  Phase 2: 150W push-pull inverter (working!)                │
│     ↓                                                        │
│  Phase 3: H-bridge & IR2110 gate drivers                    │
│     ↓                                                        │
│  Phase 4: 500W modified sine inverter (working!)            │
│     ↓                                                        │
│  Phase 5: Transformer winding practice                       │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

## Skills Acquired

| Skill | Phase |
|-------|-------|
| RC oscillator calculation | 1 |
| MOSFET fundamentals | 2 |
| Push-pull topology | 2 |
| Transformer operation | 2 |
| H-bridge topology | 3 |
| Gate driver circuits | 3 |
| Bootstrap operation | 3 |
| PWM control (SG3525) | 4 |
| Dead-time control | 4 |
| Output filtering | 4 |
| **Transformer winding** | **5** |
| **Turn ratio calculations** | **5** |
| **Wire termination** | **5** |

## Investment

| Item | Cost |
|------|------|
| Phase 1 components | $5 |
| Phase 2 (150W inverter) | $35 |
| Phase 3 (H-bridge practice) | $25 |
| Phase 4 (500W inverter) | $50 |
| Phase 5 (winding practice) | $25 |
| **Part 3 Total** | **~$140** |

## Ready for Part 4!

You now have the foundation for:
- **SPWM (Sinusoidal PWM)** - digital pure sine generation
- **EG8010** - dedicated pure sine controller
- **1kW+ power levels**
- **Eventually: OzInverter 6-15kW!**
