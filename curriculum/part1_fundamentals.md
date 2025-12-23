# Step 1: Fundamentals Course

## From Zero to Swarm Microgrid Principles

**Goal:** Build foundation skills AND understand bi-directional power flow early—the core principle behind your swarm microgrid.

---

## Overview

```
Phase 1: Pure Basics (no ICs)           ~2-3 weekends
         → Ohm's Law, switches, transistors
         → POWER FLOW: diodes, direction control

Phase 2: 555 Timer                      ~2-3 weekends
         → Timing, oscillation, relay control

Phase 3: Control & H-Bridge             ~2-3 weekends
         → PWM, sequential switching
         → MINI INVERTER: H-bridge principle!

Phase 4: Measurement                    ~1-2 weekends
         → Build your own test tools

Total: 8-12 weekends, ~$70 components
```

---

## Why This Order?

```
Project 1.1 (LED)          → You learn: current flows ONE direction
Project 1.4 (Diode)        → You learn: diodes ENFORCE one-way flow
Project 1.5 (Bi-dir)       → You learn: transistors allow CONTROL of direction
Project 1.9 (H-bridge)     → You learn: H-bridge = REVERSIBLE power flow
Project 1.10 (Auto-sense)  → You learn: automatic direction = DROOP CONTROL!

By end of Step 1, you understand the swarm microgrid principle
before touching any dangerous voltages.
```

---

# MASTER TOOL LIST (Buy Once)

| Tool | Purpose | Est. Price |
|------|---------|------------|
| Soldering iron 30-60W | Assembly | $20-40 |
| Solder 0.8mm (60/40) | Joints | $8 |
| Desoldering pump | Fix mistakes | $8 |
| Digital multimeter | Measurement | $20-40 |
| Breadboard 830 points | Prototyping | $5 |
| Jumper wire kit | Connections | $5 |
| Wire stripper/cutter | Prep wire | $10 |
| Safety glasses | Protection | $5 |

**Minimum tools: ~$80-120**

Optional but recommended:
| Tool | Purpose | Est. Price |
|------|---------|------------|
| Soldering station (temp control) | Better control | $50-80 |
| Oscilloscope (or DSO138 kit) | See waveforms | $25-150 |
| Helping hands | Hold work | $15 |

---

# PHASE 1: Pure Basics + Power Flow Principles

**Time:** 2-3 weekends
**Cost:** ~$30
**Key Learning:** Current direction, one-way vs bi-directional flow

---

## Project 1.1: LED + Resistor (Ohm's Law)

**Goal:** Understand V = I × R, current flows in ONE direction

**Schematic:**
```
    +9V
     │
    [R] 330Ω
     │
    LED  ← Current flows THIS way only
     │       (long leg = +, short leg = -)
    GND
```

**Bill of Materials:**
| Qty | Component | Value | Est. Price |
|-----|-----------|-------|------------|
| 5 | LED Red | 5mm | $1 |
| 5 | LED Green | 5mm | $1 |
| 5 | LED Yellow | 5mm | $1 |
| 10 | Resistor | 100Ω 1/4W | $0.50 |
| 10 | Resistor | 220Ω 1/4W | $0.50 |
| 10 | Resistor | 330Ω 1/4W | $0.50 |
| 10 | Resistor | 1kΩ 1/4W | $0.50 |
| 10 | Resistor | 10kΩ 1/4W | $0.50 |
| 1 | 9V battery clip | - | $1 |
| 1 | 9V battery | - | $3 |
| 1 | Breadboard | 400pts | $3 |

**Subtotal: ~$12**

**Exercises:**
1. Calculate R for 9V supply, red LED (Vf=2V), 15mA → R = 467Ω
2. Build circuit, measure current with multimeter
3. Reverse LED → no light (wrong direction!)
4. Remove resistor → LED burns ($0.10 lesson)

**Connection to Microgrid:** LEDs only work one way. Inverters have the same issue—standard ones only push power OUT, not pull power IN.

---

## Project 1.2: Switch Circuits

**Goal:** Mechanical control of current flow

**Schematics:**

