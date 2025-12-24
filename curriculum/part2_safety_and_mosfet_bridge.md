# Inverter Curriculum Part 2
## Safety Foundation & MOSFET Bridge Projects

**Prerequisites:** Completed Part 1 Fundamentals
**Output:** Safe work habits + first MOSFET-based inverter (12V AC, 20W)
**Time:** 1-2 weekends
**Cost:** ~$25

---

## Why This Module Exists

```
BEFORE Part 1B:                    AFTER Part 1B:
─────────────────                  ─────────────────
BC547 transistor (100mA)           IRF540 MOSFET (30A)
9V battery (safe)                  12V battery (higher current)
LED loads (harmless)               Small transformer
No high voltage                    Understanding of HV dangers
                                   ↓
                                   READY for Part 3 (220V)
```

This module bridges the gap between safe 9V experiments and real power electronics.

---

# SECTION A: HIGH VOLTAGE SAFETY

## A.1 Electricity Can Kill You

### The Danger Threshold

```
┌─────────────────────────────────────────────────────────────────┐
│                    CURRENT THROUGH HUMAN BODY                    │
│                                                                  │
│   1 mA    ───  Barely perceptible                               │
│   5 mA    ───  Painful but can let go                           │
│  10 mA    ───  Muscle lock (can't release grip) ⚠️              │
│  30 mA    ───  Breathing difficulty                             │
│  75 mA    ───  Ventricular fibrillation (heart stops) ☠️        │
│ 100 mA    ───  Usually fatal without immediate help             │
│                                                                  │
│   At 230V AC across dry skin (~1000Ω):                          │
│   I = 230V / 1000Ω = 230mA  ← MORE THAN DOUBLE FATAL DOSE       │
│                                                                  │
│   With wet/sweaty skin (~100Ω):                                 │
│   I = 230V / 100Ω = 2.3A   ← INSTANT DEATH                      │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Why AC is More Dangerous Than DC

```
DC (Direct Current):
├── Body can sometimes tolerate higher levels
├── Causes muscle contraction but you might push away
└── Burns are localized

AC (Alternating Current):
├── 50Hz matches body's nerve frequency
├── Causes "can't let go" muscle lock
├── Disrupts heart rhythm easily
└── MUCH more dangerous at same voltage level

220V AC IS LETHAL. RESPECT IT.
```

---

## A.2 Safe Work Practices

### The One-Hand Rule

```
ALWAYS work with one hand behind your back or in pocket!

BAD (Two hands):                  GOOD (One hand):
     ┌─────────┐                      ┌─────────┐
     │ CIRCUIT │                      │ CIRCUIT │
     └────┬────┘                      └────┬────┘
          │                                │
    ┌─────┴─────┐                    ┌─────┴─────┐
    │           │                    │           │
   LEFT       RIGHT                 LEFT        RIGHT
   HAND       HAND                  HAND        (in pocket)
    │           │                    │
    └─────┬─────┘                    │
          │                          │
       THROUGH                    THROUGH
        HEART                       ARM
       (FATAL)                   (Painful but survivable)
```

### Before Working on Any Circuit

```
MANDATORY CHECKLIST:
□ 1. Disconnect power source (unplug, remove battery)
□ 2. Wait 2 minutes (capacitors discharge)
□ 3. Verify with multimeter (should read 0V)
□ 4. Short large capacitors with resistor (1kΩ)
□ 5. Verify again with multimeter
□ 6. Only then touch the circuit
```

### Capacitor Discharge Procedure

```
Large capacitors store LETHAL energy even after power off!

Example: 4700µF at 63V
Energy = ½CV² = ½ × 0.0047 × 63² = 9.3 Joules
This is enough to kill you.

SAFE DISCHARGE METHOD:

    Capacitor ──────[1kΩ 5W]────── GND
         +            │
         │          Wait
         │        t = 5×RC
         │      = 5 × 0.0047 × 1000
         │      = 23.5 seconds
         │
         └── Then verify with meter!

