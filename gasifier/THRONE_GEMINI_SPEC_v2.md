# THRONE-GEMINI: Dual-Phase Biomass Reactor Control System

**Version:** 2.0 (Revised)
**Date:** 2025-12-28
**Status:** Engineering Specification

---

## 1. SYSTEM OVERVIEW

### 1.1 Purpose

THRONE-GEMINI is an autonomous Waste-to-Energy subsystem for NODE-ZERO. It performs two-stage thermochemical conversion:

| Phase | Process | Output | Duration |
|-------|---------|--------|----------|
| **Phase 1** | Pyrolysis (TLUD) | Heat (CWU) + Biochar | 2-4 hours |
| **Phase 2** | Gasification | Syngas (CO + H₂) for engine | 1-2 hours |

### 1.2 Physical Architecture

```
┌─────────────────────────────────────────────────────────────────────────┐
│                      STATIONARY DOCKING HEAD                            │
│  ┌───────────────────────────────────────────────────────────────────┐  │
│  │                                                                   │  │
│  │   CHIMNEY ◄──[VALVE_CHIMNEY]     AFTERBURNER CHAMBER             │  │
│  │   (Natural draft exhaust)         ┌─────────────────────┐        │  │
│  │                                   │  Secondary Combustion │        │  │
│  │   TO ENGINE ◄──[VALVE_ENGINE]◄────│  [TEMP_HEAD] ●       │        │  │
│  │   (Syngas output)                 │                      │        │  │
│  │                                   │  [VALVE_AIR_SEC] ◄───│── Air  │  │
│  │                                   └──────────┬───────────┘        │  │
│  │                                              │                    │  │
│  │                              ════════════════╧════════════════    │  │
│  │                              ║     CERAMIC ROPE SEAL        ║    │  │
│  │                              ║  [DOCK_LIMIT_SWITCH] ●       ║    │  │
│  └──────────────────────────────╨────────────────────────────────╨──┘  │
│                                 │          LIFT & LOCK          │      │
│  ┌──────────────────────────────┴────────────────────────────────┴──┐  │
│  │                     MOBILE REACTOR VESSEL                        │  │
│  │                    (Octagonal Steel Container)                   │  │
│  │  ┌─────────────────────────────────────────────────────────────┐ │  │
│  │  │                                                             │ │  │
│  │  │     ░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░              │ │  │
│  │  │     ░░░░░░░░░░  BIOMASS / BIOCHAR  ░░░░░░░░░░              │ │  │
│  │  │     ░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░              │ │  │
│  │  │                                                             │ │  │
│  │  │  [VALVE_SIDE_AIR] ──► ●────────────● ◄── [WATER_INJECTION] │ │  │
│  │  │       (Tuyeres)       │  REDUCTION  │        (PWM)         │ │  │
│  │  │       (PWM Servo)     │    ZONE     │                      │ │  │
│  │  │                       └──────┬──────┘                      │ │  │
│  │  │                              │                              │ │  │
│  │  │                    ══════════╧══════════                   │ │  │
│  │  │                    ║   GRATE / ASH    ║                    │ │  │
│  │  │                    ║  [TEMP_BOTTOM] ● ║                    │ │  │
│  │  │                    ══════════╤════════                     │ │  │
│  │  │                              │                              │ │  │
│  │  │              [DAMPER_PRIMARY_AIR] ◄── Air (Draft)          │ │  │
│  │  │                    (PWM Servo)                             │ │  │
│  │  └─────────────────────────────────────────────────────────────┘ │  │
│  └──────────────────────────────────────────────────────────────────┘  │
│                              ║ TURNTABLE ║                             │
│                              ║ (Revolver)║                             │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 2. STATE MACHINE (Revised)

### 2.1 State Diagram

```
                            ┌──────────────────┐
                            │                  │
              ┌─────────────│   STATE 0: IDLE  │◄────────────────────────┐
              │             │    / DOCKING     │                         │
              │             │                  │                         │
              │             └────────┬─────────┘                         │
              │                      │                                   │
              │    Conditions met:   │                                   │
              │    • dock_switch=ON  │                                   │
              │    • temp_head<100°C │                                   │
              │    • user_cmd=START  │                                   │
              │                      ▼                                   │
              │             ┌──────────────────┐                         │
              │             │                  │                         │
              │             │  STATE 1: PYRO   │                         │
   EMERGENCY  │             │   (TLUD Mode)    │                         │
     STOP     │             │                  │                         │
              │             └────────┬─────────┘                         │
              │                      │                                   │
              │    Triggers (ALL):   │                                   │
              │    • temp_head<500°C │                                   │
              │    • temp_head_rate  │                                   │
              │      < -5°C/min      │                                   │
              │    • temp_bottom     │                                   │
              │      > 400°C         │                                   │
              │                      ▼                                   │
              │             ┌──────────────────┐                         │
              │             │                  │                         │
              │             │ STATE 2: TRANS   │                         │
              │             │  (Switchover)    │                         │
              │             │                  │                         │
              │             └────────┬─────────┘                         │
              │                      │                                   │
              │    Conditions:       │                                   │
              │    • sequence done   │                                   │
              │    • pressure test OK│                                   │
              │    • engine_running  │                                   │
              │                      ▼                                   │
              │             ┌──────────────────┐                         │
              │             │                  │                         │
              └────────────►│  STATE 3: GASIF  │─────────────────────────┘
                            │  (Power Mode)    │   Triggers:
                            │                  │   • temp_bottom<500°C
                            └──────────────────┘   • char_exhausted
                                                   • user_cmd=STOP
                                                   • engine_timeout