**1.2a: Simple ON/OFF**
```
    +9V ── Switch ── R ── LED ── GND
```

**1.2b: AND Gate (Series switches)**
```
    +9V ── Sw.A ── Sw.B ── R ── LED ── GND
    (Both needed to light LED)
```

**1.2c: OR Gate (Parallel switches)**
```
         ┌─ Sw.A ─┐
    +9V ─┤        ├─ R ── LED ── GND
         └─ Sw.B ─┘
    (Either switch lights LED)
```

**Bill of Materials:**
| Qty | Component | Value | Est. Price |
|-----|-----------|-------|------------|
| 5 | Tactile switch | 6×6mm | $1 |
| 2 | Toggle switch | SPST | $2 |
| 2 | Toggle switch | SPDT | $3 |

**Subtotal: ~$6**

---

## Project 1.3: Transistor as Switch

**Goal:** Electronic switching (foundation for MOSFETs and H-bridge)

**Schematic:**
```
        +9V
         │
        [R1] 330Ω
         │
        LED
         │
         C (collector)
        ╱
   B ──●    NPN (BC547)
        ╲
         E (emitter)
         │
        GND

   Base connects via 10kΩ to button
```

**Bill of Materials:**
| Qty | Component | Value | Est. Price |
|-----|-----------|-------|------------|
| 10 | NPN Transistor | BC547 | $1 |
| 5 | NPN Transistor | 2N2222 | $1 |
| 5 | PNP Transistor | BC557 | $1 |
| 5 | Resistor | 10kΩ | $0.30 |
| 5 | Resistor | 100kΩ | $0.30 |
| 2 | LDR photoresistor | 5mm | $1 |
| 2 | Thermistor | 10kΩ NTC | $1 |

**Subtotal: ~$6**

**Experiments:**
1. Button-controlled LED
2. Light-controlled LED (LDR + transistor)
3. Touch-activated switch (skin resistance)
4. Temperature switch (thermistor)

**Connection to Microgrid:** Transistors let you control HIGH power with LOW power signals. This is exactly how inverters work—small control signal switches big currents.

---

## Project 1.4: Diode - One-Way Power Flow

**Goal:** Understand why standard inverters can't charge batteries backwards

> **THIS IS THE KEY INSIGHT FOR BI-DIRECTIONAL SYSTEMS**

**Schematic:**
```
    Battery A (9V)              Battery B (6V)
        (+)                         (+)
         │                           │
         │      ┌───────┐            │
         └──────┤►  D1  ├────────────┘
                │ 1N4007│
                └───┬───┘
                    │
                   LED  ← Shows current direction
                    │
                   [R] 330Ω
                    │
    GND ────────────┴──────────────── GND
```

**What happens:**
- A (9V) is higher than B (6V)
- Current flows A → B through diode
- LED lights up
- **Reverse batteries:** Diode blocks! No current. LED off.

**Bill of Materials:**
| Qty | Component | Value | Est. Price |
|-----|-----------|-------|------------|
| 10 | Diode | 1N4007 | $1 |
| 10 | Diode | 1N4001 | $0.50 |
| 1 | 4×AA battery holder | 6V | $1 |
| 4 | AA batteries | - | $2 |

**Subtotal: ~$5**

**Experiments:**
1. Build circuit → LED lights (9V powers 6V side)
2. Swap battery positions → LED off (diode blocks)
3. Remove diode, use wire → current flows both ways (dangerous in real circuits!)
4. Add second LED reversed → shows which direction current flows

**Connection to Microgrid:**
```
Standard OzInverter = has "diodes" in the MOSFETs (body diodes)
                    = power can only flow OUT
                    = cannot charge battery from AC bus

This is why you need BI-DIRECTIONAL control!
```

---

## Project 1.5: Transistor Bi-Directional Control

**Goal:** Control power direction with transistors (foundation for bi-directional inverter)

