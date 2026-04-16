# Indoor Lighting

**Last Updated:** February 2026

---

## Overview

The Indoor Lighting subsystem manages automated control for living room lamps, seasonal decorative lighting, and master bedroom lights. The system uses Insteon native scenes coordinated with Home Assistant automations that respond to sunset timing, occupancy sensing, and user preferences.

All indoor lighting operates through Insteon scenes, ensuring lights can be controlled manually via Insteon switches even when Home Assistant is offline, or the Insteon PLM malfunctions. **Insteon N-way switch control always works, controller independent.** Automations add intelligence through sunset-synchronized activation, occupancy-aware shutoff, and seasonal decoration scheduling.

---

## Living Room Scene Control

### What It Does

Automatically fades up living room lamps 40 minutes before sunset for ambient evening lighting (Scene 31 gradual fade), then turns them off at night (11:20 PM + random delay) via Scene 24 instant toggle after confirming the room is unoccupied via Ecobee sensor. Additionally, lamps are automatically faded off in the morning (35 minutes after the photocell turns on at dawn) to catch any lamps that were manually turned on and left on overnight.

When `input_boolean.long_term_away` is on (set automatically after 24 hours of no main floor occupancy), lamps also activate in the early morning at randomized times (weekdays 5:27-6:48 AM, weekends 7:13-7:58 AM) to simulate occupancy for security purposes.

**Rationale:** Sunset-based activation ensures lights come on at the right time year-round without manual schedules. Occupancy check prevents lights from turning off while someone is present. Random delay creates natural variation rather than predictable timing. 2-hour timeout prevents indefinite waiting if occupancy sensor fails. Morning fade-off prevents lamps from staying on all day if manually activated and forgotten overnight. Away mode simulation provides security by making the house appear occupied during absences.

### Hardware

**Living Room Lamps:**
- Scene 31 (ON): Gradual fade-up activation
- Scene 24 (ON/OFF): Instant toggle, controlled by PS01 light switch in living room
- Multiple Insteon lamp modules
- See: insteon-scenes.md

**Ecobee Thermostat:**
- Location: Main floor dining room
- Provides: Main floor occupancy sensing, away mode detection
- Entity: `binary_sensor.main_floor_occupancy`
- Entity: `select.main_floor_current_mode`

**Photocell Sensor:**
- Location: Indoor, south-facing second-floor bedroom window
- Provides: Dawn/dusk detection for morning automation
- Entity: `switch.photocell_micro_module`
- See: lighting-premise.md

### Key Entities

**Sensors:**
- `binary_sensor.main_floor_occupancy` - Ecobee occupancy sensor
- `sun.sun` - Sun integration for sunset timing
- `switch.photocell_micro_module` - Indoor photocell for dawn/dusk detection

**Helpers:**
- `input_boolean.long_term_away` - Set after 24h no main floor occupancy (declared in garage.yaml)

**User Controls:**
- None (fully automatic, but manual Insteon switch control always available)

### Automations

**Living Room Lamps Fade Up:**
- `living_room_lamps_fade_up`
- Trigger: 50 minutes before sunset OR photocell turns off (dusk detected)
- Condition: After 2:00 PM (guards against 1 PM daily photocell reset triggering a false fade-up)
- Action: Activate Insteon Scene 31 (lamps ON)
- Rationale: Gradual ambient lighting before darkness; dual trigger ensures activation even if sun timing and actual darkness diverge

**Living Room Lamps Off (Night):**
- `living_room_lamps_off_1120_random`
- Trigger: 11:20 PM + random 0-13 minute delay
- Condition: Main floor unoccupied
- Action: Activate Scene 24 (instant toggle to OFF)
- Safety: 2-hour timeout if occupancy never clears
- Retry: Waits continuously for occupancy to clear (wait_for_trigger with 2-hour timeout); always turns off at timeout regardless

**Living Room Lamps Morning Fade Off:**
- `living_room_lamps_morning_fade_off`
- Trigger: Photocell turns on (daylight detected), after 35-minute delay
- Condition: Between 5:00 AM and 10:00 AM (prevents false triggers outside dawn window)
- Action: Deactivate Scene 31 (gradual 2.5 minute fade-down)
- Rationale: Catches lamps manually turned on overnight, gradually fading them off as natural daylight increases

