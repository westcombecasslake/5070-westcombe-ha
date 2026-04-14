# Garage Systems

**Last Updated:** April 2026

---

## Overview

The Garage Systems subsystem provides three independent functions:

- **Motion/door-activated lighting** for safety and convenience
- **Freezing temperature monitoring** to protect against pipe freeze damage
- **Away mode sensor monitoring** with Pushover alerts when vacation mode is active

Lighting and temperature alerts operate continuously. Away monitoring is user-controlled via the `vacation_mode` helper and only activates when enabled.

---

## Motion/Door-Activated Lighting

### What It Does

Automatically turns on all garage lights when motion is detected or a door is opened, using Insteon Scene 20 for immediate device-to-device response plus Home Assistant enforcement. Lights automatically turn off 10 minutes after the last triggering event.

**Rationale:** Device-to-device Insteon scene triggering provides instant light response without waiting for Home Assistant to process. HA enforces the timed shutoff to prevent lights being left on indefinitely. Restart mode resets the timer on each new trigger, keeping lights on during continuous activity while ensuring they turn off after the garage is vacant.

### Hardware

**YoLink Motion Sensor:**
- Location: Garage
- Entity: `binary_sensor.garage_motion`
- Detection: PIR motion sensor

**YoLink Door Sensors:**
- Back Yard Garage Door: `binary_sensor.back_yard_garage`
- Garage Entry Door: `binary_sensor.entry_door_garage`
- Detection: Open/close contact sensor

**Garage Lights:**
- Controlled via Insteon Scene 20
- Multiple Insteon switches
- See: insteon-scenes.md

### Key Entities

**Sensors:**
- `binary_sensor.garage_motion` - YoLink motion detector
- `binary_sensor.back_yard_garage` - YoLink back yard garage door sensor
- `binary_sensor.entry_door_garage` - YoLink garage entry door sensor

**User Controls:**
- None (fully automatic, but Scene 20 can be manually controlled)

### Automations

**Garage Motion/Door Lights via Insteon Scene:**
- ID: `garage_motion_lights`
- Alias: `Garage - Motion Lights via Insteon Scene`
- Trigger: Motion detected OR any door opened (state = on)
- Mode: Restart (resets timer on each new trigger)
- Action Sequence:
  1. Activate Scene 20 (backup to device-to-device trigger)
  2. Wait 3 seconds
  3. Activate Scene 20 (double-send for reliability)
  4. Wait 10 minutes
  5. Deactivate Scene 20 (first attempt)
  6. Wait 3 seconds
  7. Deactivate Scene 20 (backup attempt)
- Rationale: 10-minute delay allows activity to complete, 3-second backup ensures lights actually turn off

**YAML:** garage.yaml

---

## Freezing Temperature Alerts

### What It Does

Monitors garage temperature and sends repeating alerts if temperature remains at or below 32F (freezing) for 1+ hours. Alerts repeat every 2 hours while temperature stays below freezing to ensure continued awareness of potential pipe freeze risk.

**Rationale:** The garage contains materials that can be damaged by freezing, camper batteries on trickle charger that need monitoring, and is adjacent to the east suite bathroom with water pipes nearby. Prolonged freezing temperatures risk pipe damage and flooding. 1-hour confirmation period prevents false alerts from brief cold snaps. Repeating alerts ensure the condition isn't forgotten during extended cold periods. HIGH priority ensures alerts are noticed and acted upon.

### Hardware

**YoLink Temperature Sensor:**
- Location: Garage
- Entity: `sensor.garage_smoke_alarm_temperature`
- Monitors: Ambient garage temperature (built into YoLink smoke/CO detector)

### Key Entities

**Sensors:**
- `sensor.garage_smoke_alarm_temperature` - YoLink smoke/CO detector thermometer

**User Controls:**
- None (fully automatic)

### Automations

**Garage Below Freezing Alert:**
- ID: `garage_below_freezing_alert`
- Alias: `Garage - Below Freezing Alert`
- Trigger: Temperature <= 32F for 1+ hours
- Repeat: Every 2 hours while below freezing
- Action: Send HIGH priority alert
- Notification: Pushover House Alerts

**YAML:** garage.yaml

---

## Away Mode Sensor Monitoring

### What It Does

When vacation mode is active, monitors all three garage sensors (motion and both doors) and sends Pushover alerts via House Alerts. Designed to inform without spamming — the first trigger sends an immediate alert, and subsequent activity is counted and summarized every 30 minutes. The monitoring session automatically resets if no new activity occurs for 1.5 hours, and fully resets when vacation mode is turned off.

**Rationale:** The garage is locked and closed when away. Any sensor activity warrants awareness but repeated alerts for sustained activity (wind, sensor drift, wildlife) would be disruptive. The summary approach balances awareness with noise control.

### Alert Behavior

