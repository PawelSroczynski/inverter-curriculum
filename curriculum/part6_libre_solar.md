# Inverter Curriculum - Libre Solar Integration
## Step 6.5: Open Source MPPT & BMS

---

## Overview

This module integrates **Libre Solar** open-source hardware into your power system:
- **MPPT 2420 HC** - Solar charge controller
- **BMS C1** - Battery management system

Both are fully open source - schematics, PCB layouts, and firmware available on GitHub!

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                                                                             │
│                    COMPLETE OPEN SOURCE POWER SYSTEM                        │
│                                                                             │
│   ┌─────────────┐     ┌─────────────┐     ┌─────────────┐                  │
│   │   SOLAR     │     │   BATTERY   │     │  INVERTER   │                  │
│   │             │     │             │     │             │                  │
│   │ MPPT 2420   │────▶│   BMS C1    │────▶│ OzInverter  │                  │
│   │    HC       │     │             │     │             │                  │
│   │             │     │             │     │             │                  │
│   │ Libre Solar │     │ Libre Solar │     │ Your Build  │                  │
│   │ Open Source │     │ Open Source │     │ Open Source │                  │
│   └─────────────┘     └─────────────┘     └─────────────┘                  │
│                                                                             │
│                    100% OPEN SOURCE POWER SYSTEM!                          │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Why Libre Solar?

| Aspect | Commercial | Libre Solar |
|--------|------------|-------------|
| **Cost** | $150-300 per device | $40-80 DIY build |
| **Schematics** | Proprietary | ✅ Open (GitHub) |
| **Firmware** | Locked | ✅ Open (modify freely) |
| **Repairability** | Limited | ✅ Full access |
| **Customization** | None | ✅ Modify anything |
| **Learning** | Black box | ✅ Understand everything |
| **Community** | Vendor support only | ✅ Active open source |

---

# PART 1: MPPT 2420 HC

## What is MPPT?

```
WITHOUT MPPT:                        WITH MPPT:
Solar panel output varies            Controller finds optimal point

Power                                Power
  ▲                                    ▲
  │      ●                             │      ●  ← Maximum Power Point
  │    ●   ●                           │    ●   ●
  │  ●       ●                         │  ●       ●
  │●           ●                       │●           ●
  └──────────────▶ Voltage             └──────────────▶ Voltage

  You might operate here: ●            MPPT finds this: ●
  (wasting 20-30% power)               (maximum power extraction)
```

**MPPT** = Maximum Power Point Tracking
- Continuously adjusts to extract maximum power from solar panels
- 20-30% more energy vs simple PWM controllers
- Essential for efficient solar systems

---

## MPPT 2420 HC Specifications

| Parameter | Value |
|-----------|-------|
| Battery voltage | 12V or 24V (configurable) |
| Max charge current | 20A |
| Max PV voltage | 80V |
| Max PV power | ~520W (24V) / ~260W (12V) |
| Processor | STM32G431 (ARM Cortex-M4) |
| Communication | CAN bus (RJ45 jacks) |
| Efficiency | >95% |
| Protections | OVP, UVP, OCP, short circuit, reverse polarity |

### Block Diagram

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         MPPT 2420 HC BLOCK DIAGRAM                          │
│                                                                             │
│   SOLAR INPUT                                                               │
│   (up to 80V)                                                              │
│       │                                                                     │
│       ▼                                                                     │
│   ┌───────────────┐                                                        │
│   │  INPUT FILTER │                                                        │
│   │  + Protection │                                                        │
│   └───────┬───────┘                                                        │
│           │                                                                 │
│           ▼                                                                 │
│   ┌───────────────┐     ┌───────────────┐     ┌───────────────┐           │
│   │   BUCK        │     │   STM32G431   │     │    CAN        │           │
│   │   CONVERTER   │◀───▶│   MCU         │◀───▶│  INTERFACE    │           │
│   │   (DC-DC)     │     │               │     │   (RJ45)      │           │
│   └───────┬───────┘     │  • MPPT algo  │     └───────────────┘           │
│           │             │  • Charging   │                                   │
│           │             │  • Protection │     ┌───────────────┐           │
│           │             │  • Comms      │◀───▶│    UEXT       │           │
│           │             └───────────────┘     │  (I2C/SPI)    │           │
│           │                    │              └───────────────┘           │
│           │                    │                                           │
│           │              ┌─────┴─────┐                                     │
│           │              │  VOLTAGE  │                                     │
│           │              │  CURRENT  │                                     │
│           │              │  SENSING  │                                     │
│           │              └───────────┘                                     │
│           │                                                                 │
│           ▼                                                                 │
│   ┌───────────────┐                                                        │
│   │OUTPUT FILTER  │                                                        │
│   │  + Protection │                                                        │
│   └───────┬───────┘                                                        │
│           │                                                                 │
│           ▼                                                                 │
│      BATTERY OUTPUT                                                         │
│      (12V or 24V)                                                          │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Building MPPT 2420 HC