**Living Room Lamps Away Mode (Weekday):**
- `living_room_lamps_away_weekday`
- Trigger: Random time 5:27-6:48 AM Monday-Friday
- Condition: `input_boolean.long_term_away` is on (24h no main floor occupancy)
- Action: Activate Scene 24 (instant ON)
- Rationale: Simulates occupancy for security when house is unoccupied on weekdays

**Living Room Lamps Away Mode (Weekend):**
- `living_room_lamps_away_weekend`
- Trigger: Random time 7:13-7:58 AM Saturday-Sunday
- Condition: `input_boolean.long_term_away` is on (24h no main floor occupancy)
- Action: Activate Scene 24 (instant ON)
- Rationale: Simulates occupancy for security when house is unoccupied on weekends, reflects typical later weekend wake time

**YAML:** lightingindoor.yaml

---

## Seasonal Decorations (Scene 23)

### What It Does

Provides dashboard toggle control for seasonal decorative lighting (typically holiday lights) with automatic daily scheduling. When enabled, lights turn on at 6:00 AM and off at 11:55 PM, but only when the main floor is unoccupied to avoid disrupting activity.

**Rationale:** Single helper toggle provides easy seasonal control without modifying automations. Daily scheduling ensures decorative lights operate consistently without manual intervention. Occupancy awareness for nighttime shutoff prevents lights from turning off while someone is present in the room.

### Hardware

**Seasonal Lighting:**
- Controlled via Insteon Scene 23
- Typically holiday/decorative lights on Insteon outlets
- See: insteon-scenes.md

**Ecobee Thermostat:**
- Entity: `binary_sensor.main_floor_occupancy`
- Used for nighttime shutoff occupancy check

### Key Entities

**Sensors:**
- `binary_sensor.main_floor_occupancy` - Ecobee occupancy sensor

**User Controls:**
- `input_boolean.seasonal_lighting_enabled` - Master enable/disable toggle

### Automations

**Helper ON - Immediate Activation:**
- `seasonal_lighting_helper_on`
- Trigger: `input_boolean.seasonal_lighting_enabled` turned ON
- Action: Immediately activate Scene 23
- Provides instant feedback when toggle enabled

**Helper OFF - Immediate Deactivation:**
- `seasonal_lighting_helper_off`
- Trigger: `input_boolean.seasonal_lighting_enabled` turned OFF
- Action: Immediately deactivate Scene 23
- Ensures lights turn off when toggle disabled

**Daily ON at 6:00 AM:**
- `seasonal_lighting_on_6am`
- Trigger: 6:00 AM daily
- Condition: Helper toggle enabled
- Action: Activate Scene 23

**Daily OFF at 11:55 PM (Occupancy-Aware):**
- `seasonal_lighting_off_2355_with_occupancy`
- Trigger: 11:55 PM daily
- Condition: Helper enabled AND main floor unoccupied
- Action: Deactivate Scene 23
- Retry: Checks occupancy every 31 minutes if occupied

**YAML:** lightingindoor.yaml

---

## Occupancy-Based Control

### What It Does

Automatically turns off master bedroom lights when the room becomes unoccupied, as detected by the Ecobee remote sensor. Light activation is handled manually or via motion detection (external to this package).

When `input_boolean.long_term_away` is on (set automatically after 24 hours of no main floor occupancy), the bedroom lights also activate in the evening (random time 9:22-9:34 PM) and remain on for 17-26 minutes to simulate bedroom activity for security purposes.

**Rationale:** Prevents lights from being left on when room is empty, saving energy. Only controls OFF behavior since ON is typically intentional (manual switch or separate motion automation). Away mode simulation creates realistic evening activity patterns to deter potential intruders.

### Hardware

**Master Bedroom Lights:**
- Controlled via Insteon Scene 30
- Insteon switches and dimmers
- See: insteon-scenes.md

**Ecobee Remote Sensor:**
- Location: Master bedroom
- Entity: `binary_sensor.master_bedroom_occupancy`

**Ecobee Thermostat:**
- Provides: Away mode detection for evening simulation
- Entity: `select.main_floor_current_mode`

### Key Entities

**Sensors:**
- `binary_sensor.master_bedroom_occupancy` - Ecobee remote sensor

