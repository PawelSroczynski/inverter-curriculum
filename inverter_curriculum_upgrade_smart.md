# Inverter Curriculum - Smart Upgrade Module
## Making OzInverter "Victron-Like" with ESP32 Monitoring

---

## Overview: Bridging the Gap to Victron

### What Victron Has (That OzInverter Lacks)

| Victron Feature | Status in Basic OzInverter |
|-----------------|---------------------------|
| Remote monitoring app | ❌ None |
| Voltage/Current display | ❌ None |
| Data logging | ❌ None |
| WiFi/Bluetooth | ❌ None |
| Temperature monitoring | ❌ Basic (thermal shutdown only) |
| Overcurrent alerts | ❌ Basic (fuse only) |
| Energy metering (kWh) | ❌ None |
| Home Assistant integration | ❌ None |
| Historical graphs | ❌ None |
| Remote ON/OFF | ❌ None |

### What We Can Add with ESP32/SmartBob

| Feature | With Smart Upgrade |
|---------|-------------------|
| Remote monitoring app | ✅ Via Home Assistant / custom web UI |
| Voltage/Current display | ✅ OLED + web dashboard |
| Data logging | ✅ InfluxDB / local storage |
| WiFi/Bluetooth | ✅ ESP32 built-in |
| Temperature monitoring | ✅ Multiple sensors |
| Overcurrent alerts | ✅ Push notifications |
| Energy metering (kWh) | ✅ Calculated from V×I |
| Home Assistant integration | ✅ Native ESPHome |
| Historical graphs | ✅ Grafana / HA dashboards |
| Remote ON/OFF | ✅ Relay control |

---

## Architecture: Smart OzInverter

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         SMART OZINVERTER SYSTEM                             │
│                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                      MONITORING & CONTROL                            │   │
│  │                                                                      │   │
│  │   ┌──────────────┐    ┌──────────────┐    ┌──────────────┐         │   │
│  │   │   ESP32      │    │    OLED      │    │   RELAYS     │         │   │
│  │   │  SmartBob    │───▶│   Display    │    │   Control    │         │   │
│  │   │  Controller  │    │  Local UI    │    │  ON/OFF/ATS  │         │   │
│  │   └──────┬───────┘    └──────────────┘    └──────────────┘         │   │
│  │          │                                                          │   │
│  │          │ ADC / I2C / 1-Wire                                       │   │
│  │          │                                                          │   │
│  │   ┌──────┴───────────────────────────────────────────────┐         │   │
│  │   │                    SENSORS                            │         │   │
│  │   │  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐ │         │   │
│  │   │  │Voltage  │  │Current  │  │  Temp   │  │  Temp   │ │         │   │
│  │   │  │Sensor   │  │Sensor   │  │Heatsink │  │Ambient  │ │         │   │
│  │   │  │(ADC)    │  │(ACS758) │  │(DS18B20)│  │(DS18B20)│ │         │   │
│  │   │  └─────────┘  └─────────┘  └─────────┘  └─────────┘ │         │   │
│  │   └──────────────────────────────────────────────────────┘         │   │
│  │                                                                      │   │
│  └──────────────────────────────────────────────────────────────────────┘   │
│                              │                                              │
│                              │ WiFi / LAN                                   │
│                              ▼                                              │
│  ┌──────────────────────────────────────────────────────────────────────┐  │
│  │                      HOME ASSISTANT / CLOUD                          │  │
│  │   ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐            │  │
│  │   │Dashboard │  │  Alerts  │  │  Graphs  │  │  Remote  │            │  │
│  │   │   App    │  │Telegram/ │  │ History  │  │ Control  │            │  │
│  │   │          │  │  Push    │  │          │  │          │            │  │
│  │   └──────────┘  └──────────┘  └──────────┘  └──────────┘            │  │
│  └──────────────────────────────────────────────────────────────────────┘  │
│                                                                             │
│  ┌──────────────────────────────────────────────────────────────────────┐  │
│  │                         OZINVERTER POWER                             │  │
│  │                                                                      │  │
│  │   Battery ──▶ Control ──▶ H-Bridge ──▶ Transformer ──▶ AC Output   │  │
│  │     48V        Board      MOSFETs       Toroid         220V        │  │
│  │                                                                      │  │
│  └──────────────────────────────────────────────────────────────────────┘  │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Option A: SmartBob SM-MINI-0808R Integration

### Why SmartBob?

