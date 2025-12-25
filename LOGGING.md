# Microgrid Logging & Debugging Strategy

Deep observability for distributed microgrid systems. Enables AI-assisted failure analysis and root cause debugging.

---

## 3-Tier Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           LOGGING ARCHITECTURE                               │
│                                                                              │
│  TIER 3: SWARM (Central)                                                    │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │  Raspberry Pi 4 / Mini PC                                           │    │
│  │  ├── Home Assistant (coordination, alerts, dashboards)              │    │
│  │  ├── InfluxDB (time-series storage, 1+ year retention)              │    │
│  │  ├── Grafana (visualization, anomaly dashboards)                    │    │
│  │  └── AI Analysis (Claude API / local LLM for failure analysis)      │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                              ▲                                               │
│                              │ WiFi/Ethernet (aggregated data)              │
│         ┌────────────────────┼────────────────────┐                         │
│         ▼                    ▼                    ▼                         │
│  TIER 2: CLUSTER GATEWAYS                                                   │
│  ┌──────────────┐     ┌──────────────┐     ┌──────────────┐                │
│  │ Gateway A    │     │ Gateway B    │     │ Gateway C    │                │
│  │ ESP32 + SD   │     │ ESP32 + SD   │     │ ESP32 + SD   │                │
│  │ Local buffer │     │ Local buffer │     │ Local buffer │                │
│  └──────┬───────┘     └──────┬───────┘     └──────┬───────┘                │
│         │ CAN                │ CAN                │ CAN                     │
│    ┌────┴────┐          ┌────┴────┐          ┌────┴────┐                   │
│    ▼    ▼    ▼          ▼    ▼    ▼          ▼    ▼    ▼                   │
│                                                                              │
│  TIER 1: HOUSE NODES                                                        │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐                           │
│  │ House 1 │ │ House 2 │ │ House 3 │ │ House 4 │  (× 3 clusters)          │
│  │ ESP32   │ │ ESP32   │ │ ESP32   │ │ ESP32   │                           │
│  │ SD card │ │ SD card │ │ SD card │ │ SD card │                           │
│  │ 7 days  │ │ 7 days  │ │ 7 days  │ │ 7 days  │                           │
│  └─────────┘ └─────────┘ └─────────┘ └─────────┘                           │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

**Key principle:** Each tier stores locally first, then aggregates upward. If network fails, no data is lost.

---

## Tier 1: House Node Logging

Each house ESP32 logs to local SD card at high frequency.

### Data Points

| Data Point | Frequency | Source | Purpose |
|------------|-----------|--------|---------|
| Inverter frequency | 100Hz | EG8010 | Droop state, load indicator |
| Inverter voltage | 10Hz | ADC | Voltage stability |
| Inverter current | 10Hz | CT sensor | Power output |
| DC bus voltage | 10Hz | ADC | Battery/MPPT state |
| MOSFET temperatures | 1Hz | NTC | Thermal protection |
| BMS cell voltages (×16) | 1Hz | UART/CAN | Cell health |
| BMS cell temps | 1Hz | BMS | Thermal monitoring |
| BMS current | 10Hz | BMS | Charge/discharge rate |
| BMS SOC | 0.1Hz | BMS | State of charge |
| MPPT input V/I | 1Hz | MPPT | Solar production |
| MPPT output V/I | 1Hz | MPPT | Charge rate |
| CAN bus errors | event | MCP2515 | Communication health |
| Fault events | event | firmware | Protection triggers |

### Storage

- **Medium:** 32GB SD card
- **Retention:** 7-day rolling buffer
- **Size:** ~50MB/day
- **Format:** Binary log with timestamps (efficient)

### Hardware Addition

```
ESP32 (existing in inverter)
├── SD card module (SPI) ─── $5
├── 32GB SD card ─────────── $5
└── RTC DS3231 ───────────── $3 (battery-backed time)
```

---

## Tier 2: Cluster Gateway Logging

Gateways aggregate house data and monitor inter-cluster state.

### Data Points