### Option A: Order Assembled (Easiest)

Check Libre Solar shop or community group buys.
**Estimated cost:** €80-120

### Option B: Build Yourself (Recommended for Learning)

#### Step 1: Get the Design Files

```bash
# Clone the repository
git clone https://github.com/LibreSolar/mppt-2420-hc.git
cd mppt-2420-hc

# Design files location:
# /hardware - KiCad schematics and PCB
# /firmware - Source code (separate repo)
```

#### Step 2: Order PCBs

Upload Gerber files to:
- **JLCPCB** (cheapest): ~$10 for 5 boards
- **PCBWay**: ~$15 for 5 boards
- **Aisler** (EU): ~$20 for 3 boards

**PCB Specs:**
- Layers: 4
- Thickness: 1.6mm
- Copper: 2oz outer (for current handling)
- Surface finish: HASL or ENIG

#### Step 3: Order Components

**Bill of Materials:**

| Qty | Component | Value/Part | Package | Est. Price |
|-----|-----------|------------|---------|------------|
| 1 | MCU | STM32G431CBT6 | LQFP-48 | $5 |
| 1 | DC-DC Controller | (integrated in MCU) | - | - |
| 2 | Power MOSFET | CSD19534Q5A | SON 5x6 | $3 ea |
| 1 | Inductor | 22µH 20A | Shielded | $5 |
| 1 | Gate driver | LM5109B | SOIC-8 | $2 |
| 2 | Current sensor | INA181 | SOT-23-6 | $1 ea |
| 1 | CAN transceiver | SN65HVD230 | SOIC-8 | $2 |
| 4 | Capacitor | 100µF 80V | Electrolytic | $1 ea |
| 10 | Capacitor | 10µF 50V | 0805 | $0.50 |
| 20 | Capacitor | 100nF | 0603 | $0.50 |
| 1 | Crystal | 8MHz | HC49 | $0.50 |
| 2 | RJ45 jack | CAN interface | Through-hole | $1 ea |
| 1 | Screw terminal | 4-pos high current | 5mm pitch | $2 |
| - | Resistors | Various | 0603 | $2 |
| - | LEDs | Status indicators | 0805 | $0.50 |
| 1 | TVS diode | SMBJ58A | SMB | $0.50 |

**Component total: ~$35-45**

#### Step 4: Assembly

**Required equipment:**
- Soldering station with fine tip
- Hot air rework station (for QFN parts)
- Solder paste + stencil (recommended)
- Flux
- Magnification (microscope or loupe)

**Assembly order:**
1. Apply solder paste (stencil)
2. Place SMD components (smallest first)
3. Reflow (hot air or reflow oven)
4. Hand solder through-hole parts
5. Inspect all joints
6. Clean flux residue

#### Step 5: Flash Firmware

```bash
# Clone firmware repository
git clone https://github.com/LibreSolar/charge-controller-firmware.git
cd charge-controller-firmware

# Install PlatformIO (if not installed)
pip install platformio

# Build for MPPT 2420 HC
pio run -e mppt_2420_hc

# Flash via ST-Link or USB DFU
pio run -e mppt_2420_hc -t upload
```

**Total DIY cost: ~$50-60** (vs $150-250 commercial)

---

## MPPT 2420 HC Pinout & Connections

```
┌─────────────────────────────────────────────────────────────────┐
│                    MPPT 2420 HC CONNECTIONS                     │
│                                                                 │
│   SOLAR INPUT (left side):                                     │
│   ┌─────────────────────┐                                      │
│   │  PV+  │  PV-        │  Max 80V, observe polarity!          │
│   └─────────────────────┘                                      │
│                                                                 │
│   BATTERY OUTPUT (right side):                                 │
│   ┌─────────────────────┐                                      │
│   │  BAT+ │  BAT-       │  12V or 24V battery                 │
│   └─────────────────────┘                                      │
│                                                                 │
│   CAN BUS (RJ45 jacks):                                        │
│   ┌─────────────────────┐                                      │
│   │  CAN_H │ CAN_L │ GND │  ThingSet protocol                 │
│   └─────────────────────┘                                      │
│                                                                 │
│   UEXT CONNECTOR:                                              │
│   ┌─────────────────────┐                                      │
│   │ 3.3V │ GND │ TX │ RX │ SCL │ SDA │ MISO │ MOSI │ CS │    │
│   └─────────────────────┘                                      │
│   For expansion modules (display, WiFi, etc.)                  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

# PART 2: BMS C1

## What is a BMS?

```
WITHOUT BMS:                         WITH BMS:
Cells can become unbalanced          All cells protected & balanced

