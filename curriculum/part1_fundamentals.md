# Complete Inverter Building Curriculum
## From Zero to OzInverter (6-15kW Pure Sine Wave)

---

# OVERVIEW

```
Step 1: Fundamentals (12 projects, 4 phases)    ~8-12 weekends
Step 2: CD4047 Square Wave Inverter (150W)      ~2-3 weekends
Step 3: SG3525 Modified Sine Wave (500W)        ~3-4 weekends
Step 4: EG8010 Pure Sine Wave (1kW)             ~1-2 months
Step 5: OzInverter (6-15kW)                     ~2-3 months

Total: 6-12 months, ~$500 budget
```

---

# MASTER TOOL LIST (Buy Once)

| Tool | Purpose | Est. Price |
|------|---------|------------|
| Soldering iron 30-60W | Assembly | $20-40 |
| Soldering station (temp control) | Better for SMD | $50-80 |
| Solder 0.8mm (60/40 or lead-free) | Joints | $8 |
| Flux pen | Better flow | $5 |
| Desoldering pump | Fix mistakes | $8 |
| Solder wick | Remove solder | $5 |
| Digital multimeter | Measurement | $20-40 |
| Oscilloscope (or DSO138 kit) | Waveforms | $25-150 |
| Breadboard 830 points | Prototyping | $5 |
| Jumper wire kit | Connections | $5 |
| Wire stripper/cutter | Prep wire | $10 |
| Helping hands | Hold work | $15 |
| Heat shrink tubing set | Insulation | $8 |
| Hot glue gun | Mechanical | $10 |
| Safety glasses | Protection | $5 |
| Fire extinguisher (Class C) | Safety | $25 |

**Total tools: ~$200-400**

---

# STEP 1: FUNDAMENTALS COURSE

## Phase 1: No ICs (Pure Basics)

---

### Project 1.1: LED + Resistor (Ohm's Law)

**Goal:** Understand V = I × R in practice

**Schematic:**
```
    +9V
     │
    [R] 330Ω
     │
    LED
     │
    GND
```

**Bill of Materials:**
| Qty | Component | Value | Package | Est. Price |
|-----|-----------|-------|---------|------------|
| 5 | LED | Red 5mm | Through-hole | $1 |
| 5 | LED | Green 5mm | Through-hole | $1 |
| 5 | LED | Yellow 5mm | Through-hole | $1 |
| 10 | Resistor | 100Ω 1/4W | Axial | $0.50 |
| 10 | Resistor | 220Ω 1/4W | Axial | $0.50 |
| 10 | Resistor | 330Ω 1/4W | Axial | $0.50 |
| 10 | Resistor | 1kΩ 1/4W | Axial | $0.50 |
| 10 | Resistor | 10kΩ 1/4W | Axial | $0.50 |
| 1 | 9V battery clip | - | - | $1 |
| 1 | 9V battery | - | - | $3 |
| 1 | Breadboard | 400 points | - | $3 |

**Subtotal: ~$12**

**Learning Outcomes:**
- Calculate current: I = V/R
- Calculate resistor: R = (Vsupply - Vled) / I
- Understand LED forward voltage (Vf)
- Observe: lower R = brighter, higher R = dimmer

**Exercises:**
1. Calculate R for 9V supply, red LED (Vf=2V), 15mA
2. Build circuit, measure actual current
3. Try different resistors, observe brightness
4. Remove resistor - watch LED burn (cheap lesson!)

---

### Project 1.2: Switch Circuits

**Goal:** Understand mechanical switching, series/parallel logic

**Schematics:**

**1.2a: Simple ON/OFF**
```
    +9V ── Switch ── R ── LED ── GND
```

**1.2b: AND Gate (Series)**
```
    +9V ── Sw.A ── Sw.B ── R ── LED ── GND
    (Both needed)
```

**1.2c: OR Gate (Parallel)**
```
         ┌─ Sw.A ─┐
    +9V ─┤        ├─ R ── LED ── GND
         └─ Sw.B ─┘
    (Either works)
```

**Bill of Materials:**
| Qty | Component | Value | Package | Est. Price |
|-----|-----------|-------|---------|------------|
| 5 | Tactile switch | 6×6mm | PCB mount | $1 |
| 2 | Toggle switch | SPST | Panel mount | $2 |
| 2 | Toggle switch | SPDT | Panel mount | $3 |
| 1 | Slide switch | DPDT | PCB mount | $1 |

**Subtotal: ~$7**