| Feature | Benefit for Inverter |
|---------|---------------------|
| 8× relay outputs (10A) | Main ON/OFF, ATS control, fan control, load shedding |
| 8× 24V inputs | Status signals, fault detection, battery BMS signals |
| 2× ADC channels | Voltage monitoring (battery + output) |
| ESP32 | WiFi, Bluetooth, processing power |
| OLED display | Local status without phone |
| DIN rail mount | Professional installation |
| ESPHome ready | Easy Home Assistant integration |

### SmartBob Wiring for OzInverter

```
┌─────────────────────────────────────────────────────────────────┐
│                    SM-MINI-0808R CONNECTIONS                    │
│                                                                 │
│  INPUTS (directly to SmartBob logic inputs):                   │
│  ─────────────────────────────────────────────                 │
│  IN1 ◄─── EG8010 FAULT output (inverter fault signal)         │
│  IN2 ◄─── Overcurrent comparator output                        │
│  IN3 ◄─── Overvoltage comparator output                        │
│  IN4 ◄─── Grid present signal (for ATS)                        │
│  IN5 ◄─── Battery BMS fault signal                             │
│  IN6 ◄─── Fan tachometer (RPM feedback)                        │
│  IN7 ◄─── Spare                                                │
│  IN8 ◄─── Spare                                                │
│                                                                 │
│  ADC CHANNELS (voltage dividers required):                     │
│  ─────────────────────────────────────────                     │
│  ADC1 ◄─── Battery voltage (48V → 3.3V via divider)           │
│  ADC2 ◄─── AC output voltage (via transformer + rectifier)    │
│                                                                 │
│  RELAY OUTPUTS:                                                │
│  ──────────────                                                │
│  REL1 ───► Main inverter ON/OFF (enable EG8010)               │
│  REL2 ───► Battery disconnect contactor                        │
│  REL3 ───► AC output contactor                                 │
│  REL4 ───► Grid bypass contactor (ATS)                        │
│  REL5 ───► Fan 1 control                                       │
│  REL6 ───► Fan 2 control                                       │
│  REL7 ───► Load shedding relay 1                              │
│  REL8 ───► Load shedding relay 2                              │
│                                                                 │
│  I2C BUS:                                                      │
│  ────────                                                      │
│  → ACS758 current sensor (via ADC board)                       │
│  → INA226 power monitor (alternative)                          │
│                                                                 │
│  1-WIRE BUS:                                                   │
│  ───────────                                                   │
│  → DS18B20 #1 (MOSFET heatsink temperature)                   │
│  → DS18B20 #2 (Transformer temperature)                        │
│  → DS18B20 #3 (Ambient temperature)                            │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### SmartBob Bill of Materials

| Qty | Component | Purpose | Est. Price |
|-----|-----------|---------|------------|
| 1 | SM-MINI-0808R | Main controller | ~€80 |
| 3 | DS18B20 | Temperature sensors | $3 |
| 1 | ACS758-100A | Current sensor | $8 |
| 2 | Voltage divider PCB | ADC input scaling | $5 |
| 1 | Contactor 80A | Main battery disconnect | $25 |
| 1 | Contactor 40A | AC output | $15 |
| 1 | Contactor 40A | Grid bypass (ATS) | $15 |
| - | Wiring, terminals | Connections | $10 |

**SmartBob Upgrade Total: ~€160 / $175**

---

## Option B: DIY ESP32 Monitoring (Budget Alternative)

### If SmartBob Is Too Expensive

```
┌─────────────────────────────────────────────────────────────────┐
│              DIY ESP32 MONITORING BOARD                         │
│                                                                 │
│   ┌─────────────┐                                              │
│   │   ESP32     │                                              │
│   │   DevKit    │◄─── $5                                       │
│   └──────┬──────┘                                              │
│          │                                                      │
│   ┌──────┴──────┐                                              │
│   │             │                                              │
│   ▼             ▼                                              │
│ ┌─────┐     ┌─────────┐                                        │
│ │OLED │     │ Relay   │                                        │
│ │0.96"│     │ Module  │                                        │
│ │ $3  │     │ 4ch $4  │                                        │
│ └─────┘     └─────────┘                                        │
│                                                                 │
│ Sensors:                                                        │
│ ├── ACS758 current sensor ──── $8                              │
│ ├── Voltage divider (DIY) ──── $1                              │
│ ├── DS18B20 ×3 ──────────────── $3                             │
│ └── INA226 (optional) ───────── $5                             │
│                                                                 │
│ Total: ~$25-35                                                 │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### DIY ESP32 BOM