Cell 1: ████████████ 4.2V           Cell 1: ████████ 3.5V
Cell 2: ██████ 3.2V   ← DANGER!     Cell 2: ████████ 3.5V
Cell 3: ████████████ 4.2V           Cell 3: ████████ 3.5V
Cell 4: ██████████ 3.8V             Cell 4: ████████ 3.5V

Over-discharge Cell 2 = DEAD CELL    All cells balanced = LONG LIFE
Overcharge Cell 1 = FIRE RISK        Safe charging = SAFE SYSTEM
```

**BMS** = Battery Management System
- Monitors individual cell voltages
- Balances cells (passive or active)
- Protects against over/under voltage
- Measures current and temperature
- Calculates State of Charge (SOC)

---

## BMS C1 Specifications

| Parameter | Value |
|-----------|-------|
| Cell count | 3-16 series (configurable) |
| Cell types | LiFePO4, Li-ion NMC, LTO |
| Continuous current | 70-100A (MOSFET dependent) |
| Peak current | 150A+ (brief) |
| Balance current | ~150mA (passive) |
| Processor | ESP32-C3 (WiFi + BLE) |
| BMS chip | TI bq76952 |
| Communication | CAN, WiFi, Bluetooth, USB, I2C |
| Temperature sensors | 3× (cells + MOSFETs) |

### Block Diagram

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          BMS C1 BLOCK DIAGRAM                               │
│                                                                             │
│   BATTERY PACK                                                              │
│   (3-16 cells)                                                             │
│       │                                                                     │
│       ▼                                                                     │
│   ┌───────────────────────────────────────────────────────────────────┐    │
│   │                        CELL CONNECTIONS                           │    │
│   │   Cell 1+ ─┬─ Cell 2+ ─┬─ Cell 3+ ─┬─ ... ─┬─ Cell 16+ ─┬─ Pack+ │    │
│   │            │           │           │       │            │         │    │
│   │           ─┴─         ─┴─         ─┴─     ─┴─          ─┴─        │    │
│   │         Cell 1      Cell 2      Cell 3    ...       Cell 16      │    │
│   │           ─┬─         ─┬─         ─┬─     ─┬─          ─┬─        │    │
│   │            │           │           │       │            │         │    │
│   │   Cell 1- ─┴─ Cell 2- ─┴─ Cell 3- ─┴─ ... ─┴─ Cell 16- ─┴─ Pack- │    │
│   └───────────────────────────────────────────────────────────────────┘    │
│                              │                                              │
│                              ▼                                              │
│   ┌───────────────┐     ┌───────────────┐     ┌───────────────┐           │
│   │   bq76952     │     │   ESP32-C3    │     │   HIGH-SIDE   │           │
│   │   BMS IC      │◀───▶│   MCU         │────▶│   SWITCH      │           │
│   │               │     │               │     │  (MOSFETs)    │           │
│   │ • Cell monitor│     │ • WiFi/BLE   │     │               │           │
│   │ • Balancing   │     │ • CAN        │     │ 70-100A       │           │
│   │ • Protection  │     │ • USB        │     │ continuous    │           │
│   └───────────────┘     │ • ThingSet   │     └───────┬───────┘           │
│                         └───────────────┘             │                    │
│                                │                      │                    │
│                         ┌──────┴──────┐               │                    │
│                         │  CURRENT    │               │                    │
│                         │   SHUNT     │◀──────────────┘                    │
│                         │  (sensing)  │                                    │
│                         └─────────────┘                                    │
│                                │                                            │
│                                ▼                                            │
│                          LOAD OUTPUT                                        │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Building BMS C1

### Step 1: Get Design Files

```bash
# Clone the repository
git clone https://github.com/LibreSolar/bms-c1.git
cd bms-c1