```

### 2.2 State Definitions

#### STATE 0: IDLE / DOCKING

**Purpose:** Safe state for reactor swap or system standby.

| Output | State | Notes |
|--------|-------|-------|
| valve_chimney | CLOSED | - |
| valve_engine | CLOSED | - |
| damper_primary_air | CLOSED | - |
| valve_air_secondary | CLOSED | - |
| valve_side_air | CLOSED | - |
| water_injection | OFF | - |

**Entry Conditions:** Default state on boot, or after shutdown.

**Exit Conditions (to STATE 1):**
- `dock_limit_switch == TRUE`
- `temp_head < 100°C` (cold start only)
- `user_command == START`

---

#### STATE 1: PYROLYSIS (TLUD Mode)

**Purpose:** Generate heat for domestic hot water, convert biomass to biochar.

**Physics:** Top-Lit Up-Draft - fire burns downward through fuel bed. Volatiles combust in secondary chamber (afterburner).

| Output | State | Control |
|--------|-------|---------|
| valve_chimney | OPEN | Full open |
| valve_engine | CLOSED | - |
| damper_primary_air | OPEN | 20-40% (PWM) |
| valve_air_secondary | OPEN | Full open |
| valve_side_air | CLOSED | - |
| water_injection | OFF | - |

**Control Logic:**
```
damper_primary_air = PID(
    setpoint = 750°C,
    input = temp_head,
    output_min = 10%,
    output_max = 50%
)
```

**Exit Conditions (to STATE 2):**

All three must be TRUE:
1. `temp_head < 500°C` (volatiles exhausted)
2. `temp_head_rate < -5°C/min` (confirmed falling, not fluctuation)
3. `temp_bottom > 400°C` (fire reached grate level)

**Safety Cutoffs:**
- `temp_bottom > 600°C` → WARNING (early burnout)
- `temp_head > 1000°C` → Reduce damper_primary_air
- `dock_limit_switch == FALSE` → EMERGENCY STOP

---

#### STATE 2: TRANSITION (Switchover)

**Purpose:** Safely switch from pyrolysis to gasification without extinguishing char bed.

**Duration:** 15-30 seconds (sequenced)

**Sequence (CRITICAL ORDER):**

```
Step 1 (t=0s):
    damper_primary_air = 0%      # Stop bottom air

Step 2 (t=2s):
    valve_air_secondary = CLOSED  # Stop head air

Step 3 (t=5s):
    # Reactor now in "smolder" mode
    # Char retains heat, minimal gas production

Step 4 (t=8s):
    # PRESSURE TEST - verify seal integrity
    valve_engine = OPEN (briefly, 2 sec)
    valve_engine = CLOSED
    monitor pressure for 5 seconds
    IF pressure_rise > threshold:
        → ABORT (seal leak detected)
        → GOTO STATE 0

Step 5 (t=15s):
    # Confirmed: seal is good
    # Start engine (external command or automatic)
    WAIT for engine_running == TRUE (timeout: 30s)
    IF timeout:
        → ABORT
        → GOTO STATE 0

Step 6 (t=20s):
    valve_engine = OPEN           # Connect to engine suction

Step 7 (t=22s):
    valve_chimney = CLOSED        # Redirect all gas to engine

Step 8 (t=25s):
    → GOTO STATE 3
