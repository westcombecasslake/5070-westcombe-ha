# Premise Lighting

**Last Updated:** February 2026

---

## Overview

The Premise Lighting subsystem manages all exterior lighting through photocell-based automation that responds to actual darkness rather than fixed schedules. The system includes seasonal adjustments to compensate for varying light levels at dusk throughout the year, motion-activated overrides for security, and randomized shutoff timing to prevent predictable patterns.

The photocell sensor (located indoors at a south-facing second-floor bedroom window) provides reliable dusk detection while avoiding false triggers from outdoor weather conditions. All premise lights are controlled through Insteon native scenes, ensuring synchronized operation even if Home Assistant is offline.

---

## Photocell Automation

### What It Does

Turns on all exterior premise lights when the photocell detects dusk, with seasonal delay adjustments to ensure lights activate at a consistent darkness level year-round. Includes daily photocell reset to prevent analog sensor drift and backup failsafe automation in case of photocell failure.

**Rationale:** Photocells trigger at different ambient light levels depending on the season (brighter in summer, darker in winter). Seasonal delays compensate for this variation so lights turn on when it's actually dark enough, not just when the photocell triggers. Indoor placement prevents weather interference while south-facing window provides reliable dusk detection.

### Hardware

**Photocell Sensor:**
- Type: Analog photocell with Insteon micro module interface
- Location: Indoor, south-facing second-floor bedroom window
- Entity: `switch.photocell_micro_module`

**Premise Lights:**
- Controlled via Insteon Scene 21 (all exterior lights)
- Multiple Insteon switches throughout exterior
- See: insteon-scenes.md for complete scene details

### Key Entities

**Sensors:**
- `switch.photocell_micro_module` - Photocell state (on = dusk detected)
- `sun.sun` - Sun integration for backup failsafe

**Helper Sensors:**
- None (uses input_number helpers for seasonal delays)

**User Controls:**
- `input_number.premise_summer_delay` - Delay for May-July (0-45 min)
- `input_number.premise_shoulder_delay` - Delay for April/August (0-45 min)
- `input_number.premise_spring_fall_delay` - Delay for March/September (0-45 min)
- `input_number.premise_winter_delay` - Delay for October-February (0-45 min)

### Automations

**Daily Photocell Sensor Reset:**
- `premise_dusk_sensor_daily_reset`
- Trigger: 1:00 PM daily
- Action: Cycles photocell module (off-on-off sequence)
- Rationale: Clears analog sensor drift/confusion for reliable evening triggering

**Dusk On (Primary):**
- `premise_lights_dusk_on`
- Trigger: Photocell detects dusk (`switch.photocell_micro_module` turns on)
- Condition: Between 3:45 PM and 7:30 PM (prevents false triggers)
- Action: Apply seasonal delay, activate Insteon Scene 21 (double-send for reliability)
- Notification: Pushover Operational Log with season, delay time, and actual dusk time

**Dusk Backup (Failsafe):**
- `premise_lights_dusk_backup`
- Trigger: 15 minutes after sunset
- Condition: Photocell still off AND premise lights still off AND sun below horizon
- Action: Activate Scene 21 (double-send), send HIGH priority alert
- Notification: Pushover House Alerts - photocell failure detected

**Dawn Off:**
- `premise_lights_dawn_off`
- Trigger: 15 minutes before sunrise
- Action: Deactivate Scene 21 (double-send)
- Rationale: Turns off lights at dawn to save energy

**YAML:** lightingpremise.yaml

---

## Motion Overrides

### What It Does

Provides motion-activated lighting at the front porch when Ring doorbell detects motion during dark hours (after premise lights have shut off for the night). Creates an 8-14 minute randomized timer for security lighting without requiring manual intervention.

**Rationale:** After scheduled nighttime shutoff, motion detection provides security lighting when someone approaches the front door. The randomized timer ensures lights stay on long enough for safe entry/exit while automatically shutting off to save energy. Timer restarts on continued motion to keep lights on as long as activity continues.