# Design files:
# /hardware - KiCad schematics and PCB
# Firmware is separate: github.com/LibreSolar/bms-firmware
```

### Step 2: Order PCBs

**PCB Specs:**
- Layers: 4
- Thickness: 1.6mm
- Copper: 2oz (all layers for current)
- Size: ~100mm × 80mm

**Cost:** ~$15-25 for 5 boards

### Step 3: Order Components

**Main Components:**

| Qty | Component | Value/Part | Package | Est. Price |
|-----|-----------|------------|---------|------------|
| 1 | BMS IC | bq76952 | TQFP-44 | $8 |
| 1 | MCU | ESP32-C3-WROOM-02 | Module | $3 |
| 4 | Power MOSFET | PSMN4R0-30YL | LFPAK56 | $2 ea |
| 1 | Current shunt | 100µΩ | SMD | $3 |
| 1 | Shunt amplifier | INA181 | SOT-23 | $1 |
| 1 | CAN transceiver | SN65HVD230 | SOIC-8 | $2 |
| 3 | NTC thermistor | 10kΩ | 0603 | $0.30 ea |
| 16 | Cell tap resistor | 100Ω | 0603 | $0.50 |
| 16 | Cell filter cap | 100nF | 0603 | $0.50 |
| 1 | M5 terminal | Pack connections | High current | $5 |
| - | Capacitors | Various | 0603/0805 | $3 |
| - | Resistors | Various | 0603 | $2 |

**Component total: ~$45-60**

### Step 4: Assembly

**Critical notes:**
- bq76952 requires precise soldering (0.5mm pitch)
- Power MOSFETs need good thermal connection
- Current shunt must be Kelvin connected
- Cell sense wiring must be exact

**Assembly steps:**
1. Solder paste + stencil
2. Place BMS IC and MCU first (inspect carefully!)
3. Place remaining SMD components
4. Reflow
5. Hand solder power terminals
6. Attach heatsink to backside
7. Test before connecting to battery!

### Step 5: Flash Firmware

```bash
# Clone firmware
git clone https://github.com/LibreSolar/bms-firmware.git
cd bms-firmware

# Build for BMS C1
pio run -e bms_c1

# Flash via USB (ESP32-C3 has USB bootloader)
pio run -e bms_c1 -t upload
```

### Step 6: Configuration

Configure via serial console or ThingSet:

```
# Set cell count (example: 16S for 48V)
conf/bat_num_cells = 16

# Set cell type (LiFePO4)
conf/bat_type = 1

# Set voltage limits
conf/cell_ov_limit = 3.65    # Overvoltage (V)
conf/cell_uv_limit = 2.50    # Undervoltage (V)

# Set current limits
conf/dis_oc_limit = 100      # Discharge overcurrent (A)
conf/chg_oc_limit = 50       # Charge overcurrent (A)
```

**Total DIY cost: ~$70-90** (vs $200-400 commercial)

---

## BMS C1 Connections

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        BMS C1 WIRING DIAGRAM                                │
│                                                                             │
│   BATTERY CELLS (16S LiFePO4 example):                                     │
│                                                                             │
│   Cell 1+ ───────────────────────────────────────────── B+ (Pack positive) │
│         │                                                                   │
│        ─┴─ Cell 1                                                          │
│         │                                                                   │
│   Cell 1-/2+ ─────────────────────────────────────────── Tap 1            │
│         │                                                                   │
│        ─┴─ Cell 2                                                          │
│         │                                                                   │
│   Cell 2-/3+ ─────────────────────────────────────────── Tap 2            │
│         │                                                                   │
│        ─┴─ Cell 3                                                          │
│         │                                                                   │
│        ...  (continue for all 16 cells)                                    │
│         │                                                                   │
│   Cell 16- ──────────────────────────────────────────── B- (Pack negative)│
│                                                                             │
│                                                                             │
│   BMS BOARD TERMINALS:                                                     │
│   ┌─────────────────────────────────────────────────────────────────────┐ │
│   │                                                                     │ │
│   │   B+    B-    P+    P-    T1   T2   T3   CAN_H  CAN_L  GND         │ │
│   │   │     │     │     │     │    │    │      │      │     │          │ │
│   │   │     │     │     │     │    │    │      │      │     │          │ │
│   └───┼─────┼─────┼─────┼─────┼────┼────┼──────┼──────┼─────┼──────────┘ │
│       │     │     │     │     │    │    │      │      │     │            │
│       │     │     │     │     │    │    │      └──────┴─────┴─► CAN Bus │
│       │     │     │     │     │    │    │                                │
│       │     │     │     │     │    │    └─► Thermistor 3 (ambient)      │
│       │     │     │     │     │    └──────► Thermistor 2 (MOSFETs)      │
│       │     │     │     │     └───────────► Thermistor 1 (cells)        │
│       │     │     │     │                                                │
│       │     │     │     └─────────────────► Pack negative (to load)     │
│       │     │     └───────────────────────► Pack positive (to load)     │
│       │     │                                                            │
│       │     └─────────────────────────────► Battery negative            │
│       └───────────────────────────────────► Battery positive            │
│                                                                          │
│   CELL TAP CONNECTOR (16-pin):                                          │
│   ┌─────────────────────────────────────────────────────────────────┐   │
│   │  0   1   2   3   4   5   6   7   8   9  10  11  12  13  14  15  │   │
│   │  │   │   │   │   │   │   │   │   │   │   │   │   │   │   │   │  │   │
│   │  B-  ├───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┤  │   │
│   │     C1  C2  C3  C4  C5  C6  C7  C8  C9 C10 C11 C12 C13 C14 C15 B+ │   │
│   └─────────────────────────────────────────────────────────────────┘   │
│                                                                          │
└──────────────────────────────────────────────────────────────────────────┘
```

