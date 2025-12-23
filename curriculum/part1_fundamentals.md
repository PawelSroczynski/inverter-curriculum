# Step 1: Fundamentals Course

## From Zero to Mini Swarm Microgrid

**Goal:** Build foundation skills with every project pointing toward the swarm microgrid. By the end, you'll have TWO mini inverters sharing power automatically—the full concept working at safe 9V before scaling to 48V/230V.

---

## Learning Path

```
PHASE 1: DC Fundamentals                    → Foundation for everything
PHASE 2: Power Flow & Direction             → Core microgrid concept
PHASE 3: Oscillation & AC                   → What IS an inverter?
PHASE 4: Build the Inverter                 → H-bridge in your hands
PHASE 5: Mini Swarm Microgrid              → THE GOAL working at 9V!

Each phase builds directly on the previous.
No wasted projects. Every circuit matters.
```

---

## Milestone Preview

| After Phase | You Can Do |
|-------------|------------|
| 1 | Understand any DC circuit schematic |
| 2 | Explain why power flows and how to control direction |
| 3 | Build oscillators, understand AC and frequency |
| 4 | Build a working H-bridge inverter |
| **5** | **Run two inverters sharing power automatically** |

---

# TOOLS (Buy Once)

## Essential Tools (~$80)

| Tool | Purpose | Price |
|------|---------|-------|
| Soldering iron 30-60W | Assembly | $25 |
| Solder 0.8mm 60/40 | Joints | $8 |
| Desoldering pump | Fix mistakes | $8 |
| Digital multimeter | Measurement | $25 |
| Breadboard 830pts | Prototyping | $5 |
| Jumper wire kit | Connections | $5 |
| Wire stripper | Prep wire | $8 |

## Recommended Additions (~$50)

| Tool | Purpose | Price |
|------|---------|-------|
| DSO138 oscilloscope kit | See waveforms | $25 |
| Helping hands | Hold work | $15 |
| Component tester (TC1) | Identify parts | $10 |

---

# PHASE 1: DC Fundamentals

**Time:** 2 weekends | **Cost:** ~$18 | **Projects:** 6

**Why this phase:** Every inverter circuit uses these concepts. Skip nothing.

---

## 1.1 LED + Resistor (Ohm's Law)

**The most important law in electronics: V = I × R**

```
    +9V
     │
    [R] 330Ω  ← Limits current
     │
    LED       ← Long leg = positive
     │
    GND
```

**Build it:**
1. Insert 330Ω resistor (orange-orange-brown)
2. Connect LED long leg to resistor
3. Connect LED short leg to GND
4. Connect resistor to +9V
5. LED lights!

**Measure:**
- Voltage across LED: ~2V
- Voltage across resistor: ~7V
- Current: 7V ÷ 330Ω = 21mA

**Experiments:**
| Resistor | Current | Brightness |
|----------|---------|------------|
| 220Ω | 32mA | Bright |
| 330Ω | 21mA | Normal |
| 1kΩ | 7mA | Dim |
| 10kΩ | 0.7mA | Barely visible |

**Key insight:** Resistor controls current. In inverters, we control HUGE currents (100A+) with the same principle.

**BOM:**
| Qty | Component | Price |
|-----|-----------|-------|
| 5 | LED Red 5mm | $0.50 |
| 5 | LED Green 5mm | $0.50 |
| 5 | LED Yellow 5mm | $0.50 |
| 1 | Resistor kit (100Ω-1MΩ) | $3 |
| 1 | 9V battery + clip | $3 |
| 1 | Breadboard 400pts | $3 |

---

## 1.2 Voltage Divider (CRITICAL for Sensing!)

**Every sensor circuit in your inverter uses this.**

```
    +9V
     │
    [R1] 10kΩ
     │
     ├──── Vout (measure here)
     │
    [R2] 10kΩ
     │
    GND

    Vout = Vin × R2/(R1+R2) = 9V × 10k/(10k+10k) = 4.5V
```

**Why this matters:**
```
Real inverter problem:
- Battery is 48V
- ESP32 ADC max is 3.3V
- How to measure 48V safely?

Solution: Voltage divider!
48V × 10k/(100k+10k) = 4.36V → add protection → 3.3V safe
```

**Build it:**
1. Two 10kΩ resistors in series
2. Measure voltage at middle point
3. Should read ~4.5V (half of 9V)

**Experiments:**
| R1 | R2 | Vout |
|----|----|----- |
| 10kΩ | 10kΩ | 4.5V |
| 10kΩ | 20kΩ | 6V |
| 20kΩ | 10kΩ | 3V |
| 10kΩ | 1kΩ | 0.82V |

**Key insight:** You can scale ANY voltage down to safe levels. This is how you'll measure 48V battery and 230V AC in your real inverter.

