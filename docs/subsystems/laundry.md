# Laundry Automation

**Last Updated:** April 2026

---

## Overview

The Laundry Automation subsystem manages the laundry room exhaust fan based on dryer operation and temperature monitoring. The system uses adaptive baseline temperature tracking to intelligently manage fan runtime, running the fan during dryer operation and continuing through a temperature-based cooldown period after the dryer stops.

The baseline temperature approach allows the system to work correctly regardless of seasonal room temperature variations, and coordinates with HVAC exhaust boost to optimize whole-house ventilation after laundry cycles.

---

## Dryer Monitoring & Auto Fan Control

### What It Does

Automatically detects dryer operation through power monitoring, sets a temperature baseline when the dryer starts, runs the exhaust fan throughout the drying cycle, and continues fan operation during cooldown until room temperature returns to near-baseline levels (baseline + 3°F) or 45-minute timeout.

**Rationale:** Dryer operation increases laundry room temperature and humidity. The exhaust fan removes heat and moisture during operation, then continues running during cooldown to complete moisture removal. Baseline tracking adapts to seasonal room temperature changes - what's "hot" in summer vs winter varies by 20°F+, so absolute temperature thresholds don't work. The 45-minute timeout prevents indefinite fan operation if temperature sensor fails or room doesn't cool for other reasons.

### Hardware

**Power Monitor:**
- Entity: `sensor.dryer_monitor_current_power`
- Monitors: Dryer power draw
- Thresholds: >200W = ON, <2W = OFF

**Temperature Sensor:**
- Entity: `sensor.laundry_temperature`
- Location: Laundry room
- Purpose: Baseline and cooldown tracking

**Fan Control:**
- Entity: `light.laundry_fan_timer`
- Controls: Laundry room exhaust fan
- Type: Insteon switch (represented as light entity)

### Key Entities

**Sensors:**
- `sensor.dryer_monitor_current_power` - Current power draw
- `sensor.laundry_temperature` - Room temperature

**Switches:**
- `light.laundry_fan_timer` - Exhaust fan control

**Helper Sensors (created by this package):**
- `input_number.laundry_temp_baseline` - Starting temperature when dryer turns on
- `input_number.laundry_dryer_last_off` - Timestamp of last dryer shutdown (for baseline reset)

**User Controls:**
- None (fully automatic, but fan can be manually controlled)

### Automations

**Dryer Fan Control (Main Logic):**
- `laundry_dryer_fan_control`
- **Mode: `restart`** — if a new dryer_off trigger fires while cooldown is running (e.g. power fluctuation crossing the 2W threshold), the current cooldown is cancelled and restarts with a fresh 45-minute deadline. A dryer_on trigger during cooldown similarly cancels and re-evaluates baseline.
- Trigger: Power state change (numeric_state on sensor.dryer_monitor_current_power)
- Detection Logic:
  - Dryer ON (>200W): Set temperature baseline (if needed), turn fan ON
  - Dryer OFF (<2W): Begin temperature-based cooldown
- Cooldown Logic:
  - Repeat loop checks every 2 minutes
  - Continue until temp drops to baseline + 3°F OR 45 min timeout
  - Calls `script.hvac_exhaust_boost` when fan turns off
- Baseline Update Conditions:
  - Current temp < stored baseline (room has cooled), OR
  - Baseline = 0 (uninitialized), OR
  - Dryer off for 4+ hours

**Baseline Timeout Reset:**
- `laundry_baseline_timeout`
- **Mode: `single`** — if a reset is already in progress when the 30-minute trigger fires again, the duplicate run is discarded.
- Trigger: Every 30 minutes
- Condition: `baseline > 0` AND dryer has been off for 4+ hours
  - The `baseline > 0` guard prevents redundant resets when the baseline is already cleared, and avoids the reset firing on HA startup before any dryer cycle has run.
- Action: Resets baseline to 0
- Rationale: Clears stale baseline data when dryer hasn't been used

**YAML:** laundry.yaml

---

## User Controls

**Access:** Direct entity control (no helper inputs for user adjustment)

**Manual Fan Control:**
- `light.laundry_fan_timer` can be manually controlled via:
  - Home Assistant dashboard
  - Insteon switch
  - Voice control (if configured)

**Advanced (typically no adjustment needed):**
- `input_number.laundry_temp_baseline` - Current baseline temperature (auto-set)
- `input_number.laundry_dryer_last_off` - Last dryer shutdown timestamp (auto-set)

---

## Notifications

No Pushover notifications configured for laundry automations.

---

## Related Systems

**HVAC Coordination:**
- Calls `script.hvac_exhaust_boost` when fan turns off
- Optimizes whole-house ventilation after laundry cycle
- Script defined in separate HVAC coordination package

**Insteon Controls:**
- Fan switch entity: `light.laundry_fan_timer`
- See: insteon-integration.md

**Power Monitoring:**
- Dryer power monitor required for operation detection
- Entity: `sensor.dryer_monitor_current_power`

---

## Troubleshooting

**Fan not turning on when dryer starts:**
- Verify `sensor.dryer_monitor_current_power` is reporting correctly
- Check power reading exceeds 200W when dryer running
- Ensure `light.laundry_fan_timer` responds to manual control

**Fan not turning off after dryer stops:**
- Check `sensor.laundry_temperature` is updating
- Verify temperature is dropping (should reach baseline + 3°F)
- 45-minute timeout will force off if temp doesn't drop
- Manual off via `light.laundry_fan_timer` always available

**Fan turning off too quickly:**
- Check if temperature is dropping very quickly (possible in winter)
- Baseline + 3°F threshold may be met quickly in cold conditions
- Review baseline temperature setting in `input_number.laundry_temp_baseline`

**Baseline seems incorrect:**
- Baseline auto-resets after 4 hours of dryer inactivity
- Baseline updates when dryer turns on if current temp < stored baseline
- Manual reset possible by setting `input_number.laundry_temp_baseline` to 0
- Wait for next dryer cycle to establish new baseline

**Fan runs for full 45 minutes every time:**
- Temperature may not be dropping to baseline + 3°F
- Check if temperature sensor is working correctly
- Verify sensor placement near dryer (should sense heat)
- 45-minute timeout is safety feature, not indication of failure

---

## YAML Packages

- `/config/packages/laundry.yaml`

---