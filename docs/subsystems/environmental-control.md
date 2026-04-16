# Environmental Control

**Last Updated:** April 2026

---

## Overview

The Environmental Control subsystem manages indoor humidity and air quality through automated control of humidification, dehumidification, and ventilation systems. These systems work together to maintain optimal comfort levels while maximizing energy efficiency through peak-hour avoidance and intelligent triggering based on actual conditions rather than fixed schedules.

The subsystem prevents common household issues like mold growth, dry air discomfort, and excessive energy costs by responding to real-time sensor data and seasonal patterns.

---

## Humidifier Pad Monitoring

### What It Does

Tracks the performance of the whole-house humidifier pad over time by monitoring main floor humidity trends. Alerts when the pad is degrading or needs replacement based on 7-day and 14-day rolling averages that filter out short-term HVAC cycling patterns.

**Rationale:** Humidifier pads degrade gradually, causing declining humidity levels. Early detection prevents discomfort and allows planned replacement rather than emergency response during winter heating season. Proper humidity levels also protect house construction materials, wood trim, and furniture from damage caused by dry air.

### Hardware

**Ecobee Thermostat:**
- Location: Main floor dining room
- Provides: Main floor humidity reading
- Entity: `sensor.main_floor_current_humidity`

**HVAC Humidifier:**
- Type: Whole-house bypass humidifier
- Control: Independent humidistat (not controlled by HA)
- HA Role: Monitoring only

### Key Entities

**Sensors:**
- `sensor.main_floor_current_humidity` - Current humidity from Ecobee

**Helper Sensors (created by this package):**
- `sensor.main_floor_humidity_7_day_average` - 7-day rolling average
- `sensor.main_floor_humidity_14_day_average` - 14-day rolling average
- `binary_sensor.main_humidity_declining` - Trend detector

**User Controls:**
- `input_number.humidifier_baseline` - Expected humidity with good pad (default: 44%)
- `input_datetime.last_humidifier_pad_change` - Tracks pad replacement date

### Automations

**Low Humidity Critical Alert:**
- `humidifier_pad_low_humidity_alert`
- Trigger: 7-day average below 36% for 48+ hours
- Action: HIGH priority alert
- Message: "Main floor humidity critically low - humidifier pad likely needs replacement"

**Below Baseline Warning:**
- `humidifier_pad_below_baseline_warning`
- Trigger: 7-day average falls 8%+ below baseline for 72+ hours
- Action: NORMAL priority
- Message: "Humidity below baseline - humidifier pad may be degrading"

**Declining Trend Notice:**
- `humidifier_pad_declining_trend_notice`
- Trigger: Negative 7-day trend sustained 5+ days AND 7-day average below 40%
- Action: LOW priority early warning via house_notices

**Pad Change Detection:**
- `humidifier_pad_change_detected`
- Trigger: Humidity spike >50% sustained for 30+ minutes
- Condition: HVAC must be in heat mode (prevents false triggers from summer humidity)
- Action: Auto-updates `last_humidifier_pad_change` timestamp; resets baseline 24 hours later
- Rationale: New pad causes immediate humidity jump - auto-resets tracking

**YAML:** hvachumiditypad.yaml

---

## Basement Dehumidifier Control

### What It Does

Automatically controls basement dehumidifier outlet to maintain 45-50% humidity while avoiding weekday peak electricity hours (3-7 PM Mon-Fri). Saves approximately $120-192 annually by blocking operation during peak rate periods.

**Rationale:** Basements require consistent humidity control to prevent mold and musty odors. The system uses setpoint control - it runs for best effort to dehumidify when humidity exceeds 50%, though it may not always maintain exactly 50% depending on conditions. The 5% hysteresis (45-50%) prevents excessive on/off cycling while the peak-hour avoidance provides significant cost savings without compromising humidity control since basement humidity changes slowly.

### Hardware

**YoLink Humidity Sensor:**
- Location: Basement
- Entity: `sensor.basement_humidity_sensor_humidity`

**Insteon Smart Outlet:**
- Entity: `switch.scheduled_power_save`
- Controls: Dehumidifier AC power

**Dehumidifier:**
- Type: Standard portable unit
- HA Role: On/off control only

### Key Entities

**Sensors:**
- `sensor.basement_humidity_sensor_humidity` - Current basement humidity