```

**Safety:** Always maintain exhaust path until engine suction confirmed.

---

#### STATE 3: GASIFICATION (Power Mode)

**Purpose:** Convert glowing biochar to syngas (CO + H₂) for engine fuel.

**Physics:**
- Engine suction creates negative pressure (-5 to -20 kPa)
- Air drawn through side tuyeres (controlled)
- Water injection produces H₂ via water-gas reaction: `C + H₂O → CO + H₂`

| Output | State | Control |
|--------|-------|---------|
| valve_chimney | CLOSED | - |
| valve_engine | OPEN | - |
| damper_primary_air | CLOSED | - |
| valve_air_secondary | CLOSED | **CRITICAL: Must stay closed** |
| valve_side_air | OPEN | PWM 30-80% |
| water_injection | PWM | 0-100% based on temp |

**Control Logic - Air:**
```
valve_side_air = PID(
    setpoint = 850°C,
    input = temp_bottom,
    output_min = 20%,
    output_max = 90%,
    direction = DIRECT  # More air = higher temp
)
```

**Control Logic - Water:**
```
IF temp_bottom < 700°C:
    water_injection = 0%  # Too cold, water won't react

ELIF temp_bottom >= 700°C AND temp_bottom < 850°C:
    water_injection = map(engine_load, 0-100%) * 0.5

ELIF temp_bottom >= 850°C AND temp_bottom < 1000°C:
    water_injection = map(engine_load, 0-100%) * 1.0

ELIF temp_bottom >= 1000°C:
    water_injection = map(engine_load, 0-100%) * 1.5  # Cool down
```

**Exit Conditions (to STATE 0):**
- `temp_bottom < 500°C` for >60 seconds (char exhausted)
- `user_command == STOP`
- `engine_running == FALSE` for >30 seconds (engine stopped)
- EMERGENCY STOP triggered

---

## 3. SAFETY INTERLOCKS

### 3.1 Critical Interlocks (Always Active)

| ID | Rule | Action |
|----|------|--------|
| S1 | `valve_engine == OPEN` → `valve_air_secondary = CLOSED` | Prevent fresh air in gas line |
| S2 | `dock_limit_switch == FALSE` → ALL VALVES CLOSED | Seal lost = safe state |
| S3 | `temp_bottom > 1100°C` | EMERGENCY STOP (clinker risk) |
| S4 | `temp_head > 1100°C` | EMERGENCY STOP (structural damage) |

### 3.2 State-Dependent Interlocks

| State | Rule | Action |
|-------|------|--------|
| PYRO | `temp_bottom > 600°C` | WARNING: Early burnout |
| PYRO | `temp_head < 200°C` for >5min | WARNING: Fire dying |
| GASIF | `temp_bottom < 600°C` | WARNING: Gasification failing |
| GASIF | `temp_head < 300°C` | WARNING: Tar condensation risk |
| GASIF | `engine_running == FALSE` for >30s | Close valve_engine, GOTO IDLE |

### 3.3 Startup Interlocks

| Rule | Reason |
|------|--------|
| `temp_head < 100°C` required for START | Prevent igniting hot reactor |
| `temp_bottom < 100°C` required for START | Confirm cold start |
| `dock_limit_switch == TRUE` | Seal confirmed |
| `pressure_test == PASS` before GASIF | Seal integrity verified |

### 3.4 Emergency Stop Procedure

```
EMERGENCY_STOP:
    1. valve_engine = CLOSED       # Isolate engine
    2. valve_side_air = CLOSED     # Stop gasification air
    3. damper_primary_air = CLOSED # Stop primary air
    4. valve_air_secondary = CLOSED
    5. valve_chimney = OPEN        # Vent any pressure
    6. water_injection = OFF
    7. state = IDLE
    8. alarm = ACTIVE
    9. REQUIRE manual reset to restart
