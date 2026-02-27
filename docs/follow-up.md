# Post-Review Follow-Up Items

**Purpose:** Deferred improvements and low-priority verifications identified during
subsystem reviews. These are not bugs -- implement or investigate when ready.

---

## Sump Pump

**Verify rainfall sensor attribute**
- `sensor.rainfall_24_hours` reads `state_attr('weather.forecast_home_2', 'precipitation')`
- Depending on Met.no, this may be current precipitation rate rather than 24h accumulation
- Automations use it for idle thresholds (>0.1", >0.5") -- verify the attribute returns the expected value
- File: `/homeassistant/packages/sumppump.yaml`

---

## HVAC Humidity Pad

**Confirm climate entity name**
- `hvachumiditypad.yaml` checks `climate.main_floor` for heat mode
- Other packages use `climate.ecobee`
- Verify these are the same entity with a different friendly name, or two distinct entities
- File: `/homeassistant/packages/hvachumiditypad.yaml`

---

## Air Quality Doc

**Encoding corruption in air-quality-monitoring.md**
- UTF-8 characters (ug/m3, +/-, emoji, arrows) are double-encoded throughout
- Left as-is -- notification examples in doc represent working YAML strings
- Address when doc is rewritten or significant edits are needed
- File: `/homeassistant/docs/subsystems/air-quality-monitoring.md`

---

## Irrigation

**Wire PoP threshold into skip condition**
- Helper `input_number.irrigation_pop_threshold` exists and displays in automatic cycle notifications
- Not currently wired into any skip logic -- weather state check (rainy/pouring/etc.) is the only skip condition
- Improvement: add condition to `irrigation_automatic_morning_cycle` to skip if forecast
  precipitation probability exceeds the threshold value
- File: `/homeassistant/packages/irrigation.yaml`

---
