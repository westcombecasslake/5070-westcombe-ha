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

**RESOLVED — Emergency Response uses `climate.ecobee` entity for fan shutoff**
Confirmed intentional. Fan mode (on/auto) is exposed via the climate entity and `climate.set_fan_mode` is the correct call for shutoff. The `device_id` approach in `hvac_exhaust_boost` is required only for the boost feature, which is not exposed as an entity. No conflict with builddecisions rule 8.

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

**RESOLVED — Helper documented in safety-monitoring.md does not exist in YAML**
`input_number.sump_pump_average_cycle_seconds` confirmed no longer used. Removed from safety-monitoring.md.

**INFO — Rainfall sensor reads current precipitation attribute, not 24h accumulation**
`sensor.rainfall_24_hours` reads `state_attr('weather.forecast_home_2', 'precipitation')`. Depending on what Met.no returns in that attribute, this may be current precipitation rate rather than 24h accumulation. The automations use it to determine idle thresholds (>0.1", >0.5") — worth verifying the attribute returns the expected value. Low risk but worth confirming.

---

### safety-monitoring.md (doc)

**RESOLVED — Battery alert threshold shows 15% only**
safety-monitoring.md updated February 2026 to reflect two-tier system: 30% to battery_alerts, 15% to house_alerts.

**INFO — Notification examples outdated**
Smoke/CO detection examples in the doc reflect v1 behavior. Current YAML sends a consolidated Emergency Response with HVAC/fan shutdown messaging. Minor doc drift.

---

### Summary for Walkthrough

| File | IDs OK | Doc Matches | Flags |
|------|--------|-------------|-------|
| smokeco.yaml | Yes -- IDs added | Partially | 1 open |
| basementleaksensors.yaml | Yes | Yes | 0 |
| sumppump.yaml | Yes | Updated | 0 open |
| safety-monitoring.md | -- | See above | 1 |

**Priority discussion items:**
1. smokeco.yaml -- Emergency Response fan control method (climate entity vs device_id -- is this intentional?)

---

## Safety Subsystems -- systemhealth.yaml, batteryalerts.yaml, systemstartup.yaml

**Reviewed:** February 2026

---

### systemhealth.yaml

**RESOLVED -- Missing automation IDs (all 9 automations)**
IDs added February 2026:
- system_health_insteon_offline
- system_health_insteon_online
- system_health_yolink_offline
- system_health_yolink_online
- system_health_ecobee_offline
- system_health_ecobee_online
- system_health_ring_offline
- system_health_ring_online
- system_health_integrations_status_update

**INFO -- Notification routing: all 4 integration offline alerts route to house_alerts**
YoLink offline is arguably safety-relevant (silences smoke/CO sensors). Intentional by owner design.

**INFO -- No recovery notification on online recovery**
Recovery automations only update the status helper. No Pushover on recovery. Intentional -- avoids noise.

**RESOLVED -- safety-monitoring.md System Health section has doc drift**
Doc updated February 2026: removed binary_sensor.ha_startup_pending, database size monitoring, network connectivity monitoring, and Recorder Configuration section. Startup Monitoring section rewritten to reflect actual systemstartup.yaml behavior.

---

### batteryalerts.yaml

**INFO -- All 3 automation IDs present. Clean.**

**INFO -- Two-level alert system confirmed**
30% warning -> notify.battery_alerts (normal priority). 15% critical -> notify.house_alerts (high priority). Matches builddecisions rule 12. Condition on 30% alert (> 15%) prevents double-alerting.

**RESOLVED -- safety-monitoring.md battery section documents only 15% threshold**
Doc updated February 2026 to reflect two-tier system with correct routing.

---

### systemstartup.yaml

**INFO -- Single automation ID present. Clean.**

**INFO -- Startup notification routes to house_alerts**
All restarts (not just unexpected ones) route to house_alerts. Rationale: every restart = monitoring gap for safety sensors. Doc says house_notices for unusual restarts -- doc is wrong, YAML is intentional.

**INFO -- Status refresh sequence is clean**
30-second delay + script refresh + reset integration helpers to Online. Correct approach -- offline automations re-fire if integration doesn't come back.

---

### Summary

| File | IDs OK | Doc Matches | Flags |
|------|--------|-------------|-------|
| systemhealth.yaml | Yes -- IDs added | No -- doc drift | 0 open |
| batteryalerts.yaml | Yes | Partially | 1 open (doc) |
| systemstartup.yaml | Yes | Partially | 0 open |

---

## Environmental Control -- hvachumiditypad, basementdehumidifiercontrol, masterbathhumidity, middlebathhumidity, eastbathhumidity, entryfan, scripts.yaml

**Reviewed:** February 2026

---

### hvachumiditypad.yaml

**INFO -- All 4 automation IDs present. Clean.**

