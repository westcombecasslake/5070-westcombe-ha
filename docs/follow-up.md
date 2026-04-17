# Post-Review Follow-Up Items

**Purpose:** Deferred improvements and low-priority verifications identified during
subsystem reviews. These are not bugs -- implement or investigate when ready.

---

## Device Inventory

**Verify Smoke CO Siren (YS7103-UC)**
- TSV row 68: Middle Bathroom, `switch.smoke_co_siren`, SN: 281B4FEB47
- Entity returns 404 and device has no entry in HA device registry
- Device is referenced in `smokeco.yaml` -- confirm whether it is physically installed and operational, or removed
- If removed: delete TSV row and remove from smokeco.yaml
- File: `/homeassistant/docs/device-inventory.tsv`, `/homeassistant/packages/smokeco.yaml`

---

## Sump Pump

**Verify depth sensor polarity**
- Hardware wiring indicates `on` should be normal depth, `off` should be HIGH WATER ALERT
- Doc and YAML header comment say `on = Normal, off = HIGH` (consistent with wiring)
- But YAML automation code does the opposite: `if off → "Normal", else → "HIGH!"`
- Live sensor shows `off` during normal conditions -- contradicts wiring expectation
- Needs physical verification: manually trigger the float, observe entity state, confirm which value is correct
- If code is wrong: fix the `if` condition in all 4 sump pump notification templates in `sumppump.yaml`
- If doc/comment is wrong: update doc and YAML header comment
- File: `/homeassistant/packages/sumppump.yaml`, `/homeassistant/docs/subsystems/safety-monitoring.md`

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

## Power Infrastructure

**Create subsystem doc**
- `powerinfrastructure.yaml` has no doc -- skipped during initial review as single-automation package
- If the package grows, create `docs/subsystems/powerinfrastructure.md`
- File: `/homeassistant/packages/powerinfrastructure.yaml`

---
