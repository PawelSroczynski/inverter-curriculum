# Part 1: Fundamentals Course

## From Zero to Mini Swarm Microgrid

**Goal:** Build foundation skills with every project pointing toward the swarm microgrid. By the end, you'll have TWO mini inverters sharing power automatically with SmartBob (ESP32) monitoring and Home Assistant integration—the full concept working at safe 9V before scaling to 48V/230V.

---

## Learning Path

```
PHASE 1: DC Fundamentals                    → Foundation for everything
PHASE 2: Power Flow & SmartBob             → Core concept + ESP32 introduction
PHASE 3: Oscillation & AC                   → What IS an inverter?
PHASE 4: Build the Inverter                 → H-bridge in your hands
PHASE 5: Mini Swarm Microgrid              → Two inverters sharing power
PHASE 6: Advanced Synchronization          → Phase alignment & frequency droop

Each phase builds directly on the previous.
No wasted projects. Every circuit matters.
SmartBob introduced early for monitoring throughout.
```

---

## Milestone Preview

| After Phase | You Can Do |
|-------------|------------|
| 1 | Understand any DC circuit schematic |
| 2 | Read sensors with SmartBob, send data to Home Assistant |
| 3 | Build oscillators, understand AC and frequency |
| 4 | Build a working H-bridge inverter |
| 5 | Run two inverters sharing power with droop control |
| **6** | **Synchronize inverters with phase alignment for island grid** |

---

# TOOLS (Buy Once)

## Essential Tools (~200 PLN)

| Tool | Purpose | Price |
|------|---------|-------|
| Soldering station (YIHUA 995D+) | Assembly + hot air | 250 PLN |
| Digital multimeter | Measurement | 80 PLN |
| Breadboard 830pts (×2) | Prototyping | 30 PLN |
| Jumper wire kit | Connections | 25 PLN |
| Wire stripper | Prep wire | 25 PLN |

## Recommended Additions (~150 PLN)

| Tool | Purpose | Price |
|------|---------|-------|
| DSO138 oscilloscope kit | See waveforms | 80 PLN |
| Helping hands | Hold work | 40 PLN |
| Component tester (TC1) | Identify parts | 30 PLN |

---

# PHASE 1: DC Fundamentals

**Time:** 2 weekends | **Cost:** ~70 PLN | **Projects:** 6

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

**→ INVERTER CONNECTION:**
```
Resistor controls current. In inverters, we control HUGE currents (100A+)
with the same principle. Gate resistors on MOSFETs = same Ohm's Law.
```

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
- SmartBob (ESP32) ADC max is 3.3V
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

**→ INVERTER CONNECTION:**
```
You can scale ANY voltage down to safe levels.
This is how SmartBob will measure:
- 48V battery bank
- 230V AC output (with isolation!)
- Individual cell voltages
```

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

**→ INVERTER CONNECTION:**
```
In your inverter:
- DC bus capacitors (4700µF × 4) → smooth the power
- Filter capacitors → clean the output
- Timing capacitors → control frequency
- Decoupling capacitors → prevent noise

Without capacitors = noisy, unstable inverter
OzInverter uses ~20,000µF on DC bus!
```

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

**→ INVERTER CONNECTION:**
```
MOSFETs have built-in "body diodes"
Standard inverter: power can only flow OUT
This is why you need ACTIVE CONTROL for bi-directional!

Your swarm microgrid MUST overcome this limitation.
```

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

**→ INVERTER CONNECTION:**
```
H-bridge control signals use this logic:
- Q1 AND Q4 = forward current
- Q2 AND Q3 = reverse current
- Q1 AND Q2 = FORBIDDEN (shoot-through!)

Dead-time logic prevents forbidden states.
```

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

**→ INVERTER CONNECTION:**
```
This is EXACTLY how inverters work:
- Small signal (3.3V from SmartBob)
- Controls BIG current (100A through MOSFETs)
- Same principle, bigger scale

MOSFET = voltage-controlled transistor (even better!)
```

---

## Phase 1 Summary

| Project | Key Concept | Inverter Application |
|---------|-------------|---------------------|
| 1.1 LED | Ohm's Law | Current limiting, gate resistors |
| 1.2 Voltage Divider | Scaling voltage | ALL sensor circuits |
| 1.3 Capacitor | Energy storage | DC bus, filtering |
| 1.4 Diode | One-way flow | Why bi-directional needs active control |
| 1.5 Switches | Logic gates | H-bridge control signals |
| 1.6 Transistor | Electronic switch | Foundation for MOSFETs |

**Phase 1 Cost: ~70 PLN**

---

# PHASE 2: Power Flow & SmartBob Introduction

**Time:** 3 weekends | **Cost:** ~120 PLN | **Projects:** 9

**Why this phase:** This IS the core concept of your swarm microgrid. Plus: SmartBob (ESP32) enters the picture for monitoring and Home Assistant integration.

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

**→ SWARM CONNECTION:**
```
This is EXACTLY what happens in your microgrid:

House 1: Battery full (52V)  ─────→ EXPORTS power
House 2: Battery low (48V)   ←───── IMPORTS power

No controller needed for basic sharing!
Just voltage difference drives current.
Physics handles the coordination.
```

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

**→ INVERTER CONNECTION:**
```
Standard inverter = has body diodes in MOSFETs
Power can only flow ONE direction (battery → AC output)
Battery CANNOT be charged from AC bus

This is why:
- Grid-tie inverters need special design
- Bi-directional inverters need active rectification
- Your swarm needs CONTROLLABLE power flow
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

**→ SWARM CONNECTION:**
```
This IS the bi-directional inverter principle!