**Schematic:**
```
                 MODE SELECT
           ┌────────┴────────┐
        Button A          Button B
        (A→B)             (B→A)
           │                 │
         [10k]             [10k]
           │                 │
    9V ────┼────┐     ┌──────┼──── 6V
    Bat A  │   B│     │B     │     Bat B
           │  ╱ │     │ ╲    │
           │ ●Q1│     │Q2●   │
           │  ╲ │     │ ╱    │
           │   E│     │E     │
           │    └──┬──┘      │
           │       │         │
           │    LED + R      │
           │    (shows       │
           │     flow)       │
           │       │         │
    GND ───┴───────┴─────────┴──── GND
```

**Operation:**
- Press Button A → Q1 ON → Current flows A→B → LED lights
- Press Button B → Q2 ON → Current flows B→A → LED lights
- **YOU control the direction!**

**Bill of Materials:**
*(Uses transistors from 1.3)*
| Qty | Component | Value | Est. Price |
|-----|-----------|-------|------------|
| 2 | Push button | 6×6mm | (have) |
| 2 | Resistor | 10kΩ | (have) |

**Subtotal: ~$0** (already have parts)

**Experiments:**
1. Press A button → verify current A→B
2. Press B button → verify current B→A
3. Press both → what happens? (short circuit - be quick!)
4. Try with motors instead of LED → motor spins both directions!

**Connection to Microgrid:**
```
This IS the bi-directional principle!

Button A = "EXPORT mode" (send power to grid)
Button B = "IMPORT mode" (receive power, charge battery)

Real inverter uses MOSFETs instead of BC547,
and ESP32 instead of buttons.
```

---

## Project 1.6: Voltage Auto-Sensing (Droop Control Principle!)

**Goal:** Automatic direction control based on voltage difference

> **THIS IS DROOP CONTROL IN MINIATURE**

**Schematic:**
```
    Higher Voltage                    Lower Voltage
    (has excess)                      (needs power)
         │                                 │
    9V ──┼───[1kΩ]────┬────────────────────┼── 6V
         │            │                    │
         │           B│                    │
         │          ╱ │                    │
         └──[10k]──●  │   LED (direction)  │
                    ╲ │        │           │
                     E├────────┼───────────┤
                      │       [R]          │
                      │      330Ω          │
                      │        │           │
    GND ──────────────┴────────┴───────────┴── GND
```

**How it works:**
- 9V > 6V → Base voltage is HIGH → Transistor ON
- Current automatically flows from HIGH to LOW
- **No buttons needed!** Voltage difference controls direction.

**Swap batteries:**
- Now 6V side is "A" and 9V side is "B"
- Transistor turns OFF (base voltage too low)
- Current stops (or reverses if you add second transistor)

**Bill of Materials:**
*(Uses parts from previous projects)*

**Subtotal: ~$0**

**Experiments:**
1. Build with 9V and 6V → LED lights (auto-flow high→low)
2. Swap batteries → LED off or reversed
3. Use potentiometer to simulate "variable solar" → watch LED respond
4. Add second transistor for reverse path → full bi-directional!

**Connection to Microgrid:**
```
This IS droop control!

Real system:
- Battery FULL = higher voltage → pushes power OUT
- Battery LOW = lower voltage → pulls power IN
- No communication needed between inverters!
- Just physics: power flows high→low automatically

Your 9V/6V batteries = two houses in the microgrid
Transistor = the bi-directional inverter
LED = power flowing between them
```

---

## Phase 1 Complete BOM

| Category | Items | Est. Total |
|----------|-------|------------|
| LEDs | 15 pcs various | $3 |
| Resistors | 50+ pcs various | $3 |
| Switches | 10 pcs | $6 |
| Transistors | 20 pcs (NPN+PNP) | $3 |
| Diodes | 20 pcs | $2 |
| Sensors | LDR, thermistor | $2 |
| 9V battery + clip | 1 set | $4 |
| 6V (4×AA) + holder | 1 set | $3 |
| Breadboard | 1 pc | $3 |

**Phase 1 Total: ~$30**

---

# PHASE 2: 555 Timer

**Time:** 2-3 weekends
**Cost:** ~$18
**Key Learning:** Timing, oscillation, frequency control

---

## Project 2.1: 555 Monostable (One-Shot Timer)

**Goal:** Triggered timing, RC time constants