**Switches:**
- `switch.scheduled_power_save` - Dehumidifier outlet power control

### Automations

**Turn On:**
- `basement_dehumidifier_on`
- Trigger: Humidity rises above 50%
- Condition: Outside peak hours (Mon-Fri 3-7 PM blocked)
- Action: Turn on outlet, send notification
- Operates anytime on weekends

**Turn Off:**
- `basement_dehumidifier_off`
- Trigger: Humidity falls below 45% OR peak hours start (3 PM Mon-Fri)
- Action: Turn off outlet, send notification
- Prevents operation during expensive peak rate periods

**YAML:** basementdehumidifiercontrol.yaml

---

## Bathroom Humidity Control

### What It Does

Automatically detects showers by monitoring humidity rise patterns and runs the exhaust fan to remove moisture, followed by turning on the HVAC fan to further equalize humidity throughout the house. Uses relative humidity tracking (baseline vs. peak) to work correctly regardless of seasonal outdoor humidity levels. Includes a comfort delay before the fan starts and a fixed runtime per HVAC guidelines. Includes manual timer for non-shower ventilation needs.

**Rationale:** Prevents mold growth and moisture damage by removing shower humidity. Traditional fixed timers waste energy running fan when not needed, while humidity-based control responds only to actual moisture events. The relative rise method works year-round without seasonal adjustments. A flat 20-minute runtime is used based on observed sensor data showing the bulk of humidity removal occurs in the first few minutes; continuing beyond 20 minutes provides minimal additional benefit. The comfort delay prevents the fan from starting immediately, avoiding a sudden rush of cold air during showering. After the exhaust fan completes its cycle, the HVAC fan is activated to help distribute and equalize humidity throughout the house.

This pattern is deployed as separate YAML packages per bathroom, each with its own helpers and entities to allow fully independent monitoring and control.

### How It Works

**Detection Phase (polling every 30 seconds):**
- Tracks current humidity and maintains a rolling peak value
- Detection fires when two conditions are both met:
  1. Humidity has risen above the peak by the threshold (8% below 50% ambient, 5% above 50% ambient)
  2. The last sensor reading was within the past 15 minutes (rate-of-rise gate)
- The 15-minute gate is critical: YoLink sensors report hourly in stable conditions, so slow ambient humidity drift accumulates between reports and can appear as a single large jump. Requiring a recent reading confirms the rise happened quickly — the signature of a shower rather than gradual drift
- Baseline and peak are locked at detection time — baseline to the pre-rise peak, peak to the current humidity
- A 5-minute comfort delay runs before the fan turns on (prevents cold air during showering)
- Fan turns on unconditionally after the delay — no re-check

**Fan Running Phase:**
- Fan runs for a flat 20 minutes per HVAC guidelines
- After 20 minutes, fan turns off, all tracking values reset, and HVAC boost is triggered
- 2-hour safety timeout acts as a backstop in case the sequence is interrupted mid-run

**Timeout Reset:**
- If no humidity updates for 15+ minutes with fan off, resets all tracking values
- Prevents stale baseline/peak from causing false triggers

**Manual Timer:**
- If fan is manually turned on (baseline is 0, meaning not automation-triggered), auto-off after 20 minutes
- Independent of humidity-based control; useful for ventilation without showering

**Startup Reset:**
- On HA restart, all tracking helpers are zeroed and the fan is turned off
- Prevents stale helper values from triggering false detection after a restart

**HVAC Coordination:**
- When fan cycle completes normally, calls `script.hvac_exhaust_boost`
- Script runs HVAC fan for 20 minutes to distribute remaining humidity
- Script uses `mode: restart` so overlapping calls from multiple bathrooms reset the timer (last bathroom to finish sets the full 20-minute window)

### Bathroom Instances

#### Master Bath

**Hardware:**
- YoLink Humidity Sensor: `sensor.master_bath_humidity`
- Insteon Switch (exhaust fan): `light.master_bath_fan`

**Helpers (auto-managed, typically no adjustment needed):**
- `input_number.master_bath_humidity_baseline` - Starting humidity when shower begins
- `input_number.master_bath_humidity_peak` - Highest humidity at detection time
- `input_number.master_bath_humidity_last_update` - Timestamp for stale detection
- `input_number.master_bath_fan_loop_start` - 2-hour safety timeout tracking

