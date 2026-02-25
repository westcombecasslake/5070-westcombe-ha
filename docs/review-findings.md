# System Review Findings

**Purpose:** Notes from subsystem reviews for walkthrough and discussion. No changes made.
**Format:** Flag = needs discussion before any action. Info = observation only.

---

## Safety Subsystems — smokeco.yaml, basementleaksensors.yaml, sumppump.yaml

**Reviewed:** February 2026

---

### smokeco.yaml

**RESOLVED — Missing automation IDs (all 9 automations)**
IDs added February 2026:
- smoke_co_detector_offline
- smoke_co_detector_online
- smoke_co_smoke_detected
- smoke_co_co_detected
- smoke_co_smoke_cleared
- smoke_co_co_cleared
- smoke_co_emergency_response
- smoke_co_all_clear_recovery
- smoke_co_daily_status_summary

**FLAG — Emergency Response uses `climate.ecobee` entity for fan shutoff**
The Emergency Response automation calls `climate.set_fan_mode` on `climate.ecobee` (entity_id) to shut down the HVAC fan during a smoke/CO event. Builddecisions rule 8 states fan control must use `device_id: c38113c2d02da790b689d4a3bdc4b5e1` only via the HomeKit Device integration. Needs clarification: are these two different service calls (climate entity vs. HomeKit Device fan boost), or is this a conflict? The hvac_exhaust_boost script is not used here.

**INFO — Notification format in safety-monitoring.md is outdated**
Doc shows older single-line notification examples (v1 format). Actual YAML uses consolidated Emergency Response with HVAC/fan shutdown confirmation in message. Doc examples do not reflect current behavior. Not a functional issue — doc needs updating at some point.

---

### basementleaksensors.yaml

**INFO — All 6 automations have IDs. Clean.**

**INFO — Uses `action:` keyword instead of `service:`**
Newer HA syntax. Both work. Inconsistent with smokeco.yaml which uses `service:`. Not a problem, just noted for awareness.

**INFO — No daily summary automation in this package**
The safety-monitoring.md Battery Status section implies a daily summary but that is handled by batteryalerts.yaml, not this file. Confirmed as correct — no gap.

---

### sumppump.yaml

**INFO — All automations have IDs. Clean.**

**FLAG — Helper documented in safety-monitoring.md does not exist in YAML**
The doc lists `input_number.sump_pump_average_cycle_seconds` as a user-adjustable helper. This helper is not defined anywhere in sumppump.yaml. Only `sump_pump_gallons_per_cycle` and `sump_pump_monthly_gallons` exist. Either the doc is outdated (helper was removed) or it was never built. Needs clarification.

**INFO — Rainfall sensor reads current precipitation attribute, not 24h accumulation**
`sensor.rainfall_24_hours` reads `state_attr('weather.forecast_home_2', 'precipitation')`. Depending on what Met.no returns in that attribute, this may be current precipitation rate rather than 24h accumulation. The automations use it to determine idle thresholds (>0.1", >0.5") — worth verifying the attribute returns the expected value. Low risk but worth confirming.

---

### safety-monitoring.md (doc)

**FLAG — Battery alert threshold shows 15% only**
The Battery Status section documents only a 15% threshold. Builddecisions rule 12 defines a two-level system (30% to battery_alerts, 15% to house_alerts). The doc is incomplete on this. Needs update when battery section is reviewed.

**INFO — Notification examples outdated**
Smoke/CO detection examples in the doc reflect v1 behavior. Current YAML sends a consolidated Emergency Response with HVAC/fan shutdown messaging. Minor doc drift.

---

### Summary for Walkthrough

| File | IDs OK | Doc Matches | Flags |
|------|--------|-------------|-------|
| smokeco.yaml | Yes — IDs added | Partially | 1 open |
| basementleaksensors.yaml | Yes | Yes | 0 |
| sumppump.yaml | Yes | Partially | 1 |
| safety-monitoring.md | — | See above | 1 |

**Priority discussion items:**
1. smokeco.yaml — 9 missing IDs (confirm intent to add, then add with confirmation)
2. smokeco.yaml — Emergency Response fan control method (climate entity vs device_id — is this intentional?)
3. sumppump.yaml — `sump_pump_average_cycle_seconds` helper — exists or removed?