**Schematic:**
```
                +9V
                 │
          ┌──────┼──────┐
          │      8      │
       ┌──┤4    VCC    7├──────────┐
       │  │   RESET     │          │
       │  │             │         [R] 470kΩ
       │  │    555      │          │
       │  │             │          │
       │  │     3       │          │
       │  │    OUT──────┼──[1kΩ]──LED──GND
       │  │             │          │
       │  │  2      6   │         [C] 10µF
       │  │  │      └───┼──────────┤
       │  │  │     5    │          │
       │  │  │    ─┴─   │          │
       │  └──┼──100nF───┼──────────┤
       │     1          │          │
       └─────┴──────────┴──────────┴── GND
             │
          Button → +9V (trigger)
```

**Time = 1.1 × R × C = 1.1 × 470kΩ × 10µF = 5.17 seconds**

**Bill of Materials:**
| Qty | Component | Value | Est. Price |
|-----|-----------|-------|------------|
| 5 | 555 Timer IC | NE555 | $1 |
| 5 | IC Socket | 8-pin DIP | $0.50 |
| 5 | Resistor | 100kΩ | $0.30 |
| 5 | Resistor | 470kΩ | $0.30 |
| 5 | Resistor | 1MΩ | $0.30 |
| 10 | Capacitor | 100nF ceramic | $0.50 |
| 5 | Capacitor | 1µF electrolytic | $0.30 |
| 5 | Capacitor | 10µF electrolytic | $0.30 |
| 5 | Capacitor | 100µF electrolytic | $0.50 |

**Subtotal: ~$4**

---

## Project 2.2: 555 Astable (Oscillator)

**Goal:** Continuous oscillation—this generates the switching signal for inverters!

**Schematic:**
```
                +9V
                 │
          ┌──────┼──────────┐
          │      8          │
       ┌──┤4    VCC        7├──┐
       │  │                 │  │
       │  │      555        │ [R1] 10kΩ
       │  │                 │  │
       │  │       3         │  ├──┐
       │  │      OUT────────┼──┤ [R2] 1kΩ
       │  │                 │  │  │
       │  │    2       6    │  │ [C] 100µF
       │  │    └───────┴────┼──┴──┤
       │  │       5         │     │
       │  │      ─┴─ 100nF  │     │
       └──┴─────────────────┴─────┴── GND
```

**Frequency ≈ 1.44 / ((R1 + 2×R2) × C)**

**Bill of Materials:**
| Qty | Component | Value | Est. Price |
|-----|-----------|-------|------------|
| 5 | Capacitor | 47µF | $0.30 |
| 5 | Capacitor | 220µF | $0.50 |
| 2 | Potentiometer | 10kΩ | $1 |
| 2 | Potentiometer | 100kΩ | $1 |

**Subtotal: ~$3**

**Connection to Microgrid:** This oscillator is the "heartbeat" of an inverter. It determines the AC frequency (50Hz or 60Hz). In your swarm, matching frequencies = synchronized power sharing.

---

## Project 2.3: 555 with Relay

**Goal:** Switch high-power loads, understand inductive protection

**Schematic:**
```
        +9V                     LOAD CIRCUIT
         │
    555 OUT───[1kΩ]──┐         ┌─────────┐
                    B│         │ RELAY   │
                   ╱ │    ┌────┤ NO ● ───┼── Load+
              NPN ●  │    │    │ COM● ───┼── Load-
                   ╲ │    │    └─────────┘
                    E│    │
         ┌──────────┤    │
    +9V ─┤  RELAY   ├────┘
         │  COIL    │
         └────┬─────┘
           ┌──┴──┐
           │ ◄── │ Flyback diode (1N4007)
           └──┬──┘
              │
             GND
```

**Bill of Materials:**
| Qty | Component | Value | Est. Price |
|-----|-----------|-------|------------|
| 2 | Relay | 5V coil, 10A | $3 |
| 2 | Relay | 9V coil, 10A | $3 |
| 5 | Transistor | BC337 | $1 |
| 2 | Transistor | TIP120 | $1 |

**Subtotal: ~$8**