---

## 1.3 Capacitor Basics (Energy Storage)

**Capacitors are EVERYWHERE in inverters. Understand them now.**

```
    +9V ──[Switch]──┬── +
                    │
                   [C] 1000µF
                    │
    GND ────────────┴── -
```

**Experiment 1: Charging**
1. Connect capacitor (observe polarity! stripe = negative)
2. Close switch briefly
3. Open switch
4. Connect LED across capacitor
5. LED lights then fades (capacitor discharging)

**Experiment 2: Timing**
| Capacitor | LED fade time |
|-----------|---------------|
| 100µF | ~0.5 sec |
| 1000µF | ~5 sec |
| 2200µF | ~10 sec |

**Why this matters:**
```
In your inverter:
- DC bus capacitors (smooth the power)
- Filter capacitors (clean the output)
- Timing capacitors (control frequency)
- Decoupling capacitors (prevent noise)

Without capacitors = noisy, unstable inverter
```

**BOM:**
| Qty | Component | Price |
|-----|-----------|-------|
| 5 | Capacitor 100µF 25V | $0.50 |
| 5 | Capacitor 1000µF 25V | $1 |
| 2 | Capacitor 2200µF 25V | $1 |
| 10 | Capacitor 100nF ceramic | $0.50 |

---

## 1.4 Diode - One Way Flow

**This explains why standard inverters can't charge batteries.**

```
    +9V ────►├──── LED ──── GND
           Diode
         (1N4007)

    Current flows → LED lights


    +9V ────├◄──── LED ──── GND
           Diode
         reversed

    Current blocked → LED off
```

**Band on diode = cathode (negative) side**

**Experiment: Power flow direction**
```
    Battery A (9V)         Battery B (6V)
        +                      +
        │                      │
        └────►├────LED────┬────┘
             Diode        │
                         [R]
                          │
        -                 -
        └─────────────────┘

    9V > 6V → Current flows A to B → LED lights
    Swap batteries → Diode blocks → LED off
```

**Key insight:**
```
MOSFETs have built-in "body diodes"
Standard inverter: power can only flow OUT
This is why you need ACTIVE CONTROL for bi-directional!
```

**BOM:**
| Qty | Component | Price |
|-----|-----------|-------|
| 20 | Diode 1N4007 | $1 |
| 10 | Diode 1N4001 | $0.50 |

---

## 1.5 Switch Circuits

**Manual control of current flow**

**Simple ON/OFF:**
```
    +9V ──[Switch]── R ── LED ── GND
```

**AND logic (series):**
```
    +9V ── Sw.A ── Sw.B ── R ── LED ── GND

    Both switches must be ON for LED to light
```

**OR logic (parallel):**
```
         ┌── Sw.A ──┐
    +9V ─┤          ├── R ── LED ── GND
         └── Sw.B ──┘

    Either switch lights LED
```

**BOM:**
| Qty | Component | Price |
|-----|-----------|-------|
| 10 | Tactile button 6×6mm | $1 |
| 4 | Toggle switch SPST | $2 |

---

## 1.6 Transistor as Switch

**Electronic switching - foundation for MOSFETs**

```
        +9V
         │
        [R] 330Ω
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

   B connects via 10kΩ to button/signal
```

**How it works:**
- Button pressed → small current into Base (~1mA)
- Transistor turns ON → large current through LED (~20mA)
- Current gain: 20mA ÷ 1mA = 20× amplification

**Experiments:**
1. Button control → LED follows button
2. LDR control → LED responds to light
3. Touch control → your skin resistance triggers it
4. Thermistor → LED responds to temperature

**Key insight:**
```
This is EXACTLY how inverters work:
- Small signal (5V from controller)
- Controls BIG current (100A through MOSFETs)
- Same principle, bigger scale
```

**BOM:**
| Qty | Component | Price |
|-----|-----------|-------|
| 10 | BC547 NPN | $1 |
| 5 | BC557 PNP | $1 |
| 5 | 2N2222 NPN | $1 |
| 2 | LDR photoresistor | $0.50 |
| 2 | Thermistor 10kΩ NTC | $0.50 |

---

## Phase 1 Summary

| Project | Key Concept | Inverter Application |
|---------|-------------|---------------------|
| 1.1 LED | Ohm's Law | Current limiting, protection |
| 1.2 Voltage Divider | Scaling voltage | ALL sensor circuits |
| 1.3 Capacitor | Energy storage | DC bus, filtering |
| 1.4 Diode | One-way flow | Why bi-directional needs active control |
| 1.5 Switches | Manual control | Understanding logic |
| 1.6 Transistor | Electronic switch | Gate drivers, power stage |

**Phase 1 Cost: ~$18**

