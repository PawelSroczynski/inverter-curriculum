# THRONE-GEMINI: Dual-Phase Biomass Gasifier

Part of the NODE-ZERO microgrid project.

## Overview

THRONE-GEMINI is an autonomous Waste-to-Energy subsystem that performs two-stage thermochemical conversion:

| Phase | Process | Output |
|-------|---------|--------|
| **Phase 1** | Pyrolysis (TLUD) | Heat + Biochar |
| **Phase 2** | Gasification | Syngas for engine |

## Architecture

- **Topology:** Stationary docking head + mobile reactor vessels on turntable ("Revolver")
- **Controller:** SmartBOB (ESP32-based PLC) with Home Assistant integration
- **Process:** Automatic transition from heat generation to power generation

## Files

- `THRONE_GEMINI_SPEC_v2.md` - Complete engineering specification with state machine, I/O mapping, safety interlocks

## Status

- [x] Physics analysis and critical review
- [x] State machine design (v2)
- [ ] ESPHome configuration
- [ ] Home Assistant dashboard
- [ ] Hardware build
