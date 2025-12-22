# Inverter Building Curriculum - Master Index

## From Zero to OzInverter (6-15kW Pure Sine Wave)

---

## Document Structure

| File | Contents |
|------|----------|
| `inverter_curriculum_part1.md` | Step 1: Fundamentals (12 projects, 4 phases) |
| `inverter_curriculum_part2.md` | Step 2: CD4047 (150W) + Step 3: SG3525 (500W) |
| `inverter_curriculum_part3.md` | Step 4: EG8010 (1kW) + Step 5: OzInverter (6-15kW) |

---

## Quick Reference

### Timeline
```
Step 1: 8-12 weekends  (Fundamentals)
Step 2: 2-3 weekends   (First Inverter)
Step 3: 3-4 weekends   (H-Bridge)
Step 4: 1-2 months     (Pure Sine)
Step 5: 2-3 months     (OzInverter)
────────────────────────────────────
Total:  6-12 months
```

### Budget
```
Step 1:    $64   (12 projects)
Step 2:    $40   (150W inverter)
Step 3:   $110   (500W inverter)
Step 4:   $150   (1kW inverter)
Step 5:   $600   (6kW inverter)
Tools:    $300   (one-time)
────────────────────────────────────
Total: ~$1,264
```

---

## Step 1: Fundamentals (Part 1)

### Phase 1: No ICs
- 1.1 LED + Resistor (Ohm's Law)
- 1.2 Switch Circuits (AND/OR logic)
- 1.3 Transistor as Switch

### Phase 2: 555 Timer
- 1.4 Monostable (One-Shot)
- 1.5 Astable (Blinker)
- 1.6 Relay Control

### Phase 3: Control Concepts
- 1.7 PWM Dimmer
- 1.8 LED Chaser (4017)
- 1.9 Power Switching (TIP120)

### Phase 4: Measurement
- 1.10 Continuity Tester
- 1.11 Voltage Indicator
- 1.12 Audio Oscillator

---

## Step 2: CD4047 Square Wave (Part 2)

- Push-pull topology
- 2 MOSFETs (IRF3205)
- Center-tap transformer
- 150W output
- Square wave

---

## Step 3: SG3525 Modified Sine (Part 2)

- H-bridge topology
- 4 MOSFETs
- IR2110 gate drivers
- Dead-time control
- 500W output
- Modified sine wave

---

## Step 4: EG8010 Pure Sine (Part 3)

- SPWM generation
- Same IC as OzInverter
- LC output filter
- Voltage feedback
- 1kW output
- Pure sine wave (<5% THD)

---

## Step 5: OzInverter (Part 3)

- Full OzInverter build
- Hand-wound toroidal transformer
- 16-32 paralleled MOSFETs
- Advanced cooling system
- 6-15kW output
- Production quality

---

## Skills Progression

| After Step | You Can Build |
|------------|---------------|
| 1 | Any basic electronics project |
| 2 | 150W backup inverter |
| 3 | 500W tool/workshop inverter |
| 4 | 1kW home electronics inverter |
| 5 | 6-15kW whole-house inverter |

---

## Key Components Across All Steps

| Component | Step 2 | Step 3 | Step 4 | Step 5 |
|-----------|--------|--------|--------|--------|
| Controller | CD4047 | SG3525 | EG8010 | EG8010 |
| Gate Driver | Direct | IR2110 | IR2110 | IR2110 |
| MOSFETs | 2× IRF3205 | 4× IRF3205 | 4× IRFP460 | 16× IRFP4668 |
| Topology | Push-pull | H-bridge | H-bridge | H-bridge |
| Transformer | 100VA | 300VA | 500VA | 5000VA |
| Output | Square | Modified | Pure Sine | Pure Sine |

---

## How to Use This Curriculum

1. **Start with Part 1** - Complete all 12 fundamentals projects
2. **Build Part 2 projects** - Steps 2 and 3 in sequence
3. **Progress to Part 3** - Steps 4 and 5
4. **Don't skip steps** - Each builds on previous knowledge
5. **Ask for help** - Use forums, this assistant, YouTube

---

## Resources

- **OzInverter Official:** https://www.bryanhorology.com/ozinverter.php
- **YouTube:** GreatScott!, EEVBlog, ElectroBOOM
- **Free Textbook:** AllAboutCircuits.com
- **Simulator:** Falstad Circuit Simulator

---

*Created: December 2024*
*Goal: Build OzInverter 6-15kW Pure Sine Wave Inverter*