---

# PHASE 2: Power Flow & Direction

**Time:** 2 weekends | **Cost:** ~$15 | **Projects:** 5

**Why this phase:** This IS the core concept of your swarm microgrid—controlling power direction.

---

## 2.1 Two-Battery Power Flow

**Power flows from HIGH to LOW - like water**

```
    Battery A (9V)              Battery B (6V)
        (+)                         (+)
         │                           │
         └───────┬───────────────────┘
                 │
                LED  (with 330Ω)
                 │
         ┌───────┴───────────────────┐
         │                           │
        (-)                         (-)
        GND ─────────────────────── GND
```

**Observation:**
- Current flows from 9V to 6V naturally
- LED lights, showing power transfer
- A is "discharging", B is "charging" (in theory)

**Swap batteries:**
- Now 6V is on left, 9V on right
- Current reverses direction
- Same physics, opposite flow

**Key insight:**
```
This is EXACTLY what happens in your microgrid:

House 1: Battery full (higher voltage)  ─→ exports power
House 2: Battery low (lower voltage)    ←─ imports power

No controller needed for basic sharing!
Just voltage difference drives current.
```

**BOM:**
| Qty | Component | Price |
|-----|-----------|-------|
| 1 | 4×AA holder | $1 |
| 4 | AA batteries | $2 |
| 1 | Extra 9V battery | $2 |

---

## 2.2 Diode Enforces Direction

**Add diode to PREVENT reverse flow**

```
    9V ────►├────┬──── 6V
          Diode │
                LED
                │
    GND ────────┴──── GND

    Current flows 9V → 6V (diode allows)

    If you swap batteries:
    6V ────►├────┬──── 9V
          Diode │
                LED (OFF!)
                │
    GND ────────┴──── GND

    Diode BLOCKS reverse flow (9V can't push back)
```

**Key insight:**
```
Standard inverter = has body diodes in MOSFETs
Power can only flow ONE direction (battery → AC output)
Battery CANNOT be charged from AC bus

This is the LIMITATION you'll overcome with bi-directional control!
```

---

## 2.3 Transistor Bi-Directional Control

**Now YOU control which direction power flows**

```
        EXPORT Button         IMPORT Button
             │                     │
           [10k]                 [10k]
             │                     │
    9V ──────┼─────┐       ┌───────┼────── 6V
    (A)      │    B│       │B      │       (B)
             │   ╱ │       │ ╲     │
             │  ●Q1│       │Q2●    │
             │   ╲ │       │ ╱     │
             │    E│       │E      │
             │     └───┬───┘       │
             │         │           │
             │    LED + 330Ω       │
             │         │           │
    GND ─────┴─────────┴───────────┴────── GND
```

**Operation:**
| Button | Q1 | Q2 | Current Flow | LED |
|--------|----|----|--------------|-----|
| None | OFF | OFF | None | Off |
| EXPORT | ON | OFF | A → B | On (one direction) |
| IMPORT | OFF | ON | B → A | On (reversed!) |
| Both | ON | ON | SHORT CIRCUIT! | Danger! |

**This IS the bi-directional inverter principle!**

```
Real inverter:
- EXPORT button = ESP32 sets "grid-forming" mode
- IMPORT button = ESP32 sets "grid-following" mode
- Q1/Q2 = MOSFET H-bridge
- Same concept, higher power
```

---

## 2.4 Voltage Auto-Sensing (DROOP CONTROL!)

**Automatic direction based on voltage difference - NO BUTTONS!**

```
    Comparator using transistors:

    9V ──[10k]──┬──[10k]── 6V
                │
                │ Voltage at this point depends
                │ on which battery is stronger
                │
               B│
              ╱ │
             ●  │──── LED (shows which way power flows)
              ╲ │
               E│
                │
    GND ────────┴────────── GND
```

**How it works:**
- If 9V battery is stronger → middle voltage is higher → transistor conducts more → power flows A→B
- If 6V battery were replaced with 12V → middle voltage shifts → transistor behavior changes

**Better version with two transistors:**
```
    9V ──┬────────────────────────┬── 6V
         │                        │
        [R]                      [R]
         │    ┌──── LED A ────┐   │
         └────┤               ├───┘
              │    Q1    Q2   │
              └──── LED B ────┘

    Whichever battery is HIGHER lights its LED
    Automatic sensing! No buttons!
```

**Key insight:**
```
THIS IS DROOP CONTROL!

Real system:
- Battery full → voltage slightly higher → exports
- Battery empty → voltage slightly lower → imports
- No communication between inverters needed
- Physics handles the coordination

Your swarm microgrid will use this principle.
```

---

## 2.5 Current Sensing Introduction

**Know HOW MUCH power flows, not just direction**