Real inverter:
- EXPORT button = SmartBob sets "grid-forming" mode
- IMPORT button = SmartBob sets "grid-following" mode
- Q1/Q2 = MOSFET H-bridge
- Same concept, higher power

Grid-Forming: "I set the voltage and frequency, others follow me"
Grid-Following: "I match whatever voltage/frequency I see on the bus"
```

---

## 2.4 Voltage Droop Control (CRITICAL!)

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
- If 9V battery is stronger → middle voltage higher → more current flows A→B
- If you replace 6V with 12V → current reverses automatically
- No controller needed!

**Better version with indicator LEDs:**
```
    9V ──┬────────────────────────┬── 6V
         │                        │
        [R]                      [R]
         │    ┌──── LED A ────┐   │
         └────┤   (EXPORT)    ├───┘
              │    Q1    Q2   │
              └──── LED B ────┘
                  (IMPORT)

    Whichever battery is HIGHER lights its LED
    Automatic sensing! No buttons!
```

**→ THIS IS VOLTAGE DROOP CONTROL:**
```
Real swarm microgrid:
┌─────────────────────────────────────────────────────────┐
│  DROOP CONTROL = Automatic power sharing via physics   │
│                                                         │
│  Battery 100% (52.0V) ───────→ EXPORTS maximum         │
│  Battery 75%  (51.0V) ───────→ EXPORTS some            │
│  Battery 50%  (50.0V) ───────→ NEUTRAL (balanced)      │
│  Battery 25%  (49.0V) ←─────── IMPORTS some            │
│  Battery 0%   (48.0V) ←─────── IMPORTS maximum         │
│                                                         │
│  No communication needed between houses!               │
│  Voltage difference DRIVES the current flow.           │
└─────────────────────────────────────────────────────────┘
```

---

## 2.5 Current Sensing Introduction

**Know HOW MUCH power flows, not just direction**

```
    Simple current sense (shunt resistor):

    +9V ────[0.1Ω]────┬──── Load
               │      │
               └──Measure voltage here

    V = I × R
    If I = 1A, R = 0.1Ω → V = 0.1V = 100mV
```

**Better: Use ACS712 Hall Effect Sensor**
```
    +9V ───┬────[ACS712]────┬──── Load
           │                │
           │     ┌──────────┘
           │     │ Signal out (66-185mV per Amp)
           │     ▼
           │  To SmartBob ADC
           │
    GND ───┴─────────────────────── GND
```

**ACS712 Variants:**
| Model | Range | Sensitivity |
|-------|-------|-------------|
| ACS712-05B | ±5A | 185 mV/A |
| ACS712-20A | ±20A | 100 mV/A |
| ACS712-30A | ±30A | 66 mV/A |

**→ SWARM CONNECTION:**
```
Energy metering needs:
- Voltage (from voltage divider) → V
- Current (from ACS712) → I
- Power = V × I → Watts
- Energy = Power × Time → Wh → kWh

This is how you'll track:
- kWh EXPORTED to neighbors
- kWh IMPORTED from neighbors
- Net balance for billing
```

---

## 2.6 SmartBob First Light (ESP32 Introduction)

**Meet your microgrid brain: SmartBob (ESP32)**

```
    ┌─────────────────────────────────────┐
    │         SMARTBOB (ESP32)            │
    │                                     │
    │  WiFi ──────→ Home Assistant        │
    │  GPIO ──────→ Control MOSFETs       │
    │  ADC ───────→ Read sensors          │
    │  I2C ───────→ OLED display          │
    │                                     │
    │  This is your inverter's brain!     │
    └─────────────────────────────────────┘
```

**First project: Read voltage divider**

```
    +9V
     │
    [R1] 100kΩ
     │
     ├──────────→ SmartBob GPIO34 (ADC)
     │
    [R2] 33kΩ
     │
    GND

    Vout = 9V × 33k/(100k+33k) = 2.24V (safe for 3.3V ADC)
```

**SmartBob Code:**
```cpp
// First SmartBob program - Read battery voltage

const int VOLTAGE_PIN = 34;
const float DIVIDER_RATIO = (100.0 + 33.0) / 33.0;  // 4.03

void setup() {
  Serial.begin(115200);
  analogReadResolution(12);  // 0-4095
  Serial.println("SmartBob Battery Monitor v1.0");
}

void loop() {
  int raw = analogRead(VOLTAGE_PIN);
  float voltage_adc = (raw / 4095.0) * 3.3;
  float battery_voltage = voltage_adc * DIVIDER_RATIO;

  Serial.print("Battery: ");
  Serial.print(battery_voltage, 2);
  Serial.println("V");

  // Determine state
  if (battery_voltage > 8.5) {
    Serial.println("Status: FULL - Ready to EXPORT");
  } else if (battery_voltage > 7.0) {
    Serial.println("Status: MEDIUM - Balanced");
  } else {
    Serial.println("Status: LOW - Need to IMPORT");
  }

  delay(1000);
}
```

**→ INVERTER CONNECTION:**
```
This exact code scales to real inverter:
- Change DIVIDER_RATIO for 48V battery
- Add calibration offset
- Same principle: ADC → Voltage → Decision
```

---

## 2.7 SmartBob Current Monitor (ACS712)

**Add current sensing to SmartBob**

```
    +9V ────[ACS712]────┬──── Load (LED + resistor)
                        │
           ┌────────────┘
           │ Vout (2.5V at 0A, changes with current)
           ▼
    SmartBob GPIO35 (ADC)