**CRITICAL Learning:** Flyback diode protects against voltage spikes from inductors. Transformer = big inductor. **Every inverter needs flyback protection!**

---

## Phase 2 Complete BOM

| Category | Items | Est. Total |
|----------|-------|------------|
| 555 ICs + sockets | 5 each | $1.50 |
| Resistors | Various | $2 |
| Capacitors | Various | $3 |
| Potentiometers | 4 pcs | $2 |
| Relays | 4 pcs | $6 |
| BC337, TIP120 | 7 pcs | $2 |

**Phase 2 Total: ~$18**

---

# PHASE 3: Control & Mini H-Bridge

**Time:** 2-3 weekends
**Cost:** ~$15
**Key Learning:** PWM, H-bridge topology—the actual inverter circuit!

---

## Project 3.1: 555 PWM (Brightness Control)

**Goal:** Pulse Width Modulation—the basis of SPWM in pure sine inverters

**Schematic:**
```
            +9V
             │
      ┌──────┼──────────┐
      │      8          │
   ┌──┤4    VCC        7├──────────┐
   │  │                 │          │
   │  │      555        │         [R] 1kΩ
   │  │                 │          │
   │  │       3         │          │
   │  │      OUT────────┼──[220Ω]──LED
   │  │                 │          │
   │  │    2       6    │      ┌───┤
   │  │    └───────┴────┼──────┤POT│ 10kΩ
   │  │       5         │      └───┤
   │  │      ─┴─ 100nF  │          │
   └──┴─────────────────┴──────────┴── GND
```

**Bill of Materials:**
| Qty | Component | Value | Est. Price |
|-----|-----------|-------|------------|
| 2 | Potentiometer | 10kΩ linear | $1 |
| 1 | Potentiometer | 50kΩ linear | $0.50 |

**Subtotal: ~$2**

**Connection to Microgrid:** SPWM (Sinusoidal PWM) varies the pulse width to create a sine wave average. This project teaches the principle.

---

## Project 3.2: LED Chaser (4017 Counter)

**Goal:** Sequential switching—H-bridge needs precise timing sequence

**Schematic:**
```
        +9V
         │
    ┌────┴────┐          ┌────┴────┐
    │   555   │          │  4017   │
    │         │   CLK    │         │
    │    3────┼──────────┤14    Q0─┼──[330Ω]──LED0
    │         │          │      Q1─┼──[330Ω]──LED1
    │    8────┤──VCC VCC─┤16    Q2─┼──[330Ω]──LED2
    │    1────┤──GND GND─┤8     Q3─┼──[330Ω]──LED3
    │    4────┤──VCC RST─┤15       │
    └─────────┘          └─────────┘
```

**Bill of Materials:**
| Qty | Component | Value | Est. Price |
|-----|-----------|-------|------------|
| 3 | CD4017 | Decade counter | $1 |
| 3 | IC Socket | 16-pin DIP | $0.60 |
| 10 | LED | Various | $1 |

**Subtotal: ~$3**

---

## Project 3.3: Mini H-Bridge (THE INVERTER PRINCIPLE!)

**Goal:** Build the actual inverter topology with safe, low voltage

> **THIS IS THE CORE OF EVERY INVERTER**

**Schematic:**
```
                      +9V
                       │
            ┌──────────┼──────────┐
            │                     │
           Q1                    Q2
          (PNP)                 (PNP)
         BC557                 BC557
            │                     │
            ├─────── LOAD ────────┤
            │      (Motor or      │
            │       LED pair)     │
           Q3                    Q4
          (NPN)                 (NPN)
         BC547                 BC547
            │                     │
            └──────────┼──────────┘
                       │
                      GND

Control Logic:
- Q1+Q4 ON (Q2+Q3 OFF): Current flows LEFT→RIGHT
- Q2+Q3 ON (Q1+Q4 OFF): Current flows RIGHT→LEFT
- Alternate = AC output!
```

**Detailed wiring:**
```
Control A (Q1+Q4):                Control B (Q2+Q3):

  +9V──[10k]──●──Button A         +9V──[10k]──●──Button B
              │                               │
         Q1 Base                         Q2 Base
              │                               │
         Q4 Base (via inverter)          Q3 Base (via inverter)
```