| Qty | Component | Purpose | Est. Price |
|-----|-----------|---------|------------|
| 1 | ESP32 DevKit V1 | Main controller | $5 |
| 1 | OLED 0.96" I2C | Local display | $3 |
| 1 | 4-channel relay module | Control outputs | $4 |
| 1 | ACS758-100A | Current sensing | $8 |
| 3 | DS18B20 waterproof | Temperature | $3 |
| 1 | INA226 module | Precision V/I | $5 |
| 4 | Resistor 100kΩ 1% | Voltage dividers | $0.50 |
| 4 | Resistor 10kΩ 1% | Voltage dividers | $0.50 |
| 1 | PCB prototype | Assembly | $2 |
| 1 | DC-DC converter | 48V→5V isolated | $5 |

**DIY ESP32 Total: ~$36**

---

## Sensor Details

### 1. Voltage Sensing (Battery 48V)

```
Battery 48V cannot connect directly to ESP32 (max 3.3V)!

Voltage Divider:

    48V ────┬──[100kΩ]──┬──[10kΩ]──┬── GND
            │           │          │
            │         ESP32       │
            │         ADC         │
            │         (4.36V max) │
            │                     │
            └─────────────────────┘

Calculation:
Vout = 48V × (10k / (100k + 10k)) = 48 × 0.0909 = 4.36V

Scale factor in code:
actual_voltage = adc_reading × 11.0
```

### 2. Current Sensing (ACS758)

```
┌─────────────────────────────────────────┐
│              ACS758-100A                │
│                                         │
│   HIGH CURRENT PATH                     │
│   ════════════════                      │
│   From battery ─────┬───── To inverter  │
│                     │                   │
│                  [sensor]               │
│                     │                   │
│   OUTPUT ───────────┴───── To ESP32 ADC│
│   (66mV per Amp)                        │
│                                         │
│   At 0A:   Vout = 2.5V (center)        │
│   At 100A: Vout = 2.5V + 6.6V = 9.1V   │
│   (needs voltage divider for ESP32!)    │
│                                         │
└─────────────────────────────────────────┘

Code calculation:
current = (adc_voltage - 2.5) / 0.066
```

### 3. Temperature Sensing (DS18B20)

```
1-Wire Bus (multiple sensors on one wire):

ESP32           ┌──────────┐
GPIO4 ──────────┤ DS18B20  │ Heatsink
    │     4.7kΩ │ #1       │
    ├────[R]────┤          │
    │           └──────────┘
    │
    │           ┌──────────┐
    ├───────────┤ DS18B20  │ Transformer
    │           │ #2       │
    │           └──────────┘
    │
    │           ┌──────────┐
    └───────────┤ DS18B20  │ Ambient
                │ #3       │
                └──────────┘

Each sensor has unique address - auto-detected.
```

---

## Software: ESPHome Configuration

### Basic ESPHome YAML

