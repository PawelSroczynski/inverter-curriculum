# WarpVerter: A Practical Build Guide
## Book Outline v0.1

---

### Preface
- Why another DIY inverter book?
- WarpVerter vs OzInverter: when to choose which
- About this guide (build-log methodology, AI-assisted documentation)
- Acknowledgments (Tony Le Grip / Warpspeed, BackShed & DIYSolar communities)

---

## Part I: Understanding the WarpVerter

### Chapter 1: Step-Synthesis Fundamentals
- The problem with PWM at high power (paralleling MOSFETs, current sharing nightmares)
- Step-synthesis concept explained
- Why 4 transformers? The voltage building blocks
- Comparison with traditional H-bridge inverters
- The "staircase" waveform and why it works

### Chapter 2: Design Decisions Before You Start
- Choosing your DC voltage (24V / 48V / 96V / 100V+ considerations)
- Target power output (5kW, 10kW, 15kW+)
- Single-phase vs 3-phase configurations
- Split-phase 120/240V considerations (for North American builders)
- Budget vs performance trade-offs
- Your load profile analysis

### Chapter 3: System Architecture Overview
- Block diagram walkthrough
- The four inverter stages and their voltage contributions
- Control board options (Arduino/STM32/dedicated)
- Synchronization and timing
- Bidirectional capability explained
- AC coupling integration points

---

## Part II: Component Sourcing & Preparation

### Chapter 4: Transformer Strategy
- Why transformers are the critical path
- New vs salvaged/second-hand toroids
- Sourcing from scrapped grid-tie inverters (Aerosharp, SMA, etc.)
- Calculating requirements for your target voltage/power
- Evaluating unknown toroids (testing methodology)
- Supplier options by region

### Chapter 5: Winding Your Own Transformers
- Tools and workspace setup
- Wire gauge selection for primary/secondary
- Winding calculations for each of the 4 stages
- Step-by-step winding process with photos
- Common mistakes and how to avoid them
- Testing your finished transformers

### Chapter 6: Power Semiconductors
- MOSFETs vs IGBTs: the real trade-offs
- Recommended parts (HY4008, HY5208, IGBT modules)
- Why IGBTs make sense at higher power despite voltage drop
- Sourcing genuine vs counterfeit components
- Gate driver requirements
- Heatsinking calculations

### Chapter 7: Supporting Components
- Electrolytic capacitors (sizing, voltage ratings)
- Chokes and inductors
- Busbars vs PCB traces
- Connectors and wiring
- Enclosure considerations
- Safety components (fuses, breakers, contactors)

---

## Part III: The Build

### Chapter 8: Control System Assembly
- Control board options comparison
- Mack-e-OzInverter controller adaptation
- STM32-based WarpVerter control
- Wiring the control section
- Initial bench testing (no power stage)

### Chapter 9: Power Stage Construction
- Layout planning (minimize inductance)
- Busbar construction techniques
- MOSFET/IGBT mounting and isolation
- Gate driver connections
- Capacitor bank assembly
- Thermal management integration

### Chapter 10: Transformer Integration
- Mounting the 4 toroids
- Primary connections (the switching side)
- Secondary connections (building the output voltage)
- Output choke installation
- Wiring best practices for high current

### Chapter 11: System Integration
- Connecting control to power stages
- DC input connections
- AC output connections
- Grounding and safety
- Pre-power checklist

---

## Part IV: Testing & Commissioning

### Chapter 12: Safe Power-Up Sequence
- The light bulb limiter method
- Variac testing approach
- Step-by-step first power procedure
- What to monitor and measure
- Expected vs abnormal behavior

### Chapter 13: Testing Under Load
- Resistive load testing
- Reactive load behavior
- Efficiency measurements
- Thermal performance verification
- Waveform analysis (oscilloscope views)
- Fine-tuning output voltage

### Chapter 14: Integration Testing
- Battery bank connection
- AC coupling with grid-tie inverters
- Backfeed charging verification
- Transfer switch integration
- Load testing with real appliances

---

## Part V: Living with Your WarpVerter

### Chapter 15: Troubleshooting Guide
- Symptom-based diagnosis
- Common failure modes
- The "blow-up" post-mortem (what went wrong?)
- Replacing components
- Preventive maintenance

### Chapter 16: Monitoring & Protection
- Adding basic instrumentation
- Over-current protection
- Over-temperature shutdown
- Battery voltage cutoffs
- Optional: remote monitoring

### Chapter 17: Scaling and Expansion
- Adding more WarpVerter units
- Parallel operation considerations
- 3-phase from multiple units
- Hybrid systems (WarpVerter + commercial inverter)

---

## Part VI: Reference

### Appendix A: Complete Bill of Materials
- Base 5kW build
- 10kW build
- 15kW+ build

### Appendix B: Wiring Diagrams
- Control board schematics
- Power stage layouts
- Full system interconnect

### Appendix C: Transformer Winding Tables
- 24V input configurations
- 48V input configurations  
- 96V+ input configurations
- 50Hz vs 60Hz considerations

### Appendix D: Calculations & Formulas
- Transformer sizing
- Wire gauge selection
- Heat dissipation
- Efficiency estimation

### Appendix E: Supplier Directory
- Transformer cores by region
- MOSFET/IGBT sources
- Control board options
- Recommended tools

### Appendix F: Community Resources
- Forum links (BackShed, DIYSolar, Fieldlines)
- Tony's original posts compilation
- GitHub repositories
- Video resources

---

## Notes for Build Documentation

### Photo/Video Checklist
- [ ] All incoming components (with part numbers visible)
- [ ] Workspace setup
- [ ] Each transformer winding session
- [ ] Control board assembly stages
- [ ] Power stage layout progression
- [ ] Critical wiring connections
- [ ] Testing setup (oscilloscope, meters)
- [ ] First power-up moment
- [ ] Load testing sessions
- [ ] Final installed system

### Decision Log Template
```
Date: 
Decision: 
Options considered:
Why this choice:
Cost impact:
References/sources:
```

### Problem/Solution Log Template
```
Date:
Problem observed:
Symptoms:
Investigation:
Root cause:
Solution:
Time to resolve:
Lessons learned:
```
