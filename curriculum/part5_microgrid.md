# Inverter Curriculum - Microgrid Expansion Module
## Step 7: DC Microgrid with AC Tie (Hybrid Architecture)

---

## Overview

This module expands your single OzInverter into a 4-unit community microgrid using:
- **48V DC bus** within clusters (efficient, simple, safe)
- **230V AC tie** between clusters (for longer distances, grid option)

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                                                                             │
│                    HYBRID DC/AC MICROGRID ARCHITECTURE                      │
│                                                                             │
│   ┌─────────────────────────────┐     ┌─────────────────────────────┐      │
│   │       CLUSTER A             │     │       CLUSTER B             │      │
│   │     (Houses 1-2)            │     │     (Houses 3-4)            │      │
│   │                             │     │                             │      │
│   │   ══ 48V DC BUS ══          │     │   ══ 48V DC BUS ══          │      │
│   │         │                   │     │         │                   │      │
│   │    ┌────┴────┐              │     │    ┌────┴────┐              │      │
│   │    │         │              │     │    │         │              │      │
│   │  ┌─┴─┐     ┌─┴─┐            │     │  ┌─┴─┐     ┌─┴─┐            │      │
│   │  │INV│     │INV│            │     │  │INV│     │INV│            │      │
│   │  │ 1 │     │ 2 │            │     │  │ 3 │     │ 4 │            │      │
│   │  │3kW│     │3kW│            │     │  │3kW│     │3kW│            │      │
│   │  └─┬─┘     └─┬─┘            │     │  └─┬─┘     └─┬─┘            │      │
│   │    │         │              │     │    │         │              │      │
│   │  House1   House2            │     │  House3   House4            │      │
│   │    │         │              │     │    │         │              │      │
│   │  ┌─┴─────────┴─┐            │     │  ┌─┴─────────┴─┐            │      │
│   │  │   Battery   │            │     │  │   Battery   │            │      │
│   │  │   15kWh     │            │     │  │   15kWh     │            │      │
│   │  └─────────────┘            │     │  └─────────────┘            │      │
│   │         │                   │     │         │                   │      │
│   │    ┌────┴────┐              │     │    ┌────┴────┐              │      │
│   │    │ TIE     │              │     │    │ TIE     │              │      │
│   │    │ INVERTER│              │     │    │ INVERTER│              │      │
│   │    │ 6kW     │              │     │    │ 6kW     │              │      │
│   │    └────┬────┘              │     │    └────┬────┘              │      │
│   │         │                   │     │         │                   │      │
│   └─────────┼───────────────────┘     └─────────┼───────────────────┘      │
│             │                                   │                           │
│             │         ═══ 230V AC TIE ═══       │                           │
│             └───────────────────────────────────┘                           │
│                              │                                              │
│                         (Optional)                                          │
│                      Grid Connection                                        │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Why Hybrid Architecture?

| Benefit | Explanation |
|---------|-------------|
| **DC efficiency within cluster** | 95% efficiency, no sync needed |
| **AC for long distances** | 230V needs thinner cables |
| **Cluster independence** | If AC tie fails, clusters still work |
| **Grid-ready** | AC tie can connect to utility grid |
| **Simpler sync** | Only 2 tie inverters need to sync (not 4) |
| **Scalable** | Add more clusters easily |

---

## System Components

### Per Cluster (×2)

```
CLUSTER COMPONENTS:
├── 48V DC Bus
│   ├── Bus bars (copper, 100A rated)
│   ├── Fuses/breakers
│   └── DC distribution panel
│
├── Solar Input
│   ├── 4kW panels (8× 500W)
│   └── 2× MPPT chargers (48V/60A)
│
├── Battery Bank
│   ├── 48V 300Ah LiFePO4 (15kWh)
│   └── BMS with communication
│
├── House Inverters (×2)
│   ├── OzInverter 3kW each
│   └── Smart monitor (ESP32)
│
└── Tie Inverter (×1)
    ├── Bi-directional 6kW
    ├── Grid-forming capable
    └── Sync communication port
```

### Shared Infrastructure