**INFO -- climate.main_floor vs climate.ecobee**
Pad change detection checks climate.main_floor (heat mode). Other packages reference climate.ecobee. May be same entity with different friendly name. No action -- noted for awareness.

**INFO -- humidifier_pad_below_baseline_warning routes to house_alerts**
Doc says NORMAL priority / house_notices. YAML uses house_alerts. Intentional -- below baseline for 3 days warrants higher attention.

---

### basementdehumidifiercontrol.yaml

**INFO -- All 2 automation IDs present. Clean.**

**INFO -- No flags. Logic, routing, and peak-hour avoidance match doc exactly.**

---

### masterbathhumidity.yaml / middlebathhumidity.yaml / eastbathhumidity.yaml

**INFO -- All 9 automation IDs present across 3 files. Clean.**

**RESOLVED -- Non-ASCII em dashes in comment lines (all three files)**
7 em dashes per file (21 total) replaced with plain hyphens February 2026. Rule 17 compliant.

**INFO -- Safety timeout does not call hvac_exhaust_boost**
Normal fan shutoff calls the script; 2-hour timeout does not. Intentional -- timeout is a failure condition, not a normal cycle end.

**INFO -- Fan entity types differ by bathroom**
Master bath: light.master_bath_fan (Insteon). Middle/east bath: fan.* (Matter/Thread). Intentional and matches doc.

---

### entryfan.yaml

**INFO -- Single automation ID present. Clean.**

**RESOLVED -- Doc placeholder text replaced**
Entity name (light.entry_bath_fan) and duration (12 minutes) filled in environmental-control.md.

---

### scripts.yaml (hvac_exhaust_boost)

**INFO -- Clean. No flags.**
Uses device_id correctly per builddecisions rule 8. mode: restart correct. PM2.5 filtration check correct.

---

### environmental-control.md doc

**RESOLVED -- East bath sensor entity ID corrected**
Doc had sensor.east_suite_bathroom; corrected to sensor.east_suite_bathroom_humidity.

**RESOLVED -- Entry fan placeholder text replaced**
Entity and duration now documented.

---

### Summary

| File | IDs OK | Doc Matches | Flags |
|------|--------|-------------|-------|
| hvachumiditypad.yaml | Yes | Mostly | 0 open |
| basementdehumidifiercontrol.yaml | Yes | Yes | 0 |
| masterbathhumidity.yaml | Yes -- em dashes fixed | Yes | 0 |
| middlebathhumidity.yaml | Yes -- em dashes fixed | Yes | 0 |
| eastbathhumidity.yaml | Yes -- em dashes fixed | Yes | 0 |
| entryfan.yaml | Yes | Updated | 0 |
| scripts.yaml | -- | Yes | 0 |

---

## Premise Lighting -- lightingpremise.yaml

**Reviewed:** February 2026

---

### lightingpremise.yaml

**INFO -- All 11 automation IDs present. Clean.**

**INFO -- action: syntax confirmed working for all Insteon scene calls**
All 20 Insteon scene calls use action: syntax. Confirmed working in production for weeks. insteon-integration.md corrected to reflect this (action: correct, service: does not work).

**RESOLVED -- insteon-integration.md warning was backwards**
Doc said service: required, action: fails. Correct is the opposite. Doc updated February 2026.

**RESOLVED -- Scene 31 listed under lightingpremise.yaml in insteon-integration.md**
lightingpremise.yaml does not use Scene 31. Corrected in insteon-integration.md: Scene 29 (Front Porch Override) added, Scene 31 moved to lightingindoor.yaml.

**RESOLVED -- premise_front_porch_manual_override not documented**
Automation, Scene 29, and light.front_porch_override added to lighting-premise.md.

**RESOLVED -- Motion timer condition simplified in doc**
Doc said "photocell on" -- actual condition is input_boolean.front_motion_armed (more nuanced than photocell state). Doc updated with accurate condition.

---

### ring-integration.md

**INFO -- Clean. Accurately documents integration role.**

---

### Summary

| File | IDs OK | Doc Matches | Flags |
|------|--------|-------------|-------|
| lightingpremise.yaml | Yes | Updated | 0 open |
| insteon-integration.md | -- | Corrected | 0 open |
| ring-integration.md | -- | Yes | 0 |

---

## Indoor Lighting -- lightingindoor.yaml

**Reviewed:** February 2026

---

### lightingindoor.yaml

**INFO -- All 11 automation IDs present. Clean.**

**INFO -- Double-send pattern correct on all Insteon calls. All use service: syntax.**

**INFO -- Scene numbers all decimal. Non-ASCII clean.**

**INFO -- Doc matches YAML exactly. Random delay ranges, entity names, scene numbers all verified.**

---

### insteon-integration.md

**INFO -- Scene 31 listed under lightingpremise.yaml in "Finding Scene References" section**
Scene 31 (Living Room Lamps Fade-up) is used in lightingindoor.yaml but only appears under lightingpremise.yaml in the example table. May also be in lightingpremise.yaml (to be confirmed during that review). No action until lightingpremise.yaml is reviewed.