```
    Simple current sense:

    +9V ────[0.1Ω]────┬──── Load
               │      │
               └──Measure voltage here

    V = I × R
    If I = 1A, R = 0.1Ω → V = 0.1V = 100mV
```

**Better: Use dedicated sensor**
```
    +9V ───┬────[ACS712]────┬──── Load
           │                │
           │     ┌──────────┘
           │     │ Signal out (66mV per Amp)
           │     ▼
           │  To voltmeter/ADC
           │
    GND ───┴─────────────────────── GND
```

**Why this matters:**
```
Energy metering needs:
- Voltage (from voltage divider)
- Current (from ACS712 or shunt)
- Power = V × I
- Energy = Power × Time (Wh)

This is how you'll track kWh sent/received in your microgrid!
```

**BOM:**
| Qty | Component | Price |
|-----|-----------|-------|
| 2 | ACS712 5A module | $3 |
| 5 | Resistor 0.1Ω 1W | $1 |

---

## Phase 2 Summary

| Project | Concept | Microgrid Application |
|---------|---------|----------------------|
| 2.1 Two-Battery | Power flows high→low | Basic sharing principle |
| 2.2 Diode Direction | One-way enforcement | Why standard inverters can't import |
| 2.3 Bi-Directional | Controlled direction | Import/Export mode switching |
| 2.4 Auto-Sensing | Voltage-based control | DROOP CONTROL |
| 2.5 Current Sensing | Measure flow amount | Energy metering |

**Phase 2 Cost: ~$15**

---

# PHASE 3: Oscillation & AC

**Time:** 2 weekends | **Cost:** ~$12 | **Projects:** 5

**Why this phase:** An inverter converts DC to AC. What IS AC? You need to deeply understand this.

---

## 3.1 What IS AC? (Manual Demo)

**AC = Alternating Current. Let's make it by hand.**

```
    Step 1: Connect battery one way

    +9V ────LED──── GND     LED lights


    Step 2: Reverse the connections

    GND ────LED──── +9V     LED lights (if bi-color)
                            or use 2 LEDs opposite polarity
```

**Better demo with motor:**
```
    +9V ────[Motor]──── GND     Motor spins clockwise

    (swap wires)

    GND ────[Motor]──── +9V     Motor spins counter-clockwise

    Swap back and forth rapidly = motor vibrates = AC!
```

**Key insight:**
```
AC is just DC that keeps reversing direction.
- 50 times per second = 50Hz (Europe)
- 60 times per second = 60Hz (USA)

Your inverter's job: automate this reversal perfectly.
```

---

## 3.2 RC Oscillator (Simplest Timing)

**Before 555, understand the basic principle**

```
         +9V
          │
         [R] 100kΩ
          │
          ├───────┬──── Output (to LED via transistor)
          │       │
         [C]    [R2]
        100µF   10kΩ
          │       │
         GND     GND
```

**Frequency ≈ 1/(R×C)**

With 100kΩ and 100µF: f ≈ 0.1 Hz (very slow, ~10 seconds per cycle)

**Experiment:** Change values, observe frequency change
| R | C | Approximate f |
|---|---|---------------|
| 100kΩ | 100µF | 0.1 Hz |
| 10kΩ | 100µF | 1 Hz |
| 10kΩ | 10µF | 10 Hz |
| 10kΩ | 1µF | 100 Hz |

---

## 3.3 555 Astable (Controlled Frequency)

**Now proper oscillator with precise control**

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
       │  │      OUT────────┼──┤ [R2] 10kΩ
       │  │                 │  │  │
       │  │    2       6    │  │ [C] 10µF
       │  │    └───────┴────┼──┴──┤
       │  │       5         │     │
       │  │      ─┴─ 100nF  │     │
       └──┴─────────────────┴─────┴── GND
```

**Frequency = 1.44 / ((R1 + 2×R2) × C)**

With values shown: f = 1.44 / ((10k + 20k) × 10µF) = 4.8 Hz

**Make it adjustable:** Replace R2 with 100kΩ potentiometer
- Full CCW: ~14 Hz
- Full CW: ~0.7 Hz

**BOM:**
| Qty | Component | Price |
|-----|-----------|-------|
| 5 | 555 Timer IC | $1 |
| 5 | IC Socket 8-pin | $0.50 |
| 4 | Potentiometer 100kΩ | $2 |

---

## 3.4 Frequency = Sound (Hear Your Oscillator!)

**Connect 555 to speaker - HEAR the frequency**

```
    555 OUT ──[100µF]──┬── Speaker +
                       │   (8Ω)
                       │
    GND ───────────────┴── Speaker -