---

# PART 3: 48V SYSTEM INTEGRATION

## Configuration for 48V

For your OzInverter and microgrid, configure for 48V (16S LiFePO4):

### Battery Pack: 16S LiFePO4

```
┌─────────────────────────────────────────────────────────────────┐
│                   16S LiFePO4 BATTERY PACK                      │
│                                                                 │
│   Single cell: 3.2V nominal (2.5V - 3.65V range)               │
│                                                                 │
│   16 cells in series:                                          │
│   • Nominal: 16 × 3.2V = 51.2V                                 │
│   • Full:    16 × 3.65V = 58.4V                                │
│   • Empty:   16 × 2.5V = 40V                                   │
│                                                                 │
│   Recommended cells:                                           │
│   • EVE LF280K (280Ah) - Good value                           │
│   • CATL 271Ah - Reliable                                      │
│   • Lishen 272Ah - Budget option                               │
│                                                                 │
│   Pack capacity example (280Ah):                               │
│   51.2V × 280Ah = 14.3 kWh                                     │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### MPPT Configuration for 48V

The MPPT 2420 HC is designed for 12V/24V. For 48V, use **two in series**:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                   DUAL MPPT FOR 48V SYSTEM                                  │
│                                                                             │
│   OPTION A: Two MPPTs in Series Output                                     │
│                                                                             │
│   Solar 1          Solar 2                                                 │
│   (2kW)            (2kW)                                                   │
│     │                │                                                      │
│     ▼                ▼                                                      │
│   ┌──────┐        ┌──────┐                                                 │
│   │MPPT 1│        │MPPT 2│                                                 │
│   │ 24V  │        │ 24V  │                                                 │
│   │ out  │        │ out  │                                                 │
│   └──┬───┘        └──┬───┘                                                 │
│      │               │                                                      │
│      │   ┌───────────┘                                                      │
│      │   │                                                                  │
│      ▼   ▼                                                                  │
│   ┌─────────┐     Note: MPPT 2 "ground" connects to MPPT 1 "positive"     │
│   │  SERIES │     Output: 24V + 24V = 48V                                  │
│   │  48V    │                                                              │
│   └────┬────┘     ⚠️ Requires isolated MPPT outputs!                       │
│        │                                                                    │
│        ▼                                                                    │
│     48V Bus                                                                │
│                                                                             │
│   OPTION B: Parallel MPPTs with DC-DC Boost (Recommended)                  │
│                                                                             │
│   Solar 1          Solar 2                                                 │
│   (2kW)            (2kW)                                                   │
│     │                │                                                      │
│     ▼                ▼                                                      │
│   ┌──────┐        ┌──────┐                                                 │
│   │MPPT 1│        │MPPT 2│                                                 │
│   │ 24V  │        │ 24V  │                                                 │
│   └──┬───┘        └──┬───┘                                                 │
│      │               │                                                      │
│      └───────┬───────┘                                                      │
│              │                                                              │
│          24V Bus                                                            │
│              │                                                              │
│              ▼                                                              │
│        ┌──────────┐                                                        │
│        │ DC-DC    │                                                        │
│        │ BOOST    │   24V → 48V converter                                  │
│        │ 24V→48V  │   (Libre Solar has designs for this too!)             │
│        └────┬─────┘                                                        │
│             │                                                               │
│             ▼                                                               │
│          48V Bus                                                           │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Alternative: Wait for Libre Solar 48V MPPT

Libre Solar is developing higher voltage controllers. Check:
- https://github.com/LibreSolar
- https://libre.solar/hardware/

---

## Complete 48V System Schematic

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                                                                             │
│                    COMPLETE 48V LIBRE SOLAR SYSTEM                          │
│                                                                             │
│   ┌─────────────────────────────────────────────────────────────────────┐  │
│   │                         SOLAR ARRAY                                  │  │
│   │                       4kW (8× 500W)                                  │  │
│   │                           │                                          │  │
│   │              ┌────────────┴────────────┐                            │  │
│   │              │                         │                            │  │
│   │         ┌────┴────┐              ┌────┴────┐                        │  │
│   │         │  MPPT   │              │  MPPT   │                        │  │
│   │         │ 2420 HC │              │ 2420 HC │                        │  │
│   │         │   #1    │              │   #2    │                        │  │
│   │         │  24V    │              │  24V    │                        │  │
│   │         └────┬────┘              └────┬────┘                        │  │
│   │              │                        │                             │  │
│   │              └───────────┬────────────┘                             │  │
│   │                          │                                          │  │
│   │                     24V DC BUS                                      │  │
│   │                          │                                          │  │
│   │                    ┌─────┴─────┐                                    │  │
│   │                    │  DC-DC    │                                    │  │
│   │                    │  BOOST    │                                    │  │
│   │                    │ 24V→48V   │                                    │  │
│   │                    └─────┬─────┘                                    │  │
│   │                          │                                          │  │
│   └──────────────────────────┼──────────────────────────────────────────┘  │
│                              │                                              │
│   ════════════════════════════════════════════════════════════════════════ │
│                           48V DC BUS                                        │
│   ════════════════════════════════════════════════════════════════════════ │
│          │                   │                        │                     │
│          │                   │                        │                     │
│   ┌──────┴──────┐    ┌───────┴───────┐       ┌───────┴───────┐            │
│   │             │    │               │       │               │            │
│   │   BMS C1    │    │  OzInverter   │       │    LOADS      │            │
│   │             │    │    6kW        │       │    (DC)       │            │
│   │  16S cells  │    │               │       │               │            │
│   │  100A       │    │   48V → 220V  │       │  Lights, USB  │            │
│   │  WiFi/CAN   │    │   Pure Sine   │       │  DC appliances│            │
│   │             │    │               │       │               │            │
│   └──────┬──────┘    └───────┬───────┘       └───────────────┘            │
│          │                   │                                              │
│          │                   ▼                                              │
│   ┌──────┴──────┐      ┌───────────┐                                       │
│   │   BATTERY   │      │  220V AC  │                                       │
│   │   PACK      │      │   LOADS   │                                       │
│   │             │      │           │                                       │
│   │ 16S 280Ah   │      │  House    │                                       │
│   │ LiFePO4     │      │  appliances│                                      │
│   │ 14.3 kWh    │      │           │                                       │
│   │             │      └───────────┘                                       │
│   └─────────────┘                                                          │
│                                                                             │
│   ┌─────────────────────────────────────────────────────────────────────┐  │
│   │                      COMMUNICATION                                   │  │
│   │                                                                      │  │
│   │   MPPT ◄──── CAN BUS ────► BMS ◄──── CAN BUS ────► ESP32 Gateway   │  │
│   │                                                         │            │  │
│   │                                                         │            │  │
│   │                                                    WiFi │            │  │
│   │                                                         │            │  │
│   │                                                         ▼            │  │
│   │                                                  Home Assistant      │  │
│   │                                                                      │  │
│   └─────────────────────────────────────────────────────────────────────┘  │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

# PART 4: CAN BUS & THINGSET

## CAN Bus Network

All Libre Solar devices communicate via **CAN bus** using **ThingSet** protocol.

### Physical Connection

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         CAN BUS WIRING                                      │
│                                                                             │
│   Use standard Ethernet cables (RJ45) for CAN connections:                 │
│                                                                             │
│   ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐            │
│   │  MPPT 1  │    │  MPPT 2  │    │  BMS C1  │    │  ESP32   │            │
│   │  ┌────┐  │    │  ┌────┐  │    │  ┌────┐  │    │ Gateway  │            │
│   │  │RJ45│  │    │  │RJ45│  │    │  │RJ45│  │    │  ┌────┐  │            │
│   │  │ IN │  │    │  │ IN │  │    │  │ IN │  │    │  │RJ45│  │            │
│   │  └──┬─┘  │    │  └──┬─┘  │    │  └──┬─┘  │    │  └──┬─┘  │            │
│   │     │    │    │     │    │    │     │    │    │     │    │            │
│   │  ┌──┴─┐  │    │  ┌──┴─┐  │    │  ┌──┴─┐  │    │     │    │            │
│   │  │RJ45│  │    │  │RJ45│  │    │  │RJ45│  │    │    ─┴─   │            │
│   │  │OUT │  │    │  │OUT │  │    │  │OUT │  │    │   120Ω   │            │
│   │  └──┬─┘  │    │  └──┬─┘  │    │  └──┬─┘  │    │  Terminator           │
│   └─────┼────┘    └─────┼────┘    └─────┼────┘    └──────────┘            │
│         │               │               │                                   │
│         └───────────────┴───────────────┘                                   │
│                    Daisy-chain topology                                     │
│                                                                             │
│   IMPORTANT:                                                               │
│   • 120Ω termination resistor at EACH END of bus                          │
│   • Max cable length: ~40m at 250kbps                                     │
│   • Use twisted pair (Cat5/Cat6 cable)                                    │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### ThingSet Protocol

Libre Solar uses **ThingSet** - an open, text-based protocol:

```
# Example ThingSet commands (via serial or CAN):