---

### sun-integration.md

**INFO -- Clean. Accurately describes its role as backup failsafe only.**

---

### Summary

| File | IDs OK | Doc Matches | Flags |
|------|--------|-------------|-------|
| lightingindoor.yaml | Yes | Yes | 0 |
| insteon-integration.md | -- | Yes | 0 |
| sun-integration.md | -- | Yes | 0 |

---

## Air Quality -- airquality.yaml

**Reviewed:** February 2026

---

### airquality.yaml

**INFO -- All 14 automation IDs present. Clean.**

**INFO -- HVAC fan control uses device_id correctly**
All fan control routes through script.hvac_exhaust_boost per builddecisions rule 8. Clean.

**RESOLVED -- Non-ASCII bullet points in YAML header comments**
Three `*` bullet points in the package header comment block were using `•` (U+2022). Replaced with `*` February 2026. Rule 17 compliance. Note: emoji and formatting characters in notification message strings are working code -- rule 17 applies to comments only.

**INFO -- Master bedroom CO2 automation interacts with masterbathhumidity.yaml**
Master bedroom CO2 high (>1000 ppm, 9 PM - 7 AM) turns on master bath fan for 20 minutes. masterbathhumidity.yaml has its own 15-minute manual timer. These are independent -- CO2 automation runs the fan directly, not through the humidity script.

---

### air-quality-monitoring.md

**KNOWN ISSUE -- Encoding corruption throughout doc**
UTF-8 characters (μg/m³, ±, emoji, arrows) are double-encoded and display as garbled Latin-1 sequences. Not corrected -- notification examples in doc represent working YAML strings and are not to be touched.

**KNOWN ISSUE -- Daily summary time wrong**
Doc says "Daily summaries at 7:00 AM." YAML automation runs at 23:55 (11:55 PM). Not corrected alongside encoding issue.

---

### Summary

| File | IDs OK | Doc Matches | Flags |
|------|--------|-------------|-------|
| airquality.yaml | Yes | Mostly | 0 open |
| air-quality-monitoring.md | -- | Encoding corrupted | 0 open |

---

## Project Docs -- insteon-scenes.md, backup-integration.md

**Reviewed:** February 2026

---

### insteon-scenes.md

**RESOLVED -- Scene 29 entity ID wrong**
Doc had `switch.front_porch_override`. Corrected to `light.front_porch_override` February 2026.

**RESOLVED -- Scene usage table errors**
- Scene 31 (Lamps Fade-up) was listed under lightingpremise.yaml -- moved to lightingindoor.yaml
- Scene 29 (Garage Front Porch Override) was missing from lightingpremise.yaml section -- added
- Scene 30 (Master Bedroom) was missing from lightingindoor.yaml section -- added

**RESOLVED -- Backup path wrong**
`/mnt/project/insteon_scenes.json` corrected to `/config/insteon_scenes.json`.

**INFO -- Scene list and hex/decimal table accurate. 13 scenes confirmed.**

---

### backup-integration.md

**INFO -- Clean. Accurately documents built-in HA backup + Nabu Casa cloud + Google Drive secondary backup.**

**KNOWN ISSUE -- Encoding corruption**
Navigation arrows (→) display as garbled sequences. Not corrected -- consistent with policy on encoding issues.

---

### Summary

| File | IDs OK | Doc Matches | Flags |
|------|--------|-------------|-------|
| insteon-scenes.md | -- | Corrected | 0 open |
| backup-integration.md | -- | Yes | 0 open |

---

## Battery Monitoring -- batteryalerts.yaml, battery-monitoring-system.md, pushover-integration.md

**Reviewed:** February 2026

---

### batteryalerts.yaml

**INFO -- All 3 automation IDs present. Clean.**

**INFO -- 24 devices monitored. Matches doc.**

---

### battery-monitoring-system.md

**RESOLVED -- battery_recovery automation documented but not built**
Removed from doc. Automation was proposed and removed due to side effects during connectivity issues.

**RESOLVED -- Battery Status Helper thresholds wrong**
Doc said Normal >20%, Low <20%, Critical <10%. Corrected to match YAML: Low <30%, Critical <15%.

**RESOLVED -- Garage Motion Sensor miscategorized under Leak Detection**
Removed from Leak Detection. Device count updated to 24. Will be documented under garage.md.

---

### pushover-integration.md

**RESOLVED -- Location field in battery notification example**
Location line removed from examples -- not present in YAML.

---

### Summary

| File | IDs OK | Doc Matches | Flags |
|------|--------|-------------|-------|
| batteryalerts.yaml | Yes | Yes | 0 open |
| battery-monitoring-system.md | -- | Corrected | 0 open |
| pushover-integration.md | -- | Corrected | 0 open |