1. **First trigger:** Immediate notification identifying which sensor fired
2. **Subsequent triggers:** Counted silently
3. **Every 30 minutes:** Summary sent only if new activity occurred since last summary ("X additional detection(s) in the last 30 minutes")
4. **No activity for 1.5 hours:** Session ends, all state resets automatically
5. **Vacation mode turned off:** Full reset regardless of session state

### Helpers

| Helper | Entity | Purpose |
|--------|--------|---------|
| Vacation Mode | `input_boolean.vacation_mode` | User-controlled master switch for all away mode features. Shared with lighting_indoor.yaml |
| Garage Away Alert Active | `input_boolean.garage_away_alert_active` | Tracks whether a monitoring session is currently armed |
| Garage Away Trigger Count | `input_number.garage_away_trigger_count` | Counts additional sensor triggers after first alert |
| Garage Away Last Trigger | `input_datetime.garage_away_last_trigger` | Timestamps last trigger for 1.5-hour inactivity check |

### Key Entities

**Sensors (shared with lighting automation):**
- `binary_sensor.garage_motion`
- `binary_sensor.back_yard_garage`
- `binary_sensor.entry_door_garage`

### Automations

**Garage Away Mode Sensor Trigger:**
- ID: `garage_away_sensor_trigger`
- Alias: `Garage - Away Mode Sensor Trigger`
- Trigger: Any of the 3 sensors goes on
- Condition: `input_boolean.vacation_mode` is on
- Mode: Queued
- Action: Stamps last trigger time; if first trigger arms the session and sends alert; if already armed increments the counter

**Garage Away Mode Summary Loop:**
- ID: `garage_away_summary_loop`
- Alias: `Garage - Away Mode Summary Loop`
- Trigger: `garage_away_alert_active` turns on
- Mode: Single
- Action: Loops every 30 minutes; exits and resets if no activity for 1.5 hours; sends summary if counter > 0; stays quiet if counter = 0

**Garage Away Mode Reset:**
- ID: `garage_away_reset_on_exit`
- Alias: `Garage - Reset Away Alert on Vacation Mode Off`
- Trigger: `vacation_mode` turns off
- Action: Turns off alert active boolean, resets trigger counter

**YAML:** garage.yaml

---

## User Controls

| Control | Entity | Access | Purpose |
|---------|--------|--------|---------|
| Vacation Mode | `input_boolean.vacation_mode` | Android home screen widget | Enables/disables all away mode monitoring and lighting simulation across the house |

**Manual Overrides:**
- Scene 20 can be manually activated/deactivated via Insteon controls
- Motion sensor can be physically disabled if needed (check YoLink app)

---

## Notifications

**Pushover House Alerts** (`notify.house_alerts`):

| Alert | Condition | Repeat |
|-------|-----------|--------|
| Garage Below Freezing | Temperature <= 32F for 1+ hours | Every 2 hours while below freezing |
| Garage Activity While Away | First sensor trigger during vacation mode | Once per session |
| Garage Activity Update | Additional triggers during vacation mode | Every 30 minutes if new activity occurred |

---

## Related Systems

**YoLink Sensors:**
- Motion sensor for lighting and away monitoring
- Door sensors for lighting and away monitoring
- Temperature sensor for freeze alerts
- See: yolink-integration.md

**Insteon Scenes:**
- Scene 20: Garage lighting
- See: insteon-scenes.md

**Vacation Mode:**
- `input_boolean.vacation_mode` is declared in garage.yaml and shared with lighting_indoor.yaml
- Away mode lighting simulation (living room, master bedroom) also uses this helper
- See: lighting-indoor.md

**Battery Monitoring:**
- Motion sensor, door sensors, and temperature sensor batteries monitored separately
- Two-level battery alerts (30% and 15%)
- See: battery-monitoring-system.md

---

## Troubleshooting

**Motion/door lights not turning on:**
- Check `binary_sensor.garage_motion`, `binary_sensor.back_yard_garage`, and `binary_sensor.entry_door_garage` are reporting correctly
- Verify Scene 20 responds to manual activation
- Check YoLink sensors have fresh batteries
- Ensure YoLink Hub is online

**Lights not turning off after 10 minutes:**
- Verify no continuous motion or door activity is being detected
- Check if automation is enabled
- Scene 20 can always be manually deactivated
- Backup deactivation occurs 3 seconds after first attempt

**Freezing alerts not received:**
- Verify temperature sensor is reporting correctly
- Check temperature must be <=32F for full hour before first alert
- Confirm Pushover House Alerts service is functioning
- Check entity name in garage.yaml matches actual sensor

**Away alerts not firing:**
- Confirm `input_boolean.vacation_mode` is on
- Check `input_boolean.garage_away_alert_active` state in Developer Tools
- If boolean is stuck on after returning home, turn vacation mode off to force reset
- Verify all three sensor entities are reporting correctly

**Away alerts firing when not away:**
- Confirm vacation mode was turned off after returning home
- Widget on Android home screen should show current state

---

## YAML Packages

- `/config/packages/garage.yaml`