```

**Experiment:**
| Component Change | Frequency | Sound |
|------------------|-----------|-------|
| C = 100µF | ~5 Hz | Clicks |
| C = 10µF | ~50 Hz | Low hum (THIS IS 50Hz AC!) |
| C = 1µF | ~500 Hz | Tone |
| C = 100nF | ~5000 Hz | High pitch |

**Key insight:**
```
When you hear 50Hz hum from a transformer = that's the AC frequency!
Your inverter will generate exactly 50.00Hz (or 60.00Hz).
Too fast or slow = appliances malfunction.
```

**BOM:**
| Qty | Component | Price |
|-----|-----------|-------|
| 2 | Speaker 8Ω 0.5W | $2 |

---

## 3.5 PWM - Pulse Width Modulation

**Control AVERAGE voltage by switching fast**

```
    555 in PWM mode:

            +9V
             │
      ┌──────┼──────────┐
      │      8          │
   ┌──┤4    VCC        7├──────────┐
   │  │                 │          │
   │  │      555        │         [R] 1kΩ
   │  │                 │          │
   │  │       3         │          │
   │  │      OUT────────┼──────── LED
   │  │                 │          │
   │  │    2       6    │      ┌───┴───┐
   │  │    │       │    │      │  POT  │ 10kΩ
   │  │    └───┴───┴────┼──────┤       │
   │  │       5         │      └───┬───┘
   │  │      ─┴─ 100nF  │          │
   └──┴─────────────────┴──────────┴── GND
```

**Turn pot:**
- One extreme: LED fully bright (100% duty)
- Middle: LED medium (50% duty)
- Other extreme: LED dim (near 0% duty)

**Oscilloscope view:**
```
100% duty:  ████████████████████
 50% duty:  ████    ████    ████
 25% duty:  ██      ██      ██
```

**Key insight:**
```
SPWM (Sinusoidal PWM) uses this principle:
- Vary the duty cycle smoothly
- Average follows a sine wave shape
- This is how EG8010 creates pure sine output!
```

---

## Phase 3 Summary

| Project | Concept | Inverter Application |
|---------|---------|---------------------|
| 3.1 What is AC | Reversing polarity | The inverter's core job |
| 3.2 RC Oscillator | Timing basics | Foundation of all control |
| 3.3 555 Astable | Controlled frequency | Generate exact 50/60Hz |
| 3.4 Frequency=Sound | Hear the frequency | Debug by ear! |
| 3.5 PWM | Average voltage control | SPWM for pure sine |

**Phase 3 Cost: ~$12**

---

# PHASE 4: Build The Inverter

**Time:** 2-3 weekends | **Cost:** ~$15 | **Projects:** 5

**Why this phase:** You'll build an actual H-bridge inverter with your hands.

---

## 4.1 Push-Pull (Two Transistors Alternating)

**Simplest "inverter" - two transistors taking turns**

```
              +9V
               │
        ┌──────┼──────┐
        │             │
       [R]           [R]
      330Ω          330Ω
        │             │
      LED A         LED B
        │             │
        C             C
       ╱             ╱
  B1──●    Q1   Q2  ●──B2
       ╲             ╲
        E             E
        │             │
        └──────┬──────┘
               │
              GND
```

**Control with 555:**
```
    555 OUT ──────────┬──[10k]── Q1 Base
                      │
                     [Inverter]
                      │
                      └──[10k]── Q2 Base
```

**Result:** LEDs alternate. When one is ON, other is OFF. This IS AC conceptually!

---

## 4.2 H-Bridge Manual Control

**Full H-bridge with 4 transistors**

```
                        +9V
                         │
              ┌──────────┼──────────┐
              │                     │
             Q1                    Q2
            (PNP)                 (PNP)
           BC557                 BC557
              │                     │
              ├──────[MOTOR]────────┤
              │                     │
             Q3                    Q4
            (NPN)                 (NPN)
           BC547                 BC547
              │                     │
              └──────────┼──────────┘
                         │
                        GND
```

**Control logic:**
| State | Q1 | Q2 | Q3 | Q4 | Motor |
|-------|----|----|----|----|-------|
| Forward | ON | off | off | ON | Spins CW |
| Reverse | off | ON | ON | off | Spins CCW |
| Brake | off | off | ON | ON | Stopped |
| Coast | off | off | off | off | Free spin |
| **FORBIDDEN** | ON | ON | - | - | SHORT CIRCUIT! |

**Build with buttons:**
```
    Button A: Activates Q1 + Q4 (forward)
    Button B: Activates Q2 + Q3 (reverse)

    Press A → motor spins one way
    Press B → motor spins other way

    NEVER press both! (shoot-through = dead transistors)
