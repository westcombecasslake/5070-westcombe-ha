# Garage Systems

**Last Updated:** February 2026

---

## Overview

The Garage Systems subsystem provides motion- and door-activated lighting for safety and convenience, plus freezing temperature monitoring to protect against pipe freeze damage. Both functions operate independently - lighting for daily usability and temperature alerts for property protection during cold weather.

Motion- and door-activated lighting uses Insteon native scenes with Home Assistant enforcing timed shutoff, ensuring lights turn on immediately when motion is detected or a door is opened, and automatically turn off after 10 minutes. Temperature monitoring sends repeating alerts if the garage remains at or below freezing for extended periods.

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

## User Controls

**Access:** No user-adjustable controls for garage systems

**Manual Overrides:**
- Scene 20 can be manually activated/deactivated via Insteon controls
- Motion sensor can be physically disabled if needed (check YoLink app)

---

## Notifications

**Pushover House Alerts** (`notify.house_alerts`):
- Garage temperature below freezing for 1+ hours
- Repeats every 2 hours while condition persists
- Priority: High (action may be required to prevent damage)

---

## Related Systems

**YoLink Sensors:**
- Motion sensor for lighting automation
- Door sensors for lighting automation
- Temperature sensor for freeze alerts
- See: yolink-integration.md

**Insteon Scenes:**
- Scene 20: Garage lighting
- See: insteon-scenes.md

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

**Lights turn off too quickly:**
- Each motion detection or door open resets the 10-minute timer
- If motion or door activity continues, lights should stay on
- Check motion sensor sensitivity in YoLink app
- 10-minute delay is fixed (not user-adjustable without YAML edit)

**Freezing alerts not received:**
- Verify temperature sensor is reporting correctly
- Check temperature must be <=32F for full hour before first alert
- Confirm Pushover House Alerts service is functioning
- Check entity name in garage.yaml matches actual sensor

**Too many freezing alerts:**
- Alerts repeat every 2 hours while below freezing
- This is intentional - ensures condition isn't forgotten
- No way to disable while still freezing (safety feature)
- Take action to warm garage or insulate pipes

---

## YAML Packages

- `/config/packages/garage.yaml`

---