```
SHARED COMPONENTS:
├── AC Tie Cable
│   ├── 3-core + earth (4mm²)
│   └── Distance: up to 500m
│
├── Central Controller
│   ├── Raspberry Pi 4
│   ├── Modbus/RS485 interface
│   └── Home Assistant server
│
├── Communication
│   ├── WiFi mesh (ESP-NOW)
│   └── RS485 backup bus
│
└── Protection
    ├── AC breakers
    ├── Surge protection
    └── Ground fault detection
```

---

## Detailed Schematic

### Cluster Internal (48V DC)

```
                            SOLAR ARRAY (4kW)
                     ┌──────────┴──────────┐
                     │                     │
                 ┌───┴───┐             ┌───┴───┐
                 │ MPPT  │             │ MPPT  │
                 │  #1   │             │  #2   │
                 │ 2kW   │             │ 2kW   │
                 └───┬───┘             └───┬───┘
                     │                     │
                     └──────────┬──────────┘
                                │
    ════════════════════════════╪════════════════════════════
                        48V DC BUS (Main)
    ════════════════════════════╪════════════════════════════
           │            │       │       │            │
           │            │       │       │            │
      ┌────┴────┐  ┌────┴────┐  │  ┌────┴────┐  ┌────┴────┐
      │  FUSE   │  │  FUSE   │  │  │  FUSE   │  │  FUSE   │
      │  60A    │  │  60A    │  │  │  100A   │  │  100A   │
      └────┬────┘  └────┬────┘  │  └────┬────┘  └────┬────┘
           │            │       │       │            │
      ┌────┴────┐  ┌────┴────┐  │  ┌────┴────┐  ┌────┴────┐
      │OzInverter│ │OzInverter│ │  │ BATTERY │  │   TIE   │
      │   #1    │  │   #2    │  │  │  BANK   │  │ INVERTER│
      │  3kW    │  │  3kW    │  │  │  15kWh  │  │   6kW   │
      └────┬────┘  └────┬────┘  │  └────┬────┘  └────┬────┘
           │            │       │       │            │
        220V AC      220V AC    │      BMS         230V AC
           │            │       │       │         (to tie)
           ▼            ▼       │       │            │
       ┌───────┐    ┌───────┐   │       │            │
       │House 1│    │House 2│   │       │            │
       │ Loads │    │ Loads │   │       │            │
       └───────┘    └───────┘   │       │            ▼
                                │       │     ┌─────────────┐
                                │       │     │  AC TIE TO  │
                                │       │     │  CLUSTER B  │
                                │       │     └─────────────┘
                                │       │
    ════════════════════════════╪═══════╪════════════════════
                        48V DC BUS (Return)
    ════════════════════════════╪═══════╪════════════════════
```

### AC Tie Between Clusters

```
    CLUSTER A                                      CLUSTER B
        │                                              │
   ┌────┴────┐                                    ┌────┴────┐
   │   TIE   │                                    │   TIE   │
   │ INVERTER│                                    │ INVERTER│
   │   6kW   │                                    │   6kW   │
   │ (Master)│                                    │ (Slave) │
   └────┬────┘                                    └────┬────┘
        │                                              │
        │    L ─────────────────────────────────── L   │
        ├────N ─────────────────────────────────── N───┤
        │    PE────────────────────────────────── PE   │
        │                                              │
        │              230V AC TIE                     │
        │           (up to 500m)                       │
        │                                              │
        │    ┌────────────────────────────────┐        │
        │    │      SYNC COMMUNICATION        │        │
        │    │                                │        │
        └────┤  RS485 (preferred) or WiFi     ├────────┘
             │                                │
             │  • Frequency reference         │
             │  • Phase angle                 │
             │  • Power setpoint              │
             │  • Status/faults               │
             │                                │
             └────────────────────────────────┘
```

---

## Tie Inverter Options

### Option A: Commercial Bi-Directional (Recommended for Beginners)

| Product | Power | Features | Price |
|---------|-------|----------|-------|
| **Victron MultiPlus-II** | 3-5kW | Grid-forming, parallel, proven | $1,200-1,800 |
| **Growatt SPF 5000ES** | 5kW | Hybrid, affordable | $600-800 |
| **MPP Solar LV6548** | 6.5kW | Popular DIY choice | $700-900 |
| **SMA Sunny Island** | 6kW | Industrial quality | $2,500+ |

