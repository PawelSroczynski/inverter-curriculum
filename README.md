# Inverter Building Curriculum

**From LED Blinker to 6-15kW Pure Sine Wave Inverter & Community Microgrid**

A complete, open-source electronics curriculum that takes you from zero knowledge to building production-quality power inverters and community-scale microgrids.

---

## Repository Structure

```
inverter-curriculum/
├── README.md                    # You are here
│
├── curriculum/                  # Learning materials
│   ├── index.md                 # Master overview & quick reference
│   ├── part1_fundamentals.md    # Step 1: 12 basic projects (4 phases)
│   ├── part2_modified_sine.md   # Steps 2-3: 150W → 500W inverters
│   ├── part3_pure_sine.md       # Steps 4-5: 1kW → 6-15kW OzInverter
│   ├── upgrade_smart.md         # Step 6: ESP32 monitoring
│   ├── microgrid.md             # Step 7: 4-household DC/AC system
│   └── libre_solar.md           # Bonus: Open source MPPT & BMS
│
└── hardware/                    # KiCad designs & firmware
    ├── ozinverter/              # OzInverter PCB designs (jdevine82)
    │   ├── original/            # Base 6kW+ main board
    │   ├── modded/              # SMPS power supply version
    │   ├── controller/          # EG8010 controller board
    │   ├── output_stage/        # LC filter & relay board
    │   ├── battery_monitor/     # Arduino Modbus BMS
    │   └── powerhouse_controller/  # Central system controller
    │
    └── libre_solar/             # Libre Solar open hardware
        ├── bms-c1/              # 16S/100A BMS (KiCad)
        ├── mppt-2420-hc/        # 20A MPPT controller (KiCad)
        ├── bms-firmware/        # BMS Zephyr firmware
        ├── charge-controller-firmware/  # MPPT firmware
        └── esp32-edge-firmware/ # CAN→WiFi gateway
```

---

## Learning Path

```
You are here                                                    Your goal
     │                                                              │
     ▼                                                              ▼
┌─────────┐   ┌─────────┐   ┌─────────┐   ┌─────────┐   ┌─────────────────┐
│   LED   │──▶│  555    │──▶│ 150W    │──▶│  6kW    │──▶│   Community     │
│ Blinker │   │ Timer   │   │Inverter │   │OzInverter│  │   Microgrid     │
└─────────┘   └─────────┘   └─────────┘   └─────────┘   └─────────────────┘
   Step 1        Step 1       Step 2        Step 5          Step 7
  (basics)      (timing)    (first inv)   (full power)    (4 households)
```

---

## What You'll Build

| Step | Project | Power | Skills |
|------|---------|-------|--------|
| 1 | 12 fundamental projects | — | Soldering, Ohm's Law, transistors, 555, PWM |
| 2 | CD4047 Square Wave | 150W | MOSFETs, transformers, push-pull |
| 3 | SG3525 Modified Sine | 500W | H-bridge, gate drivers, dead-time |
| 4 | EG8010 Pure Sine | 1kW | SPWM, LC filters, voltage feedback |
| 5 | OzInverter | 6-15kW | Toroid winding, thermal, high power |
| 6 | Smart Upgrade | — | ESP32, Home Assistant, monitoring |
| 7 | Community Microgrid | 12kW | DC bus, AC tie, swarm architecture |

---

## Investment

| Path | Time | Cost |
|------|------|------|
| Single 6kW Smart Inverter (Steps 1-6) | 6-12 months | ~$1,139 |
| Community Microgrid (Steps 1-7) | 12-24 months | ~$14,300 ($3,575/household) |

---

## vs Commercial

| Feature | Victron 5kW | Your OzInverter |
|---------|-------------|-----------------|
| Power | 5kW | 6-15kW |
| Monitoring | VRM app | Home Assistant |
| **Price** | **$2,200** | **$775** |
| Repairable | ❌ | ✅ |
| Learning | None | Immense |

---

## Getting Started

1. **Read** → `curriculum/index.md`
2. **Build** → Start with Step 1 fundamentals
3. **Hardware** → Use designs in `hardware/` folder

---

## Safety

⚠️ **This curriculum involves dangerous voltages and currents.**

- 48V DC at high current can cause fires
- 220V AC can kill
- Never work on live circuits
- Keep fire extinguisher nearby

---

## Resources

| Resource | URL |
|----------|-----|
| OzInverter Official | bryanhorology.com/ozinverter.php |
| Libre Solar | libre.solar |
| ESPHome | esphome.io |
| Home Assistant | home-assistant.io |

---

## License

Open source. Use for personal learning, teaching, community building.

---

<p align="center"><b>From LED blinker to powering communities.</b></p>