**Learning Outcomes:**
- Open circuit = no current
- Closed circuit = current flows
- Series = AND logic
- Parallel = OR logic

---

### Project 1.3: Transistor as Switch

**Goal:** Electronic switching (foundation for MOSFETs)

**Schematic:**
```
        +9V
         │
        [R1] 330Ω
         │
        LED
         │
         C
        ╱
   B ──●    NPN (BC547/2N2222)
        ╲
         E
         │
        GND

   B connects via 10kΩ to switch/button
```

**Bill of Materials:**
| Qty | Component | Value | Package | Est. Price |
|-----|-----------|-------|---------|------------|
| 10 | NPN Transistor | BC547 | TO-92 | $1 |
| 5 | NPN Transistor | 2N2222 | TO-92 | $1 |
| 5 | PNP Transistor | BC557 | TO-92 | $1 |
| 5 | Resistor | 10kΩ 1/4W | Axial | $0.30 |
| 5 | Resistor | 100kΩ 1/4W | Axial | $0.30 |
| 2 | LDR | 5mm | - | $1 |
| 2 | Thermistor | 10kΩ NTC | - | $1 |

**Subtotal: ~$6**

**Learning Outcomes:**
- Small base current controls large collector current
- Current gain (hFE) concept
- Saturation vs cutoff states
- Sensors can control transistors (LDR, thermistor)

**Experiments:**
1. Button-controlled LED
2. Light-controlled LED (LDR + transistor)
3. Touch-activated switch (skin resistance)
4. Temperature switch (thermistor)

---

## Phase 1 Combined BOM

| Category | Items | Est. Total |
|----------|-------|------------|
| LEDs | 15 pcs various | $3 |
| Resistors | 50+ pcs various | $3 |
| Switches | 10 pcs various | $7 |
| Transistors | 20 pcs | $3 |
| Sensors | LDR, thermistor | $2 |
| Battery/holder | 9V setup | $4 |
| Breadboard | 1 pc | $3 |

**Phase 1 Total: ~$25**

---

## Phase 2: 555 Timer Introduction

---

### Project 1.4: 555 Monostable (One-Shot Timer)

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
       │  │  2      6   │          │
       │  │  │      │   │         [C] 10µF
       │  │  │      └───┼──────────┤
       │  │  │          │          │
       │  │  │     5    │          │
       │  │  │    ─┴─   │          │
       │  │  │   100nF  │          │
       │  └──┼──────────┼──────────┤
       │     │     1    │          │
       └─────┴─────┴────┴──────────┴── GND
                 │
              Button
                 │
               +9V
```

**Time = 1.1 × R × C = 1.1 × 470kΩ × 10µF = 5.17 seconds**

**Bill of Materials:**
| Qty | Component | Value | Package | Est. Price |
|-----|-----------|-------|---------|------------|
| 5 | 555 Timer IC | NE555/LM555 | DIP-8 | $1 |
| 5 | IC Socket | 8-pin | DIP | $0.50 |
| 5 | Resistor | 100kΩ 1/4W | Axial | $0.30 |
| 5 | Resistor | 470kΩ 1/4W | Axial | $0.30 |
| 5 | Resistor | 1MΩ 1/4W | Axial | $0.30 |
| 10 | Capacitor | 100nF ceramic | - | $0.50 |
| 5 | Capacitor | 1µF electrolytic | 25V | $0.30 |
| 5 | Capacitor | 10µF electrolytic | 25V | $0.30 |
| 5 | Capacitor | 100µF electrolytic | 25V | $0.50 |

**Subtotal: ~$4**

**Learning Outcomes:**
- Trigger input starts timing
- RC time constant determines duration
- Threshold detection
- One-shot behavior

---

### Project 1.5: 555 Astable (Blinker)

**Goal:** Continuous oscillation at controlled frequency

**Schematic:**
```
                +9V
                 │
          ┌──────┼──────────┐
          │      8          │
       ┌──┤4    VCC        7├──┐
       │  │   RESET         │  │
       │  │                 │ [R2] 10kΩ
       │  │      555        │  │
       │  │                 │  ├──┐
       │  │       3         │  │ [R3] 1kΩ
       │  │      OUT────────┼──┼──┼──[1kΩ]──LED──GND
       │  │                 │  │  │
       │  │    2       6    │  │ [C1] 100µF
       │  │    │       │    │  │  │
       │  │    └───┬───┘    │  │  │
       │  │        │        │  │  │
       │  │        └────────┼──┴──┘
       │  │                 │
       │  │       5         │
       │  │      ─┴─        │
       │  │     100nF       │
       │  │        │        │
       └──┴────────┴────────┴── GND