| Data Point | Frequency | Purpose |
|------------|-----------|---------|
| Cluster bus frequency | 10Hz | Cluster load state |
| Swarm bus frequency | 10Hz | Swarm load state |
| Frequency delta | 10Hz | Sync monitoring |
| Contactor state | event | Gateway decisions |
| House summaries (×4) | 1Hz | Aggregated from Tier 1 |
| Inter-house power flows | 1Hz | Calculated from currents |
| CAN message statistics | 1Hz | Bus health |

### Storage

- **Medium:** 64GB SD card
- **Retention:** 30-day rolling buffer
- **Size:** ~100MB/day

### Aggregation Logic

```
Every 1 second:
  For each house in cluster:
    Pull: SOC, power, frequency, fault status
    Store: aggregated cluster state

  Calculate:
    Total cluster power = sum(house powers)
    Cluster health = min(house health scores)
    Power flow direction per house
```

---

## Tier 3: Central System

Long-term storage, visualization, and AI analysis.

### Components

| Component | Purpose |
|-----------|---------|
| Home Assistant | Coordination, alerts, automations |
| InfluxDB | Time-series database (1+ year retention) |
| Grafana | Dashboards, visualization |
| AI Pipeline | Failure analysis, pattern detection |

### Data Retention Policy

| Data Type | Resolution | Retention |
|-----------|------------|-----------|
| Real-time | 1 second | 24 hours |
| Short-term | 1 minute averages | 30 days |
| Long-term | 1 hour averages | 1 year |
| Events/faults | Full resolution | Forever |
| Fault snapshots | Full resolution | Forever |
| Daily summaries | Aggregated | Forever |

### Hardware

```
Raspberry Pi 4 (4GB+) or Mini PC
├── Home Assistant OS
├── InfluxDB addon
├── Grafana addon
├── 256GB+ SSD
└── UPS backup (critical!)
```

---

## Data Flow

```
House ESP32                    Gateway ESP32                 Central (HA + InfluxDB)
     │                              │                              │
     │──── CAN: Real-time ─────────►│                              │
     │     (100Hz sync, 10Hz data)  │                              │
     │                              │                              │
     │──── SD: Local log ──────────►│ (pulled on demand)           │
     │     (7 days buffer)          │                              │
     │                              │──── WiFi: Aggregated ───────►│
     │                              │     (1Hz summaries)          │
     │                              │                              │
     │                              │──── WiFi: Event push ───────►│
     │                              │     (faults, anomalies)      │
     │                              │                              │
     │◄─── CAN: Time sync ──────────│◄─── NTP sync ───────────────│
     │     (from master)            │                              │
```

---

## Time Synchronization

**Critical for correlating events across distributed system.**

```
NTP Server (internet or local GPS)
         │
         ▼
    Central HA (Raspberry Pi)
         │
         ▼ (WiFi broadcast every 10 sec)
    Gateway ESP32s
         │
         ▼ (CAN broadcast every 1 sec)
    House ESP32s

Target accuracy: ±10ms across entire swarm
```

Each node stores timestamps in UTC. RTC modules with battery backup maintain time during power loss.

---

## Fault Snapshot System

When ANY protection triggers, capture high-resolution snapshot:

```
┌────────────────────────────────────────────────────────────────┐
│  FAULT SNAPSHOT                                                │
│                                                                │
│  Trigger: Any protection event (overcurrent, overvoltage,     │
│           undervoltage, overtemp, communication loss, etc.)    │
│                                                                │
│  Captured:                                                     │
│  ├── Pre-fault:  10 seconds @ 100Hz                           │
│  ├── Fault:      Exact timestamp + fault code + description   │
│  └── Post-fault: 10 seconds @ 100Hz                           │
│                                                                │
│  Data included:                                                │
│  ├── Inverter: freq, V, I, temps (all channels)               │
│  ├── BMS: all 16 cell voltages, current, temps, SOC           │
│  ├── MPPT: input/output V/I                                   │
│  ├── CAN: all messages on bus during window                   │
│  └── Neighbors: state of other houses at fault time           │
│                                                                │
│  Size: ~500KB per snapshot                                     │
│  Storage: Never deleted (permanent record)                     │
└────────────────────────────────────────────────────────────────┘
```

### Fault Codes