NEVER short capacitors directly (spark, damage, injury)
```

---

## A.3 Workspace Safety

### Required Safety Equipment

```
MINIMUM REQUIREMENTS:
─────────────────────
□ Fire extinguisher (CO2 or dry powder, NOT water!)
□ First aid kit
□ Insulated tools (screwdrivers, pliers)
□ Safety glasses
□ Non-conductive shoes (rubber soles)
□ Non-conductive floor mat
□ Good lighting
□ Clean, organized workspace

RECOMMENDED ADDITIONS:
─────────────────────
□ Isolation transformer (for safer testing)
□ GFCI/RCD outlet (trips at 30mA)
□ Fume extractor (for soldering)
□ Multimeter with CAT III rating
```

### Isolation Transformer Explained

```
Without isolation:                With isolation transformer:

Grid ─── Hot ───┬── Circuit      Grid ─── Hot ───┐
                │                                │
               You                         [TRANSFORMER]
                │                                │
Grid ─── Neutral┴── Ground       Isolated ──┬── Circuit
                                            │
                                           You
                                            │
                                 Isolated ──┴── (floating)

If you touch Hot wire:            If you touch either wire:
Current flows through you         No return path to ground
to ground = ELECTROCUTION         Current can't flow = SAFE*

*Still dangerous between the two isolated wires!
```

---

## A.4 Emergency Procedures

### If Someone Gets Shocked

```
IMMEDIATE ACTIONS:
──────────────────
1. DON'T TOUCH THEM (you'll be shocked too!)
2. Disconnect power:
   - Pull plug
   - Trip breaker
   - Use DRY wood/plastic to push them away
3. Call emergency services
4. If not breathing: CPR immediately
5. If breathing: Recovery position, keep warm

CPR BASICS (if needed):
───────────────────────
• 30 chest compressions (5-6cm deep, 100-120/min)
• 2 rescue breaths
• Repeat until help arrives or they recover
```

### If Circuit Catches Fire

```
ACTIONS:
────────
1. Disconnect power if safe to do so
2. Use CO2 or dry powder extinguisher
3. NEVER use water on electrical fire!
4. If fire spreads: evacuate, call fire department
5. Don't re-enter until cleared

PREVENTION:
───────────
• Always have fuse in circuit
• Don't leave powered circuits unattended
• Check component ratings
• Use appropriate wire gauges
```

---

## A.5 Safety Quiz (Complete Before Part 2)

Answer these correctly before proceeding:

```
1. At what current level does electricity typically become fatal?
   a) 1 mA    b) 30 mA    c) 75 mA    d) 1 A

2. Why should you work with one hand in your pocket?
   a) To look professional
   b) To prevent current flowing through heart
   c) To keep hand warm
   d) To hold tools

3. Before touching a circuit, you should:
   a) Just turn off the switch
   b) Disconnect power and immediately touch
   c) Disconnect, wait, verify with meter, then touch
   d) Wear rubber gloves and touch

4. What type of fire extinguisher should you use on electrical fire?
   a) Water
   b) CO2 or dry powder
   c) Foam
   d) Any type

5. If someone is being electrocuted, you should:
   a) Grab them and pull away
   b) Throw water to interrupt the current
   c) Disconnect power using insulated object
   d) Wait for them to let go

ANSWERS: 1-c, 2-b, 3-c, 4-b, 5-c

If you got any wrong, re-read the section!
```

---

# SECTION B: MOSFET BRIDGE PROJECT

## B.1 Understanding MOSFETs

### Transistor vs MOSFET Comparison

```
BC547 (Transistor - Part 1):        MOSFET (Part 2 onwards):
────────────────────────────        ────────────────────────
• Current controlled                • Voltage controlled
• Low power (100mA max)             • High power (10-100A+)
• Requires base current             • Almost no gate current
• Linear amplification              • Used as a switch
• Gets hot easily                   • Very efficient

     BJT                                 MOSFET
      C                                    D (Drain)
      │                                    │
   B──┤                               G───┤├──
      │                                    │
      E                                    S (Source)