```yaml
# ozinverter_monitor.yaml

esphome:
  name: ozinverter
  platform: ESP32
  board: esp32dev

wifi:
  ssid: "YourWiFi"
  password: "YourPassword"

api:
  encryption:
    key: "your-api-key"

ota:
  password: "your-ota-password"

# I2C for OLED and INA226
i2c:
  sda: GPIO21
  scl: GPIO22

# 1-Wire for temperature sensors
one_wire:
  - pin: GPIO4

# OLED Display
display:
  - platform: ssd1306_i2c
    model: "SSD1306 128x64"
    address: 0x3C
    lambda: |-
      it.printf(0, 0, id(font1), "Battery: %.1fV", id(battery_voltage).state);
      it.printf(0, 16, id(font1), "Current: %.1fA", id(inverter_current).state);
      it.printf(0, 32, id(font1), "Power: %.0fW", id(inverter_power).state);
      it.printf(0, 48, id(font1), "Temp: %.1fC", id(heatsink_temp).state);

font:
  - file: "fonts/arial.ttf"
    id: font1
    size: 14

# Sensors
sensor:
  # Battery Voltage (ADC with divider)
  - platform: adc
    pin: GPIO34
    name: "Battery Voltage"
    id: battery_voltage
    update_interval: 1s
    filters:
      - multiply: 11.0  # Voltage divider ratio
    unit_of_measurement: "V"

  # Inverter Current (ACS758)
  - platform: adc
    pin: GPIO35
    name: "Inverter Current"
    id: inverter_current
    update_interval: 1s
    filters:
      - lambda: return (x - 2.5) / 0.066;
    unit_of_measurement: "A"

  # Calculated Power
  - platform: template
    name: "Inverter Power"
    id: inverter_power
    unit_of_measurement: "W"
    lambda: |-
      return id(battery_voltage).state * id(inverter_current).state;
    update_interval: 1s

  # Energy meter (kWh)
  - platform: total_daily_energy
    name: "Daily Energy"
    power_id: inverter_power
    unit_of_measurement: "kWh"
    filters:
      - multiply: 0.001

  # Temperature sensors
  - platform: dallas_temp
    address: 0x1234567890ABCDEF  # Replace with actual
    name: "Heatsink Temperature"
    id: heatsink_temp

  - platform: dallas_temp
    address: 0xFEDCBA0987654321  # Replace with actual
    name: "Transformer Temperature"
    id: transformer_temp

  - platform: dallas_temp
    address: 0xABCDEF1234567890  # Replace with actual
    name: "Ambient Temperature"
    id: ambient_temp

# Control switches (relays)
switch:
  - platform: gpio
    pin: GPIO16
    name: "Inverter Enable"
    id: inverter_enable

  - platform: gpio
    pin: GPIO17
    name: "Battery Disconnect"
    id: battery_disconnect

  - platform: gpio
    pin: GPIO18
    name: "Fan Control"
    id: fan_control

# Binary sensors (fault inputs)
binary_sensor:
  - platform: gpio
    pin: GPIO26
    name: "Inverter Fault"
    device_class: problem

  - platform: gpio
    pin: GPIO27
    name: "Overcurrent"
    device_class: problem

# Automations
automation:
  # Over-temperature protection
  - trigger:
      platform: numeric_state
      entity_id: sensor.heatsink_temperature
      above: 80
    action:
      - switch.turn_off: inverter_enable
      - logger.log: "EMERGENCY: Heatsink over 80°C - Inverter disabled"

  # Fan auto-control
  - trigger:
      platform: numeric_state
      entity_id: sensor.heatsink_temperature
      above: 45
    action:
      - switch.turn_on: fan_control

  - trigger:
      platform: numeric_state
      entity_id: sensor.heatsink_temperature
      below: 35
    action:
      - switch.turn_off: fan_control
```

---

## Home Assistant Dashboard

### Example Lovelace Card

```yaml
# Home Assistant dashboard card
type: vertical-stack
cards:
  - type: gauge
    entity: sensor.ozinverter_battery_voltage
    name: Battery
    min: 44
    max: 56
    severity:
      green: 50
      yellow: 46
      red: 44

  - type: gauge
    entity: sensor.ozinverter_inverter_power
    name: Power Output
    min: 0
    max: 6000

  - type: history-graph
    entities:
      - sensor.ozinverter_inverter_power
      - sensor.ozinverter_heatsink_temperature
    hours_to_show: 24

  - type: entities
    entities:
      - switch.ozinverter_inverter_enable
      - switch.ozinverter_fan_control
      - binary_sensor.ozinverter_inverter_fault
```

---

## Feature Comparison: After Smart Upgrade

| Feature | Victron | OzInverter Basic | OzInverter + Smart |
|---------|---------|------------------|-------------------|
| Pure sine wave | ✅ | ✅ | ✅ |
| 6kW+ power | ✅ | ✅ | ✅ |
| Remote monitoring | ✅ VRM | ❌ | ✅ Home Assistant |
| Mobile app | ✅ Victron App | ❌ | ✅ HA App |
| Voltage display | ✅ | ❌ | ✅ OLED + Web |
| Current display | ✅ | ❌ | ✅ |
| Energy metering | ✅ | ❌ | ✅ |
| Temperature monitor | ✅ | ❌ | ✅ Multi-point |
| WiFi | ✅ | ❌ | ✅ |
| Bluetooth | ✅ | ❌ | ✅ |
| Data logging | ✅ | ❌ | ✅ InfluxDB |
| Alerts/Notifications | ✅ | ❌ | ✅ Telegram/Push |
| Auto thermal protection | ✅ | Basic | ✅ Advanced |
| Remote ON/OFF | ✅ | ❌ | ✅ |
| ATS capability | ✅ Some models | ❌ | ✅ With relays |
| **Price** | **$1,500-2,500** | **$600** | **$775-800** |

---

## Automatic Transfer Switch (ATS) Logic

### With Smart Controller, Add Grid Backup