# Read battery voltage
?Bat/V
> 51.2

# Read all battery data
?Bat
> {"V": 51.2, "A": -5.3, "SOC": 85, "T": 25}

# Read cell voltages
?Bat/Cells
> [3.20, 3.21, 3.20, 3.19, 3.21, 3.20, ...]

# Set charging current limit
=Chg/A_max 30

# Read solar power
?Solar/W
> 1250

# Read all data (discovery)
?
> {"Device": {...}, "Bat": {...}, "Solar": {...}, ...}
```

---

## ESP32 CAN Gateway

Connect Libre Solar to Home Assistant via ESP32:

### Hardware

```
┌─────────────────────────────────────────────────────────────────┐
│                    ESP32 CAN GATEWAY                            │
│                                                                 │
│   ESP32 DevKit                                                  │
│   ┌─────────────────┐                                          │
│   │                 │                                          │
│   │  GPIO 21 ──────┼──► CAN TX ──► SN65HVD230 ──► CAN_H       │
│   │  GPIO 22 ──────┼──► CAN RX ──► SN65HVD230 ──► CAN_L       │
│   │  3.3V ─────────┼──► VCC                                    │
│   │  GND ──────────┼──► GND                                    │
│   │                 │                                          │
│   │  WiFi ─────────┼──► Home Assistant                        │
│   │                 │                                          │
│   └─────────────────┘                                          │
│                                                                 │
│   Components needed:                                           │
│   • ESP32 DevKit                        $5                     │
│   • SN65HVD230 CAN transceiver          $2                     │
│   • RJ45 jack                           $1                     │
│   • Perfboard + wires                   $2                     │
│                                                                 │
│   Total: ~$10                                                  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### ESPHome Configuration