### Option B: Build Your Own (Advanced)

Use larger OzInverter with modifications:
- Bi-directional H-bridge (charge + discharge)
- Phase-locked loop (PLL) for sync
- Grid-forming firmware
- **Complexity: High**
- **Recommended only after mastering basic OzInverter**

### Recommendation for Your Project

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│   START WITH:                                                   │
│   ───────────                                                   │
│   • OzInverter for house inverters (you build these)           │
│   • Commercial hybrid for tie inverter (buy this)              │
│                                                                 │
│   Example: Growatt SPF 5000ES (~$700)                          │
│   • 48V input (matches your DC bus)                            │
│   • Grid-forming capable                                       │
│   • Parallel operation supported                               │
│   • Built-in MPPT (bonus)                                      │
│                                                                 │
│   LATER (advanced):                                            │
│   ─────────────────                                            │
│   • Design custom bi-directional tie inverter                  │
│   • Add synchronization to OzInverter                          │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Communication Architecture

### Swarm Network

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        COMMUNICATION TOPOLOGY                               │
│                                                                             │
│                         ┌─────────────────┐                                │
│                         │    CENTRAL      │                                │
│                         │   CONTROLLER    │                                │
│                         │  (Raspberry Pi) │                                │
│                         └────────┬────────┘                                │
│                                  │                                          │
│                            WiFi / Ethernet                                  │
│                                  │                                          │
│           ┌──────────────────────┼──────────────────────┐                  │
│           │                      │                      │                  │
│           │                      │                      │                  │
│    ┌──────┴──────┐        ┌──────┴──────┐        ┌──────┴──────┐          │
│    │  Cluster A  │        │   AC Tie    │        │  Cluster B  │          │
│    │  Controller │        │  Monitor    │        │  Controller │          │
│    │   (ESP32)   │        │   (ESP32)   │        │   (ESP32)   │          │
│    └──────┬──────┘        └─────────────┘        └──────┬──────┘          │
│           │                                             │                  │
│     RS485 Bus                                     RS485 Bus                │
│           │                                             │                  │
│    ┌──────┼──────┐                               ┌──────┼──────┐          │
│    │      │      │                               │      │      │          │
│ ┌──┴──┐┌──┴──┐┌──┴──┐                        ┌──┴──┐┌──┴──┐┌──┴──┐       │
│ │Inv 1││Inv 2││ BMS │                        │Inv 3││Inv 4││ BMS │       │
│ │ESP32││ESP32││     │                        │ESP32││ESP32││     │       │
│ └─────┘└─────┘└─────┘                        └─────┘└─────┘└─────┘       │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Data Flow

| Data | From | To | Frequency |
|------|------|-----|-----------|
| DC bus voltage | Each inverter | Cluster controller | 1 Hz |
| Load power | Each inverter | Cluster controller | 1 Hz |
| Battery SOC | BMS | Cluster controller | 0.1 Hz |
| Temperature | Sensors | Cluster controller | 0.1 Hz |
| Tie power flow | Tie inverter | Central controller | 1 Hz |
| Commands | Central | All units | On demand |
| Alerts | Any unit | Central | Immediate |

---

## Power Flow Scenarios

### Scenario 1: Normal Day (Solar Abundant)

```
┌─────────────────────────────────────────────────────────────────┐
│   CLUSTER A                           CLUSTER B                 │
│                                                                 │
│   Solar: 4kW ☀️                        Solar: 4kW ☀️             │
│   Load:  2kW                           Load:  3kW               │
│   Battery: Charging                    Battery: Charging        │
│                                                                 │
│   Excess: 2kW ──────► AC TIE ──────► Absorbed                  │
│                    (if B needs more)                            │
│                                                                 │
│   Result: Both batteries charge, loads satisfied               │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Scenario 2: Evening (No Solar)

```
┌─────────────────────────────────────────────────────────────────┐
│   CLUSTER A                           CLUSTER B                 │
│                                                                 │
│   Solar: 0kW 🌙                        Solar: 0kW 🌙             │
│   Load:  1.5kW                         Load:  2.5kW             │
│   Battery: 80% SOC                     Battery: 40% SOC         │
│                                                                 │
│   B needs help! ◄────── AC TIE ◄────── A sends 1kW             │
│                                                                 │
│   Result: Load balanced, B battery protected                   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Scenario 3: Grid Available (Future)