| Code | Category | Description |
|------|----------|-------------|
| F001 | Inverter | Output overcurrent |
| F002 | Inverter | Output overvoltage |
| F003 | Inverter | Output undervoltage |
| F004 | Inverter | MOSFET overtemperature |
| F005 | Inverter | DC bus overvoltage |
| F006 | Inverter | DC bus undervoltage |
| F010 | BMS | Cell overvoltage |
| F011 | BMS | Cell undervoltage |
| F012 | BMS | Cell overtemperature |
| F013 | BMS | Overcurrent charge |
| F014 | BMS | Overcurrent discharge |
| F015 | BMS | Cell imbalance >100mV |
| F020 | MPPT | Input overvoltage |
| F021 | MPPT | Output overcurrent |
| F030 | CAN | Communication timeout |
| F031 | CAN | Message CRC error |
| F040 | Gateway | Sync failure |
| F041 | Gateway | Contactor failure |

---

## AI Analysis Pipeline

### Automatic Analysis on Fault

```python
def on_fault_detected(snapshot):
    # 1. Store snapshot immediately
    influxdb.store(snapshot)

    # 2. Query similar past faults
    similar = influxdb.query(f"""
        SELECT * FROM faults
        WHERE fault_code = '{snapshot.code}'
        AND house = '{snapshot.house}'
        ORDER BY time DESC LIMIT 10
    """)

    # 3. Get neighboring house state
    neighbors = influxdb.query(f"""
        SELECT * FROM house_data
        WHERE cluster = '{snapshot.cluster}'
        AND time BETWEEN '{snapshot.time - 10s}' AND '{snapshot.time + 10s}'
    """)

    # 4. Send to AI for analysis
    analysis = claude_api.analyze(f"""
    Analyze this microgrid fault for root cause:

    FAULT: {snapshot.code} - {snapshot.description}
    HOUSE: {snapshot.house} in Cluster {snapshot.cluster}
    TIME: {snapshot.time}

    PRE-FAULT DATA (10 seconds before):
    {format_timeseries(snapshot.pre_fault)}

    FAULT MOMENT:
    {format_state(snapshot.fault_state)}

    POST-FAULT DATA (10 seconds after):
    {format_timeseries(snapshot.post_fault)}

    NEIGHBORING HOUSES AT FAULT TIME:
    {format_neighbors(neighbors)}

    SIMILAR PAST FAULTS:
    {format_history(similar)}

    QUESTIONS:
    1. What is the likely root cause?
    2. Did this fault originate here or propagate from neighbor?
    3. What pattern preceded this fault?
    4. Is this a recurring issue?
    5. Recommended investigation steps?
    6. Recommended fix?
    """)

    # 5. Store analysis
    influxdb.store_analysis(snapshot.id, analysis)

    # 6. Alert operator
    home_assistant.notify(
        title=f"Fault {snapshot.code} in {snapshot.house}",
        message=analysis.summary,
        priority="high"
    )
```

### Pattern Detection Queries

```sql
-- All faults in last 30 days
SELECT * FROM faults
WHERE time > now() - 30d
ORDER BY time DESC

-- Correlation: cluster state when specific house faulted
SELECT * FROM house_data
WHERE cluster = 'A'
AND time BETWEEN '2024-01-15T10:00:00Z' AND '2024-01-15T10:00:20Z'
ORDER BY house, time

-- Pattern: cell voltage drift before BMS faults
SELECT
  house,
  max(cell_voltage) - min(cell_voltage) as delta
FROM bms_data
WHERE time > now() - 7d
GROUP BY time(1h), house
HAVING delta > 0.05

-- Anomaly: frequency deviations outside normal band
SELECT * FROM inverter_data
WHERE abs(frequency - 50.0) > 0.5
AND time > now() - 24h
ORDER BY time

-- Recurring: same fault code multiple times
SELECT
  fault_code,
  house,
  count(*) as occurrences
FROM faults
WHERE time > now() - 30d
GROUP BY fault_code, house
HAVING occurrences > 1
ORDER BY occurrences DESC

-- Energy flow: who supplied whom over time
SELECT
  house,
  mean(power_to_bus) as avg_export,
  mean(power_from_bus) as avg_import
FROM house_data
WHERE time > now() - 7d
GROUP BY time(1h), house
```