```yaml
# libre_solar_gateway.yaml

esphome:
  name: libre-solar-gateway
  platform: ESP32
  board: esp32dev

wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password

api:
mqtt:
  broker: !secret mqtt_broker

# CAN bus configuration
canbus:
  - platform: esp32_can
    id: can_bus
    tx_pin: GPIO21
    rx_pin: GPIO22
    can_id: 0x100
    bit_rate: 250kbps

# Custom component for ThingSet parsing
external_components:
  - source: github://LibreSolar/esphome-thingset

# Sensors from MPPT
sensor:
  - platform: thingset
    can_id: 0x01
    name: "MPPT Battery Voltage"
    path: "Bat/V"
    unit_of_measurement: "V"

  - platform: thingset
    can_id: 0x01
    name: "MPPT Solar Power"
    path: "Solar/W"
    unit_of_measurement: "W"

  - platform: thingset
    can_id: 0x01
    name: "MPPT Charge Current"
    path: "Bat/A"
    unit_of_measurement: "A"

# Sensors from BMS
  - platform: thingset
    can_id: 0x02
    name: "BMS State of Charge"
    path: "Bat/SOC"
    unit_of_measurement: "%"

  - platform: thingset
    can_id: 0x02
    name: "BMS Cell Voltage Min"
    path: "Bat/Cell_min"
    unit_of_measurement: "V"

  - platform: thingset
    can_id: 0x02
    name: "BMS Cell Voltage Max"
    path: "Bat/Cell_max"
    unit_of_measurement: "V"

  - platform: thingset
    can_id: 0x02
    name: "BMS Temperature"
    path: "Bat/T"
    unit_of_measurement: "°C"
```