```

**Frequency = 1.44 / ((R2 + 2×R3) × C1) ≈ 1.2 Hz**

**Bill of Materials:**
*(Uses same 555 components from 1.4, add:)*
| Qty | Component | Value | Package | Est. Price |
|-----|-----------|-------|---------|------------|
| 5 | Capacitor | 47µF electrolytic | 25V | $0.30 |
| 5 | Capacitor | 220µF electrolytic | 25V | $0.50 |
| 2 | Potentiometer | 10kΩ | Panel/trim | $1 |
| 2 | Potentiometer | 100kΩ | Panel/trim | $1 |

**Subtotal: ~$3**

---

### Project 1.6: 555 with Relay

**Goal:** Switch higher power loads, understand inductive protection

**Schematic:**
```
        +9V                    LOAD CIRCUIT
         │                     (separate)
         │     ┌─────────┐
    555 OUT───[1kΩ]──┐   │
                     │   │
                     B   │
                    ╱    │
               NPN ●     │
                    ╲    │
                     E   │
                     │   │
                    GND  │
                         │
        +9V              │
         │               │
      ┌──┴──┐            │
      │RELAY│            │
      │COIL │       ┌────┴────┐
      └──┬──┘       │   NO    │
         │          │    ●────┼─── Load+
      ┌──┴──┐       │   COM   │
      │  ◄──│ Diode │    ●────┼─── Load-
      └──┬──┘       └─────────┘
         │
         C (transistor)
```

**Bill of Materials:**
| Qty | Component | Value | Package | Est. Price |
|-----|-----------|-------|---------|------------|
| 2 | Relay | 5V coil, 10A contacts | PCB mount | $3 |
| 2 | Relay | 9V coil, 10A contacts | PCB mount | $3 |
| 5 | Diode | 1N4001 | DO-41 | $0.50 |
| 5 | Diode | 1N4007 | DO-41 | $0.50 |
| 5 | Transistor | BC337 | TO-92 | $1 |
| 2 | Transistor | TIP120 Darlington | TO-220 | $1 |

**Subtotal: ~$9**

**Learning Outcomes:**
- Relay = electromagnetic switch
- Flyback diode absorbs inductive spike
- Isolation between control and load
- **CRITICAL for inverters:** Transformer = inductive load!

---

## Phase 2 Combined BOM

| Category | Items | Est. Total |
|----------|-------|------------|
| 555 ICs + sockets | 5 each | $1.50 |
| Resistors | Various values | $2 |
| Capacitors | Various values | $3 |
| Potentiometers | 4 pcs | $2 |
| Relays | 4 pcs | $6 |
| Diodes | 10 pcs | $1 |
| Transistors | BC337, TIP120 | $2 |

**Phase 2 Total: ~$18**

---

## Phase 3: Control Concepts

---

### Project 1.7: 555 PWM (Brightness Control)

**Goal:** Pulse Width Modulation for analog-like control

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
   │  │      OUT────────┼──[220Ω]──LED──GND
   │  │                 │          │
   │  │    2       6    │          │
   │  │    │       │    │      ┌───┤
   │  │    └───────┴────┼──────┤POT│ 10kΩ
   │  │                 │      └───┤
   │  │       5         │          │
   │  │      ─┴─        │         ─┴─
   │  │     100nF       │        100nF
   │  │        │        │          │
   └──┴────────┴────────┴──────────┴── GND
```

**Bill of Materials:**
| Qty | Component | Value | Package | Est. Price |
|-----|-----------|-------|---------|------------|
| 2 | Potentiometer | 10kΩ linear | Panel | $1 |
| 1 | Potentiometer | 50kΩ linear | Panel | $0.50 |
| 5 | Resistor | 220Ω 1/4W | Axial | $0.30 |
| 2 | Capacitor | 10nF ceramic | - | $0.20 |
| 2 | Capacitor | 100nF ceramic | - | $0.20 |

**Subtotal: ~$2**

**Learning Outcomes:**
- PWM = varying ON/OFF ratio
- Average voltage controls brightness
- **CRITICAL:** SPWM in inverters uses same principle!

---

### Project 1.8: LED Chaser (4017 Counter)

**Goal:** Sequential switching, clock signals

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
    │    4────┤──VCC RST─┤15    Q4─┼──[330Ω]──LED4
    │         │          │         │    ...
    └─────────┘          └─────────┘