### Hardware

**Ring Video Doorbell:**
- Model: Ring Video Doorbell 2
- Location: Front door
- Entity: `event.front_door_motion` (event-based, more reliable than binary sensor)
- See: ring-integration.md

**Premise Lights:**
- Controlled via Insteon Scene 21 (all exterior lights)
- See: insteon-scenes.md

### Key Entities

**Sensors:**
- `event.front_door_motion` - Ring doorbell motion detection events
- `switch.photocell_micro_module` - Must be "on" (dark) for automation to trigger
- `light.entry_closet_premise` - Checks if premise lights are currently off

**User Controls:**
- None (fully automatic)

### Automations

**Front Porch Motion Timer:**
- `premise_front_porch_motion_timer`
- Trigger: Ring motion detected (`event.front_door_motion`)
- Condition:
  - `input_boolean.front_motion_armed` is on (set at dusk, cleared at dawn — blocks daytime darkness from storms/overcast)
  - `light.entry_closet_premise` is off (premise lights not already on)
  - Not during photocell daily reset (12:58-1:01 PM guard window)
- Action: 
  - Send notification if lights were off (new motion session)
  - Activate Scene 21 (double-send for reliability)
  - Wait 8-14 minutes (randomized)
  - Deactivate Scene 21 (double-send)
- Mode: `restart` (timer resets if new motion detected)
- Rationale: Lights stay on continuously during motion, then turn off 8-14 minutes after LAST motion event

**Note on Ring Motion Events:**
Ring may send duplicate motion events (same timestamp), but `mode: restart` handles this gracefully by simply restarting the timer. This ensures reliable operation without requiring debouncing logic.

**YAML:** lightingpremise.yaml

---

## Front Porch Manual Override

### What It Does

When `light.front_porch_override` is manually turned on, automatically turns it off after 3 hours. Timer restarts if turned on again while running.

### Hardware

**Insteon Switch:**
- Entity: `light.front_porch_override`
- Controlled via Insteon Scene 29

### Automations

**Front Porch Manual Override:**
- `premise_front_porch_manual_override`
- Trigger: `light.front_porch_override` turns on
- Action: Wait 3 hours, deactivate Scene 29 (double-send)
- Mode: `restart` — timer resets on repeated activation

**YAML:** lightingpremise.yaml

---

## Scheduled Shutoff

### What It Does

Turns off all premise lights at a scheduled time with weekend extension (later shutoff Friday/Saturday nights). Includes randomized delay for security and backup automations for redundancy.

**Rationale:** Scheduled shutoff saves energy overnight while randomized timing prevents predictable patterns for security. Weekend extension accommodates later activity on Friday/Saturday. Backup automations ensure lights turn off even if primary automation fails due to HA errors or Insteon scene collisions.

### Key Entities

**User Controls:**
- None (times are fixed in automation logic)

### Automations

**Lights Off Sunday-Thursday:**
- `premise_lights_off_sun_thu`
- Trigger: 9:50 PM
- Condition: Sunday-Thursday only (weekday check: `now().weekday() in [6, 0, 1, 2, 3]`)
- Action: Random 0-12 minute delay, then deactivate Scene 21 (double-send)

**Lights Off Friday-Saturday:**
- `premise_lights_off_fri_sat`
- Trigger: 11:30 PM
- Condition: Friday-Saturday only (weekday check: `now().weekday() in [4, 5]`)
- Action: Random 0-12 minute delay, then deactivate Scene 21 (double-send)

**Backup Off Sunday-Thursday:**
- `premise_lights_backup_off_sun_thu`
- Trigger: 10:15 PM (25 minutes after primary)
- Condition: Sunday-Thursday only
- Action: Deactivate Scene 21 (double-send)
- Rationale: Catches failures from HA condition errors or Insteon scene collisions