---

## Debugging Scenarios

### Scenario 1: Inverter Tripped, Unknown Cause

```
1. Pull fault snapshot from house SD card
2. Check pre-fault data:
   - Was frequency dropping? (overload)
   - Was DC voltage dropping? (battery depleted)
   - Were temps rising? (thermal)
3. Check neighbors at same time:
   - Did they also see frequency drop? (shared load issue)
   - Were they pushing power? (helping before trip)
4. AI analysis: correlate patterns
```

### Scenario 2: Cluster Disconnected from Swarm

```
1. Pull gateway logs
2. Check contactor state changes
3. Check frequency delta (cluster vs swarm)
4. Was it intentional (protection) or fault?
5. Check CAN bus for communication errors
6. AI analysis: was gateway decision correct?
```

### Scenario 3: Cell Imbalance Growing Over Weeks

```
1. Query long-term cell voltage trends
2. Identify which cell is drifting
3. Check correlation with:
   - Temperature patterns
   - Charge/discharge cycles
   - Balancer activity
4. AI analysis: predict failure timeline
```

### Scenario 4: Intermittent Power Sharing Failures

```
1. Query all frequency data across cluster
2. Look for:
   - Droop coefficient mismatches
   - Phase sync issues
   - CAN communication gaps
3. Correlate with:
   - Time of day (load patterns)
   - Weather (solar production)
4. AI analysis: identify root cause pattern
```

---

## Grafana Dashboard Panels

### Overview Dashboard

```
┌─────────────────────────────────────────────────────────────────┐
│  SWARM OVERVIEW                                    [Live] [24h] │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐               │
│  │ CLUSTER A   │ │ CLUSTER B   │ │ CLUSTER C   │               │
│  │ ████████░░  │ │ ██████████  │ │ ███████░░░  │  SOC bars     │
│  │ 78% SOC     │ │ 95% SOC     │ │ 65% SOC     │               │
│  │ 12.3 kW     │ │ 8.1 kW      │ │ 15.7 kW     │  Power        │
│  │ 49.92 Hz    │ │ 50.01 Hz    │ │ 49.85 Hz    │  Frequency    │
│  └─────────────┘ └─────────────┘ └─────────────┘               │
│                                                                 │
│  POWER FLOW:  A ←── 2.1 kW ─── B ──── 1.5 kW ──► C             │
│                                                                 │
├─────────────────────────────────────────────────────────────────┤
│  FREQUENCY (all houses)                           [Last 1 hour] │
│  50.2 ─┬─────────────────────────────────────────────────────── │
│  50.0 ─┼─══════════════════════════════════════════════════════ │
│  49.8 ─┼─────────────────────────────────────────────────────── │
│  49.6 ─┴─────────────────────────────────────────────────────── │
│        00:00    00:15    00:30    00:45    01:00                │
│                                                                 │
├─────────────────────────────────────────────────────────────────┤
│  RECENT EVENTS                                                  │
│  ● 10:23 - House 7 cell delta > 50mV (warning)                 │
│  ● 09:45 - Gateway B opened contactor (freq mismatch)          │
│  ● 08:12 - House 3 reached 100% SOC                            │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### House Detail Dashboard

```
┌─────────────────────────────────────────────────────────────────┐
│  HOUSE 7 DETAIL                                    [Live] [24h] │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  INVERTER              BMS                    MPPT              │
│  ┌──────────────┐     ┌──────────────┐      ┌──────────────┐   │
│  │ 230.2V       │     │ SOC: 72%     │      │ In: 2.3 kW   │   │
│  │ 49.95 Hz     │     │ 51.2V        │      │ Out: 2.1 kW  │   │
│  │ 3.2 kW       │     │ 45A disch    │      │ Eff: 91%     │   │
│  │ Temp: 45°C   │     │ Temp: 28°C   │      │              │   │
│  └──────────────┘     └──────────────┘      └──────────────┘   │
│                                                                 │
│  CELL VOLTAGES                                                  │
│  3.25 ─┬─────────────────────────────────────────────────────── │
│  3.20 ─┼─■■■■■■■■■■■■■■■■─────────────────────────────────────── │
│  3.15 ─┴─1─2─3─4─5─6─7─8─9─10─11─12─13─14─15─16                 │
│        Cell number                                              │
│                                                                 │
│  POWER (24h)                                                    │
│  +6kW ─┬─────────────────────────────────────────────────────── │
│     0 ─┼─────────═══════════════════════════─────────────────── │
│  -6kW ─┴─────────────────────────────────────────────────────── │
│        00:00  04:00  08:00  12:00  16:00  20:00  24:00         │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Fault Analysis Dashboard