To turn ON:                         To turn ON:
Current INTO base                   Voltage on gate (10-15V)
(wastes power)                      (almost no current needed)
```

### Your First MOSFET: IRF540

```
┌──────────────────────────────────────────────────────────────┐
│                     IRF540 MOSFET                             │
│                                                               │
│   Specifications:                                             │
│   ├── Vds (max): 100V  (drain to source voltage)             │
│   ├── Id (max): 28A    (continuous drain current)            │
│   ├── Rds(on): 0.077Ω  (on resistance)                       │
│   ├── Vgs(th): 2-4V    (threshold voltage)                   │
│   └── Vgs: 10V recommended for full turn-on                  │
│                                                               │
│   Package: TO-220                                             │
│   ┌──────────┐                                               │
│   │  ┌───┐   │                                               │
│   │  │   │   │ ← Metal tab (connected to Drain)              │
│   │  └───┘   │                                               │
│   └──┬─┬─┬───┘                                               │
│      │ │ │                                                    │
│      G D S (Gate, Drain, Source)                             │
│      1 2 3                                                    │
│                                                               │
│   Cost: ~$1                                                   │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

### MOSFET as a Switch

```
Simple MOSFET switch circuit:

    +12V
      │
      │
   [LOAD]  (bulb, motor, etc.)
      │
      D (Drain)
      │
      ├───┤├──── S (Source) ──── GND
      │
      G (Gate)
      │
      │
    [10kΩ] ← Pull-down (ensures OFF when no signal)
      │
     GND

Control signal (0V or 12V):
├── 0V on Gate → MOSFET OFF → No current through load
└── 12V on Gate → MOSFET ON → Current flows, load powered
```

---

## B.2 Baby Push-Pull Circuit

### Project Goal

Build a simple 12V push-pull circuit with two MOSFETs that creates AC from DC.

```
This is a MINIATURE version of what you'll build in Part 2!

Power: 12V DC input, 12V AC output (not 220V)
Load: 12V light bulb (5-20W)
Purpose: Learn MOSFET switching in safe environment
```

### Circuit Schematic

```
                        +12V
                          │
           ┌──────────────┼──────────────┐
           │              │              │
           │         [TRANSFORMER]       │
           │          12V CT 12V         │
           │           ┌──┴──┐           │
           │           │     │           │
           │    Pri ───┤     ├─── Pri    │
           │     A     │     │     B     │
           │           │ CT  │           │
           │           └──┬──┘           │
           │              │              │
           D              │              D
           │              │              │
    Q1 ───┤├──           │          ───┤├── Q2
    IRF540 │             GND            │ IRF540
           S                             S
           │                             │
           └──────────────┬──────────────┘
                          │
                         GND

Gate drive from CD4047 (oscillator from Part 1):

CD4047               CD4047
Pin 10 ───[100Ω]───► Q1 Gate
Pin 11 ───[100Ω]───► Q2 Gate

        Secondary (12V AC output)
             ┌────────┐
             │  LOAD  │  (12V bulb)
             └────────┘
```

### Bill of Materials

| Qty | Component | Value/Spec | Purpose | Price |
|-----|-----------|------------|---------|-------|
| 2 | MOSFET | IRF540 | Switching | $1 ea |
| 1 | CD4047 | IC | Oscillator | $0.50 |
| 1 | Transformer | 12V-0-12V CT, 1A | Step-down used backwards | $5 |
| 2 | Resistor | 100Ω 1/4W | Gate resistors | $0.10 |
| 2 | Resistor | 10kΩ 1/4W | Gate pull-down | $0.10 |
| 2 | Zener diode | 12V | Gate protection | $0.20 |
| 1 | Capacitor | 100µF/25V | Power supply filter | $0.20 |
| 2 | Capacitor | 100nF | Bypass | $0.10 |
| 1 | Bulb | 12V 10W | Test load | $1 |
| 1 | Heatsink | Small TO-220 | Cooling | $0.50 |
| 1 | Breadboard | Half-size | Assembly | $3 |
| - | Wire | 22 AWG | Connections | $1 |

**Total: ~$15**

---

## B.3 Build Instructions

### Step 1: Build CD4047 Oscillator (Review from Part 1)