**Automations:**
- `Master Bath Humidity - Shower Detection` — main polling loop
- `Master Bath Humidity - Timeout` — stale value reset
- `Master Bath Fan - Manual Timer` — 20-minute auto-off for manual use
- `Master Bath Humidity - Startup Reset` — zeroes helpers on HA restart

**YAML:** masterbathhumidity.yaml

#### Middle Bath

**Hardware:**
- YoLink Humidity Sensor: `sensor.middle_bath_humidity`
- Exhaust fan: `fan.middle_bath_humidity`

**Helpers (auto-managed, typically no adjustment needed):**
- `input_number.middle_bath_humidity_baseline` - Starting humidity when shower begins
- `input_number.middle_bath_humidity_peak` - Highest humidity at detection time
- `input_number.middle_bath_humidity_last_update` - Timestamp for stale detection
- `input_number.middle_bath_fan_loop_start` - 2-hour safety timeout tracking

**Automations:**
- `Middle Bath Humidity - Shower Detection` — main polling loop
- `Middle Bath Humidity - Timeout` — stale value reset
- `Middle Bath Fan - Manual Timer` — 20-minute auto-off for manual use
- `Middle Bath Humidity - Startup Reset` — zeroes helpers on HA restart

**YAML:** middlebathhumidity.yaml

#### East Bath

**Hardware:**
- YoLink Humidity Sensor: `sensor.east_suite_bathroom_humidity`
- Exhaust fan: `fan.east_suite_bathroom_humidity`

**Helpers (auto-managed, typically no adjustment needed):**
- `input_number.east_bath_humidity_baseline` - Starting humidity when shower begins
- `input_number.east_bath_humidity_peak` - Highest humidity at detection time
- `input_number.east_bath_humidity_last_update` - Timestamp for stale detection
- `input_number.east_bath_fan_loop_start` - 2-hour safety timeout tracking

**Automations:**
- `East Bath Humidity - Shower Detection` — main polling loop
- `East Bath Humidity - Timeout` — stale value reset
- `East Bath Fan - Manual Timer` — 20-minute auto-off for manual use
- `East Bath Humidity - Startup Reset` — zeroes helpers on HA restart

**YAML:** eastbathhumidity.yaml

#### Switch Technology Notes

The master bath exhaust fan uses an Insteon switch (`light.master_bath_fan`). The middle bath and east bath exhaust fans use Matter/Thread switches. See: insteon-integration.md, matter-thread-integration.md

#### Sensor Notes

Bathroom humidity sensors are YoLink LoRa devices (model 8003). These sensors report infrequently by design to maximize battery life — they wake up and report rapidly on fast humidity rises (typical of showers) but report hourly in stable conditions. The 15-minute rate-of-rise gate in the detection logic accounts for this behaviour, ensuring that slow ambient drift accumulating between hourly reports cannot satisfy the detection condition regardless of magnitude. Do not configure YoLink alert thresholds on these sensors — alert-triggered reporting bursts can interfere with detection logic and provide no benefit over the existing approach.

---

## Entry Fan Coordination

### What It Does

Automatically turns off the entry bathroom fan after a fixed period to prevent energy waste from forgotten manual activation.

**Rationale:** Small bathroom without humidity sensor, so uses simple timer-based control rather than intelligent humidity detection.

### Hardware

**Insteon Switch:**
- Entity: `light.entry_bath_fan`
- Controls: Entry bathroom exhaust fan

### Automations

**Fan Auto-Off:**
- `Entry Bath - Fan Auto-Off`
- Trigger: Fan turned on
- Duration: 12 minutes, then auto-off
- Action: Turns off fan to prevent waste

**YAML:** entryfan.yaml

---

## User Controls

**Access:** Settings → Helpers

**Humidifier Monitoring:**
- `input_number.humidifier_baseline` - Expected humidity with good pad (30-60%, default: 44%)
- `input_datetime.last_humidifier_pad_change` - Track pad replacement date

**Bathroom Humidity (Advanced - typically no adjustment needed):**

All bathroom humidity helpers are auto-managed by their respective automations. Each bathroom has its own independent set:

*Master Bath:*
- `input_number.master_bath_humidity_baseline` - Auto-set by detection logic
- `input_number.master_bath_humidity_peak` - Auto-set by detection logic

*Middle Bath:*
- `input_number.middle_bath_humidity_baseline` - Auto-set by detection logic
- `input_number.middle_bath_humidity_peak` - Auto-set by detection logic

*East Bath:*
- `input_number.east_bath_humidity_baseline` - Auto-set by detection logic
- `input_number.east_bath_humidity_peak` - Auto-set by detection logic

---

## Notifications

**Pushover Operational Log** (`notify.operational_log`):
- Basement dehumidifier on/off events
- Priority: Low (informational)

**Pushover House Notices** (`notify.house_notices`):
- Humidifier declining trend early warnings
- Humidifier below baseline warnings
- Priority: Normal (awareness needed)

**Pushover House Alerts** (`notify.house_alerts`):
- Humidifier critically low (pad needs replacement)
- Bathroom fan safety timeout (2-hour max exceeded) — identifies which bathroom in message
- Priority: High (action required)

---

## Related Systems

**HVAC Integration:**
- Ecobee thermostat provides humidity sensors for humidifier monitoring
- Bathroom fans coordinate with `script.hvac_exhaust_boost`
- See: homekit-device-integration.md

**YoLink Sensors:**
- Basement humidity sensor (dehumidifier control)
- Master bath humidity sensor (shower detection)
- Middle bath humidity sensor (shower detection)
- East bath humidity sensor (shower detection)
- See: yolink-integration.md

**Insteon Switches:**
- Basement dehumidifier outlet: `switch.scheduled_power_save`
- Master bath fan: `light.master_bath_fan`
- See: insteon-integration.md

**Matter/Thread Switches:**
- Middle bath fan: `fan.middle_bath_humidity`
- East bath fan: `fan.east_suite_bathroom_humidity`
- See: matter-thread-integration.md

---

## Troubleshooting

**Humidifier pad alerts but pad is new:**
- Check `input_number.humidifier_baseline` - may need adjustment
- Verify `input_datetime.last_humidifier_pad_change` is set correctly
- Statistics sensors need 7 days to populate after first setup

**Basement dehumidifier not running:**
- Check current time - blocked Mon-Fri 3-7 PM
- Verify `sensor.basement_humidity_sensor_humidity` is reporting correctly
- Check `switch.scheduled_power_save` responds to manual control

**Bathroom fan not triggering on shower** (applies to any bathroom instance):
- Check that the humidity sensor is updating — use Developer Tools → Template to check `last_reading_age`:
  `{{ (now().timestamp() - as_timestamp(states.sensor.master_bath_humidity.last_changed)) / 60 | round(1) }}` (substitute sensor name as needed)
- If age > 15 minutes at the time of the shower, the rate-of-rise gate blocked detection — this is expected behaviour when sensor reporting is slow
- Verify humidity rise exceeded threshold (8% below 50%, 5% above 50%)
- On high-humidity days, shower rises may be smaller — the rate-of-rise gate becomes the primary filter in these conditions

**Fan triggered with no shower:**
- Check sensor history for the rise pattern — was it a fast jump (minutes) or slow climb (hours)?
- Fast jump with no shower: could be steam from cooking, open windows, or other moisture source
- Slow climb that appears as a jump: rate-of-rise gate should have blocked this — check last_reading_age at time of trigger

**Fan won't turn off after shower:**
- Fan runs for a flat 20 minutes then stops automatically
- If still running past 20 minutes, the sequence may have been interrupted — check automation trace
- 2-hour safety timeout will force off if sequence is stuck
- Manual off always works regardless of automation state

**False trigger after HA restart or automation reload:**
- Helpers may retain stale values from a previous interrupted cycle
- Startup reset automation handles this on full HA restart
- For automation reload only: manually zero `*_humidity_baseline`, `*_humidity_peak`, and `*_fan_loop_start` via Developer Tools → States for the affected bathroom

---

## YAML Packages

- `/config/packages/hvachumiditypad.yaml`
- `/config/packages/basementdehumidifiercontrol.yaml`
- `/config/packages/masterbathhumidity.yaml`
- `/config/packages/middlebathhumidity.yaml`
- `/config/packages/eastbathhumidity.yaml`
- `/config/packages/entryfan.yaml`

---