```
┌─────────────────────────────────────────────────────────────────┐
│  FAULT ANALYSIS: F006 @ House 3                   [2024-01-15] │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  TIMELINE (±10 sec around fault)                                │
│                                                                 │
│  DC Bus V  ─┬────────────────╲                                  │
│    52V     ─┤                 ╲______ FAULT                     │
│    48V     ─┤                        ╲                          │
│    44V     ─┴─────────────────────────╲_____                    │
│             -10s    -5s    0s    +5s    +10s                    │
│                                                                 │
│  Inv Freq  ─┬───────────────────────────────                    │
│    50Hz    ─┤═══════════════════╗                               │
│    49Hz    ─┤                   ║                               │
│    48Hz    ─┴───────────────────╚════════════                   │
│                                                                 │
│  AI ANALYSIS:                                                   │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │ Root cause: Battery reached LVP (low voltage protection) │   │
│  │                                                          │   │
│  │ Evidence:                                                │   │
│  │ • DC bus dropped from 51.2V to 44.8V over 8 seconds     │   │
│  │ • BMS current was 95A (high discharge)                  │   │
│  │ • Cell 14 reached 2.85V first (weakest cell)            │   │
│  │ • Neighbors were not affected (isolated issue)          │   │
│  │                                                          │   │
│  │ Recommendation:                                          │   │
│  │ • Check Cell 14 capacity vs others                      │   │
│  │ • Verify active balancer is working                     │   │
│  │ • Consider reducing max discharge current               │   │
│  └──────────────────────────────────────────────────────────┘   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Hardware Cost Summary

| Component | Per Unit | Quantity | Total |
|-----------|----------|----------|-------|
| House: SD module | $5 | 12 | $60 |
| House: 32GB SD card | $5 | 12 | $60 |
| House: RTC DS3231 | $3 | 12 | $36 |
| Gateway: ESP32 | $10 | 3 | $30 |
| Gateway: SD module | $5 | 3 | $15 |
| Gateway: 64GB SD card | $10 | 3 | $30 |
| Gateway: RTC DS3231 | $3 | 3 | $9 |
| Central: Raspberry Pi 4 | $80 | 1 | $80 |
| Central: 256GB SSD | $30 | 1 | $30 |
| Central: Case + PSU | $20 | 1 | $20 |
| **Total** | | | **$370** |

---

## Implementation Phases

| Phase | What | Duration | Outcome |
|-------|------|----------|---------|
| 1 | House SD logging | 2 weeks | Local fault capture works |
| 2 | Time sync (RTC + NTP) | 1 week | All nodes synchronized |
| 3 | Central InfluxDB + Grafana | 1 week | Dashboards visible |
| 4 | Gateway aggregation | 2 weeks | Cluster-level correlation |
| 5 | Fault snapshot system | 2 weeks | Automatic capture on fault |
| 6 | AI analysis pipeline | 2 weeks | Automated debugging |

---

## References

- [Home Assistant InfluxDB Integration](https://www.home-assistant.io/integrations/influxdb/)
- [ESP32 Logging Library](https://docs.espressif.com/projects/esp-idf/en/latest/esp32/api-reference/system/log.html)
- [ESP32 CAN Logger](https://github.com/ubx/CAN-Logger)
- [HA Vibecode Agent](https://github.com/Coolver/home-assistant-vibecode-agent) - AI-assisted HA configuration
- [ESPHome JK-BMS Integration](https://github.com/syssi/esphome-jk-bms)