```

**SmartBob Code:**
```cpp
// SmartBob Current Monitor with ACS712

const int CURRENT_PIN = 35;
const float ACS712_SENSITIVITY = 0.185;  // 185mV/A for 5A version
const float ACS712_OFFSET = 2.5;         // 2.5V at 0A

void setup() {
  Serial.begin(115200);
  analogReadResolution(12);
}

void loop() {
  int raw = analogRead(CURRENT_PIN);
  float voltage = (raw / 4095.0) * 3.3;
  float current = (voltage - ACS712_OFFSET) / ACS712_SENSITIVITY;

  Serial.print("Current: ");
  Serial.print(current, 3);
  Serial.print("A  Direction: ");

  if (current > 0.05) {
    Serial.println("EXPORTING →");
  } else if (current < -0.05) {
    Serial.println("← IMPORTING");
  } else {
    Serial.println("BALANCED");
  }

  delay(500);
}
```

**→ SWARM CONNECTION:**
```
Current direction tells you:
- Positive = EXPORTING power to grid
- Negative = IMPORTING power from grid
- Zero = Balanced (not participating)

Combined with voltage:
Power = Voltage × Current
Energy = integrate Power over time
```

---

## 2.8 SmartBob Power Calculator

**Combine voltage + current = POWER monitoring**

```cpp
// SmartBob Power Monitor - The basis of energy metering

const int VOLTAGE_PIN = 34;
const int CURRENT_PIN = 35;
const float DIVIDER_RATIO = 4.03;
const float ACS712_SENSITIVITY = 0.185;
const float ACS712_OFFSET = 2.5;

float energy_exported_wh = 0;
float energy_imported_wh = 0;
unsigned long last_time = 0;

void setup() {
  Serial.begin(115200);
  analogReadResolution(12);
  last_time = millis();
  Serial.println("SmartBob Energy Monitor v1.0");
}

void loop() {
  // Read sensors
  float voltage = readVoltage();
  float current = readCurrent();
  float power = voltage * current;

  // Calculate energy (Wh)
  unsigned long now = millis();
  float hours = (now - last_time) / 3600000.0;
  last_time = now;

  if (power > 0) {
    energy_exported_wh += power * hours;
  } else {
    energy_imported_wh += abs(power) * hours;
  }

  // Display
  Serial.println("═══════════════════════════════");
  Serial.print("Voltage:  "); Serial.print(voltage, 2); Serial.println(" V");
  Serial.print("Current:  "); Serial.print(current, 3); Serial.println(" A");
  Serial.print("Power:    "); Serial.print(power, 2); Serial.println(" W");
  Serial.println("───────────────────────────────");
  Serial.print("Exported: "); Serial.print(energy_exported_wh, 4); Serial.println(" Wh");
  Serial.print("Imported: "); Serial.print(energy_imported_wh, 4); Serial.println(" Wh");
  Serial.print("Balance:  "); Serial.print(energy_exported_wh - energy_imported_wh, 4); Serial.println(" Wh");

  delay(1000);
}

float readVoltage() {
  int raw = analogRead(VOLTAGE_PIN);
  float v_adc = (raw / 4095.0) * 3.3;
  return v_adc * DIVIDER_RATIO;
}

float readCurrent() {
  int raw = analogRead(CURRENT_PIN);
  float v_adc = (raw / 4095.0) * 3.3;
  return (v_adc - ACS712_OFFSET) / ACS712_SENSITIVITY;
}
```

**→ SWARM CONNECTION:**
```
This IS your microgrid energy meter!

At end of month:
- House A exported 150 kWh, imported 50 kWh → Net +100 kWh
- House B exported 80 kWh, imported 120 kWh → Net -40 kWh
- House A credits House B for 40 kWh (fair billing!)
```

---

## 2.9 SmartBob + Home Assistant (First Integration)

**Connect SmartBob to your smart home system**

**Step 1: Install ESPHome on SmartBob**

```yaml
# smartbob_energy.yaml - ESPHome configuration

esphome:
  name: smartbob-inverter-1
  platform: ESP32
  board: esp32dev

wifi:
  ssid: "YourWiFi"
  password: "YourPassword"

api:
  encryption:
    key: "your-api-key-here"

ota:
  password: "ota-password"

sensor:
  - platform: adc
    pin: GPIO34
    name: "Battery Voltage"
    id: battery_voltage
    update_interval: 1s
    attenuation: 11db
    filters:
      - multiply: 4.03  # Voltage divider ratio
    unit_of_measurement: "V"

  - platform: adc
    pin: GPIO35
    name: "Grid Current"
    id: grid_current
    update_interval: 1s
    attenuation: 11db
    filters:
      - lambda: return (x - 2.5) / 0.185;  # ACS712 conversion
    unit_of_measurement: "A"

  - platform: template
    name: "Grid Power"
    id: grid_power
    lambda: |-
      return id(battery_voltage).state * id(grid_current).state;
    unit_of_measurement: "W"
    update_interval: 1s

  - platform: total_daily_energy
    name: "Daily Energy Exported"
    power_id: grid_power
    filters:
      - lambda: return x > 0 ? x : 0;
    unit_of_measurement: "Wh"

text_sensor:
  - platform: template
    name: "Inverter Mode"
    lambda: |-
      if (id(grid_current).state > 0.1) {
        return {"EXPORTING"};
      } else if (id(grid_current).state < -0.1) {
        return {"IMPORTING"};
      } else {
        return {"BALANCED"};
      }
    update_interval: 1s