```

**Bill of Materials:**
| Qty | Component | Value | Package | Est. Price |
|-----|-----------|-------|---------|------------|
| 3 | CD4017 | Decade counter | DIP-16 | $1 |
| 3 | IC Socket | 16-pin | DIP | $0.60 |
| 10 | LED | 5mm various colors | - | $1 |
| 10 | Resistor | 330Ω 1/4W | Axial | $0.50 |

**Subtotal: ~$3**

**Learning Outcomes:**
- Clock signals trigger state changes
- Sequential outputs
- Counter ICs
- **Relevance:** H-bridge needs sequenced switching

---

### Project 1.9: Power Switching (555 + TIP120)

**Goal:** Drive motors/high-power loads from 555

**Schematic:**
```
        +12V
         │
      ┌──┴──┐
      │MOTOR│ (or high-power LED array)
      └──┬──┘
         │
      ┌──┴──┐
      │  ◄──│ Flyback diode
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
    [1kΩ]│
         │
    555 OUT
```

**Bill of Materials:**
| Qty | Component | Value | Package | Est. Price |
|-----|-----------|-------|---------|------------|
| 3 | TIP120 | Darlington NPN | TO-220 | $1.50 |
| 3 | TIP125 | Darlington PNP | TO-220 | $1.50 |
| 2 | DC Motor | 6-12V small | - | $2 |
| 3 | Heatsink | TO-220 | Clip-on | $1 |
| 5 | Diode | 1N5408 (3A) | DO-201 | $1 |

**Subtotal: ~$7**

**Learning Outcomes:**
- Power transistors need heatsinks
- Signal chain: Control → Driver → Power stage
- **DIRECT application:** Same chain in inverters!

---

## Phase 3 Combined BOM

| Category | Items | Est. Total |
|----------|-------|------------|
| 4017 ICs + sockets | 3 each | $1.60 |
| Potentiometers | 3 pcs | $1.50 |
| TIP120/125 | 6 pcs | $3 |
| Motors | 2 pcs | $2 |
| Heatsinks | 3 pcs | $1 |
| Diodes | 5 pcs | $1 |
| LEDs | 10 pcs | $1 |
| Resistors | Various | $1 |

**Phase 3 Total: ~$12**

---

## Phase 4: Measurement & Analysis

---

### Project 1.10: Continuity Tester

**Schematic:**
```
    +9V──[1kΩ]──┬──Probe A
                │
              Buzzer
                │
                └──Probe B──GND
```

**Bill of Materials:**
| Qty | Component | Value | Package | Est. Price |
|-----|-----------|-------|---------|------------|
| 2 | Piezo buzzer | 5V active | - | $1 |
| 2 | Probe tips | - | - | $2 |
| 1 | Project box | Small | - | $2 |

**Subtotal: ~$5**

---

### Project 1.11: Voltage Indicator (LED Ladder)

**Bill of Materials:**
| Qty | Component | Value | Package | Est. Price |
|-----|-----------|-------|---------|------------|
| 10 | LED | Various colors | 5mm | $1 |
| 10 | Resistor | 10kΩ 1/4W | Axial | $0.50 |
| 5 | Zener diode | 3.3V | - | $0.50 |
| 5 | Zener diode | 5.1V | - | $0.50 |

**Subtotal: ~$3**

---

### Project 1.12: Audio Oscillator

**Uses 555 from earlier + speaker**

**Bill of Materials:**
| Qty | Component | Value | Package | Est. Price |
|-----|-----------|-------|---------|------------|
| 2 | Speaker | 8Ω 0.5W | Small | $2 |
| 2 | Capacitor | 10nF ceramic | - | $0.20 |
| 2 | Capacitor | 100nF ceramic | - | $0.20 |

**Subtotal: ~$3**

---

## Phase 4 Combined BOM

| Category | Items | Est. Total |
|----------|-------|------------|
| Buzzers | 2 pcs | $1 |
| Speakers | 2 pcs | $2 |
| Probes | 2 pcs | $2 |
| Project box | 1 pc | $2 |
| Zener diodes | 10 pcs | $1 |
| Small caps | Various | $0.50 |

**Phase 4 Total: ~$9**

---

# STEP 1 COMPLETE BOM SUMMARY

| Phase | Focus | Cost |
|-------|-------|------|
| Phase 1 | Basics (no ICs) | $25 |
| Phase 2 | 555 Timer | $18 |
| Phase 3 | Control | $12 |
| Phase 4 | Measurement | $9 |
| **Tools** | (if needed) | $200-400 |

**Step 1 Components Total: ~$64**
**Time: 8-12 weekends**