---

# PART 5: BILL OF MATERIALS

## Complete Libre Solar BOM

### MPPT 2420 HC (DIY Build)

| Qty | Component | Est. Price |
|-----|-----------|------------|
| 1 | PCB (4-layer) | $12 |
| 1 | STM32G431CBT6 | $5 |
| 2 | Power MOSFETs | $6 |
| 1 | Inductor 22µH | $5 |
| 1 | Gate driver | $2 |
| - | Capacitors, resistors | $5 |
| - | Connectors | $3 |
| - | Misc components | $5 |
| **Total** | | **~$43** |

### BMS C1 (DIY Build)

| Qty | Component | Est. Price |
|-----|-----------|------------|
| 1 | PCB (4-layer) | $15 |
| 1 | bq76952 | $8 |
| 1 | ESP32-C3-WROOM | $3 |
| 4 | Power MOSFETs | $8 |
| 1 | Current shunt | $3 |
| - | Capacitors, resistors | $5 |
| - | Connectors, terminals | $8 |
| - | Thermistors | $2 |
| **Total** | | **~$52** |

### CAN Gateway

| Qty | Component | Est. Price |
|-----|-----------|------------|
| 1 | ESP32 DevKit | $5 |
| 1 | SN65HVD230 | $2 |
| 1 | RJ45 jack | $1 |
| - | Perfboard, wires | $2 |
| **Total** | | **~$10** |

### Complete Libre Solar Setup

| Component | DIY Price | Commercial Equivalent |
|-----------|-----------|----------------------|
| 2× MPPT 2420 HC | $86 | $300+ |
| 1× BMS C1 | $52 | $200+ |
| 1× CAN Gateway | $10 | $50+ |
| **Total** | **$148** | **$550+** |

**Savings: ~$400 (73%)!**

---

## Combined System BOM (Single Home)

| Component | Specification | Price |
|-----------|---------------|-------|
| **Solar** | | |
| Panels | 4kW (8× 500W) | $800 |
| MPPT (Libre Solar) | 2× MPPT 2420 HC DIY | $86 |
| DC-DC Boost | 24V→48V (DIY or buy) | $50 |
| **Battery** | | |
| Cells | 16S 280Ah LiFePO4 | $1,800 |
| BMS (Libre Solar) | BMS C1 DIY | $52 |
| **Inverter** | | |
| OzInverter | 6kW (your build) | $600 |
| **Monitoring** | | |
| CAN Gateway | ESP32 + transceiver | $10 |
| Smart Monitor | ESP32 for inverter | $40 |
| **Infrastructure** | | |
| DC Bus | Bus bars, fuses, wiring | $150 |
| Enclosure | Weatherproof cabinet | $100 |
| **Total** | | **$3,688** |

**Compare to commercial:** Victron system = $8,000-10,000+

---

# PART 6: LEARNING OUTCOMES

After completing this module:

- [ ] Understand MPPT algorithm principles
- [ ] Can build or configure MPPT 2420 HC
- [ ] Understand BMS cell balancing and protection
- [ ] Can build or configure BMS C1
- [ ] Know CAN bus wiring and termination
- [ ] Understand ThingSet protocol
- [ ] Can integrate Libre Solar with Home Assistant
- [ ] Have 100% open source power system!

---

# RESOURCES

## Official Libre Solar

- **Website:** https://libre.solar
- **GitHub:** https://github.com/LibreSolar
- **Forum:** https://talk.libre.solar
- **Documentation:** https://libre.solar/docs/

## Specific Repositories

- MPPT 2420 HC: https://github.com/LibreSolar/mppt-2420-hc
- BMS C1: https://github.com/LibreSolar/bms-c1
- Charge Controller Firmware: https://github.com/LibreSolar/charge-controller-firmware
- BMS Firmware: https://github.com/LibreSolar/bms-firmware
- ThingSet: https://github.com/ThingSet

## Community

- Libre Solar Forum: https://talk.libre.solar
- Discord/Matrix (check website)
- OpenEnergyMonitor: https://openenergymonitor.org

---

*With Libre Solar integration, your entire power system is now 100% open source - from solar input to AC output. Full transparency, full control, full freedom!*