```

**Step 2: Home Assistant Dashboard**

```yaml
# Home Assistant configuration.yaml addition

sensor:
  - platform: template
    sensors:
      microgrid_net_power:
        friendly_name: "Microgrid Net Power"
        unit_of_measurement: "W"
        value_template: >
          {{ states('sensor.smartbob_inverter_1_grid_power') | float +
             states('sensor.smartbob_inverter_2_grid_power') | float }}
```

**Home Assistant Dashboard Card:**
```yaml
type: entities
title: Swarm Microgrid Status
entities:
  - entity: sensor.smartbob_inverter_1_battery_voltage
    name: House 1 Battery
  - entity: sensor.smartbob_inverter_1_grid_power
    name: House 1 Power Flow
  - entity: text_sensor.smartbob_inverter_1_inverter_mode
    name: House 1 Mode
  - type: divider
  - entity: sensor.microgrid_net_power
    name: Grid Total
```

**→ SWARM CONNECTION:**
```
Home Assistant becomes your MICROGRID CONTROL CENTER:

┌─────────────────────────────────────────────────────────┐
│                 HOME ASSISTANT DASHBOARD                │
├─────────────────────────────────────────────────────────┤
│  HOUSE 1          HOUSE 2          HOUSE 3          HOUSE 4  │
│  ────────         ────────         ────────         ──────── │
│  52.1V            48.3V            50.5V            49.2V    │
│  EXPORT →         ← IMPORT         BALANCED         ← IMPORT │
│  +1.2kW           -0.8kW           +0.1kW           -0.5kW   │
│                                                              │
│  GRID STATUS: BALANCED (net flow: 0.0kW)                    │
│  Total shared today: 45.2 kWh                               │
└─────────────────────────────────────────────────────────────┘
```

---

## Phase 2 Summary

| Project | Concept | Microgrid Application |
|---------|---------|----------------------|
| 2.1 Two-Battery | Power flows high→low | Basic sharing principle |
| 2.2 Diode Direction | One-way enforcement | Why standard inverters can't import |
| 2.3 Bi-Directional | Controlled direction | Import/Export mode switching |
| 2.4 Voltage Droop | Automatic sharing | DROOP CONTROL |
| 2.5 Current Sensing | Measure flow | Energy metering foundation |
| 2.6 SmartBob Voltage | ESP32 ADC | Battery monitoring |
| 2.7 SmartBob Current | ACS712 + ESP32 | Current direction sensing |
| 2.8 SmartBob Power | V × I calculation | Energy accumulation |
| 2.9 SmartBob + HA | ESPHome integration | Microgrid dashboard |

**Phase 2 Cost: ~120 PLN** (includes SmartBob + ACS712 + OLED)

---

# PHASE 3: Oscillation & AC

**Time:** 2 weekends | **Cost:** ~50 PLN | **Projects:** 5

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

**→ INVERTER CONNECTION:**
```
AC is just DC that keeps reversing direction:
- 50 times per second = 50Hz (Europe/Poland)
- 60 times per second = 60Hz (USA)

Your inverter's job: automate this reversal PERFECTLY.
Frequency must be EXACT: 50.00Hz ± 0.5Hz
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

**Proper oscillator with precise control**

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

**Make it 50Hz (for inverter):**
- R1 = 1kΩ, R2 = 6.8kΩ, C = 1µF
- f = 1.44 / ((1k + 13.6k) × 1µF) = 98.6 Hz ÷ 2 = 49.3 Hz

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

**→ INVERTER CONNECTION:**
```
When you hear 50Hz hum from a transformer = that's the AC frequency!

Your inverter must generate exactly 50.00Hz:
- Too fast (51Hz) = clocks run fast, motors overheat
- Too slow (49Hz) = clocks run slow, motors struggle
- Way off = appliances malfunction or damage

EG8010 uses crystal oscillator for precision.
```

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

**→ INVERTER CONNECTION:**
```
SPWM (Sinusoidal PWM) uses this principle:

Instead of constant duty cycle:
████    ████    ████    ████

Vary duty cycle in sine pattern:
█ ██ ███████ ██ █   █ ██ ███████ ██ █

Average follows a sine wave!
This is how EG8010 creates pure sine output.
THD < 3% = clean power for sensitive electronics.
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

**Phase 3 Cost: ~50 PLN**

---

# PHASE 4: Build The Inverter

**Time:** 2-3 weekends | **Cost:** ~60 PLN | **Projects:** 5

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
                     [Inverter CD4069]
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

**→ SWARM CONNECTION:**
```
This H-bridge topology IS the OzInverter:
- Same 4-switch arrangement
- Same control logic
- Just bigger MOSFETs (IRFP4668 instead of BC547)
- Just higher voltage (48V instead of 9V)
```

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

**→ INVERTER CONNECTION:**
```
Dead-time is CRITICAL:
- EG8010 has built-in 2µs dead-time
- IR2110 gate driver has dead-time option
- SmartBob must program dead-time if direct control

Without dead-time = dead MOSFETs = dead inverter
```

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

**→ INVERTER CONNECTION:**
```
Real OzInverter:
- 48V DC battery
- H-bridge creates 48V AC (square-ish wave)
- Transformer steps up: 48V × 5 = 240V
- LC filter smooths to sine wave