**Helpers:**
- `input_boolean.long_term_away` - Set after 24h no main floor occupancy (declared in garage.yaml)

**User Controls:**
- None (fully automatic)

### Automations

**Master Bedroom Lights Off When Unoccupied:**
- `master_bed_lights_off_unoccupied`
- Trigger: Room becomes unoccupied
- Action: Deactivate Scene 30
- Mode: Single (prevents multiple triggers)

**Master Bedroom Away Mode Evening:**
- `master_bedroom_away_evening`
- Trigger: Random time 9:22-9:34 PM daily
- Condition: `input_boolean.long_term_away` is on (24h no main floor occupancy)
- Action: Activate Scene 30, wait 17-26 minutes, deactivate Scene 30
- Rationale: Simulates evening bedroom activity for security when house is unoccupied

**YAML:** lightingindoor.yaml

---

## User Controls

**Access:** Settings -> Helpers

**Seasonal Lighting:**
- `input_boolean.seasonal_lighting_enabled` - Master toggle for Scene 23 scheduling
  - ON: Enables daily 6 AM - 11:55 PM schedule
  - OFF: Disables all automation and immediately turns off lights

---

## Notifications

No notifications configured for indoor lighting automations.

---

## Related Systems

**Premise Lighting:**
- Provides photocell dusk sensor for morning automation
- See: lighting-premise.md

**Sun Integration:**
- Provides sunset timing for living room lamp activation
- See: sun-integration.md

**Ecobee Integration:**
- Provides occupancy sensors for main floor and master bedroom
- Provides away mode detection for security simulation
- See: homekit-device-integration.md

**Insteon Scenes:**
- Scene 23: Seasonal decorative lighting
- Scene 24: Living room lamps
- Scene 30: Master bedroom lighting
- Scene 31: Living room lamps 2.5 min fade up/down
- See: insteon-scenes.md

---

## Troubleshooting

**Living room lamps not turning on at sunset:**
- Verify sun integration is functioning
- Check Scene 31 can be manually activated via Insteon
- Review automation trace for `living_room_lamps_fade_up`

**Living room lamps not turning off at night:**
- Check if main floor shows occupied (`binary_sensor.main_floor_occupancy`)
- Verify 2-hour timeout hasn't been reached (forces off regardless)
- Scene 24 can always be manually activated (PS01 switch)

**Living room lamps not turning off in morning:**
- Verify photocell sensor is functioning (`switch.photocell_micro_module`)
- Check if sensor transitions to "on" state at dawn
- Automation waits 35 minutes after photocell turns on, between 5:00-10:00 AM
- Scene 31 can always be manually deactivated

**Seasonal lights not responding to toggle:**
- Verify `input_boolean.seasonal_lighting_enabled` state
- Check Scene 23 responds to manual activation
- Ensure automations are enabled

**Seasonal lights not turning off at 11:55 PM:**
- Check main floor occupancy sensor
- Automation retries every 31 minutes if occupied
- Manual off via Scene 23 always available

**Living room lamps not activating in away mode:**
- Verify `input_boolean.long_term_away` is on (requires 24h of no main floor occupancy)
- Check automation triggered at correct day/time (M-F 5:27 AM or S-S 7:13 AM)
- Random delay means activation could be up to 81 minutes later (weekday) or 45 minutes later (weekend)
- Review automation traces for `living_room_lamps_away_weekday` or `living_room_lamps_away_weekend`
- Scene 24 can always be manually activated

**Master bedroom lights not auto-off:**
- Verify `binary_sensor.master_bedroom_occupancy` is reporting correctly
- Check Ecobee remote sensor has fresh batteries
- Ensure Scene 30 responds to manual control

**Master bedroom not activating in away mode (evening):**
- Verify `input_boolean.long_term_away` is on (requires 24h of no main floor occupancy)
- Check automation triggered at 9:22 PM
- Random delay means activation could be 0-12 minutes later (9:22-9:34 PM)
- Lights stay on for 17-26 minutes, then auto-off
- If HA restarts during sequence, lights may stay on (acceptable for security simulation)
- Review automation trace for `master_bedroom_away_evening`
- Scene 30 can always be manually activated/deactivated

---

## YAML Packages

- `/config/packages/lightingindoor.yaml`

---