```
┌─────────────────────────────────────────────────────────────────┐
│   CLUSTER A                           CLUSTER B                 │
│                                                                 │
│   Solar: 1kW (cloudy)                  Solar: 1kW (cloudy)      │
│   Load:  3kW                           Load:  4kW               │
│   Battery: 20% SOC                     Battery: 25% SOC         │
│                                                                 │
│                      ════ AC TIE ════                           │
│                            │                                    │
│                      ┌─────┴─────┐                              │
│                      │   GRID    │                              │
│                      │  230V AC  │                              │
│                      └───────────┘                              │
│                            │                                    │
│                     Importing 5kW                               │
│                                                                 │
│   Result: Grid supports both clusters during low solar         │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Bill of Materials

### Cluster A (Complete)

| Qty | Component | Specification | Price |
|-----|-----------|---------------|-------|
| **Solar** | | | |
| 8 | Solar panel | 500W mono | $100 ea = $800 |
| 2 | MPPT charger | 48V/60A | $150 ea = $300 |
| 1 | Combiner box | 4-string | $50 |
| **Battery** | | | |
| 1 | LiFePO4 battery | 48V 300Ah (15kWh) | $2,500 |
| 1 | BMS | 100A with comm | $150 |
| **Inverters** | | | |
| 2 | OzInverter | 3kW (your build) | $400 ea = $800 |
| 2 | Smart monitor | ESP32 kit | $40 ea = $80 |
| **Tie** | | | |
| 1 | Hybrid inverter | 5-6kW (Growatt/MPP) | $750 |
| **DC Bus** | | | |
| 1 | Bus bar set | Copper, 200A | $80 |
| 1 | DC breaker box | 4-way | $60 |
| 4 | DC breaker | 63A | $20 ea = $80 |
| 1 | Fuse set | ANL 100A, 60A | $30 |
| **Wiring** | | | |
| 20m | Cable | 35mm² DC | $100 |
| 50m | Cable | 4mm² solar | $50 |
| 1 | Connectors | MC4, lugs | $40 |
| **Enclosure** | | | |
| 1 | Outdoor cabinet | IP65 | $150 |
| 2 | Fan | 120mm 12V | $10 ea = $20 |

**Cluster A Total: ~$6,040**

### Cluster B (Same as A)

**Cluster B Total: ~$6,040**

### AC Tie Infrastructure

| Qty | Component | Specification | Price |
|-----|-----------|---------------|-------|
| 200m | AC cable | 4mm² 3+E | $200 |
| 2 | AC breaker | 32A 2-pole | $40 |
| 2 | Surge protector | Type 2 | $60 |
| 1 | Tie junction box | Weatherproof | $50 |

**AC Tie Total: ~$350**

### Central Controller

| Qty | Component | Specification | Price |
|-----|-----------|---------------|-------|
| 1 | Raspberry Pi 4 | 4GB | $60 |
| 1 | SD Card | 64GB | $15 |
| 1 | Case + PSU | Official | $25 |
| 2 | RS485 module | USB | $10 ea = $20 |
| 1 | UPS | Small 12V | $30 |

**Controller Total: ~$150**

### Communication

| Qty | Component | Specification | Price |
|-----|-----------|---------------|-------|
| 6 | ESP32 | DevKit V1 | $5 ea = $30 |
| 200m | RS485 cable | Twisted pair | $40 |
| 6 | RS485 module | MAX485 | $2 ea = $12 |
| 1 | WiFi router | Outdoor | $50 |

**Communication Total: ~$132**

---

## Complete System BOM Summary

| Component | Price |
|-----------|-------|
| Cluster A | $6,040 |
| Cluster B | $6,040 |
| AC Tie | $350 |
| Central Controller | $150 |
| Communication | $132 |
| **Installation materials** | $300 |
| **Contingency (10%)** | $1,300 |

**GRAND TOTAL: ~$14,312**

### Per Household Cost

```
$14,312 ÷ 4 households = $3,578 per household