**Bill of Materials:**
| Qty | Component | Value | Est. Price |
|-----|-----------|-------|------------|
| 4 | BC557 | PNP | (have from Phase 1) |
| 4 | BC547 | NPN | (have from Phase 1) |
| 2 | Small DC motor | 3-6V | $2 |
| 4 | Resistor | 1kΩ | (have) |
| 4 | Resistor | 10kΩ | (have) |

**Subtotal: ~$2**

**Experiments:**
1. Manual control with 2 buttons → motor spins left/right
2. Connect 555 oscillator → motor vibrates (AC!)
3. Add second LED pair → shows alternating current direction
4. Measure with multimeter → see voltage reversing

**Connection to Microgrid:**
```
This tiny circuit IS an inverter!

Real OzInverter:
- BC547/BC557 → IRFP4668 MOSFETs (bigger)
- 9V battery → 48V battery bank
- Small motor → 230V transformer
- Buttons → EG8010 controller (or ESP32)

Same topology. Same principle. Just scaled up.
```

---

## Project 3.4: H-Bridge with 555 (Automatic AC!)

**Goal:** Combine oscillator + H-bridge = working inverter!

**Schematic:**
```
    555 Astable                    H-Bridge
    (1-10 Hz)
         │
         │    ┌─────────────────────────┐
         │    │                         │
         └────┤ Q1+Q4 Control           │
              │         ┌───────┐       │
              │         │ Motor │       │
              │         │  or   │       │
         ┌────┤ Q2+Q3   │ LEDs  │       │
         │    │ Control └───────┘       │
         │    │                         │
    NOT Gate  └─────────────────────────┘
    (CD4069 or
     transistor)
```

**Bill of Materials:**
| Qty | Component | Value | Est. Price |
|-----|-----------|-------|------------|
| 2 | CD4069 | Hex inverter | $1 |
| 2 | IC Socket | 14-pin | $0.40 |

**Subtotal: ~$2**

**What you'll see:**
- Motor shakes back and forth (AC motion)
- LEDs alternate (shows current reversal)
- Adjust 555 pot → change "frequency"
- **You built an inverter!**

---

## Project 3.5: Power Stage (555 + TIP120)

**Goal:** Drive real loads—motor, high-power LEDs

**Schematic:**
```
        +12V
         │
      ┌──┴──┐
      │MOTOR│
      └──┬──┘
         │
      ┌──┴──┐
      │ ◄── │ Flyback diode
      └──┬──┘
         │
         C
        ╱
   ────●    TIP120
        ╲
         E
         │
        GND
         │
    [1kΩ]
         │
    555 OUT
```

**Bill of Materials:**
| Qty | Component | Value | Est. Price |
|-----|-----------|-------|------------|
| 3 | TIP120 | Darlington NPN | $1.50 |
| 3 | TIP125 | Darlington PNP | $1.50 |
| 3 | Heatsink | TO-220 clip | $1 |
| 5 | Diode | 1N5408 (3A) | $1 |

**Subtotal: ~$5**

---

## Phase 3 Complete BOM

| Category | Items | Est. Total |
|----------|-------|------------|
| 4017 ICs + sockets | 3+3 | $1.60 |
| 4069 ICs + sockets | 2+2 | $1.40 |
| Potentiometers | 3 pcs | $1.50 |
| TIP120/125 | 6 pcs | $3 |
| Motors | 2 pcs | $2 |
| Heatsinks | 3 pcs | $1 |
| Diodes | 5 pcs | $1 |
| Misc LEDs/resistors | - | $1 |

**Phase 3 Total: ~$15**

---

# PHASE 4: Measurement Tools

**Time:** 1-2 weekends
**Cost:** ~$9
**Key Learning:** Build tools you'll use throughout the curriculum

---

## Project 4.1: Continuity Tester

```
    +9V──[1kΩ]──┬──Probe A
                │
              Buzzer
                │
    GND─────────┴──Probe B
```