Same principle, bigger scale!
Toroidal transformers are most efficient.
```

---

## Phase 4 Summary

| Project | Concept | Real Inverter Application |
|---------|---------|--------------------------|
| 4.1 Push-Pull | Alternating outputs | Basic inverter topology |
| 4.2 H-Bridge | Full bridge control | OzInverter power stage |
| 4.3 + Oscillator | Automatic AC | Complete inverter |
| 4.4 Dead-Time | Shoot-through prevention | CRITICAL for MOSFETs |
| 4.5 Transformer | Voltage scaling | 48V → 230V |

**Phase 4 Cost: ~60 PLN**

---

# PHASE 5: Mini Swarm Microgrid

**Time:** 2-3 weekends | **Cost:** ~80 PLN | **Projects:** 6

**Why this phase:** Build THE GOAL in miniature. Two inverters sharing power automatically with SmartBob monitoring.

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
    "Grid-Forming"                   "Grid-Following"
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

**Grid-Forming vs Grid-Following:**
```
┌─────────────────────────────────────────────────────────────────┐
│  GRID-FORMING (Master)                                          │
│  ─────────────────────                                          │
│  • Sets the voltage level (e.g., 230V RMS)                     │
│  • Sets the frequency (e.g., 50.00Hz)                          │
│  • Other inverters must match ME                               │
│  • Like a drummer setting the beat                             │
│                                                                 │
│  GRID-FOLLOWING (Slave)                                         │
│  ──────────────────────                                         │
│  • Matches whatever voltage it sees                            │
│  • Matches whatever frequency it sees                          │
│  • Just adds/removes power to the existing grid                │
│  • Like a musician following the drummer                       │
└─────────────────────────────────────────────────────────────────┘
```

**→ SWARM CONNECTION:**
```
In your 4-house microgrid:
- ONE house is grid-forming (the "anchor")
- THREE houses are grid-following
- If anchor fails → another takes over automatically

OR use full droop control (Phase 6) where ALL form together!
```

---

## 5.4 Voltage Droop Control Demo

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

**THIS IS VOLTAGE DROOP CONTROL WORKING!**

```
VOLTAGE DROOP CURVE:
                     │
    EXPORT ────────────────
    100%             │    ╲
                     │     ╲
                     │      ╲
      0% ────────────┼───────╲──────────
                     │        ╲
                     │         ╲
    IMPORT ────────────────    ╲
    100%             │
                     └──────────────────→
                   48V  50V  52V
                     BATTERY VOLTAGE
```

---

## 5.5 SmartBob Swarm Monitor

**Monitor BOTH inverters from one SmartBob**

```cpp
// SmartBob Swarm Monitor - Monitor two inverters

const int VOLTAGE_A_PIN = 34;
const int VOLTAGE_B_PIN = 35;
const int CURRENT_A_PIN = 32;
const int CURRENT_B_PIN = 33;

void setup() {
  Serial.begin(115200);
  Serial.println("╔═══════════════════════════════════════╗");
  Serial.println("║     SMARTBOB SWARM MONITOR v1.0      ║");
  Serial.println("╚═══════════════════════════════════════╝");
}

void loop() {
  float v_a = readVoltage(VOLTAGE_A_PIN);
  float v_b = readVoltage(VOLTAGE_B_PIN);
  float i_a = readCurrent(CURRENT_A_PIN);
  float i_b = readCurrent(CURRENT_B_PIN);

  float p_a = v_a * i_a;
  float p_b = v_b * i_b;
  float p_total = p_a + p_b;

  Serial.println("┌─────────────┬─────────────┬─────────────┐");
  Serial.println("│  INVERTER A │  INVERTER B │    GRID     │");
  Serial.println("├─────────────┼─────────────┼─────────────┤");

  Serial.print("│ V: ");
  Serial.print(v_a, 1);
  Serial.print("V    │ V: ");
  Serial.print(v_b, 1);
  Serial.print("V    │             │\n");

  Serial.print("│ I: ");
  Serial.print(i_a, 2);
  Serial.print("A   │ I: ");
  Serial.print(i_b, 2);
  Serial.print("A   │             │\n");

  Serial.print("│ P: ");
  Serial.print(p_a, 1);
  Serial.print("W    │ P: ");
  Serial.print(p_b, 1);
  Serial.print("W    │ Total: ");
  Serial.print(p_total, 1);
  Serial.println("W │");

  Serial.print("│ ");
  if (i_a > 0.05) Serial.print("EXPORT →  ");
  else if (i_a < -0.05) Serial.print("← IMPORT  ");
  else Serial.print("BALANCED  ");

  Serial.print("│ ");
  if (i_b > 0.05) Serial.print("EXPORT →  ");
  else if (i_b < -0.05) Serial.print("← IMPORT  ");
  else Serial.print("BALANCED  ");

  Serial.println("│             │");
  Serial.println("└─────────────┴─────────────┴─────────────┘");

  delay(1000);
}
```

---

## 5.6 SmartBob + Home Assistant Swarm Dashboard

**ESPHome configuration for full swarm:**

```yaml
# smartbob_swarm.yaml

esphome:
  name: smartbob-swarm-controller
  platform: ESP32
  board: esp32dev

wifi:
  ssid: "YourWiFi"
  password: "YourPassword"

api:
  encryption:
    key: "your-key"