```
CD4047 wired for 50Hz square wave output:

        ┌────────────────────────┐
        │                        │
   +12V─┤14 VDD          VSS 7├──GND
        │                        │
 100kΩ──┤1 R              Q 10├──► To Q1 gate
        │                        │
 100kΩ──┤2 C             ~Q 11├──► To Q2 gate
        │                        │
 0.1µF──┤3 RC-C          OSC 13├──NC
        │                        │
   GND──┤4 AST           RET 12│──NC
        │                        │
   +12V─┤5 ASTB          TRG  8│──NC
        │                        │
        │       CD4047           │
        └────────────────────────┘

Frequency ≈ 1/(4.4 × R × C) = 1/(4.4 × 100k × 0.1µ) ≈ 23Hz per output
Combined: 50Hz (approximately)
```

### Step 2: Wire Gate Drive Circuit

```
IMPORTANT: Gate protection!

From CD4047 Pin 10:
        │
      [100Ω]  ← Limits current, slows switching
        │
        ├──────► Q1 Gate
        │
      [10kΩ]  ← Pull-down (ensures OFF)
        │
       GND

      [12V Zener] ← Clamps gate voltage
        │
       GND

Repeat for Q2 from Pin 11.
```

### Step 3: Wire Power Stage

```
CAUTION: Use thick wire for power connections!

+12V from battery or power supply
    │
    ├───────────────────────┐
    │                       │
    │    TRANSFORMER        │
    │    Primary CT         │
    │                       │
    └───► CT (center tap)   │
               │            │
         ┌─────┴─────┐      │
         │           │      │
    Pri A│           │Pri B │
         │           │      │
    D Q1─┘           └─D Q2 │
         │           │      │
    IRF540           IRF540 │
         │           │      │
    S────┼───────────┤      │
         │           │      │
         └─────┬─────┘      │
               │            │
              GND ──────────┘
```

### Step 4: Connect Load

```
Transformer Secondary:
        │
    ┌───┴───┐
    │ 12V   │
    │  AC   │
    │       │
    └───┬───┘
        │
    [12V Bulb]
        │
    ────┴────
       GND (secondary)
```

---

## B.4 Testing Procedure

### Test 1: No Power Check

```
Before applying power:
□ Verify all connections with multimeter (continuity)
□ Check no shorts between +12V and GND
□ Verify MOSFETs are oriented correctly
□ Confirm gate resistors in place
□ No loose wires
```

### Test 2: Oscillator Only

```
Disconnect MOSFETs (remove from breadboard)
Apply +12V
Check CD4047 outputs with multimeter (AC mode)
Should see ~6V AC on Pin 10 and Pin 11
Confirm they're opposite phase (one high when other low)
```

### Test 3: Full Circuit, No Load

```
Reinstall MOSFETs
Disconnect bulb (open secondary)
Apply +12V
Feel MOSFETs - should be cool/warm, NOT HOT
Listen for transformer hum (normal 50Hz buzz)
Measure secondary voltage: should be ~12V AC
```

### Test 4: With Load

```
Connect 12V bulb
Apply power
Bulb should light!
Check MOSFET temperature: warm is OK, hot = problem
Run for 5 minutes
If stable, congratulations!
```

---

## B.5 Troubleshooting

| Symptom | Likely Cause | Solution |
|---------|--------------|----------|
| No output | CD4047 not oscillating | Check power, RC values |
| MOSFETs very hot | No gate drive / shoot-through | Check gate signals |
| Low output voltage | Load too heavy | Use smaller bulb |
| Transformer buzzing loudly | Frequency wrong | Adjust RC values |
| One MOSFET hot, other cool | Asymmetric drive | Check both gate signals |
| Bulb flickers | Loose connection | Check all joints |

---

## B.6 What You Learned

```
CONCEPTS MASTERED:
─────────────────
✓ MOSFET as a high-power switch
✓ Gate drive requirements (voltage, not current)
✓ Gate protection (resistor, zener)
✓ Push-pull topology
✓ Transformer operation with switching
✓ Testing methodology (staged power-up)
✓ Thermal awareness (checking temperatures)

SKILLS FOR PART 2:
─────────────────
✓ Ready for higher power MOSFETs (IRF3205)
✓ Understand push-pull → H-bridge evolution
✓ Know how to test safely
✓ Comfortable with transformer connections
```