Includes:
• 2kW solar capacity
• 7.5kWh battery share
• 3kW inverter
• Smart monitoring
• Community tie benefits
```

### Comparison

| System | Cost | Per Household |
|--------|------|---------------|
| Your hybrid microgrid | $14,312 | $3,578 |
| 4× Individual Victron systems | $40,000+ | $10,000+ |
| Grid extension (rural Africa) | $20,000-50,000+ | Varies |

**Savings: 60-75% vs commercial alternatives!**

---

## Build Sequence

### Phase 1: Single Cluster (Month 1-3)

```
Week 1-2:   Order components
Week 3-4:   Build 2× OzInverters (you already know how)
Week 5-6:   Install solar + MPPT
Week 7-8:   Install battery + BMS
Week 9-10:  Wire DC bus
Week 11-12: Commission Cluster A
```

### Phase 2: Smart Integration (Month 4)

```
Week 1:  Install ESP32 monitors
Week 2:  Configure ESPHome
Week 3:  Set up Home Assistant
Week 4:  Test monitoring + alerts
```

### Phase 3: Second Cluster (Month 5-6)

```
Replicate Cluster A process for Cluster B
```

### Phase 4: AC Tie (Month 7)

```
Week 1:  Install tie cable
Week 2:  Configure tie inverters
Week 3:  Test synchronization
Week 4:  Commission full system
```

### Phase 5: Optimization (Month 8+)

```
• Fine-tune power sharing
• Add load shedding automation
• Monitor and improve
• Document lessons learned
```

---

## Software Configuration

### Central Controller (Home Assistant)

```yaml
# configuration.yaml

homeassistant:
  name: Microgrid Central
  unit_system: metric

# ESPHome integration (auto-discovered)
esphome:

# Energy dashboard
energy:

# Automations for power sharing
automation:
  # Share power when cluster needs help
  - alias: "AC Tie - Help Low Cluster"
    trigger:
      - platform: numeric_state
        entity_id: sensor.cluster_b_battery_soc
        below: 30
    condition:
      - condition: numeric_state
        entity_id: sensor.cluster_a_battery_soc
        above: 60
    action:
      - service: number.set_value
        target:
          entity_id: number.tie_inverter_export_power
        data:
          value: 2000  # Export 2kW from A to B

  # Load shedding when both clusters low
  - alias: "Emergency Load Shed"
    trigger:
      - platform: template
        value_template: >
          {{ states('sensor.cluster_a_battery_soc')|float < 20 and
             states('sensor.cluster_b_battery_soc')|float < 20 }}
    action:
      - service: switch.turn_off
        target:
          entity_id:
            - switch.house1_non_critical
            - switch.house2_non_critical
            - switch.house3_non_critical
            - switch.house4_non_critical
      - service: notify.all_users
        data:
          message: "⚠️ Emergency load shedding activated - batteries critical"
```

### ESPHome Cluster Controller

```yaml
# cluster_a_controller.yaml

esphome:
  name: cluster-a
  platform: ESP32
  board: esp32dev

wifi:
  ssid: "MicrogridNet"
  password: !secret wifi_password

api:
mqtt:
  broker: 192.168.1.100
  topic_prefix: microgrid/cluster_a

uart:
  - id: rs485_bus
    tx_pin: GPIO17
    rx_pin: GPIO16
    baud_rate: 9600

modbus:
  - id: modbus_hub
    uart_id: rs485_bus

sensor:
  # Read from BMS via Modbus
  - platform: modbus_controller
    modbus_controller_id: bms
    name: "Cluster A Battery SOC"
    address: 0x0100
    register_type: holding
    value_type: U_WORD
    unit_of_measurement: "%"

  # Read from inverters
  - platform: modbus_controller
    modbus_controller_id: inv1
    name: "Inverter 1 Power"
    address: 0x0010
    register_type: holding
    value_type: U_WORD
    unit_of_measurement: "W"

  # Aggregate cluster power
  - platform: template
    name: "Cluster A Total Load"
    lambda: |-
      return id(inv1_power).state + id(inv2_power).state;
    unit_of_measurement: "W"
    update_interval: 5s