sensor:
  # Inverter A
  - platform: adc
    pin: GPIO34
    name: "Inverter A Voltage"
    id: inv_a_voltage
    update_interval: 500ms
    filters:
      - multiply: 4.03

  - platform: adc
    pin: GPIO32
    name: "Inverter A Current"
    id: inv_a_current
    update_interval: 500ms
    filters:
      - lambda: return (x - 2.5) / 0.185;

  - platform: template
    name: "Inverter A Power"
    id: inv_a_power
    lambda: return id(inv_a_voltage).state * id(inv_a_current).state;
    unit_of_measurement: "W"
    update_interval: 500ms

  # Inverter B
  - platform: adc
    pin: GPIO35
    name: "Inverter B Voltage"
    id: inv_b_voltage
    update_interval: 500ms
    filters:
      - multiply: 4.03

  - platform: adc
    pin: GPIO33
    name: "Inverter B Current"
    id: inv_b_current
    update_interval: 500ms
    filters:
      - lambda: return (x - 2.5) / 0.185;

  - platform: template
    name: "Inverter B Power"
    id: inv_b_power
    lambda: return id(inv_b_voltage).state * id(inv_b_current).state;
    unit_of_measurement: "W"
    update_interval: 500ms

  # Grid totals
  - platform: template
    name: "Swarm Total Power"
    lambda: return id(inv_a_power).state + id(inv_b_power).state;
    unit_of_measurement: "W"

  - platform: template
    name: "Swarm Power Balance"
    lambda: return abs(id(inv_a_power).state) - abs(id(inv_b_power).state);
    unit_of_measurement: "W"

text_sensor:
  - platform: template
    name: "Swarm Status"
    lambda: |-
      float balance = id(inv_a_power).state + id(inv_b_power).state;
      if (abs(balance) < 0.5) return {"BALANCED"};
      else if (balance > 0) return {"NET EXPORT"};
      else return {"NET IMPORT"};
```

---

## Phase 5 Summary

| Project | Demonstrates | Real Microgrid Equivalent |
|---------|--------------|--------------------------|
| 5.1 Two Inverters | Parallel operation | Multiple houses connected |
| 5.2 Power Sharing | Load distribution | Automatic balancing |
| 5.3 Master/Slave | Grid-forming/following | Anchor + followers |
| 5.4 Voltage Droop | V-based sharing | Droop control basics |
| 5.5 SmartBob Monitor | Multi-inverter view | Control center |
| 5.6 HA Dashboard | Remote monitoring | Microgrid SCADA |

**Phase 5 Cost: ~80 PLN**

---

# PHASE 6: Advanced Synchronization (Island Grid)

**Time:** 2-3 weekends | **Cost:** ~40 PLN | **Projects:** 6

**Why this phase:** For true island grid operation, inverters must be synchronized in PHASE, not just frequency. This is what enables seamless parallel operation without circulating currents.

---

## 6.1 Why Phase Matters

**Frequency sync is NOT enough!**

```
    TWO INVERTERS AT SAME FREQUENCY BUT DIFFERENT PHASE:

    Inverter A: ~~~~∿~~~~∿~~~~∿~~~~  (0° phase)
    Inverter B: ~~∿~~~~∿~~~~∿~~~~∿   (90° out of phase)
                  ↑
                  These don't align!

    Result: CIRCULATING CURRENTS between inverters
    - Wasted energy
    - Overheating
    - Potential damage

    PROPER SYNCHRONIZATION:

    Inverter A: ~~~~∿~~~~∿~~~~∿~~~~  (0° phase)
    Inverter B: ~~~~∿~~~~∿~~~~∿~~~~  (0° phase - MATCHED!)
                  ↑
                  Perfect alignment!

    Result: Clean power sharing, no fighting
```

---

## 6.2 Phase Visualization (Two 555s)

**See phase difference with LEDs**

```
    555 A                           555 B
    (Master)                        (Independent)
      │                               │
      └──── LED A ────┬──── LED B ────┘
                      │
                   Compare!
```

**Experiment:**
1. Set both 555s to same R,C values (same frequency)
2. Watch LEDs - they drift in and out of sync
3. Sometimes both ON, sometimes alternating
4. This drift = phase difference

**With oscilloscope (DSO138):**
```
    Ch1: ████    ████    ████    ████   (555 A)
    Ch2:   ████    ████    ████    ██   (555 B - drifting)
           ↑
           Phase difference visible!
```

**→ SWARM CONNECTION:**
```
This is why parallel inverters need sync signal:
- Same frequency is not enough
- Must align the PHASE of switching
- Otherwise inverters fight each other
```

---

## 6.3 Zero-Cross Detection

**Detect when AC waveform crosses zero**

```
    AC Signal ──┬──[100kΩ]──┬──[100kΩ]──┬── GND
                │           │           │
                │          [10kΩ]       │
                │           │           │
                │           └───────────┼──► To SmartBob GPIO
                │                       │
                └───[Zener 3.3V]────────┘
                         (protection)
```

**SmartBob Zero-Cross Code:**
```cpp
// Zero-cross detection for phase synchronization

const int ZERO_CROSS_PIN = 27;
volatile unsigned long last_zero_cross = 0;
volatile unsigned long period_us = 0;
volatile float frequency = 0;

void IRAM_ATTR zeroCrossISR() {
  unsigned long now = micros();
  period_us = now - last_zero_cross;
  last_zero_cross = now;
  frequency = 1000000.0 / period_us;
}

void setup() {
  Serial.begin(115200);
  pinMode(ZERO_CROSS_PIN, INPUT);
  attachInterrupt(digitalPinToInterrupt(ZERO_CROSS_PIN), zeroCrossISR, RISING);
}