```

**BOM:**
| Qty | Component | Price |
|-----|-----------|-------|
| 4 | BC547 NPN | (have) |
| 4 | BC557 PNP | (have) |
| 2 | Small DC motor 3-6V | $2 |

---

## 4.3 H-Bridge + Oscillator = INVERTER!

**Combine 555 with H-bridge**

```
    555 Astable          NOT Gate         H-Bridge
    (2 Hz)              (CD4069)
         │                  │
         │    ┌─────────────┼─────────────┐
         │    │             │             │
         └────┤ Q1+Q4       │  Q2+Q3      │
              │ Control     │  Control    │
              │             │             │
              │      ┌──────┴──────┐      │
              │      │    Motor    │      │
              │      │    (Load)   │      │
              │      └─────────────┘      │
              │                           │
              └───────────────────────────┘
```

**What happens:**
1. 555 outputs HIGH → Q1+Q4 ON → current flows LEFT
2. 555 outputs LOW → Q2+Q3 ON → current flows RIGHT
3. Repeats at 2 Hz → motor vibrates back and forth

**THIS IS AN INVERTER!**
- DC input (9V battery)
- AC output (alternating through motor)
- Controlled frequency (555 timing)

**BOM:**
| Qty | Component | Price |
|-----|-----------|-------|
| 2 | CD4069 Hex Inverter | $1 |
| 2 | IC Socket 14-pin | $0.50 |

---

## 4.4 Dead-Time (CRITICAL Safety Concept!)

**Why you can't switch instantly**

```
    DANGER: Shoot-through

              +9V
               │
              Q1 ← If Q1 and Q3 are BOTH ON
               │    at the same time...
               │
              Q3 ← ...direct short circuit!
               │    BOOM! Dead transistors.
              GND
```

**Solution: Dead-time**
```
    Timing diagram:

    Q1:  ████████________████████________
    Q3:  ________████████________████████
                ↑       ↑
                Dead-time (both OFF)
                ~1-5 microseconds
```

**Build dead-time circuit:**
```
    555 OUT ──┬──[R]──[C]──► Q1 control (delayed ON)
              │
              └──────────► Q3 control (immediate)

    RC delay creates dead-time
