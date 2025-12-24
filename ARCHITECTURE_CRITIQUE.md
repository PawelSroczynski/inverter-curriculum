# Architecture Critique: House / Cluster / Swarm-Grid

**Status:** Review document for future work
**Scope:** Complete analysis of the three-tier microgrid architecture
**Purpose:** Identify all assumptions, risks, and recommendations before scaling
**Next Steps:** Address issues progressively as each level is implemented (inverter → house → cluster → swarm)

---

## Critique Process

**Run this critique periodically** (after major README updates, before implementation phases).

### How to Run a Critique

1. **Read the current README** architecture sections
2. **Compare against this document's recommendations** (R1-R22)
3. **Update the Latest Critique Run section** below
4. **Mark recommendations as ADDRESSED, PARTIAL, or OPEN**
5. **Add new issues discovered**
6. **Update Document History**

### Critique Checklist

```
□ Tiered bus architecture defined? (cluster + swarm buses)
□ Gateway sync logic specified?
□ Self-reliance levels documented?
□ Grounding scheme defined?
□ Cable sizing calculated?
□ Droop coefficients specified?
□ Master election protocol described?
□ High-power load connection guidance?
□ Energy metering mentioned?
□ Governance structure addressed?
□ Failure modes documented?
```

---

## Latest Critique Run

**Date:** 2024-12-24
**README Version:** After tiered bus architecture update
**Run By:** Claude + User

### Architecture Changes Since Last Review

1. **NEW: Tiered AC Bus Design** - Cluster internal buses + swarm backbone bus
2. **NEW: Gateway with Contactor** - Each cluster connects via ESP32-controlled contactor
3. **NEW: Self-Reliance Levels** - 4 modes documented (house → cluster → swarm → within-cluster)
4. **NEW: Inter-Cluster Power Flow** - 5-hop path explained
5. **NEW: Gateway Sync Logic** - Frequency matching before contactor closes
6. **NEW: High-Power Load Guide** - Where to connect >6kW loads
7. **NEW: Energy as Work** - Using surplus solar for productive tasks

### Recommendation Status

| ID | Recommendation | Status | Notes |
|----|----------------|--------|-------|
| R1 | Define grounding scheme | **OPEN** | Still not specified in README |
| R2 | Test droop stability with 3 inverters | **OPEN** | Experiments added, but no results yet |
| R3 | Implement master election | **PARTIAL** | Gateway ESP32 mentioned, election not detailed |
| R4 | Calculate/verify cable sizing | **PARTIAL** | 80m intra-cluster mentioned, but inter-cluster sizing not calculated |
| R5 | Add CAN repeaters for 800m | **OPEN** | 1000m site now, repeaters not mentioned |
| R6 | Install energy metering | **OPEN** | Not in README |
| R7 | Write governance document | **OPEN** | Not in README |
| R8 | Install cluster isolation breakers | **ADDRESSED** | Gateway contactors provide isolation |
| R9 | Install surge protection | **OPEN** | Not in README |
| R10 | Document commissioning procedure | **OPEN** | Not in README |
| R11 | Add per-inverter status display | **OPEN** | Not in README |
| R12 | Create fault diagnosis guide | **OPEN** | Failure modes table exists in critique only |
| R13 | Create Home Assistant dashboard | **OPEN** | Mentioned but not detailed |
| R14 | Implement automatic load shedding | **OPEN** | Not in README |
| R15 | Consider ring topology | **OPEN** | Linear only |
| R16 | Implement symmetrical wiring | **OPEN** | Not specified |
| R17 | Create droop calibration procedure | **PARTIAL** | Droop coefficient mentioned (k=0.25), calibration in experiments |
| R18 | Add proactive alerting | **OPEN** | Not in README |
| R19 | Designate community agents | **OPEN** | Not in README |
| R20 | Consider consumption allocation | **OPEN** | Not in README |
| R21 | Document battery end-of-life | **OPEN** | Not in README |
| R22 | Consider DC for intra-cluster | **OPEN** | AC chosen, rationale not documented |

### New Issues Identified

**N1: Gateway Contactor Coordination**

The gateway logic shows individual contactor decisions, but what if:
- Gateway A opens while power is flowing through it?
- Two gateways try to close simultaneously?
- Swarm bus has no sources (all gateways open)?

*Recommendation:* Document contactor coordination protocol. Consider "at least one gateway always closed" rule.

**N2: Two Levels of Droop - Different Coefficients?**

README mentions "two levels of droop" but doesn't specify if:
- Cluster internal droop (k₁) is different from swarm droop (k₂)?
- Gateway acts as a "virtual inverter" on swarm bus?
- How does gateway droop interact with house inverter droop?

*Recommendation:* Specify droop coefficients for both levels.

**N3: 1000m Site Exceeds CAN Limits**

README now shows 1000m site (was 800m). Standard CAN bus is marginal at 500m.

*Recommendation:* Explicitly add CAN repeaters at each gateway, or switch to different protocol for swarm backbone.

**N4: Contactor Rating for Bidirectional Flow**

63A contactor specified, but:
- Is it rated for bidirectional AC power?
- What's the inrush current when closing onto loaded bus?
- Pre-charge circuit needed?

*Recommendation:* Specify contactor requirements including inrush handling.

**N5: High-Power Load Connection Points Not Physically Defined**

README says "connect to cluster bus" but:
- Where physically is the cluster bus accessible?
- Is there a dedicated breaker panel for community equipment?
- Who owns/maintains this connection point?

*Recommendation:* Add physical diagram of cluster bus access point.

### What's Working Well

1. **Tiered architecture reduces sync complexity** - Only 3 gateways sync instead of 12 inverters
2. **Self-reliance levels clearly defined** - 99% of time no sharing needed
3. **Gateway contactor provides clean isolation** - Clusters can operate independently
4. **Energy-as-work concept** - Great insight for community productivity
5. **Power flow path documented** - Clear 5-hop explanation

### Priority for Next Phase

1. **CRITICAL:** Define grounding scheme (R1) - must be decided before any multi-house connection
2. **CRITICAL:** Specify CAN repeater locations for 1000m (R5, N3)
3. **HIGH:** Document contactor coordination protocol (N1)
4. **HIGH:** Specify both droop coefficients (N2)
5. **MEDIUM:** Add physical cluster bus access point diagram (N5)

---

## TLDR

This section provides a quick overview for those who need the key points immediately. The full analysis follows in subsequent sections.

### What Works Well

The architecture has several fundamental strengths that make it viable:

**Self-reliant houses:** Each house operates independently with its own 6-15kW inverter, 5-10kWh battery, and 6kW solar array. This means that under normal conditions (99% of the time), no house depends on any other house. The cluster and swarm layers exist purely for resilience and edge cases, not for daily operation. This is a critical design choice because it means a failure anywhere in the cluster doesn't affect normal household operation.

**Droop control for load sharing:** Instead of requiring complex digital coordination between inverters, the system uses droop control - a physics-based mechanism where inverters naturally share load based on frequency. When an inverter is heavily loaded, its output frequency drops slightly (e.g., from 50.0Hz to 49.8Hz). Other inverters on the same AC bus sense this lower frequency and automatically increase their output to compensate. This happens without any communication - it's purely based on electrical physics. The elegance is that it works even if the CAN bus fails completely.

**Isolated DC systems:** The 48V DC battery system stays entirely within each house. Only 230V AC is shared between houses. This dramatically simplifies wiring because AC at 230V can carry the same power as DC at 48V using cables that are roughly 5x thinner. It also eliminates complex DC bus balancing issues and means that a battery fault in one house cannot propagate to others.

**Scalable architecture:** The same pattern repeats at every level. A house is self-contained. A cluster is just houses connected by AC bus and CAN. A swarm is just clusters connected by the same AC bus and CAN. There's no fundamentally different technology at each level, which means lessons learned at the house level apply directly to cluster and swarm levels.

### Critical Issues That Must Be Addressed

These issues are serious enough that they could cause system failure, safety hazards, or social conflict if not addressed:

| Issue | What Goes Wrong | Consequence | Required Fix |
|-------|-----------------|-------------|--------------|
| Master ESP32 is single point of failure | If the ESP32 designated as "master" loses power or fails, no device is broadcasting sync signals | Inverters drift out of phase with each other. When phase difference exceeds ~10°, large circulating currents flow between inverters, potentially tripping breakers or damaging equipment | Implement automatic master election protocol where any ESP32 can take over if the current master stops broadcasting |
| Linear topology has no redundancy | The clusters connect in a line: A─B─C. There is only one path between any two clusters | If the cable between A and B is damaged, cluster C is completely cut off from cluster A. Cluster C must survive on its own resources, which may be insufficient | Either accept this risk (simplest), or install ring topology A─B─C─A (doubles cable cost), or add emergency wireless backup link |
| Grounding scheme not defined | The document doesn't specify where the neutral-to-ground bond is located | If every house has its own N-G bond, ground loops form and current circulates through the ground conductors, causing RCD nuisance trips and potential fire hazard. If no house has N-G bond, the neutral floats and shock hazards exist | Define exactly one N-G bond point per cluster, with all other houses having separate neutral and ground conductors |
| No energy metering | There is no measurement of how much energy each house contributes or consumes from the shared bus | House A with 2 occupants uses 8kWh/day. House B with 6 occupants and an EV uses 25kWh/day. House B constantly draws from the shared bus while House A's battery never fully charges. House A's owner becomes resentful. This social conflict has killed many community energy projects | Install kWh meters at each house AC output. Even without billing, visibility into who is contributing vs consuming dramatically reduces conflict |
| No governance structure | There is no defined process for making decisions about the shared infrastructure | Who decides where to route new cables? Who pays when shared equipment fails? Who can veto a new house joining? What happens if one house wants to leave? Without answers, conflicts escalate | Write a governance document that defines decision rights, cost sharing formulas, voting procedures, and exit procedures |