# Publish to central for coordination
interval:
  - interval: 10s
    then:
      - mqtt.publish:
          topic: "microgrid/cluster_a/status"
          payload: !lambda |-
            char buf[128];
            snprintf(buf, sizeof(buf),
              "{\"soc\":%.1f,\"load\":%.0f,\"solar\":%.0f}",
              id(battery_soc).state,
              id(total_load).state,
              id(solar_power).state);
            return std::string(buf);
```

---

## Safety & Protection

### Electrical Safety

```
┌─────────────────────────────────────────────────────────────────┐
│                    PROTECTION LAYERS                            │
│                                                                 │
│   LAYER 1: Component Level                                     │
│   ├── MOSFET gate protection (zeners)                          │
│   ├── Snubber circuits                                         │
│   └── Thermal fuses on heatsinks                               │
│                                                                 │
│   LAYER 2: Inverter Level                                      │
│   ├── Overcurrent shutdown (OCP)                               │
│   ├── Overvoltage shutdown (OVP)                               │
│   ├── Overtemperature shutdown (OTP)                           │
│   └── Short circuit protection                                 │
│                                                                 │
│   LAYER 3: DC Bus Level                                        │
│   ├── Main fuse (ANL 150A)                                     │
│   ├── Branch breakers (63A each)                               │
│   └── Battery BMS disconnect                                   │
│                                                                 │
│   LAYER 4: AC Tie Level                                        │
│   ├── AC breakers (32A)                                        │
│   ├── Surge protection                                         │
│   ├── Ground fault detection                                   │
│   └── Anti-islanding (if grid connected)                       │
│                                                                 │
│   LAYER 5: System Level                                        │
│   ├── Emergency stop button                                    │
│   ├── Smoke/fire detection                                     │
│   └── Remote shutdown capability                               │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Grounding

```
                     ┌─────────────────┐
                     │  GROUND ROD     │
                     │  (Earth stake)  │
                     └────────┬────────┘
                              │
          ┌───────────────────┼───────────────────┐
          │                   │                   │
    ┌─────┴─────┐       ┌─────┴─────┐       ┌─────┴─────┐
    │ Cluster A │       │  AC Tie   │       │ Cluster B │
    │  Cabinet  │       │  Junction │       │  Cabinet  │
    │  Ground   │       │   Ground  │       │  Ground   │
    └───────────┘       └───────────┘       └───────────┘
          │                   │                   │
          │     ════ GROUND BUS (buried) ════    │
          │                   │                   │
          └───────────────────┴───────────────────┘
```

---

## Expansion Possibilities

### Future Upgrades

| Upgrade | Benefit | Complexity |
|---------|---------|------------|
| **Add Cluster C, D** | More households | Medium |
| **Grid connection** | Backup, sell excess | Medium |
| **EV charging** | Electric vehicle support | Low |
| **Water pumping** | Shared DC pump on bus | Low |
| **Community lighting** | DC street lights | Low |
| **Productive use** | Milling, refrigeration | Medium |

### Scaling Path

```
4 Units (Current)
      │
      ▼
8 Units (Add 2 clusters)
      │
      ▼
16 Units (Add AC backbone)
      │
      ▼
Village scale (Multiple backbones + grid tie)
```

---

## Learning Outcomes

After completing Step 7:

- [ ] Understand DC vs AC coupling tradeoffs
- [ ] Can design hybrid microgrid architecture
- [ ] Know how to size system for community
- [ ] Can configure inter-cluster communication
- [ ] Understand power flow management
- [ ] Can implement load shedding automation
- [ ] Know safety requirements for multi-unit systems
- [ ] Ready to scale to larger communities

---

## Resources

### Documentation
- OzInverter Forum: Community support
- Home Assistant: https://home-assistant.io
- ESPHome: https://esphome.io
- Victron documentation (for tie inverter)

### Research Papers
- "Scalable DC Microgrid Architecture" - UC Berkeley
- "DC Microgrid for Rural Africa" - MDPI Energies
- "Swarm Electrification" - Various authors

### Communities
- DIY Solar Forum: diysolarforum.com
- Endless Sphere: endless-sphere.com
- r/SolarDIY on Reddit

---

*This module transforms your single inverter knowledge into community-scale impact. Power for 4 families, built with your own hands, using skills developed through this curriculum.*