```

**Experiment:**
1. Without dead-time: transistors get HOT (shoot-through)
2. With dead-time: transistors stay cool

**Key insight:**
```
EG8010 chip handles dead-time automatically (2µs default)
If you use ESP32, you must program dead-time yourself
This is why MOSFET drivers (IR2110) have dead-time pins
```

**BOM:**
| Qty | Component | Price |
|-----|-----------|-------|
| 5 | Capacitor 1nF | $0.30 |
| 5 | Resistor 1kΩ | (have) |

---

## 4.5 Add Transformer (Step Up!)

**Scale voltage with transformer**

```
    H-Bridge output          Transformer          Output
    (9V AC-ish)
         │                   ┌─────────┐
         ├───────────────────┤ 1:2     ├──────── ~18V AC
         │                   │         │
         │                   │  ))))   │
         │                   │  ((((   │
         │                   │         │
         └───────────────────┤         ├────────
                             └─────────┘
```

**Experiment with small transformer:**
- 9V in, measure output
- Try different transformers (1:1, 1:2, 2:1)
- Observe voltage scaling

**Key insight:**
```
Real OzInverter:
- 48V DC battery
- H-bridge creates 48V AC (square-ish wave)
- Transformer steps up to 230V AC
- LC filter smooths to sine wave

Same principle, bigger scale!
```

**BOM:**
| Qty | Component | Price |
|-----|-----------|-------|
| 1 | Small transformer 9V:9V | $3 |
| 1 | Small transformer 9V:18V | $3 |

---

## Phase 4 Summary

| Project | Concept | Real Inverter Application |
|---------|---------|--------------------------|
| 4.1 Push-Pull | Alternating outputs | Basic inverter topology |
| 4.2 H-Bridge | Full bridge control | OzInverter power stage |
| 4.3 + Oscillator | Automatic AC | Complete inverter |
| 4.4 Dead-Time | Shoot-through prevention | CRITICAL for MOSFETs |
| 4.5 Transformer | Voltage scaling | 48V → 230V |

**Phase 4 Cost: ~$15**

---

# PHASE 5: Mini Swarm Microgrid

**Time:** 2 weekends | **Cost:** ~$10 | **Projects:** 5

**Why this phase:** Build THE GOAL in miniature. Two inverters sharing power automatically. If this works at 9V, the same principle works at 48V.

---

## 5.1 Two Inverters, One AC Bus

**Connect two H-bridge inverter outputs together**

```
    Inverter A                    Inverter B
    (9V battery)                  (6V battery)
         │                             │
    ┌────┴────┐                  ┌────┴────┐
    │ H-Bridge│                  │ H-Bridge│
    │   555   │                  │   555   │
    └────┬────┘                  └────┬────┘
         │                             │
         └───────────┬─────────────────┘
                     │
                  AC BUS
                     │
                  [LOAD]
                  (motor/LED)
                     │
                   GND
```

**CRITICAL:** Both 555 oscillators must run at SAME frequency!
- Use same R and C values
- Or better: one master clock feeds both

**What you'll observe:**
- Both inverters contribute to powering the load
- If frequencies drift apart → buzzing, inefficiency

---

## 5.2 Power Sharing Observation

**See current split between inverters**

```
    Inverter A                         Inverter B
         │                                  │
        [LED A]                           [LED B]
     (shows A's                        (shows B's
      contribution)                     contribution)
         │                                  │
         └─────────────┬───────────────────┘
                       │
                    [LOAD]
                       │
                     GND
```

**Experiment:**
1. Equal batteries (9V + 9V): LEDs equally bright
2. Unequal (9V + 6V): 9V LED brighter (contributing more)
3. Disconnect B: A supplies 100%
4. Disconnect A: B supplies 100%

**This IS swarm behavior!**
- Each inverter contributes based on capacity
- System survives individual failures
- No central controller needed

---

## 5.3 Master/Slave Synchronization

**One inverter sets the beat, other follows**

```
    MASTER (Inverter A)              SLAVE (Inverter B)
    Has its own 555                  No 555! Uses master's signal
         │                                  │
    555 OUT ──────────────────────────────► H-Bridge B
         │                                  (via optocoupler
    H-Bridge A                               for isolation)
         │                                  │
         └─────────────┬───────────────────┘
                       │
                    AC BUS
```

**Why this matters:**
- Perfect synchronization
- No frequency drift
- Master can adjust frequency, slave follows

**BOM:**
| Qty | Component | Price |
|-----|-----------|-------|
| 2 | PC817 optocoupler | $1 |

---

## 5.4 Droop Control Demo

**Automatic load sharing based on voltage**

```
    Inverter A (9V)              Inverter B (9V)
    Both same voltage            but B has LOAD on its battery
         │                             │
         │                           [LOAD]
         │                          (drains B)
         │                             │
    ┌────┴────┐                  ┌────┴────┐
    │ H-Bridge│                  │ H-Bridge│
    └────┬────┘                  └────┬────┘
         │                             │
         └───────────┬─────────────────┘
                     │
                  AC BUS
```

**What happens:**
1. Initially: both batteries 9V, equal sharing
2. B's battery drains to 8V (due to extra load)
3. A's voltage now higher than B's
4. A automatically contributes MORE
5. Current flows A → B through AC bus

**Measure with multimeter:**
- Current from A increases
- Current from B decreases
- Total to load stays constant

**THIS IS DROOP CONTROL WORKING!**

---

## 5.5 Energy Counter (Mini Metering)

**Track energy transferred between inverters**

```
    Simple Amp-hour counter:

    Inverter A ──[ACS712]──┬── AC Bus
                           │
                     Signal to
                     Arduino/ESP
                     (count pulses
                      over time)
```

**Arduino sketch concept:**
```cpp
float current = readACS712();  // Amps
float power = 9.0 * current;   // Watts (V × I)
energy_wh += power * (1.0/3600.0);  // Accumulate Wh

if (current > 0) {
  Serial.println("EXPORTING");
  export_wh += power * (1.0/3600.0);
} else {
  Serial.println("IMPORTING");
  import_wh += abs(power) * (1.0/3600.0);
}
```

**Display:**
```
    ┌────────────────────────┐
    │ Inverter A Status      │
    │                        │
    │ Mode: EXPORTING        │
    │ Power: 2.3W            │
    │ Exported: 0.15 Wh      │
    │ Imported: 0.02 Wh      │
    │ Balance: +0.13 Wh      │
    └────────────────────────┘
```

**BOM:**
| Qty | Component | Price |
|-----|-----------|-------|
| 1 | Arduino Nano | $3 |
| 1 | OLED 0.96" I2C | $3 |
| 1 | ACS712 5A module | (have from 2.5) |

---

## Phase 5 Summary

| Project | Demonstrates | Real Microgrid Equivalent |
|---------|--------------|--------------------------|
| 5.1 Two Inverters | Parallel operation | Multiple houses connected |
| 5.2 Power Sharing | Load distribution | Automatic balancing |
| 5.3 Master/Slave | Synchronization | Frequency coordination |
| 5.4 Droop Demo | Voltage-based sharing | Droop control |
| 5.5 Energy Counter | Metering | kWh billing system |

**Phase 5 Cost: ~$10**

---

# COMPLETE STEP 1 SUMMARY

## Cost Breakdown

| Phase | Focus | Cost |
|-------|-------|------|
| Phase 1 | DC Fundamentals | $18 |
| Phase 2 | Power Flow & Direction | $15 |
| Phase 3 | Oscillation & AC | $12 |
| Phase 4 | Build the Inverter | $15 |
| Phase 5 | Mini Swarm Microgrid | $10 |
| **Total Components** | | **~$70** |
| **Tools** (if needed) | | **~$80-130** |

---

## Skills Achieved

```
After Step 1, you can:

✓ Read and understand any DC circuit schematic
✓ Explain Ohm's Law and use it for calculations
✓ Design voltage dividers for any sensing need
✓ Explain why power flows from high to low voltage
✓ Explain why standard inverters can't import power
✓ Build bi-directional power control circuits
✓ Explain droop control without complex math
✓ Build oscillators at any frequency
✓ Explain what AC is and how it's generated
✓ Build a working H-bridge inverter
✓ Explain dead-time and why it matters
✓ Connect two inverters to share load
✓ Demonstrate automatic power sharing
✓ Build a simple energy meter

YOU'VE BUILT A WORKING MINI SWARM MICROGRID!
```

---

## Connection to Full-Scale System

| Mini Version (Step 1) | Real Version (Steps 2-7) |
|-----------------------|--------------------------|
| 9V battery | 48V battery bank |
| BC547/BC557 transistors | IRFP4668 MOSFETs |
| 555 timer | EG8010 SPWM controller |
| Small motor | 5kVA transformer |
| Manual buttons | ESP32 automation |
| Two breadboard inverters | Four household inverters |
| LED current indicator | ACS712 + ESP32 metering |
| ~$70 total | ~$3,500 per household |

**Same principles. Same topology. Just scaled up.**

---

## Complete Shopping List

```
PHASE 1-5 COMPONENTS:

Semiconductors:
- 10× BC547 NPN transistor
- 10× BC557 PNP transistor
- 5× 2N2222 NPN transistor
- 5× 555 timer IC
- 2× CD4069 hex inverter IC
- 2× CD4017 decade counter (optional)
- 20× 1N4007 diode
- 5× 1N5408 3A diode
- 15× LED (red, green, yellow 5mm)
- 2× PC817 optocoupler
- 2× LDR
- 2× Thermistor 10kΩ

Resistors (or buy kit):
- 10× each: 100Ω, 330Ω, 1kΩ, 10kΩ, 100kΩ

Capacitors:
- 10× 100nF ceramic
- 5× 1µF electrolytic
- 5× 10µF electrolytic
- 5× 100µF electrolytic
- 5× 1000µF electrolytic

Potentiometers:
- 4× 100kΩ linear

Other:
- 5× IC socket 8-pin (for 555)
- 2× IC socket 14-pin (for CD4069)
- 10× Tactile button 6×6mm
- 2× Small DC motor 3-6V
- 2× Speaker 8Ω 0.5W
- 2× Small transformer 9V
- 2× ACS712 5A current sensor
- 1× Arduino Nano (or clone)
- 1× OLED 0.96" I2C
- 2× Breadboard 830 points
- 1× Jumper wire kit
- 2× 9V battery + clip
- 1× 4×AA battery holder + batteries
```

**Estimated total: $60-80** (depending on supplier)

---

## Recommended Order

**AliExpress (cheapest, 2-4 week shipping):**
- Component kits
- Arduino clones
- Sensors

**Amazon/Local (faster):**
- Batteries
- Basic tools
- Breadboards

---

## Time Investment

| Phase | Projects | Weekends |
|-------|----------|----------|
| Phase 1 | 6 | 2 |
| Phase 2 | 5 | 2 |
| Phase 3 | 5 | 2 |
| Phase 4 | 5 | 2-3 |
| Phase 5 | 5 | 2 |
| **Total** | **26** | **10-12** |

---

## Next Steps

After completing Step 1:

1. **Step 2:** CD4047 Square Wave Inverter (150W)
   - Real MOSFETs (IRF3205)
   - Real transformer
   - 12V → 230V

2. **Step 3:** SG3525 Modified Sine (500W)
   - H-bridge topology
   - IR2110 gate drivers
   - Dead-time in hardware

3. **Step 4:** EG8010 Pure Sine (1kW)
   - SPWM controller
   - LC filtering
   - <3% THD

4. **Step 5:** OzInverter (6-15kW)
   - Full power build
   - Toroidal transformer
   - Production quality

5. **Step 6:** Smart Upgrade
   - ESP32 monitoring
   - Home Assistant
   - Energy metering

6. **Step 7:** Full Swarm Microgrid
   - 4 households
   - Bi-directional tie inverters
   - Automatic power sharing

---

*You've built the foundation. The principles you learned in Step 1 scale directly to the full system. Onward!*