### Issues Organized by Implementation Phase

This helps you know what to worry about at each stage of building:

**Phase 1 - Building Your First Inverter:**
No blocking issues at this phase. Focus on getting the OzInverter working correctly with stable output voltage, clean sine wave, and proper overcurrent protection. All the cluster and swarm issues are irrelevant until you have working inverters.

**Phase 2 - House Scale (one inverter + solar + battery):**
- You must define where the neutral-to-ground bond is located for your house
- You must verify that the BMS correctly communicates with the inverter (or that the inverter correctly responds to battery voltage dropping)
- You should test that the inverter reduces output gracefully when battery SOC is low rather than cutting off abruptly

**Phase 3 - First Cluster (3-4 houses connected):**
- You must test droop control stability with multiple inverters before trusting it. Start with 2 inverters, verify stability, then add a 3rd
- You must implement the master election protocol so the system survives master failure
- You must add isolation breakers so any house can be disconnected for maintenance without affecting others
- You should test all failure scenarios: What happens when the master loses power? When one inverter is overloaded? When one inverter faults?

**Phase 4 - Full Swarm (12 houses across 3 clusters):**
- You must add CAN bus repeaters because 800m exceeds reliable CAN distance
- You must install energy metering to prevent social conflict
- You must create the governance document
- You should commission houses one at a time, verifying stability after each addition

---

## Detailed Analysis

### 1. ASSUMPTIONS

Every engineering design is built on assumptions - beliefs about how things work that may or may not be true. If an assumption is wrong, the design may fail in unexpected ways. This section explicitly lists all assumptions embedded in the swarm-grid design so they can be validated.

#### Electrical Assumptions

**A1: All inverters can operate in parallel via droop control without instability**

This assumption is that multiple EG8010-based inverters, when connected to the same AC bus and configured with droop control, will naturally find a stable operating point where they share load proportionally.

The risk is that this may not be true. Droop-controlled systems can oscillate if:
- The droop slopes (how much frequency changes per kW of load) are not matched between inverters
- The cable impedance between inverters creates phase shifts that confuse the droop response
- The inverters have different response speeds, so one reacts before another has finished responding

If this assumption is wrong, the system could oscillate at 1-10Hz, with inverters fighting each other - one increasing output while another decreases, then reversing. This stresses components and can damage equipment.

Validation required: Test with 2 inverters first, measure frequency stability, then add a 3rd and verify stability holds. Document the exact droop coefficient that works.

**A2: The EG8010 chip supports droop control natively or can be modified to support it**

The EG8010 is a dedicated SPWM (sinusoidal pulse width modulation) generator chip designed for single inverters. The assumption is that either:
- It has built-in droop functionality, or
- The ESP32 can measure output current and adjust the EG8010's frequency reference to implement droop

If this assumption is wrong, you would need a different control approach - perhaps the ESP32 generating the SPWM directly, or a different driver chip.

Validation required: Review EG8010 datasheet for frequency adjustment capability. Test whether ESP32 can adjust frequency reference in real-time based on load current measurement.

**A3: CAN bus can reliably communicate over 800m with simple RJ45 cabling**

Standard CAN bus is typically rated for distances up to about 500m at 125kbps, or longer distances at lower bitrates. The assumption is that your 800m site can work with either:
- Standard CAN at reduced bitrate (e.g., 50kbps)
- CAN with repeaters at cluster boundaries

If this assumption is wrong, you'll experience communication errors: dropped messages, corrupted data, or complete communication failure. This would cause inverters to lose synchronization.

The risk is increased by:
- Using standard Ethernet cable (RJ45) instead of proper twisted-pair CAN cable
- Not properly terminating the bus (120Ω resistors at each end)
- Running the CAN cable parallel to power cables (electromagnetic interference)

Validation required: Test CAN communication at your actual distances before relying on it. Use a CAN bus analyzer to measure error rates.

**A4: 4mm² cable is sufficient for 200-400m inter-cluster runs at expected power levels**

This assumption needs mathematical verification. Let's calculate:

For copper cable, resistance is approximately 4.61 mΩ/m for 4mm² cross-section.

For a 400m run (one direction), the round-trip distance for current is 800m.
Total resistance: 4.61 mΩ/m × 800m = 3.69Ω

If 20A flows through this cable (about 4.6kW at 230V):
Voltage drop = 20A × 3.69Ω = 73.8V

This is a 32% voltage drop, which is completely unacceptable. The receiving end would see only 156V instead of 230V.

This assumption appears to be WRONG for the specified distances at meaningful power levels.

For acceptable voltage drop (say, 5% or 11.5V at 20A):
Required resistance = 11.5V / 20A = 0.575Ω
Required cable size for 800m round trip: approximately 25mm²

Alternatively, if you keep 4mm² cable, maximum current for 5% drop:
Max current = 11.5V / 3.69Ω = 3.1A (about 700W)

This means inter-cluster power sharing would be limited to about 700W with 4mm² cable over 400m, which may be acceptable since sharing is only for emergencies.

Validation required: Decide what power level you need to transfer between clusters, then size cable accordingly.

**A5: Single-phase 230V is sufficient for all loads (no 3-phase equipment needed)**

This assumption means no household or shared equipment requires 3-phase power. Typical 3-phase loads include:
- Large motors (>3kW)
- Some EV chargers (11kW+ models)
- Commercial kitchen equipment
- Workshops with large machines

If this assumption is wrong and someone needs 3-phase, they cannot be served by this system without significant modification.

Validation required: Survey all planned loads across all houses to confirm none require 3-phase.

**A6: The frequency range 49.7-50.0Hz used by droop control is acceptable for all connected loads**

Most equipment is designed for ±1% frequency tolerance (49.5-50.5Hz in a 50Hz system), so the 0.3Hz droop range should be acceptable. However, some equipment is sensitive:
- Clocks and timers that count mains cycles
- Some audio/video equipment
- Precision motors

If this assumption is wrong, sensitive equipment may malfunction or show errors.

Validation required: Review specifications of any precision equipment that will be connected.

**A7: The frequency deviations from droop control will not cause cumulative problems**

Even if individual equipment tolerates ±0.3Hz, there may be cumulative effects:
- Electric clocks will run slow when frequency is below 50Hz
- Some equipment phase-locks to mains frequency

In grid-connected systems, grid operators ensure that time spent above 50Hz equals time spent below, so clocks stay accurate over 24 hours. In an islanded microgrid, there's no such guarantee.

If this assumption is wrong, clocks will drift and some equipment may experience long-term issues.

Mitigation: The ESP32 could track cumulative frequency deviation and bias the system slightly high or low to compensate over time.

#### Battery and Solar Assumptions

**B1: 5-10 kWh battery storage per house is sufficient for overnight operation and cloudy days**

This assumption depends heavily on:
- Household consumption patterns (how much is used at night?)
- Local climate (how many consecutive cloudy days?)
- Seasonal variation (winter has shorter days and often higher heating loads)

A typical household might use 10kWh/day total, with perhaps 5kWh at night (when solar is unavailable). A 5kWh battery would be depleted by morning. A 10kWh battery provides some margin.

However, in winter with 2-3 cloudy days in a row:
- Daily consumption: 10kWh × 3 days = 30kWh
- Solar production: maybe 2kWh/day × 3 days = 6kWh
- Net deficit: 24kWh

