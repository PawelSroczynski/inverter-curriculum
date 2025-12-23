# OzInverter PCB Designs & Supporting Systems

Complete KiCad PCB designs, Gerber files, and Arduino code for building an OzInverter-based off-grid power system.

**Source:** [jdevine82 GitHub repositories](https://github.com/jdevine82) (Public Domain)

---

## Repository Contents

| Folder | Type | Description |
|--------|------|-------------|
| `original/` | PCB | Original OzInverter main board |
| `modded/` | PCB | Modified OzInverter with SMPS power supply |
| `controller/` | PCB | Inverter controller board (EG8010-based) |
| `output_stage/` | PCB | Output filter and relay board |
| `battery_monitor/` | Arduino | Modbus battery cell monitoring system |
| `powerhouse_controller/` | Arduino | Central power system controller |

---

## PCB Designs

### `original/` - OzInverter Main Board

The original OzInverter PCB design - a 6kW+ pure sine wave inverter.

**Key Files:**
| File | Purpose |
|------|---------|
| `ozinverterkicad.sch` | Schematic (KiCad) |
| `ozinverterkicad.kicad_pcb` | PCB layout |
| `ozinverterkicad.pdf` | Schematic PDF for reference |
| `output.pdf` | PCB output documentation |
| `ozinverterlib.lib` | Custom component library |
| `ozinverterfootprints.pretty/` | Custom footprints |

**Gerber Files (for PCB fabrication):**
| File | Layer |
|------|-------|
| `ozinverterkicad-F.Cu.gtl` | Front copper |
| `ozinverterkicad-B.Cu.gbl` | Back copper |
| `ozinverterkicad-F.Mask.gts` | Front solder mask |
| `ozinverterkicad-B.Mask.gbs` | Back solder mask |
| `ozinverterkicad-F.SilkS.gto` | Front silkscreen |
| `ozinverterkicad-B.SilkS.gbo` | Back silkscreen |
| `ozinverterkicad.drl` | Drill file |

---

### `modded/` - OzInverter with SMPS Power Supply

Enhanced version with integrated switching power supply for control circuits.

**Improvements over original:**
- Built-in SMPS for 12V/5V control power
- More compact design
- Improved thermal layout
- Additional protection features

**Additional Files:**
| File | Purpose |
|------|---------|
| `voltagemodule.lib` | SMPS voltage regulator module |
| `ozinverterkicad modded.zip` | Complete design archive |
| `README.md` | Original modification notes |

**Gerber Files:** Same naming convention as original, includes both `.gbr` (newer format) and legacy extensions.

---

### `controller/` - Inverter Controller Board

Separate controller PCB for the EG8010 SPWM driver and related control electronics.

**Key Files:**
| File | Purpose |
|------|---------|
| `invertercontroller.sch` | Controller schematic |
| `invertercontroller.kicad_pcb` | Controller PCB |
| `invertercontroller.pdf` | Schematic PDF (41 pages) |

**Features:**
- EG8010 SPWM controller interface
- IR2110 gate driver connections
- Voltage feedback circuits
- Protection input/outputs

---

### `output_stage/` - Output Filter Board

Separate PCB for LC filter, output relay, and metering connections.

**Key Files:**
| File | Purpose |
|------|---------|
| `inverteroutputboard.sch` | Output stage schematic |
| `inverteroutputboard.kicad_pcb` | Output stage PCB |
| `output.pdf` | Documentation |

**Features:**
- LC filter mounting
- Output relay control
- AC voltage sensing
- Current transformer interface

---

## Arduino Projects

### `battery_monitor/` - Modbus Battery Monitor

Arduino Mega-based battery monitoring system using Modbus RTU protocol.

**Files:**
| File | Purpose |
|------|---------|
| `modbusmasterv2.ino.ino` | Master controller (Mega) |
| `modbusslave.ino/` | Slave node firmware |

**Features:**
- 12-cell voltage monitoring
- Individual cell alarms (high/low)
- Dump load control per cell
- LCD display
- Modbus RTU communication
- Optoisolated serial bus

**Hardware Required:**
- Arduino Mega (master)
- Arduino (any) per slave node
- LCD display
- RS485 or optoisolated bus

---

### `powerhouse_controller/` - Central Power Controller

Complete off-grid power system controller with multi-source management.

**Files:**
| File | Purpose |
|------|---------|
| `Jasonspowerhousecontroller.ino` | Main controller firmware |
| `data/` | Web interface files |

**Features:**
- Multi-source management:
  - Solar input monitoring (INA226)
  - Wind turbine control
  - Generator auto-start
  - Grid-tie inverter control
- Dual inverter support (main + backup)
- Battery voltage monitoring
- Line voltage monitoring (240V)
- SD card data logging
- ESP8266 WiFi + MQTT
- LCD display with 3-button interface
- Storm protection (wind turbine shutdown)
- Dump load control

**Hardware Required:**
- Arduino
- 4× INA226 current/voltage sensors
- 75mV shunts
- ESP8266 module
- SD card logger with RTC
- LCD display
- CAN bus module (optional)

---

## How to Use These Files

### PCB Fabrication

1. **Open in KiCad** (v4.x or later):
   ```
   Open the .pro file in KiCad
   Review schematic (.sch)
   Review PCB layout (.kicad_pcb)
   ```

2. **Order PCBs:**
   - Use the Gerber files directly with JLCPCB, PCBWay, or similar
   - Files are production-ready
   - Board size: ~200mm × 150mm (main board)

3. **Required Files for Fabrication:**
   ```
   *-F.Cu.gtl      Front copper
   *-B.Cu.gbl      Back copper
   *-F.Mask.gts    Front solder mask
   *-B.Mask.gbs    Back solder mask
   *-F.SilkS.gto   Front silkscreen
   *.drl           Drill file
   *-Edge.Cuts.*   Board outline (if present)
   ```

### Arduino Projects

1. Install required libraries:
   - `SimpleModbusMaster` (battery monitor)
   - `INA226` by Korneliusz Jarzębski
   - `espduino` for ESP8266 control

2. Upload to appropriate Arduino boards

---

## Integration with OzInverter Curriculum

These designs support the curriculum in:

| Curriculum Step | Relevant Files |
|-----------------|----------------|
| Step 5 (OzInverter Build) | `original/`, `modded/` |
| Smart Upgrade Module | `controller/`, `powerhouse_controller/` |
| Microgrid (Step 7) | `battery_monitor/`, `powerhouse_controller/` |

---

## Bill of Materials

See the schematic PDFs in each folder for component values. Key components:

**Main Inverter Board:**
- 8-16× IRFP4668 MOSFETs
- 4× IR2110 gate drivers
- EG8010/EGS002 controller
- 5kVA+ toroidal transformer

**Controller Board:**
- EG8010 SPWM IC
- IR2110 gate drivers
- LM7812/LM7805 regulators
- Optocouplers

---

## License

**Public Domain** - These designs are released to the public domain by the original author.

---

## Resources

- [OzInverter Official](https://www.bryanhorology.com/ozinverter.php)
- [Original GitHub: jdevine82](https://github.com/jdevine82)
- [EG8010 Datasheet](https://www.google.com/search?q=EG8010+datasheet)
- [IR2110 Application Notes](https://www.infineon.com/dgdl/ir2110.pdf)

---

*Added to curriculum: December 2024*