```

---

## 4. I/O MAPPING

### 4.1 Inputs

| Pin | ID | Type | Range | Purpose |
|-----|----|------|-------|---------|
| ADC1 | temp_head | Thermocouple K + MAX31855 | 0-1200°C | Afterburner temperature |
| ADC2 | temp_bottom | Thermocouple K + MAX31855 | 0-1200°C | Grate/gasification zone temp |
| GPIO | dock_limit_switch | Binary (NO) | 0/1 | Reactor docked and sealed |
| GPIO | engine_running | Binary (AC detect) | 0/1 | Engine/suction active |
| ADC3 | pressure_reactor | Differential pressure | -50 to +10 kPa | Vacuum monitoring (optional) |

### 4.2 Outputs

| Pin | ID | Type | Range | Purpose |
|-----|----|------|-------|---------|
| RELAY1 | valve_chimney | Solenoid/Servo | ON/OFF | Main exhaust to chimney |
| RELAY2 | valve_engine | Solenoid | ON/OFF | Gas line to engine |
| PWM1 | damper_primary_air | Servo | 0-100% | Bottom air intake control |
| RELAY3 | valve_air_secondary | Solenoid | ON/OFF | Afterburner air injection |
| PWM2 | valve_side_air | Servo | 0-100% | Tuyere air (gasification) |
| PWM3 | water_injection | MOSFET + solenoid | 0-100% | Water drip rate |

### 4.3 Derived Values (Calculated)

| ID | Formula | Purpose |
|----|---------|---------|
| temp_head_rate | `(temp_head[t] - temp_head[t-60]) / 60` | °C/min rate of change |
| temp_bottom_rate | `(temp_bottom[t] - temp_bottom[t-60]) / 60` | °C/min rate of change |
| gasification_efficiency | `f(temp_bottom, water_rate, air_rate)` | Process quality indicator |

---

## 5. CONTROL PARAMETERS

### 5.1 Temperature Setpoints

| Parameter | Value | Notes |
|-----------|-------|-------|
| PYRO_HEAD_TARGET | 750°C | Secondary combustion optimal |
| PYRO_HEAD_MIN | 400°C | Below this = fire dying |
| PYRO_HEAD_MAX | 950°C | Above this = reduce air |
| GASIF_BOTTOM_TARGET | 850°C | Optimal gasification temp |
| GASIF_BOTTOM_MIN | 700°C | Below = water-gas reaction stops |
| GASIF_BOTTOM_MAX | 1050°C | Above = clinker/slag risk |
| TRANSITION_TRIGGER | 500°C | Head temp to trigger switchover |

### 5.2 Timing Parameters

| Parameter | Value | Notes |
|-----------|-------|-------|
| PYRO_MIN_DURATION | 30 min | Minimum pyrolysis time |
| TRANSITION_DURATION | 25 sec | Total switchover sequence |
| ENGINE_START_TIMEOUT | 30 sec | Max wait for engine |
| ENGINE_STOP_TIMEOUT | 30 sec | Max engine-off before abort |
| TEMP_SAMPLE_INTERVAL | 1 sec | Sensor polling rate |
| RATE_CALC_WINDOW | 60 sec | Window for rate calculation |

### 5.3 PID Tuning (Starting Values)

| Controller | Kp | Ki | Kd | Notes |
|------------|----|----|----| ------|
| damper_primary_air | 2.0 | 0.1 | 0.5 | Conservative, slow response |
| valve_side_air | 1.5 | 0.05 | 0.3 | Moderate response |
| water_injection | 0.5 | 0.02 | 0.1 | Very conservative |

---

## 6. PROCESS CHEMISTRY REFERENCE

### 6.1 Pyrolysis Reactions (STATE 1)

```
Biomass → Char + Volatiles + Heat

Volatiles (in afterburner):
  CₓHᵧ + O₂ → CO₂ + H₂O + Heat (combustion)

Temperature: 400-800°C
Atmosphere: Oxidizing (excess air in afterburner)
Product: Biochar (~25% of input mass)
```

### 6.2 Gasification Reactions (STATE 3)

```
Primary (desired):
  C + H₂O → CO + H₂         (Water-gas, +131 kJ/mol, endothermic)
  C + CO₂ → 2CO             (Boudouard, +172 kJ/mol, endothermic)

Heat source:
  C + ½O₂ → CO              (Partial oxidation, -111 kJ/mol, exothermic)

Undesired (too much air):
  C + O₂ → CO₂              (Complete combustion - no syngas!)

Optimal equivalence ratio (λ): 0.25-0.35
Temperature: 800-1000°C
Atmosphere: Reducing (limited air)
Product: Syngas (CO 20-25%, H₂ 15-20%, N₂ 50-55%)
```

### 6.3 Syngas Composition Target

| Component | Target | Acceptable Range |
|-----------|--------|------------------|
| CO | 22% | 18-28% |
| H₂ | 18% | 12-22% |
| CH₄ | 2% | 1-4% |
| CO₂ | 8% | 5-12% |
| N₂ | 50% | 45-55% |
| LHV | 5.5 MJ/m³ | 4.5-6.5 MJ/m³ |

---

## 7. REVISION HISTORY

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2025-12-28 | Initial specification from user |
| 2.0 | 2025-12-28 | Critical review: Fixed transition sequence, added rate-of-change triggers, state-dependent safety thresholds, water injection logic, pressure test, PWM air control |

---

## 8. NEXT STEPS

- [ ] ESPHome YAML configuration
- [ ] Home Assistant dashboard design
- [ ] Test bench simulation mode
- [ ] Sensor calibration procedure
- [ ] Commissioning checklist