A single 10kWh battery cannot cover this. The cluster sharing helps (if your neighbor has sun and you don't), but if the entire region is cloudy, everyone has the same problem.

Validation required: Calculate for your specific location's worst-case solar production and your actual consumption patterns.

**B2: 6kW of solar panels per house matches local solar irradiance and consumption needs**

Solar production depends on:
- Geographic location (latitude affects sun angle)
- Panel orientation (south-facing in northern hemisphere is ideal)
- Shading (trees, buildings, other structures)
- Local climate (cloud cover, rain, snow)

In Central Europe, 6kW of panels might produce 5,000-7,000 kWh/year (average 14-19 kWh/day). But production varies dramatically:
- Summer day: 30-40 kWh
- Winter day: 2-5 kWh

If this assumption is wrong (panels produce less than expected), batteries will chronically undercharge and the system will struggle.

Validation required: Use a solar calculator (like PVGIS for Europe) with your exact location and panel orientation to estimate production.

**B3: All houses have similar consumption patterns**

The droop control and power sharing work best when houses have roughly similar consumption. If consumption is wildly unequal:
- High-consumption houses constantly draw from the shared bus
- Low-consumption houses never benefit from sharing
- Resentment builds

If this assumption is wrong, you need either metering/billing or consumption limits to maintain fairness.

Validation required: Estimate consumption for each household before they join. Set expectations about what "normal" consumption looks like.

**B4: LiFePO4 battery cells are available at reasonable cost**

LiFePO4 (lithium iron phosphate) is assumed because:
- It's safer than other lithium chemistries (no thermal runaway)
- It has excellent cycle life (3000+ cycles)
- It tolerates being held at partial charge

The assumption is that cells are available at approximately $100-150/kWh. If prices increase or availability decreases, the economics change significantly.

Alternative: Lead-acid batteries are cheaper upfront but have 3-4x lower cycle life and are heavier. They might be acceptable for a prototype but increase long-term costs.

Validation required: Price actual cells from suppliers (e.g., EVE, CATL) and factor into budget.

**B5: The BMS will correctly limit discharge current and communicate this to the inverter**

The Libre Solar BMS-C1 is assumed to:
- Monitor cell voltages and temperatures
- Limit discharge current when SOC is low (e.g., reduce to 1kW below 20% SOC)
- Communicate status via CAN bus

If the BMS fails to limit discharge, the inverter could over-drain the battery, potentially damaging cells or triggering BMS disconnect (sudden blackout).

If the BMS limits discharge but doesn't communicate this, the system won't know to draw from the shared bus - it will just see reduced capacity.

Validation required: Test the specific BMS-inverter communication during house-level commissioning. Verify the inverter responds correctly to BMS signals.

#### Social and Organizational Assumptions

**S1: All households agree to share power without individual metering or billing**

This is perhaps the most dangerous assumption. It assumes:
- Everyone consumes roughly the same amount
- Everyone produces roughly the same amount
- Nobody feels taken advantage of
- This remains true over years as circumstances change

Real-world experience with community energy projects shows that unmetered sharing frequently leads to conflict. Even small perceived inequities grow into resentment over time.

Common failure modes:
- One household gets an EV and triples their consumption
- One household's solar gets shaded by a new tree and production drops
- A frugal household is annoyed at a wasteful neighbor
- Someone new moves in with different expectations

If this assumption is wrong, relationships between neighbors deteriorate and the project fails socially even if it works technically.

Validation required: Have explicit conversations with all participants about expectations. Consider metering even if you don't bill - transparency alone reduces conflict significantly.

**S2: Sufficient technical skills exist within the community for ongoing maintenance**

The system requires someone who can:
- Diagnose inverter faults
- Replace MOSFETs, capacitors, and control boards
- Debug CAN bus communication issues
- Update ESP32 firmware
- Repair or replace cables and connectors

If the original builder leaves or becomes unavailable, and no one else has these skills, the system will gradually degrade as faults accumulate without repair.

If this assumption is wrong, you'll eventually face a choice between expensive outside help or system abandonment.

Mitigation: Train at least 2-3 people during the build process. Document everything thoroughly. Keep spare parts inventory.

**S3: No one will abuse the shared resource (tragedy of the commons)**

The classic tragedy of the commons: a shared resource is overused because individuals benefit from overuse while costs are distributed to all.

In this context:
- Running an energy-intensive home business while "paying" only your share of shared infrastructure costs
- Leaving lights/heating on wastefully because "the solar is free anyway"
- Charging your EV every night because "there's always power available"

If this assumption is wrong, the most conscientious users subsidize the least conscientious.

Mitigation: Social norms, visible metering, or actual billing can align individual incentives with collective good.

**S4: The household with the master ESP32 is trusted by all other participants**

The master ESP32:
- Controls synchronization timing for all inverters
- Could potentially manipulate the system to favor certain houses
- Has access to data about everyone's consumption and production

Even if the master owner is trustworthy, the perception of unequal control can create social friction.

If this assumption is wrong, disputes arise about master location or functionality.

Mitigation: Master election protocol means any house can become master, removing the "special" status. Alternatively, locate master in a neutral location (cluster junction box) rather than in any house.

**S5: Faults in one house will not cause legal liability issues for other participants**

If House A's inverter catches fire and damages House A, is House B liable because they were sharing power? What if the fire spreads to House B - can they claim against the installer? What about insurance?

If this assumption is wrong, a single accident could result in lawsuits that destroy relationships and finances.

Mitigation: Consult with an insurance agent about coverage for DIY electrical systems. Consider forming a legal entity (cooperative) that provides liability shielding. Have clear written agreements about responsibility.

#### Physical and Site Assumptions

**P1: The site layout is linear and approximately 800m long**

The architecture is designed for a linear site where clusters are arranged in a line. If the actual site is:
- Scattered (houses not in convenient clusters)
- Branching (T-shaped or star-shaped layout)
- Much longer than 800m

Then cable routing becomes more complex, costs increase, and the simple A-B-C model may not apply.

Validation required: Map actual house locations and cable routes before finalizing design.

**P2: Right-of-way exists for inter-house and inter-cluster cabling**

Running cables between houses requires either:
- Ownership of the land the cables cross
- Easements granting permission to cross others' land
- Public right-of-way that allows utility installation

If this assumption is wrong, you may be unable to physically install the cables, or may face legal challenges later.

Validation required: Confirm land ownership or obtain easements before beginning cable installation.

**P3: All houses have adequate roof space and orientation for 6kW of solar panels**

6kW of solar panels requires approximately 30-40m² of roof space, ideally south-facing (in northern hemisphere) with good sun exposure.

If actual roofs are:
- Smaller
- North-facing
- Shaded by trees or buildings
- Unsuitable for mounting (weak structure, heritage restrictions)

Then solar production will be less than expected, and some houses may become net consumers rather than producers.

Validation required: Assess each roof individually with a solar surveying tool or professional assessment.

**P4: The climate allows meaningful solar production year-round**

Solar production varies dramatically with latitude and climate. In northern locations:
- Winter days are very short (6-8 hours of weak daylight)
- Snow may cover panels
- Fog and low clouds reduce production

If winter production is negligible, the system must either:
- Have enough battery storage to survive long dark periods
- Accept reduced functionality in winter
- Have a backup power source

Validation required: Research solar production data for your specific location across all seasons.

---

### 2. ANALYSIS BY STANDPOINT

This section analyzes the architecture from different professional perspectives, each bringing different concerns and priorities.

#### Electrical Engineering Standpoint

**Droop Control Stability Analysis**

Droop control is elegant in theory: each inverter adjusts its output frequency based on load, and power naturally flows from lightly-loaded inverters (higher frequency) to heavily-loaded inverters (lower frequency). No communication required.

However, stability is not guaranteed. The mathematical analysis of parallel droop-controlled inverters shows that oscillation can occur when:

1. **Droop slopes are mismatched:** If Inverter A drops 0.1Hz per kW while Inverter B drops 0.2Hz per kW, they won't share load equally and may hunt for equilibrium.

2. **Cable impedance creates phase shifts:** The cable between inverters has inductance and resistance. When current flows through this impedance, it creates a phase shift between the voltage at each inverter. This can confuse the droop response because inverters see different voltages.

3. **Response time mismatch:** If Inverter A responds to frequency changes in 10ms but Inverter B responds in 100ms, they're operating on different timescales. Inverter A might overshoot while waiting for B to respond.

4. **Too many inverters:** As the number of parallel inverters increases, the frequency band must be divided among more units. With 12 inverters sharing a 0.3Hz band, each has only 0.025Hz to work with. Small measurement errors become significant.

The consequence of instability is oscillation. The system might oscillate at 0.5-5Hz, with frequency and power swinging back and forth. This is audible (you can hear transformers buzzing), stresses components, and can damage equipment.

Testing protocol:
1. Start with 2 inverters on a test bench with resistive load
2. Measure frequency with an oscilloscope or frequency counter
3. Verify stable frequency under varying load
4. Add 3rd inverter, repeat testing
5. Document the droop coefficient that provides stability
6. Only then proceed to field deployment

**CAN Bus Distance and Reliability Analysis**

CAN (Controller Area Network) was designed for automotive applications where distances are tens of meters. Using it over 800m requires careful consideration.

Standard CAN physical layer characteristics:
- Maximum distance at 1Mbps: ~40m
- Maximum distance at 125kbps: ~500m
- Maximum distance at 50kbps: ~1000m

Your 800m site is within range at lower bitrates, but marginally so. Factors that reduce effective range:
- Using Ethernet cable instead of proper CAN cable (Ethernet has different characteristic impedance)
- Improper termination (must have exactly 120Ω at each end of the bus)
- Running CAN cable parallel to power cables (electromagnetic interference)
- Stub connections (CAN should be a single line, not a star)
- Poor connections or damaged cables

When CAN fails partially, you see:
- Increasing error frame count
- Retransmitted messages
- Eventually, nodes going "bus off" (disconnecting themselves due to too many errors)

When CAN fails completely, sync messages stop arriving and inverters lose phase synchronization.

Recommendations:
1. Use proper CAN cable (twisted pair, 120Ω characteristic impedance) for long runs
2. Install CAN repeaters at cluster boundaries (isolates segments, regenerates signals)
3. Terminate bus correctly with 120Ω resistors
4. Monitor CAN error counts and investigate if they increase
5. Route CAN cables away from power cables

**Voltage Drop Analysis for Long Cables**

Voltage drop on AC cables is a serious concern for long runs. Let's analyze properly.

For single-phase AC, current flows through the line conductor and returns through neutral. Both conductors have resistance, so total resistance is double the single-conductor resistance.

For copper cable:
- 4mm² has resistance of approximately 4.61 mΩ/m
- 10mm² has resistance of approximately 1.83 mΩ/m
- 16mm² has resistance of approximately 1.15 mΩ/m

For a 400m inter-cluster cable (800m round trip) at various sizes:

| Cable Size | Round Trip Resistance | Voltage Drop at 10A | Voltage Drop at 20A |
|------------|----------------------|---------------------|---------------------|
| 4mm² | 3.69Ω | 36.9V (16%) | 73.8V (32%) |
| 10mm² | 1.46Ω | 14.6V (6.4%) | 29.3V (12.7%) |
| 16mm² | 0.92Ω | 9.2V (4%) | 18.4V (8%) |

Acceptable voltage drop for power distribution is typically 3-5%. At 400m distances, even 16mm² cable can only carry about 12A (2.8kW) within 5% drop.

This has implications:
- Inter-cluster power sharing is limited by cable size
- For emergency sharing (inverter failure, low battery), a few kW may be enough
- For heavy sharing (EV charging), larger cables or shorter distances are needed

Recommendations:
1. Calculate required power sharing level realistically
2. Size inter-cluster cables for that power level plus margin
3. Accept that inter-cluster sharing may be limited to a few kW
4. Use larger cables (10mm² or 16mm²) if budget allows

**Neutral-Ground Bonding Analysis**

In any AC distribution system, the relationship between neutral and ground is critical for safety.

The neutral conductor carries return current during normal operation. The ground conductor should carry zero current during normal operation - it's there only for fault protection.

A neutral-to-ground bond at one point (and only one point) in the system:
- Provides a reference point so the neutral is at ground potential
- Ensures ground fault current has a return path (for breakers/RCDs to detect)

If multiple N-G bonds exist:
- Current can flow through the ground conductor during normal operation
- This current flowing through ground can cause:
  - Voltage differences between ground points (shock hazard)
  - Nuisance RCD trips
  - Corrosion at ground connections
  - Electromagnetic interference

If no N-G bond exists:
- The neutral floats at whatever voltage circumstances create
- Touching neutral could shock you
- Ground faults may not be detected

For the swarm-grid, the recommended approach:
1. Each cluster has ONE N-G bond at the cluster junction box (or at the "master" house)
2. All houses in the cluster connect to this grounding point
3. Houses do NOT have individual N-G bonds
4. Each cluster has a ground rod for local earth reference
5. Cluster ground rods are interconnected by the PE conductor in the inter-cluster cable

This creates a "single ground reference per cluster" system with clusters interconnected.

**Inrush Current and Transient Analysis**

Large motors have high inrush current at startup - typically 5-7x their running current. A 3kW motor might draw 15-21kW for 0.5-2 seconds during startup.

In a single-inverter system, the inverter must be sized to handle this inrush or the motor won't start.

In a paralleled system, droop control should automatically share inrush current among all inverters. But there are concerns:

1. **Response time:** Droop control might not respond fast enough. Inrush lasts 0.5-2 seconds. If droop response time is 0.5 seconds, the local inverter sees the full inrush before neighbors help.

2. **Which inverter responds first?** If multiple inverters are at similar load levels, the one that responds first "wins" and provides more power. This could lead to inconsistent behavior.

3. **Current limiting:** If the local inverter hits current limit, it might trip before neighbors can help.

Mitigation:
- Size local inverters to handle expected inrush from local loads
- Use soft-start devices on large motors
- Test motor starting behavior during commissioning

#### Reliability and Resilience Standpoint

**Single Points of Failure Analysis**

A single point of failure (SPOF) is any component whose failure causes system-wide failure. The architecture should be analyzed for SPOFs:

| Component | Scope of Failure | Severity | Mitigation |
|-----------|------------------|----------|------------|
| Individual inverter | One house loses internal power (but can draw from bus) | Low | By design - houses can run from bus |
| Individual battery | One house loses storage (but solar still works) | Low | By design - still has solar + bus |
| Master ESP32 | All inverters lose sync | HIGH | Master election protocol |
| Inter-cluster cable | Downstream clusters isolated | HIGH | Ring topology or accept risk |
| Intra-cluster cable segment | Houses beyond break are isolated | Medium | Multiple paths or accept risk |
| Individual house breaker | One house isolated | Low | By design - breaker is per-house |

The two significant SPOFs are:
1. Master ESP32 - addressed by master election protocol
2. Inter-cluster cable - harder to address without ring topology

**Master Election Protocol Design**

When the master ESP32 fails, another ESP32 must take over. Here's a detailed protocol:

Normal operation:
- Master broadcasts sync message every 10ms (100Hz)
- Each sync message contains: master ID, timestamp, sequence number
- Slaves verify they're receiving sync, adjust their EG8010 phase to match

Master failure detection:
- Each slave maintains a counter: "time since last sync received"
- If counter exceeds 100ms (10 missed messages), master is presumed dead

Election process:
1. When a slave detects master timeout, it waits a random backoff period (0-50ms)
2. After backoff, it broadcasts "election" message with its node ID
3. If it receives an election message with higher ID, it yields
4. If it receives no higher-ID election message within 50ms, it declares itself master
5. New master immediately begins broadcasting sync messages

Tie-breaking:
- Higher node ID wins
- Node IDs must be unique (could be based on ESP32 MAC address)

Rejoining:
- If old master recovers, it sees sync messages from new master
- It becomes a slave (does not try to reclaim master role)
- This prevents oscillation between masters

Implementation considerations:
- Code is approximately 50-100 lines
- Must be tested thoroughly - master failure is rare but critical
- Consider adding "master health" monitoring (temperature, voltage, error counts)

**Failure Mode and Effects Analysis (FMEA)**

| Failure Mode | Detection Method | Local Effect | System Effect | Recovery Action |
|--------------|------------------|--------------|---------------|-----------------|
| Inverter MOSFET short-circuit | Overcurrent trips breaker | House loses internal generation | House draws from bus; neighbors supply | Replace MOSFET, reset breaker |
| Inverter MOSFET open-circuit | Output voltage collapses | House loses internal generation | House draws from bus | Replace MOSFET |
| Inverter control board failure | No AC output | House loses internal generation | House draws from bus | Replace control board |
| Battery cell failure | BMS detects voltage imbalance | Reduced capacity or BMS disconnect | House operates from solar + bus | Replace cell |
| BMS failure (stuck off) | No battery power available | House operates from solar only | During night, draws from bus | Replace BMS |
| BMS failure (stuck on, no limiting) | Over-discharge damages battery | Battery damage over time | Other houses may notice low power quality | Monitor and replace BMS |
| MPPT charger failure | Battery not charging | Battery depletes over days | Increasing draw from bus | Replace MPPT |
| CAN bus cable break | Communication errors, timeout | Affected nodes lose sync | Nodes beyond break are islanded | Repair cable |
| CAN bus termination failure | Increasing error counts | Unreliable communication | Potential sync problems | Add/fix termination |
| Master ESP32 hardware failure | Slaves timeout on sync | No sync messages | All inverters gradually drift | Master election activates |
| Master ESP32 software crash | Slaves timeout on sync | No sync messages | All inverters gradually drift | Watchdog should restart; election if not |
| Inter-cluster cable cut | Cluster B loses contact with A | Cluster C completely isolated | C operates autonomously | Repair cable |
| Main breaker trip (one house) | House offline | One house isolated | Neighbors continue normally | Investigate cause, reset breaker |
| Ground fault | RCD trips | Affected area offline | Depends on RCD location | Find and repair fault |
| Lightning strike | SPD operates, may sacrifice | SPD needs replacement | Possible equipment damage | Replace SPD, check all equipment |

#### Scalability Standpoint

**CAN Bus Scaling Analysis**

CAN bus can support many nodes (theoretical limit is 127), but practical limits are lower due to message collisions.

In CAN, if two nodes transmit simultaneously, the one with lower ID wins (arbitration). The losing node must wait and retry. As more nodes are added:
- Collision probability increases
- Low-priority messages may be delayed
- Bus utilization must be kept below ~50% for reasonable latency

For the swarm-grid:
- Sync messages are highest priority (lowest ID)
- Status messages (SOC%, load, faults) are lower priority
- With 12 nodes sending status every second, bus load is minimal

Scaling is probably not an issue for the designed system size. If scaling to 50+ nodes, consider:
- Reducing status message frequency
- Implementing message filtering
- Segmenting the CAN bus with bridges

**Droop Control Scaling Analysis**

As more inverters are added, the available frequency range must be divided among them.

If the acceptable frequency range is 49.7-50.0Hz (0.3Hz total) and each inverter needs some frequency margin to indicate its load level:
- With 3 inverters: 0.1Hz per inverter
- With 12 inverters: 0.025Hz per inverter
- With 48 inverters: 0.00625Hz per inverter

At small frequency margins, measurement accuracy becomes critical. If an inverter can only measure frequency to ±0.01Hz accuracy, it can't reliably position itself within a 0.025Hz window.

Practical limit is probably 20-30 inverters in a single droop-controlled group. Beyond that, the system should be split into multiple independently-controlled groups with tie lines between them.

For the planned 12-house swarm, scaling is acceptable but approaching limits.

#### Economics Standpoint

**Cost Allocation Without Metering**

Without metering, costs are typically shared equally - each household pays 1/N of shared costs regardless of consumption.

This works if:
- Consumption is roughly equal across households
- No one significantly over- or under-consumes
- Everyone trusts that others are being fair

This fails when:
- One household uses dramatically more (e.g., EV, home business, many occupants)
- Resentment builds over time
- Arguments occur about who is "using too much"

Even without billing, installing kWh meters provides:
- Visibility - everyone can see their own usage
- Accountability - over-consumers know they're over-consuming
- Data for decisions - know where energy goes, when demand peaks

Cost of simple kWh meters: approximately $20-30 per house.

**Maintenance Cost Allocation**

Shared infrastructure will eventually need maintenance:
- Cable repairs (damage from digging, rodents, weather)
- Breaker replacements
- CAN repeater replacement
- System-wide configuration changes

How are these costs shared?
- Equal shares? (Fair if equal benefit)
- By consumption? (Those who use more pay more)
- By proximity? (Those closer to the problem pay more)
- By the responsible party? (If House A's digging damaged the cable, House A pays)

Without a governance document, these questions cause conflict when they arise.

Recommendation: Establish a shared maintenance fund with regular contributions, plus rules for extraordinary expenses.

**Return on Investment Analysis**

Investment per household (from the document): approximately $3,575

Value of energy produced per year:
- 6kW solar × 1000kWh/kWp/year (typical for Central Europe) = 6,000kWh
- At $0.30/kWh grid price = $1,800/year value

Simple payback: $3,575 / $1,800 = 2.0 years

This is attractive compared to grid electricity. However, the calculation assumes:
- You would have paid grid prices for all produced energy
- The system lasts long enough to provide payback
- No major repair costs

More realistic analysis would include:
- Battery replacement (every 10-15 years)
- Inverter repairs
- Maintenance labor value

Even with these factors, community solar with storage typically has 5-8 year payback, which is acceptable for long-term infrastructure.

#### Safety and Compliance Standpoint

**Grid Code and Certification Issues**

In most jurisdictions, electrical equipment connected to buildings must meet safety standards:
- CE marking in Europe
- UL listing in North America
- Local electrical codes

DIY inverters typically don't have these certifications because:
- Testing and certification costs $10,000-50,000+
- It requires formal documentation and quality control
- Few DIY projects go through this process

Implications:
- Insurance companies may not cover damage from uncertified equipment
- If a fire occurs, investigators may attribute it to the DIY equipment
- Local authorities might order removal if they become aware
- Property sale might be complicated by unpermitted electrical work

Mitigation strategies:
- Have a licensed electrician inspect and sign off on AC wiring (even if they don't certify the inverter itself)
- Document everything: design calculations, testing results, safety measures
- Keep inverters in fire-resistant enclosures (metal boxes)
- Maintain fire extinguishers nearby
- Consider forming a legal entity to provide some liability protection

**Grounding and Earth Fault Protection**

Proper grounding is critical for safety. The system needs:

1. **Equipment grounding:** All metal enclosures connected to ground so that a fault to enclosure trips a breaker rather than electrifying the enclosure.

2. **System grounding:** The neutral-to-ground bond that provides a reference for the AC system.

3. **Ground fault detection:** RCDs (residual current devices) or GFCIs that detect current imbalance between line and neutral, indicating a ground fault.

For the swarm-grid:
- Each house should have an RCD at the main breaker
- Each cluster should have a master RCD at the cluster junction
- Ground conductors must be continuous and properly sized
- Ground rods should be tested for adequate earth resistance

**Arc Fault Risk**

Arc faults occur when electricity jumps across a gap, typically due to:
- Loose connections
- Damaged cable insulation
- Wet conditions

With 800m of cable and many junction points, the probability of an arc fault somewhere in the system is significant.

Arc faults can:
- Start fires (the arc is extremely hot)
- Be intermittent and hard to detect
- Not trip standard breakers (current may be normal)

Mitigation:
- Use proper cable glands and junction boxes (IP65 or better)
- Torque all connections to manufacturer specifications
- Annual thermographic inspection (infrared camera shows hot spots)
- Consider AFCI (arc fault circuit interrupter) breakers at key points

**Lightning Protection**

If the cable network is struck by lightning (directly or nearby):
- Surge voltage can be thousands of volts
- Current can destroy electronics instantly
- Fire can result from overheated conductors

Mitigation:
- Install SPDs (surge protective devices) at cluster entry points
- Ground the cable shield (if shielded cable is used)
- Ensure good grounding throughout (low impedance to earth)
- Consider lightning rods for prominent structures

SPDs sacrifice themselves to protect equipment - they must be replaced after a significant surge event.

#### Social Dynamics Standpoint

**Free Rider Problem Analysis**

The free rider problem occurs when someone benefits from a shared resource without contributing fairly. In the swarm-grid:

Scenario: House B has an EV and charges it every night, consuming 10kWh. House A has no EV and uses only 5kWh at night. Both houses have the same solar and battery setup.

What happens:
- House B drains its battery by midnight
- House B then draws from the shared AC bus
- Houses A, C, and D supply House B's demand from their batteries
- By morning, everyone's batteries are lower than they would be
- House A effectively subsidizes House B's transportation

Over months, House A notices their battery never reaches full charge. They investigate and realize they're powering House B's car. Resentment builds.

Solutions:
1. **Metering + billing:** House B pays for what they consume. Market solution.
2. **Consumption caps:** Each house limited to X kWh from shared bus per day. Technical solution.
3. **Social norms:** Community agreement about acceptable consumption levels. Social solution.
4. **Transparency only:** Everyone sees the data, social pressure does the rest. Information solution.

Recommendation: At minimum, implement transparency (metering with visible dashboard). Consider whether billing is appropriate for your community.

**Governance Structure Requirements**

A functioning community infrastructure needs governance - agreed-upon ways to make decisions. Without it, every issue becomes an ad-hoc negotiation.

Questions governance must answer:

1. **Expansion:** Can new houses join? Who decides? What do they pay for existing infrastructure?

2. **Modification:** If someone wants to upgrade their system, who approves? What if it affects others?

3. **Maintenance:** Who is responsible for maintaining shared infrastructure? How are costs allocated?

4. **Disputes:** If two households disagree about something (noise from someone's inverter, cable routing through someone's property), how is it resolved?

5. **Exit:** If someone wants to leave, what happens? Do they get refunded for infrastructure contributions? How is the system reconfigured?

6. **Liability:** If something goes wrong and causes damage, who is responsible?

Recommendation: Write a simple governance document covering these points before connecting houses. It's much easier to agree when the questions are hypothetical than when there's an active conflict.

**Exit Strategy Considerations**

People's circumstances change. Someone might:
- Sell their house
- Have a falling out with neighbors
- Decide they want grid connection instead
- Die (heirs may not want to participate)

The system must handle departure gracefully:

Technical considerations:
- Can a house physically disconnect without affecting others?
- How is the system reconfigured? (New cable routing? Different master location?)
- What happens to their CAN node ID?

Financial considerations:
- Do they get refunded for infrastructure contributions?
- What about their share of the maintenance fund?
- Who pays for reconfiguration?

Social considerations:
- How do you prevent a departure from becoming acrimonious?
- What if someone wants to leave but can't afford the exit cost?

Recommendation: Include exit procedures in the governance document. Define costs and processes before anyone needs to use them.

#### Practical Implementation Standpoint

**Commissioning Complexity Analysis**

Commissioning the full swarm requires bringing 12 inverters into synchronization, which is non-trivial.

Recommended commissioning sequence:

1. **Individual testing:** Test each inverter individually, verifying output voltage, frequency, and waveform quality.

2. **House-level commissioning:** Connect inverter to its battery and solar. Verify charging, discharging, and load operation. Run for one week.

3. **Two-inverter parallel test:** Connect two houses on a test bench (or two inverters in one house). Verify stable parallel operation under varying loads. Measure frequency stability.

4. **Cluster commissioning:** Connect houses one at a time. After each addition:
   - Verify frequency stability
   - Test load sharing (apply load to one house, verify others help)
   - Test failure recovery (turn one off, verify others continue)

5. **Inter-cluster commissioning:** Connect clusters one at a time, following the same verify-after-each-addition approach.

The key principle is: never connect something new without verifying the system is stable first. If instability occurs, you know it was caused by the most recent addition.

**Debugging Oscillation Problems**

If the system oscillates (frequency swinging up and down), debugging is challenging because:
- Multiple inverters are interacting
- The problem might only appear under certain load conditions
- Oscillation can be fast (hard to observe) or slow (takes time to develop)

Debugging approach:

1. **Observe:** Use a frequency counter or oscilloscope to measure AC frequency. Is it stable? If swinging, what's the period and amplitude?

2. **Isolate:** Disconnect inverters one at a time until oscillation stops. The last-disconnected inverter is likely the problem. (Or, the problem was in the interaction between it and the remaining system.)

3. **Compare:** Compare the suspect inverter to working ones. Are droop settings identical? Is control board the same revision?

4. **Test pairwise:** Connect only two inverters - the suspect and one known-good. Does oscillation occur? Try different pairings to find which combination oscillates.

5. **Adjust droop:** Try reducing droop slope (smaller frequency change per kW). This slows response and may eliminate oscillation at the cost of less precise load sharing.

6. **Add damping:** If available in firmware, increase the response time constant. This filters out rapid changes that could cause oscillation.

**Cable Installation Practical Considerations**

Installing 800m of cable is a significant civil engineering project:

**Route planning:**
- Survey the entire route before beginning
- Identify obstacles (roads, streams, existing utilities)
- Obtain any necessary permissions or permits

**Burial depth:**
- Minimum 0.6m depth for direct burial (check local codes)
- Deeper under roads or driveways
- Shallower if using conduit

**Cable protection:**
- Use conduit in areas with risk of damage
- Mark cable location with warning tape above the cable
- Consider armored cable for extra protection

**Junction boxes:**
- Needed wherever cable changes direction significantly or where branches connect
- Must be waterproof (IP65 or better)
- Must be accessible for future maintenance

**Testing:**
- Test cable continuity and insulation resistance after installation
- Document test results for future reference

---

### 3. RECOMMENDATIONS

#### Critical Priority (must address before connecting multiple houses)

**R1: Define and implement grounding scheme**

Why critical: Improper grounding creates shock hazards and can cause nuisance trips or equipment damage.

Action:
1. Designate one location per cluster as the N-G bond point (the cluster junction box is ideal)
2. At this location, install a bond between neutral and ground
3. All other locations have neutral and ground as separate conductors
4. Install a ground rod at each cluster, measured for adequate earth resistance (<10Ω)
5. Document the grounding scheme clearly

Effort: 1-2 days of design work, plus installation time.

**R2: Test droop stability with 3 inverters before scaling**

Why critical: Droop instability could damage equipment or prevent the system from working at all.

Action:
1. Set up a test bench with 3 inverters connected in parallel
2. Load with resistive load (heaters or resistor bank)
3. Measure frequency with oscilloscope or frequency counter
4. Vary load from 0% to 100% and back, observing frequency stability
5. If oscillation occurs, adjust droop coefficients until stable
6. Document the final droop settings that work

Effort: 1-2 weeks of testing.

**R3: Implement master election protocol**

Why critical: Without it, master failure takes down the entire swarm.

Action:
1. Design the election protocol (see detailed design earlier)
2. Implement in ESP32 firmware
3. Test by powering off the master and verifying a slave takes over
4. Test by having the master send corrupt sync messages (should be ignored)
5. Test recovery when old master rejoins

Effort: 1-2 days of coding, plus 1-2 days of testing.

**R4: Calculate and verify cable sizing**

Why critical: Undersized cables create voltage drop, overheating, and limit power transfer.

Action:
1. Measure actual distances between houses and clusters
2. Estimate realistic power transfer requirements (not maximum, but realistic)
3. Calculate voltage drop for proposed cable sizes
4. Select cable size that gives acceptable voltage drop (<5% for main runs)
5. If budget doesn't allow proper cable size, document the limitation

Effort: 1 day of calculation.

#### High Priority (must address before scaling to full swarm)

**R5: Add CAN repeaters for 800m distance**

Why: Standard CAN is marginal at 800m; repeaters ensure reliable communication.

Action:
1. Install a CAN repeater at each cluster boundary
2. Each repeater regenerates the signal for the next segment
3. Test communication reliability across the full distance

Effort: $50-100 per repeater, plus 1 day installation and testing.

**R6: Install energy metering at each house**

Why: Prevents social conflict by providing transparency about consumption.

Action:
1. Install kWh meter at each house's AC output
2. Consider smart meters that can report to a central dashboard
3. Make data accessible to all participants
4. Decide whether metering is for transparency only or for billing

Effort: $20-50 per meter, plus 2-4 hours installation per house.

**R7: Write governance document**

Why: Prevents disputes by establishing decision processes in advance.

Action:
1. Gather all participants for a governance discussion
2. Address all the questions listed in the Social Dynamics section
3. Write down the agreed answers
4. Have all participants sign the document
5. Review and update annually

Effort: 1-2 meetings plus writing time.

**R8: Install cluster isolation breakers**

Why: Allows any cluster to be disconnected for maintenance without affecting others.

Action:
1. Install a breaker at each end of each inter-cluster cable
2. Breakers should be lockable in the off position (LOTO - lockout/tagout)
3. Document the procedure for isolating a cluster

Effort: $50-100 per breaker, plus installation.

#### Medium Priority (enhances robustness)

**R9: Install surge protection (SPD)**

Why: Protects against lightning-induced surges.

Action:
1. Install SPD at each cluster entry point
2. Select SPD rated for your system voltage (230V) and expected surge current
3. Include indicator that shows when SPD has operated (needs replacement)
4. Test annually

Effort: $50-100 per SPD.

**R10: Document commissioning procedure**

Why: Ensures consistent, safe commissioning of new houses.

Action:
1. Write step-by-step procedure for adding a house to an existing cluster
2. Include test points and acceptance criteria
3. Include troubleshooting guidance
4. Train at least 2 people on the procedure

Effort: 1 day of documentation.

**R11: Add per-inverter status display**

Why: Makes it easier to see what's happening during operation and debugging.

Action:
1. Add small display (OLED or LCD) to each inverter showing:
   - Output voltage and frequency
   - Current load (watts or amps)
   - Battery SOC (if available via CAN)
   - Fault codes if any
2. Consider LED indicators for key states (running, fault, master/slave)

Effort: $20-50 per inverter, plus firmware work.

**R12: Create fault diagnosis guide**

Why: Helps anyone diagnose problems, not just the original builder.

Action:
1. List all common fault conditions
2. For each, describe:
   - Symptoms (what you see/hear/measure)
   - Possible causes
   - Diagnosis steps
   - Repair procedure
3. Include wiring diagrams and reference photos

Effort: 1-2 days of documentation.

#### Nice to Have (improves user experience)

**R13: Create Home Assistant dashboard**

Why: Provides visibility into whole-system status for all participants.

Action:
1. Set up Home Assistant server (can be Raspberry Pi)
2. Connect to CAN bus (via ESP32 gateway or USB-CAN adapter)
3. Create dashboard showing:
   - Status of each inverter
   - Battery SOC for each house
   - Solar production
   - Consumption per house
   - Historical graphs
4. Make accessible to all participants

Effort: 2-3 days of setup.

**R14: Implement automatic load shedding**

Why: Graceful degradation when capacity is limited.

Action:
1. Define "non-critical" loads that can be shed (e.g., water heater, EV charger)
2. Add smart switches or relays to these loads
3. Program ESP32 to disconnect non-critical loads when:
   - Battery SOC falls below threshold
   - Inverter is overloaded
   - Manual override by user
4. Reconnect automatically when conditions improve

Effort: 1-2 weeks of development plus hardware.

**R15: Consider ring topology for critical installations**

Why: Provides redundant path if one cable is cut.

Action:
1. Run additional cable connecting last cluster back to first: A─B─C─A
2. Requires about 2x the inter-cluster cable length
3. System continues to work even if any one cable is cut
4. More complex: need to handle current flow control to prevent loops

Effort: 2x inter-cluster cable cost plus additional complexity.

---

### 4. IMPLEMENTATION PHASES

This section maps recommendations to the project phases, so you know what to address at each stage.

#### Phase 1: Single Inverter

**Focus:** Get one OzInverter working reliably.

**No cluster/swarm issues at this phase.** Concentrate on:
- Clean sine wave output (low THD)
- Stable voltage under varying load
- Proper overcurrent protection
- Thermal management (heatsink sizing, fan operation)

**Testing to perform:**
- Pure resistive load (heaters): verify voltage regulation
- Motor load (fan, pump): verify startup capability and steady-state operation
- Overload test: verify overcurrent trip protects the inverter

**Success criteria:** Inverter runs reliably for 24 hours under typical household load profile.

#### Phase 2: House Scale

**Focus:** Integrate inverter with battery, solar, and house distribution.

**Key tasks:**
- Define grounding for the house (R1 - partial)
- Connect BMS, verify communication or safe behavior without communication
- Connect MPPT, verify charging works correctly
- Install house distribution board with appropriate breakers
- Connect typical loads, run for extended period

**Testing to perform:**
- Full charge cycle: solar → battery → inverter → loads
- Night operation: battery → inverter → loads
- Low battery: verify BMS limits discharge, inverter reduces output gracefully
- Overload: verify breaker trips, not inverter damage

**Success criteria:** House runs entirely on solar/battery for 1 week without intervention.

#### Phase 3: First Cluster (3-4 houses)

**Focus:** Verify parallel operation of multiple inverters.

**Key tasks:**
- Complete grounding scheme for cluster (R1 - complete)
- Test droop stability with 2 inverters, then 3 (R2)
- Implement master election (R3)
- Calculate and install correct cable sizing (R4)
- Install cluster isolation breakers (R8)
- Add metering (R6)

**Testing to perform:**
- Stable parallel operation under varying load
- Load sharing: apply heavy load to one house, verify others help
- Master failure: turn off master, verify election works
- Inverter failure: turn off one inverter, verify house draws from bus
- Cable fault: disconnect cable, verify isolation works

**Success criteria:** Cluster runs reliably for 1 month, including simulated fault scenarios.

#### Phase 4: Full Swarm (12 houses across 3 clusters)

**Focus:** Scale to full system, establish governance and monitoring.

**Key tasks:**
- Install CAN repeaters (R5)
- Complete metering for all houses (R6)
- Write governance document (R7)
- Document commissioning procedure (R10)
- Create fault diagnosis guide (R12)
- Set up Home Assistant dashboard (R13)

**Commissioning approach:**
1. Commission each cluster independently first
2. Then connect clusters one at a time
3. After each addition, verify stability for 24 hours before adding more

**Testing to perform:**
- Inter-cluster power sharing
- Communication across full 800m
- System behavior when one cluster is isolated

**Success criteria:** Full swarm operates reliably for 3 months. All participants understand governance and can access monitoring.

---

### 5. OPEN QUESTIONS

These questions need answers before or during implementation. Document the answers as decisions are made.

**Q1: Where exactly is the neutral-to-ground bond located?**
Options:
- One N-G bond per cluster, at the cluster junction box
- One N-G bond per cluster, at the designated "master" house
- One N-G bond for the entire swarm, at a central location

Current recommendation: One per cluster at junction box. Document your decision.

**Q2: What is the master election tiebreaker if two ESP32s have the same priority?**
Options:
- Higher MAC address wins
- First to broadcast wins
- Random backoff resolves it statistically

Current recommendation: Use ESP32 MAC address as unique ID. Higher MAC wins. Document your decision.

**Q3: What droop coefficient provides stability?**
This must be determined experimentally. Starting point:
- Try 0.1Hz per kW of load (frequency drops 0.1Hz for each kW the inverter supplies)
- Adjust based on stability testing

Document the final value that works.

**Q4: Is 4mm² cable actually sufficient for your distances?**
Calculate for your actual distances and required power transfer. You may need 10mm² or 16mm² for inter-cluster runs.

Document the calculation and selected cable size.

**Q5: Is metering for transparency only, or will there be billing?**
Options:
- Display only (everyone sees their data, no payments)
- Periodic balancing (net consumers pay net producers)
- Real-time billing (smart meter tracks and bills continuously)

Document your community's decision.

**Q6: What is the governance decision process?**
Options:
- Unanimous consent (everyone must agree)
- Majority vote (>50% agreement)
- Supermajority (>67% agreement)
- Designated leader decides

Different issues might have different processes. Document the rules.

**Q7: What happens when a house wants to leave the swarm?**
Issues to address:
- Do they get refunded for infrastructure contributions?
- Who pays for system reconfiguration?
- What is the notice period?

Document the exit process.

**Q8: What does homeowner insurance cover?**
Contact your insurance provider and ask specifically:
- Is DIY electrical equipment covered?
- What documentation do they require?
- Are there exclusions or conditions?

Document the answer and any required actions.

---

### 6. FAILURE MODE REFERENCE TABLE

This table serves as a quick reference for diagnosing problems.

| Symptom | Possible Cause | Check This | Likely Fix |
|---------|---------------|------------|------------|
| One house has no power | Inverter fault | Inverter LEDs/display, breaker position | Reset breaker, check inverter fault codes |
| One house has no power, inverter OK | Main breaker tripped | Breaker position | Investigate cause (overload? fault?), reset |
| Multiple houses have no power | Inter-cluster cable issue | Cable continuity, breaker positions | Repair cable or reset breakers |
| Frequency unstable (±0.2Hz+) | Droop instability | Droop settings, number of inverters | Reduce droop slope, check settings match |
| Frequency slowly drifting | Master not syncing | CAN communication, master status | Check master election, CAN connectivity |
| Inverters out of phase | Lost sync | CAN communication | Verify CAN, check for master |
| Low voltage at far end of cluster | Voltage drop in cable | Cable size, distance, current | Reduce current or upgrade cable |
| Battery not charging | MPPT fault or solar issue | MPPT display, solar voltage | Check MPPT, check panel connections |
| Battery draining fast | Excessive load or BMS issue | Load measurement, BMS status | Check for unexpected loads, check BMS |
| CAN communication errors | Cable issue or termination | Error counters, termination resistors | Repair cable, fix termination |
| Breakers nuisance tripping | Ground fault or overload | RCD indicators, load measurement | Find fault, reduce load |
| Buzzing sound from inverter | Overload or loose connection | Load measurement, physical inspection | Reduce load, tighten connections |
| House getting power but not contributing | Inverter not paralleling | Inverter sync status | Check CAN, verify sync working |
| One house always low battery | Higher consumption or less solar | kWh meter, solar production | Reduce consumption or improve solar |

---

### 7. LESSONS FROM VICTRON AND AFRICAN MICROGRIDS

This section compares the swarm-grid architecture against proven strategies from Victron Energy (the industry leader in off-grid systems) and lessons learned from African DC microgrid deployments (the largest real-world laboratory for community solar).

#### What Victron Does That We Should Learn From

**Symmetrical Wiring Requirement**

Victron's parallel installation manual is explicit: "For units in parallel, both the DC and AC wiring needs to be symmetrical per phase: use the same length, type and cross-section to every unit." They recommend using busbars before and after inverters to ensure equal current distribution.

*Lesson for swarm-grid:* Your current design doesn't specify wiring symmetry. Within a cluster, the cable from House 1 to the bus might be 10m while House 3 is 30m. This asymmetry means House 1's inverter "sees" a different impedance than House 3's, which can cause unequal load sharing and potential oscillation.

*Recommendation:* Either use a central busbar in each cluster with equal-length cables to each house, or document and accept the asymmetry (since sharing is rare anyway).

**Identical Hardware and Firmware**

Victron requires: "All units in one system must be the same type and firmware version, including the same size, system voltage, and feature set."

*Lesson for swarm-grid:* Your DIY inverters will inevitably have slight variations - different component batches, different assembly, different calibration. These variations may cause load sharing problems.

*Recommendation:* Develop a calibration procedure that measures and adjusts each inverter's droop coefficient to match others. Document firmware versions and ensure all ESP32s run identical code.

**Individual Fusing with Mechanical Linking**

Victron recommends: "Each unit needs to be fused individually. Consider using mechanically connected fuses, so that if one trips, they all trip."

*Lesson for swarm-grid:* Your 63A breakers per house are individual but not linked. If one house has a fault, its breaker trips but others continue feeding power into the fault.

*Recommendation:* For intra-cluster connections, consider adding a shunt-trip mechanism where any house breaker tripping sends a signal to trip the cluster bus. Or accept this risk given the rarity of faults.

**VE.Bus Daisy-Chain Communication**

Victron uses proprietary VE.Bus (RJ-45) to daisy-chain inverters, similar to your CAN bus approach. They explicitly state: "Do not use terminators in the VE.Bus network."

*Lesson for swarm-grid:* CAN bus does require termination (120Ω at each end), which is different from VE.Bus. Ensure you're following CAN specifications, not Victron VE.Bus specifications.

**The 1:1 Rule (Factor of One)**

Victron's critical rule: "The inverter power on the output of a Victron inverter cannot be greater than the inverter's rating." This prevents AC-coupled systems from having more generation than the inverter can absorb.

*Lesson for swarm-grid:* In your architecture, each house's inverter is the only generation source for that house, so this rule is automatically satisfied. But be aware that if you ever add AC-coupled solar inverters to the system, this rule applies.

**Fleet Management with VRM**

Victron provides the VRM (Victron Remote Monitoring) portal that allows: "For installers and fleet managers, the installation overview can provide high level summary data and filtering for thousands of systems."

*Lesson for swarm-grid:* Your Home Assistant dashboard idea is good, but Victron's approach shows the value of:
- Alerts and alarms (email/push when something is wrong)
- Remote firmware updates
- Historical data logging
- Geofencing (know if equipment moves)

*Recommendation:* Design your monitoring system to include proactive alerts, not just passive dashboards. The ESP32 should send alerts via MQTT/Telegram when anomalies occur.

**Training Requirement**

Victron explicitly warns: "Parallel and Multiphase systems are complex. We do not support or recommend that untrained and/or inexperienced installers work on these size systems."

*Lesson for swarm-grid:* Your assumption S2 (sufficient technical skills exist) is critical. Victron requires formal training for their systems. Your DIY system is arguably more complex because it lacks Victron's automated configuration.

*Recommendation:* Create a training program for your community. Document everything extensively. Consider having at least one person attend actual Victron training to understand parallel system principles.

---

#### What African DC Microgrids Teach Us

**Okra Solar's DC Mesh Network (Cambodia, deployed in Africa)**

Okra Solar created a system where 140 households share power through a DC mesh network. Key features:

- **Peer-to-peer topology:** Each node links "directly and dynamically to each other without going through a top-down hierarchy"
- **Extra-low voltage DC:** Uses safe voltage levels, reducing shock hazard and enabling untrained locals to maintain the system
- **Minimal cabling:** "Cables interconnecting households can be a tenth the size of AC mini-grid cabling"
- **40% cost reduction** compared to AC microgrids
- **Local maintenance:** "Operations don't require trained electricians—local residents handle maintenance using the remote monitoring platform"

*Lessons for swarm-grid:*

1. **DC might actually be simpler:** Your decision to use AC for inter-house sharing requires careful synchronization. DC mesh networks avoid this entirely. The trade-off is that AC allows standard appliances while DC requires efficient DC appliances.

2. **Remote monitoring is essential:** Okra's software "diagnoses issues, optimizes network profitability and communicates operational issues to local community agents." Your system needs similar capabilities.

3. **Local agents matter:** Having designated community members trained to respond to alerts dramatically reduces downtime and cost.

**Pay-As-You-Go (PAYGO) Solar Lessons**

African PAYGO solar companies (M-KOPA, Azuri, d.light) have electrified millions of households. Key lessons:

**Prepaid metering solves the free-rider problem:**
- In Mali, switching to prepaid increased payment rates from 82% to 99%
- "Poorer households buy electricity more often, in smaller increments, and are most likely to buy on payday"
- The prepaid model aligns consumption with ability to pay

*Lesson for swarm-grid:* Your governance document should consider a prepaid or credit-based system rather than pure sharing. Even if money doesn't change hands, a "credit" system where each house has an allocation prevents abuse.

**Mobile money integration is crucial:**
- M-PESA (Kenya's mobile money) enables payments without bank accounts
- Scratch cards work where mobile money doesn't
- Remote lockout/unlock based on payment status

*Lesson for swarm-grid:* Your ESP32 could implement a lockout function - if a house exceeds its allocation from the shared bus, it gets reduced priority or temporary disconnection. This is technically complex but solves the tragedy of commons.

**Customer education is essential:**
- "In the early days of adoption, consumers face hurdles using the solar systems, hence the need to invest in education"
- Companies invested heavily in user training

*Lesson for swarm-grid:* Plan for user education. Create simple guides, hold training sessions, establish support channels.

**Rwanda Microgrid Feasibility Studies**

Research on village microgrids in Rwanda found:

- "There is no 'one size fits all' approach that will work in every African country, let alone every small community"
- Network topology must match village geography
- "Field deployment of DC microgrids in rural places is often expensive and logistically difficult. Hence, it is necessary to develop a test bench to test and thoroughly validate the proper operation of a microgrid before its deployment"

*Lesson for swarm-grid:* Your Phase 3 (cluster testing) before Phase 4 (full swarm) is correct. Never deploy untested configurations.

**Environmental and End-of-Life Concerns**

Multiple sources noted: "Disposal of energy storage devices and other components raise environmental concerns. Challenges associated with recycling devices like batteries."

*Lesson for swarm-grid:* Plan for battery end-of-life from the beginning. LiFePO4 cells last 10-15 years. Where will 60-120kWh of batteries go? Who pays for disposal?

---

#### Specific Recommendations from This Research

**R16: Implement symmetrical wiring within clusters**

Draw a wiring diagram with equal cable lengths from each house to a central busbar. If true symmetry is impractical, at minimum document the asymmetry and monitor for load sharing problems.

**R17: Create calibration procedure for droop matching**

Each inverter should be bench-tested and its droop coefficient adjusted to match a reference. Document the procedure and required test equipment.

**R18: Add proactive alerting to monitoring system**

Don't rely on humans checking dashboards. ESP32s should send alerts via MQTT, Telegram, or email when:
- Frequency deviation exceeds threshold
- Any house draws from bus for more than X minutes continuously
- CAN communication errors increase
- Battery SOC falls below threshold
- Any fault code appears

**R19: Designate and train "community agents"**

Following Okra Solar's model, designate 2-3 community members as first responders for system issues. Train them on:
- Reading the monitoring dashboard
- Basic troubleshooting (breaker resets, visual inspection)
- When to escalate to the technical expert

**R20: Consider consumption allocation system**

Rather than unlimited sharing, give each house a monthly "credit" of kWh they can draw from the bus. Track actual usage. Houses that exceed their allocation:
- Get a warning
- Lose priority (droop coefficient adjusted so they contribute more than they draw)
- Pay into the maintenance fund

This aligns incentives without requiring true billing.

**R21: Document battery end-of-life plan**

Include in governance document:
- Expected battery lifetime (10-15 years for LiFePO4)
- Who pays for replacement?
- How are dead batteries disposed/recycled?
- Is there a sinking fund for future replacement?

**R22: Consider DC for intra-cluster sharing**

Your current AC-based sharing requires phase synchronization. An alternative for future consideration: keep DC buses within each cluster for sharing, use AC only for inter-cluster and household loads. This eliminates sync complexity at cluster level but requires DC-DC converters.

This is a major architectural change - not recommended for initial deployment, but worth considering for future versions if sync proves problematic.

---

#### Comparison Table: Your Architecture vs. Victron vs. African Microgrids

| Aspect | Your Swarm-Grid | Victron Best Practice | African Microgrid Lessons |
|--------|-----------------|----------------------|---------------------------|
| **Synchronization** | CAN bus master/slave | VE.Bus proprietary | DC eliminates need for sync |
| **Wiring** | Not specified symmetry | Requires symmetry | Mesh allows any topology |
| **Metering** | Optional (recommended) | Per-device monitoring | Prepaid is essential |
| **Communication** | CAN + ThingSet | VE.Bus + VRM cloud | IoT mesh + cloud platform |
| **Fault isolation** | Individual breakers | Linked breakers recommended | Per-node isolation |
| **User training** | Assumed available | Formal training required | Local agents + education |
| **Governance** | To be defined | N/A (commercial product) | Critical success factor |
| **Remote monitoring** | Home Assistant planned | VRM portal (mature) | Essential, drives maintenance |
| **Cost sharing** | Equal or metered | N/A | PAYGO/prepaid proven |

---

#### Sources

**Victron Energy:**
- [Parallel and Three-Phase VE.Bus Systems Manual](https://www.victronenergy.com/live/ve.bus:manual_parallel_and_three_phase_systems)
- [ESS Design and Installation Manual Rev 11 (10/2024)](https://www.victronenergy.com/upload/documents/Energy_Storage_System/6292-ESS_design_and_installation_manual-pdf-en.pdf)
- [VRM Portal Documentation](https://www.victronenergy.com/media/pg/VRM_Portal_manual/en/introduction.html)
- [Cerbo GX Product Page](https://www.victronenergy.com/communication-centres/cerbo-gx)
- [Mini-grid for Extended Family Case Study](https://www.victronenergy.com/blog/2024/01/19/mini-grid-for-extended-family/)

**African Microgrids:**
- [Okra Solar DC Mesh Network in Cambodia](https://microgridnews.com/dc-mesh-network-provides-low-cost-electricity-in-rural-cambodia/) - also deployed in Africa
- [ODI: How Solar Mini-Grids Can Bring Cheap Electricity to Rural Africa](https://odi.org/en/insights/how-solar-mini-grids-can-bring-cheap-green-electricity-to-rural-africa/)
- [IRENA: Pay-As-You-Go Models Innovation Brief](https://www.irena.org/-/media/Files/IRENA/Agency/Publication/2020/Jul/IRENA_Pay-as-you-go_models_2020.pdf)
- [Energypedia: Fee-For-Service and PAYGO Concepts](https://energypedia.info/wiki/Fee-For-Service_or_Pay-As-You-Go_Concepts_for_Photovoltaic_Systems)
- [WRI: Pay-As-You-Go Solar Could Electrify Rural Africa](https://www.wri.org/insights/pay-you-go-solar-could-electrify-rural-africa)
- [IEEE: DC Microgrid Network Topologies for Rwanda](https://ieeexplore.ieee.org/document/9364514/)
- [ResearchGate: Low Voltage DC Microgrid for South Africa](https://www.researchgate.net/publication/279023684_Design_of_a_low_voltage_DC_microgrid_system_for_rural_electrification_in_South_Africa)
- [MDPI: Peer-to-Peer Energy Trading in AC and DC Microgrids](https://www.mdpi.com/1996-1073/12/19/3709)

---

### 8. DOCUMENT HISTORY

| Date | Change | Author |
|------|--------|--------|
| 2024-12-24 | Initial version | Claude + User |
| 2024-12-24 | Added exhaustive detail to all sections | Claude + User |
| 2024-12-24 | Added Victron and African microgrid lessons (R16-R22) | Claude + User |
| 2024-12-24 | Added Critique Process section with periodic run procedure | Claude + User |
| 2024-12-24 | First critique run: reviewed tiered bus architecture, identified N1-N5 new issues | Claude + User |

---

### 9. NEXT CRITIQUE SCHEDULED

**Trigger:** Run critique when any of these occur:
- Major README architecture update
- Before starting Phase 3 (first cluster)
- Before starting Phase 4 (full swarm)
- After completing droop stability testing (R2)
- After defining grounding scheme (R1)

---

*This document is a living critique. Update it as issues are resolved, new ones are discovered, or decisions are made.*