void loop() {
  Serial.print("Frequency: ");
  Serial.print(frequency, 2);
  Serial.print(" Hz  Period: ");
  Serial.print(period_us);
  Serial.println(" µs");
  delay(500);
}
```

**→ SWARM CONNECTION:**
```
Zero-cross detection is used for:
1. Measuring grid frequency precisely
2. Synchronizing to existing grid
3. Detecting phase angle between inverters
4. Timing TRIAC/SCR firing for soft-start
```

---

## 6.4 Frequency Droop Control (P-f Droop)

**Active power sharing via frequency adjustment**

```
FREQUENCY DROOP CONCEPT:
────────────────────────

Voltage Droop (Phase 5):  More power → Lower voltage
Frequency Droop (NEW):    More power → Lower frequency

Both work together in real inverters!

┌─────────────────────────────────────────────────────────────┐
│                    P-f DROOP CURVE                          │
│                                                             │
│  Frequency                                                  │
│     ▲                                                       │
│ 50.5Hz ─────╲                                              │
│             ╲                                              │
│ 50.0Hz ──────╳─────────────  (nominal)                     │
│              ╲                                             │
│ 49.5Hz ───────╲────────────                                │
│                ╲                                           │
│                 └──────────────────────────▶ Power (kW)    │
│               0     2kW    4kW    6kW                      │
│                                                             │
│  Higher load → Inverter droops frequency slightly          │
│  Other inverters see lower f → They increase output        │
│  System self-balances!                                     │
└─────────────────────────────────────────────────────────────┘
```

**Mini Demo Circuit:**
```
    Load Sensor                     555 Frequency Control
    (current)
         │
    ┌────┴────┐
    │ ACS712  │──► Voltage ──► Controls 555 timing capacitor
    └─────────┘                     (via transistor/FET)

    More current = More load = Lower 555 frequency
```

**→ SWARM CONNECTION:**
```
In your 4-house microgrid with P-f droop:

House 1 heavily loaded → frequency drops to 49.8Hz
Houses 2,3,4 see 49.8Hz → "Grid needs help!"
Houses 2,3,4 increase their output
Frequency rises back toward 50.0Hz
System stabilizes automatically

NO COMMUNICATION NEEDED between houses!
Physics coordinates everything.
```

---

## 6.5 Soft Parallel Connection

**Safely connect inverter to running bus**

```
    WRONG WAY (dangerous):
    ──────────────────────
    Bus: ~~~~∿~~~~∿~~~~  (running at 50Hz, 0°)

    New inverter: ~~∿~~~~∿~~~~  (50Hz but random phase)

    *CONNECT*

    Result: Huge surge current! Possible damage.


    RIGHT WAY (soft sync):
    ─────────────────────
    1. New inverter starts, measures bus frequency
    2. Adjusts own frequency to match
    3. Measures phase difference
    4. Slowly adjusts until phase matches
    5. When phase < 5° difference → safe to connect
    6. Connect via soft-start (current limiting)
```

**SmartBob Sync Algorithm:**
```cpp
// Soft synchronization before parallel connection

const int BUS_ZERO_CROSS = 27;
const int INV_ZERO_CROSS = 26;
const int RELAY_PIN = 25;

volatile unsigned long bus_zc_time = 0;
volatile unsigned long inv_zc_time = 0;

void IRAM_ATTR busZC() { bus_zc_time = micros(); }
void IRAM_ATTR invZC() { inv_zc_time = micros(); }

float calculatePhaseAngle() {
  // Phase difference in degrees
  long diff = inv_zc_time - bus_zc_time;
  float period = 20000;  // 50Hz = 20ms = 20000µs
  float phase = (diff / period) * 360.0;

  // Normalize to -180 to +180
  while (phase > 180) phase -= 360;
  while (phase < -180) phase += 360;

  return phase;
}

void syncAndConnect() {
  Serial.println("Starting synchronization...");

  while (true) {
    float phase = calculatePhaseAngle();
    Serial.print("Phase difference: ");
    Serial.print(phase, 1);
    Serial.println("°");

    if (abs(phase) < 5.0) {
      Serial.println("SYNCHRONIZED! Connecting...");
      digitalWrite(RELAY_PIN, HIGH);  // Connect to bus
      break;
    }

    // Adjust inverter frequency slightly
    if (phase > 0) {
      // We're ahead, slow down slightly
      adjustFrequency(-0.1);
    } else {
      // We're behind, speed up slightly
      adjustFrequency(+0.1);
    }

    delay(100);
  }
}
```

---

## 6.6 Full Island Grid Demo

**Complete mini swarm with all features**

```
    ┌─────────────────────────────────────────────────────────────┐
    │                    MINI ISLAND GRID                         │
    │                                                             │
    │   Inverter A              Inverter B                       │
    │   (GRID-FORMING)          (GRID-FOLLOWING)                 │
    │   ┌─────────┐             ┌─────────┐                      │
    │   │ 555     │             │ Synced  │                      │
    │   │ Master  │────SYNC────►│ Slave   │                      │
    │   │ Clock   │             │         │                      │
    │   └────┬────┘             └────┬────┘                      │
    │        │                       │                           │
    │   ┌────┴────┐             ┌────┴────┐                      │
    │   │H-Bridge │             │H-Bridge │                      │
    │   │ + Droop │             │ + Droop │                      │
    │   └────┬────┘             └────┬────┘                      │
    │        │                       │                           │
    │        └───────────┬───────────┘                           │
    │                    │                                       │
    │              ══════╪══════  AC BUS  ═══════                │
    │                    │                                       │
    │              ┌─────┴─────┐                                 │
    │              │   LOAD    │                                 │
    │              │  (Motor)  │                                 │
    │              └───────────┘                                 │
    │                                                             │
    │   SmartBob monitors everything:                            │
    │   • Frequency: 50.0Hz                                      │
    │   • Phase A: 0°                                            │
    │   • Phase B: 2° (synced!)                                  │
    │   • Power A: 1.2W (EXPORT)                                 │
    │   • Power B: 0.8W (EXPORT)                                 │
    │   • Grid Status: ISLAND MODE - STABLE                      │
    │                                                             │
    └─────────────────────────────────────────────────────────────┘