---

## B.7 Bridge to Part 3

### What Changes in Part 3

| Baby MOSFET (This project) | Part 3 Modified Sine |
|----------------------------|---------------------|
| 12V DC input | 12-24V DC input |
| 12V AC output | **220V AC output** |
| 20W power | 150-500W power |
| IRF540 (28A) | IRF3205 (110A) |
| Push-pull | Push-pull → H-bridge |
| No heatsink needed | Proper heatsinking required |
| Breadboard | Perfboard/PCB |

### Key Safety Difference

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│   THIS PROJECT: 12V output                                      │
│   ─────────────────────────                                     │
│   • Cannot hurt you                                             │
│   • Safe to touch (don't, but won't kill you)                  │
│   • Good for learning                                           │
│                                                                  │
│   PART 2: 220V output                                           │
│   ──────────────────────                                        │
│   • LETHAL VOLTAGE                                              │
│   • Never touch when powered                                    │
│   • Always discharge before touching                            │
│   • Follow ALL safety procedures from Section A                 │
│                                                                  │
│   You MUST internalize safety before proceeding!                │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## B.8 Checklist Before Part 2

Complete all items before proceeding:

```
SAFETY:
□ I can explain why 220V is dangerous
□ I know the one-hand rule and why it matters
□ I know how to discharge capacitors safely
□ I have a fire extinguisher in my workspace
□ I passed the safety quiz with 100%

TECHNICAL:
□ I built the baby MOSFET push-pull successfully
□ I understand how MOSFETs switch on/off
□ I know why gate resistors are needed
□ I can measure AC voltage with multimeter
□ I understand push-pull transformer operation

PRACTICAL:
□ I'm comfortable with soldering (from Part 1)
□ I can read a schematic and build from it
□ I know how to troubleshoot systematically
□ I have appropriate tools for Part 2

If any box is unchecked, go back and complete it!
```

---

# SECTION C: OPTIONAL INTERMEDIATE PROJECT

## C.1 H-Bridge LED Driver (No Transformer)

For extra practice before Part 2, build this H-bridge:

```
This teaches H-bridge topology without high voltage.

                    +12V
                      │
         ┌───────────┬┴┬───────────┐
         │           │ │           │
         D           │ │           D
         │           │ │           │
   Q1───┤├───────────┘ └───────────┤├───Q3
   IRF540│                         │IRF540
         S                         S
         │                         │
         ├─────[LED STRING]────────┤
         │                         │
         D                         D
         │                         │
   Q2───┤├─────────────────────────┤├───Q4
   IRF540│                         │IRF540
         S                         S
         │                         │
         └───────────┬─────────────┘
                     │
                    GND

Drive pattern:
Phase 1: Q1+Q4 ON, Q2+Q3 OFF → Current left-to-right
Phase 2: Q2+Q3 ON, Q1+Q4 OFF → Current right-to-left

LED string sees AC! (flickers at 50Hz, appears steady)
```

This is the exact topology used in Part 2-3, but at safe 12V.

---

## Summary

```
PART 1B COMPLETION:
═══════════════════

□ Section A: Safety Foundation
  └── Understanding of electrical dangers
  └── Safe work practices memorized
  └── Emergency procedures known
  └── Safety quiz passed

□ Section B: Baby MOSFET Project
  └── First MOSFET circuit working
  └── Push-pull topology understood
  └── Transformer switching mastered
  └── Testing methodology learned

□ Section C: Optional H-Bridge (if time permits)
  └── H-bridge topology understood
  └── Ready for Part 2!

TOTAL TIME: 1-2 weekends
TOTAL COST: ~$25
CONFIDENCE GAINED: Priceless
```

---

**You are now ready for Part 3: Modified Sine Wave Inverter!**

*Remember: 220V will not forgive mistakes. Apply everything you learned here.*