**Backup Off Friday-Saturday:**
- `premise_lights_backup_off_fri_sat`
- Trigger: Midnight (30 minutes after primary)
- Condition: Friday-Saturday only
- Action: Deactivate Scene 21 (double-send)
- Rationale: Catches failures from HA condition errors or Insteon scene collisions

**YAML:** lightingpremise.yaml

---

## Maintenance Reminders

### What It Does

Sends biannual reminders (February 1 and August 1) to unplug and replug the photocell sensor module for maintenance. Ensures continued reliable operation throughout the year.

**Rationale:** Periodic power cycle helps prevent analog sensor drift and ensures reliable evening triggering. Biannual timing aligns with seasonal transitions when lighting patterns change most.

### Automations

**Photocell Maintenance Reminder:**
- `premise_photocell_maintenance_reminder`
- Trigger: February 1 and August 1 at 10:00 AM
- Action: Send maintenance reminder via Pushover
- Notification: Pushover House Alerts

**YAML:** lightingpremise.yaml

---

## User Controls

**Access:** Settings → Helpers

**Seasonal Photocell Delays:**
- `input_number.premise_summer_delay` - May-July delay (0-45 min)
- `input_number.premise_shoulder_delay` - April/August delay (0-45 min)
- `input_number.premise_spring_fall_delay` - March/September delay (0-45 min)
- `input_number.premise_winter_delay` - October-February delay (0-45 min)

**Adjustment Guide:**
- Lights turn on too LATE? → DECREASE the delay value
- Lights turn on too EARLY? → INCREASE the delay value

---

## Notifications

**Pushover Operational Log** (`notify.operational_log`):
- Dusk triggered events with season and delay information
- Example: "Season: Winter, Delay: 15 min, Lights activated: 05:45 PM, Actual dusk: 05:30 PM"
- Priority: Silent (informational)

**Pushover House Alerts** (`notify.house_alerts`):
- Photocell failure detected (backup automation activated)
- Photocell maintenance reminders (February 1, August 1)
- Front motion at night (when motion triggers lights after scheduled shutoff)
- Priority: High (attention required)

---

## Related Systems

**Sun Integration:**
- Provides backup failsafe timing (sunset/sunrise)
- See: sun-integration.md

**Ring Integration:**
- Motion detection for security lighting override
- Uses event entity (`event.front_door_motion`) for reliable detection
- See: ring-integration.md

**Insteon Scenes:**
- Scene 21: Main premise lighting (all exterior lights)
- All scene commands use double-send for Insteon reliability
- See: insteon-scenes.md

---

## Troubleshooting

**Lights not turning on at dusk:**
- Check photocell sensor state (`switch.photocell_micro_module`)
- Verify seasonal delay isn't too long for current conditions
- Check if backup automation ran (indicates photocell failure)
- Inspect physical photocell sensor at bedroom window

**Lights turning on too early or too late:**
- Adjust appropriate seasonal delay helper (increase to delay, decrease to advance)
- Wait 1-2 days to observe results before further adjustment
- Biannual reminders (Feb 1, Aug 1) prompt seasonal review

**Lights not turning off at night:**
- Verify scheduled shutoff automations are enabled
- Check backup automations ran (10:15 PM Sun-Thu, midnight Fri-Sat)
- Manual off always available via Insteon Scene 21

**Motion override not working:**
- Verify Ring doorbell motion sensor is online
- Check if `event.front_door_motion` is firing (Developer Tools → Events)
- Verify photocell is "on" (must be dark for override to trigger)
- Check if premise lights are already on (override only works after scheduled shutoff)
- Motion detection must occur after nighttime shutoff to trigger
- Not during 1:00 PM photocell reset window (12:58 PM - 1:01 PM)

**Motion override triggering during the day:**
- Automation blocked by photocell condition (must be "on" = dark)
- Also blocked during 1:00 PM reset window
- Should not trigger during daylight hours

---

## YAML Packages

- `/config/packages/lightingpremise.yaml`

---