**Bill of Materials:**
| Qty | Component | Value | Est. Price |
|-----|-----------|-------|------------|
| 2 | Piezo buzzer | 5V active | $1 |
| 2 | Probe tips | - | $2 |
| 1 | Project box | Small | $2 |

**Subtotal: ~$5**

---

## Project 4.2: Voltage Indicator (LED Ladder)

| Qty | Component | Value | Est. Price |
|-----|-----------|-------|------------|
| 5 | Zener diode | 3.3V | $0.50 |
| 5 | Zener diode | 5.1V | $0.50 |

**Subtotal: ~$2** (use LEDs/resistors from kit)

---

## Project 4.3: Audio Probe (Signal Tracer)

```
    Input ──[100nF]──┬──[10kΩ]──+9V
                     │
                    Speaker
                     │
                    GND
```

| Qty | Component | Value | Est. Price |
|-----|-----------|-------|------------|
| 2 | Speaker | 8Ω 0.5W | $2 |

**Subtotal: ~$2**

---

## Phase 4 Complete BOM

| Category | Items | Est. Total |
|----------|-------|------------|
| Buzzers | 2 pcs | $1 |
| Speakers | 2 pcs | $2 |
| Probes | 2 pcs | $2 |
| Project box | 1 pc | $2 |
| Zener diodes | 10 pcs | $1 |

**Phase 4 Total: ~$9**

---

# STEP 1 COMPLETE SUMMARY

## Bill of Materials

| Phase | Focus | Cost |
|-------|-------|------|
| Phase 1 | Basics + Power Flow | $30 |
| Phase 2 | 555 Timer | $18 |
| Phase 3 | Control + H-Bridge | $15 |
| Phase 4 | Measurement | $9 |

**Components Total: ~$72**
**Tools (if needed): ~$80-150**

---

## Skills Progression

| After Project | You Understand |
|---------------|----------------|
| 1.1-1.3 | Basic electronics, transistor switching |
| 1.4 | Why standard inverters are one-way |
| 1.5-1.6 | Bi-directional control, droop principle |
| 2.1-2.3 | Timing, oscillation, relay driving |
| 3.3-3.4 | **H-bridge = inverter topology** |
| 3.5 | Power stage design |

---

## Connection to Swarm Microgrid

```
By completing Step 1, you understand:

✓ Current direction (LED polarity)
✓ One-way flow (diodes) → why standard inverters can't import
✓ Bi-directional control (transistors) → how to enable import/export
✓ Auto-sensing (voltage comparison) → droop control principle
✓ H-bridge topology → the actual inverter circuit
✓ Oscillator + H-bridge → working inverter!

You're ready for Step 2: Real inverters with MOSFETs and transformers.
```

---

## Shopping List (All Phases)

**Order from AliExpress/Amazon:**

```
SEMICONDUCTORS:
- 10× BC547 NPN transistor
- 5× BC557 PNP transistor
- 5× 2N2222 NPN transistor
- 3× TIP120 Darlington NPN
- 3× TIP125 Darlington PNP
- 5× 555 timer IC
- 3× CD4017 decade counter
- 2× CD4069 hex inverter
- 20× 1N4007 diode
- 5× 1N5408 diode (3A)
- 10× Zener diode (3.3V, 5.1V)
- 15× LED (red, green, yellow)

PASSIVE:
- Resistor kit (100Ω to 1MΩ)
- Capacitor kit (100nF to 220µF)
- 4× Potentiometer (10kΩ, 100kΩ)

MISC:
- 5× IC socket 8-pin
- 3× IC socket 14-pin
- 3× IC socket 16-pin
- 5× Tactile button
- 4× Relay (5V and 9V)
- 2× Small DC motor
- 2× Piezo buzzer
- 2× Small speaker
- 2× LDR
- 2× Thermistor 10kΩ
- 1× Breadboard 830pts
- 1× Jumper wire kit
- 1× 9V battery + clip
- 1× 4×AA holder + batteries
- 3× Heatsink TO-220
```

**Estimated total: $50-70** (varies by supplier)

---

*Next: Step 2 - CD4047 Square Wave Inverter (150W)*