```
                         ┌─────────────────┐
                         │   SmartBob /    │
                         │     ESP32       │
                         └────────┬────────┘
                                  │
              Grid detect ────────┤
              signal              │
                                  │
    ┌─────────────────────────────┼─────────────────────────────┐
    │                             │                             │
    ▼                             ▼                             ▼
┌────────┐                  ┌──────────┐                  ┌────────┐
│  Grid  │                  │   ATS    │                  │Inverter│
│  230V  │────────┬─────────│  Relays  │─────────┬───────│  230V  │
│        │        │         │          │         │        │        │
└────────┘        │         └──────────┘         │        └────────┘
                  │                              │
                  └──────────────┬───────────────┘
                                 │
                                 ▼
                            ┌─────────┐
                            │  LOAD   │
                            │ (House) │
                            └─────────┘

Logic:
1. Grid OK → Load fed from Grid
2. Grid fails → Wait 5 sec → Switch to Inverter
3. Grid returns → Wait 30 sec → Switch back to Grid
```

### ESPHome ATS Logic

```yaml
# ATS Automation
globals:
  - id: grid_stable_counter
    type: int
    initial_value: '0'

binary_sensor:
  - platform: gpio
    pin: GPIO25
    name: "Grid Present"
    id: grid_present
    on_press:  # Grid returns
      then:
        - delay: 30s  # Wait for stability
        - if:
            condition:
              binary_sensor.is_on: grid_present
            then:
              - switch.turn_on: grid_relay
              - delay: 100ms
              - switch.turn_off: inverter_relay
              - logger.log: "Switched to GRID"
    on_release:  # Grid fails
      then:
        - delay: 5s  # Wait to confirm failure
        - if:
            condition:
              binary_sensor.is_off: grid_present
            then:
              - switch.turn_off: grid_relay
              - delay: 100ms
              - switch.turn_on: inverter_relay
              - logger.log: "Switched to INVERTER"
```

---

## Bill of Materials Summary: Complete Smart OzInverter

### Option A: With SmartBob

| Component | Price |
|-----------|-------|
| OzInverter (Step 5) | $600 |
| SmartBob SM-MINI-0808R | $175 |
| Sensors (current, temp) | $15 |
| Contactors (3×) | $55 |
| Wiring & misc | $15 |
| **Total** | **$860** |

### Option B: DIY ESP32

| Component | Price |
|-----------|-------|
| OzInverter (Step 5) | $600 |
| ESP32 + sensors | $36 |
| Relay modules | $15 |
| Contactors (3×) | $55 |
| Isolated DC-DC | $10 |
| Wiring & misc | $15 |
| **Total** | **$731** |

---

## Comparison: Final Cost vs Victron

```
┌─────────────────────────────────────────────────────────────┐
│                    COST COMPARISON                          │
│                                                             │
│  Victron MultiPlus 48/5000            $2,200               │
│  ████████████████████████████████████████████              │
│                                                             │
│  Smart OzInverter (SmartBob)            $860               │
│  █████████████████                                         │
│                                                             │
│  Smart OzInverter (DIY ESP32)           $731               │
│  ██████████████                                            │
│                                                             │
│  Basic OzInverter                       $600               │
│  ████████████                                              │
│                                                             │
│  SAVINGS vs Victron:              $1,340 - $1,600          │
│                                   (61% - 73% less!)        │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## Learning Path: When to Add Smart Features

```
Recommended progression:

Step 1-4: Focus on electronics fundamentals
    │
    ▼
Step 5: Build basic OzInverter (working first!)
    │
    ▼
Step 5.1: Add ESP32 monitoring (this module)
    │       - Voltage/Current sensing
    │       - Temperature monitoring
    │       - Basic OLED display
    │
    ▼
Step 5.2: Add Home Assistant integration
    │       - WiFi connectivity
    │       - Dashboard
    │       - Mobile app access
    │
    ▼
Step 5.3: Add ATS and advanced features
    │       - Grid detection
    │       - Auto transfer switch
    │       - Load shedding
    │
    ▼
Complete: Victron-like Smart Inverter!
```

---

## Resources

### SmartBob
- Website: https://smartbob.pl
- ESPHome configs: Available on their GitHub

### ESPHome
- Documentation: https://esphome.io
- Home Assistant: https://home-assistant.io

### Required Skills (From Earlier Steps)
- Basic soldering (Step 1)
- Voltage dividers (Step 1.1)
- I2C communication (Step 1.8 counter project)
- Understanding ADC (Step 1.11 voltage indicator)

---

*This module transforms your OzInverter from a "dumb" power converter into a smart, monitored, remotely-controlled system rivaling commercial Victron units at 1/3 the cost!*
