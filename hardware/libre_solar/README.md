# Libre Solar Open Source Hardware

100% open source solar charge controllers and battery management systems.

**Source:** [LibreSolar GitHub](https://github.com/LibreSolar) (Various open licenses)

---

## Contents

| Folder | Type | Description |
|--------|------|-------------|
| `bms-c1/` | Hardware | 16S/100A Battery Management System |
| `mppt-2420-hc/` | Hardware | 20A MPPT charge controller with CAN |
| `bms-firmware/` | Firmware | BMS firmware (Zephyr RTOS) |
| `charge-controller-firmware/` | Firmware | MPPT/PWM controller firmware |
| `esp32-edge-firmware/` | Firmware | CAN/UART to WiFi/BLE gateway |

---

## Hardware Designs

### `bms-c1/` - Battery Management System

**Specs:**
- 3-16S cells (LiFePO4 or Li-ion)
- 100A continuous current
- CAN bus communication (ThingSet protocol)
- Cell balancing
- Temperature monitoring
- KiCad design files

**Use Case:** 48V (16S) LiFePO4 battery pack for microgrid

---

### `mppt-2420-hc/` - MPPT Charge Controller

**Specs:**
- 20A output current
- 12V/24V battery (boost to 48V with DC-DC)
- Maximum Power Point Tracking
- High-side load switch
- CAN bus interface
- KiCad design files

**Use Case:** Solar panel charging for off-grid system

---

## Firmware

### `bms-firmware/`

Zephyr RTOS-based firmware supporting:
- bq769x0, bq769x2, ISL94202 BMS ICs
- CAN bus (ThingSet protocol)
- UART interface
- Cell balancing algorithms
- Safety protections

### `charge-controller-firmware/`

Zephyr RTOS-based firmware for:
- MPPT algorithm
- PWM charging
- Load management
- CAN bus communication
- Data logging

### `esp32-edge-firmware/`

Gateway firmware for:
- CAN bus to WiFi bridge
- MQTT publishing
- Home Assistant integration
- Web interface
- BLE connectivity

---

## Integration with Curriculum

| Curriculum Section | Libre Solar Component |
|-------------------|----------------------|
| Smart Upgrade Module | `esp32-edge-firmware` |
| Microgrid (Step 7) | `bms-c1` + `mppt-2420-hc` |
| Libre Solar Module | All components |

---

## Resources

- [Libre Solar Documentation](https://libre.solar)
- [Learn DC Energy Systems](https://learn.libre.solar)
- [ThingSet Protocol](https://thingset.io)

---

*License: Apache 2.0 / CC-BY-SA (see individual repos)*