```

**→ REAL SWARM OPERATION:**
```
This mini demo proves the concept for your 4-house system:

1. One house is "grid-forming" (anchor)
   - Sets 50Hz, 230V reference
   - All others sync to it

2. Other houses are "grid-following"
   - Match frequency and phase
   - Add/remove power based on droop

3. If anchor fails:
   - Detect loss of grid-forming inverter
   - Another house takes over as anchor
   - Seamless transition (< 20ms)

4. Home Assistant shows:
   - All 4 houses' status
   - Power flow between houses
   - Net import/export
   - Alerts for issues

Same principles at 9V or 48V!
```

---

## Phase 6 Summary

| Project | Concept | Island Grid Application |
|---------|---------|------------------------|
| 6.1 Why Phase | Understanding the problem | Avoid circulating currents |
| 6.2 Phase Visual | See phase difference | Debugging tool |
| 6.3 Zero-Cross | Detect AC timing | Sync measurement |
| 6.4 P-f Droop | Frequency-based sharing | Active power balance |
| 6.5 Soft Sync | Safe connection | Hot-swap inverters |
| 6.6 Full Demo | Everything together | Prove the concept! |

**Phase 6 Cost: ~40 PLN**

---

# COMPLETE PART 1 SUMMARY

## Cost Breakdown

| Phase | Focus | Cost |
|-------|-------|------|
| Phase 1 | DC Fundamentals | ~70 PLN |
| Phase 2 | Power Flow & SmartBob | ~120 PLN |
| Phase 3 | Oscillation & AC | ~50 PLN |
| Phase 4 | Build the Inverter | ~60 PLN |
| Phase 5 | Mini Swarm Microgrid | ~80 PLN |
| Phase 6 | Advanced Synchronization | ~40 PLN |
| **Total Components** | | **~420 PLN** |
| **Tools** (soldering station, etc.) | | **~400 PLN** |

---

## Skills Achieved

```
After Part 1, you can:

DC FUNDAMENTALS:
✓ Read and understand any DC circuit schematic
✓ Apply Ohm's Law for any calculation
✓ Design voltage dividers for sensor circuits
✓ Understand capacitor charging/timing

POWER ELECTRONICS:
✓ Explain why power flows from high to low voltage
✓ Explain why standard inverters can't import power
✓ Build bi-directional power control circuits
✓ Build a working H-bridge inverter
✓ Explain dead-time and implement it

SWARM/MICROGRID:
✓ Explain voltage droop control
✓ Explain frequency droop control
✓ Understand grid-forming vs grid-following
✓ Implement phase synchronization
✓ Connect inverters safely in parallel

SMARTBOB/ESP32:
✓ Read analog sensors (voltage, current)
✓ Calculate power and energy
✓ Send data to Home Assistant via ESPHome
✓ Monitor multiple inverters simultaneously
✓ Implement zero-cross detection

YOU'VE BUILT A WORKING MINI SWARM MICROGRID
WITH SMARTBOB MONITORING AND HOME ASSISTANT!
```

---

## Connection to Full-Scale System

| Mini Version (Part 1) | Real Version (Parts 2-7) |
|-----------------------|--------------------------|
| 9V battery | 48V/280Ah LiFePO4 bank |
| BC547/BC557 transistors | IRFP4668 MOSFETs |
| 555 timer | EG8010 SPWM controller |
| Small motor as load | 5kVA transformer + household |
| LED current indicators | ACS712 + SmartBob metering |
| Two breadboard inverters | Four OzInverter units |
| Home Assistant dashboard | Full SCADA system |
| ~420 PLN total | ~$3,500 per household |

**Same principles. Same topology. Just scaled up.**

---

## Time Investment

| Phase | Projects | Weekends |
|-------|----------|----------|
| Phase 1 | 6 | 2 |
| Phase 2 | 9 | 3 |
| Phase 3 | 5 | 2 |
| Phase 4 | 5 | 2-3 |
| Phase 5 | 6 | 2-3 |
| Phase 6 | 6 | 2-3 |
| **Total** | **37** | **13-16** |

---

## Next Steps

After completing Part 1:

1. **Part 2:** CD4047 Square Wave Inverter (150W)
   - Real MOSFETs (IRF3205)
   - Real transformer
   - 12V → 230V

2. **Part 3:** SG3525 Modified Sine (500W)
   - H-bridge topology
   - IR2110 gate drivers
   - Dead-time in hardware

3. **Part 4:** EG8010 Pure Sine (1kW)
   - SPWM controller
   - LC filtering
   - <3% THD

4. **Part 5:** OzInverter (6-15kW)
   - Full power build
   - Toroidal transformer
   - Production quality

5. **Part 6:** Smart Integration
   - SmartBob full monitoring
   - Home Assistant automation
   - Energy metering & billing

6. **Part 7:** Full Swarm Microgrid
   - 4 households connected
   - Bi-directional power flow
   - Automatic coordination
   - Fair energy sharing

---

*You've built the foundation. SmartBob is now part of your toolkit. The principles you learned scale directly to the full 4-house swarm microgrid. Onward!*
