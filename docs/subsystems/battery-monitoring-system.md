# Battery Monitoring System

## Overview

The battery monitoring system tracks all battery-powered IoT devices in the home automation system and sends alerts based on battery levels. Devices are auto-discovered by `device_class: battery` — no hardcoded entity list is required, and new devices are included automatically. Non-IoT devices (phones, tablets, speakers, etc.) are excluded from all monitoring.

---

## System Architecture

### Components

**Package File:** `/config/packages/batteryalerts.yaml`

**Key Components:**
1. **Input Text Helper** - `input_text.battery_status` for Android Auto display
2. **Scripts** - Manual reporting and status updates
3. **Automations** - Automatic battery monitoring and alerting (1 automation)

**Dependencies:**
- Pushover integration (`notify.battery_alerts`)
- All battery sensors discovered via `device_class: battery` (non-IoT devices excluded)

---

## Alert System

### Alert Routing Strategy

```
+-------------------------------------------------------------+
|                    BATTERY MONITORING                        |
+-------------------------------------------------------------+
                            |
                            v
              Nightly poll at 23:55 — all sensors
                            |
            +---------------+---------------+
            |                               |
            v                               v
    Battery <= 25% or unavailable       All batteries > 25%
            |                               v
            v                          No notification
   +-----------------+                 (silent night)
   | Battery Alerts  |
   |   (Normal 0)    |
   +-----------------+
            |
            v
   Consolidated alert listing
   all affected devices
```

### 25% Battery Alert

**Service:** `notify.battery_alerts`
**Priority:** Normal (0)
**Trigger:** Nightly poll at 23:55 finds one or more sensors at ≤25% or unavailable
**Purpose:** Early warning to order replacement batteries
**Time to Act:** Several weeks to months

**Notification Example:**
```
Title: Battery Alert
Message: Front Door: 22%
         Sump Pump Running Sensor: 18%
         Hot Water Heater Leak Sensor: Unavailable
```

**Characteristics:**
- Single consolidated notification listing all affected devices
- Standard notification tone
- Silent when all batteries are above 25%
- Gives advance planning time

---

## Device Exclusions

### Automatic Exclusions (Integration-Based)
Mobile devices are excluded automatically via the `mobile_app` integration filter. Any phone or tablet using the Home Assistant companion app is excluded without any manual configuration.

### Manual Exclusions (Entity-Based)
Non-IoT consumer devices such as phones and speakers are excluded individually. These devices report battery state but are not home automation sensors — including them would generate noise.

To add, remove, or review manually excluded entities, search for `rejectattr('entity_id', 'in',` in `/config/packages/batteryalerts.yaml`. The same filter appears in **3 places** (both scripts and the automation) and all 3 must be kept in sync.

---

## Monitored Devices

Devices are auto-discovered via `device_class: battery`. The list below reflects current known devices but does not need to be maintained manually — any new IoT device with a battery sensor is included automatically.

### Safety-Critical Devices (9)
1. Basement Smoke/CO
2. Dan Bedroom Smoke Alarm
3. Den Smoke Alarm
4. Dining Room Smoke/CO
5. Fire CO Alarm
6. Garage Smoke Alarm
7. Master Bedroom Smoke/CO
8. Office Smoke Alarm
9. Sarah Bedroom Smoke Alarm

**Battery Type:** YoLink replaceable batteries (typically AA)
**Expected Life:** 5+ years
**Critical:** These devices provide life safety monitoring

### Door Sensors (2)
10. Back Yard Garage Door (`sensor.back_yard_garage_battery`)
11. Garage Entry Door (`sensor.garage_entry_door_battery`)

**Battery Type:** YoLink AA batteries
**Expected Life:** 2-3 years
**Purpose:** Trigger garage lighting and provide open/close status

### Leak Detection Devices (3)
12. Hot Water Heater Leak Sensor
13. Sump Leak Sensor
14. Well Leak Sensor

**Battery Type:** YoLink AA batteries
**Expected Life:** 2-3 years
**Critical:** Early leak detection prevents major water damage

### Environmental Monitoring (7)
15. Basement Humidity Sensor
16. Garage Freezer Temp
17. Garage Fridge Temp
18. Kitchen Freezer Temp
19. Kitchen Fridge Temp
20. Laundry Temp/Humidity
21. Master Bath Temp/Humidity

**Battery Type:** YoLink AA batteries
**Expected Life:** 2-3 years
**Purpose:** Monitor HVAC performance and appliance status

### Sump Pump System (2)
22. Sump Pump Depth Alert
23. Sump Pump Running Sensor

**Battery Type:** YoLink AA batteries
**Expected Life:** 2-3 years
**Critical:** Backup pump monitoring prevents basement flooding

### Access Control (1)
24. Front Door (Ring 2 Doorbell)

**Battery Type:** Ring rechargeable battery pack
**Recharge Method:** Micro USB adapter
**Expected Life:** 6-12 months depending on activity/temperature
**Purpose:** Security, motion detection for premise lighting, video doorbell

**Note on Ecobee Remote Sensor (Master Bedroom):**
- Uses CR2032 coin cell battery (3-5 year life per Ecobee support)
- **NOT monitored** - HomeKit integration reports binary status (100% = good, 0% = dead)
- No gradual battery decline available through integration
- Manual replacement recommended when sensor stops responding
- Check periodically or replace proactively every 4 years

---

## Automation Details

### 1. Nightly Battery Check
```yaml
ID: battery_daily_check
Alias: "Battery - Daily Check"
Trigger: 23:55 daily
Action:
  1. Run script.update_battery_status (updates helper regardless of battery state)
  2. Build consolidated message of all sensors <= 25% or unavailable
  3. Condition: only proceed if message is non-empty
  4. Send notification to Battery Alerts
```

### Design Patterns

**Scheduled Poll (not event-driven):**
- Runs once nightly at 23:55
- Checks all battery sensors at that moment
- No per-event triggers or delays

**Consolidated Notification:**
- All affected devices listed in a single message
- Notification only sent when at least one device needs attention
- Silent when everything is healthy — no false reassurance messages

**Auto-Discovery:**
- Uses `device_class: battery` to find all battery sensors automatically
- Non-IoT devices excluded via integration filter and manual entity exclusions
- New IoT devices included without any configuration changes

**Template Usage:**
- Iterates `states.sensor` filtered by `device_class: battery`
- `device_name(sensor.entity_id)` — human-readable device name
- `sensor.state` — current battery percentage or `unavailable`

---

## Battery Status Helper

### Purpose
Provides at-a-glance battery system status for:
- Android Auto dashboard display
- Quick status checks
- Automation conditions

### States
- **Normal** - All batteries above 25% and online
- **Low** - One or more batteries at or below 25%
- **Offline** - One or more sensors reporting unavailable (takes priority over Low)

### Update Schedule
- Updated by the nightly battery check automation (23:55)
- Updated whenever `script.update_battery_status` is run manually

---

## Testing & Verification

### Manual Testing

**Test Battery Alerts:**
```
Developer Tools -> Services
Service: notify.battery_alerts
Data:
  title: "Test: Battery Alert"
  message: "Testing 25% alert routing"
```

### Verification Checklist
1. Check Pushover app — Battery Alerts bucket
2. Confirm test notification appears in **Battery Alerts** bucket
3. Verify tone and delivery behavior

### Automation Traces
```
Developer Tools -> Automations
Find: "Battery - Daily Check"
View Traces -> Check last trigger details
```

---

## Battery Replacement Guide

### When to Replace

**25% Alert Received:**
- Order replacement batteries
- No immediate action required
- Plan replacement within 4-8 weeks

**Sensor Reporting Unavailable:**
- Check device is powered and within range
- Replace battery promptly — device may have already failed

### Battery Types Required

**AA (YoLink Devices):**
- Smoke/CO detectors (9x)
- Temp/humidity sensors (7x)
- Leak sensors (4x)
- Sump pump sensors (2x)
- Motion sensors (1x)
- Door sensors (2x)
- **Quantity to keep on hand:** 32-37 batteries

**Ring Rechargeable:**
- Front door doorbell (1x)
- Recharge every 6-12 months
- Keep micro USB charging cable accessible

**CR2032 Coin Cell (Not Monitored):**
- Ecobee remote sensor (Master bedroom)
- Replace when sensor stops responding
- Check periodically or replace every 4 years
- **Quantity to keep on hand:** 1-2 batteries

### Replacement Procedure

**General Process:**
1. Receive notification with device name
2. Locate device (see device-inventory.md)
3. Remove old battery
4. Install fresh battery
5. Test device operation
6. Verify recovery (device no longer appears in next nightly report)

**Safety-Critical Devices (Smoke/CO):**
1. Replace battery
2. Press test button to verify
3. Confirm sensor LED flash
4. Monitor for recovery in next nightly report
5. Document replacement date

---

## Troubleshooting

### No Notifications Received

**Check Automation Status:**
```
Settings -> Automations
Search: "Battery"
Verify automation is enabled
```

**Check Battery Sensor:**
```
Developer Tools -> States
Search: sensor.*_battery
Verify sensor is reporting numeric value
Check "unavailable" or "unknown" states
```

**Check Pushover Integration:**
```
Settings -> Integrations -> Pushover
Verify integration loaded successfully
Test with Developer Tools -> Services
```

**Check Notification Condition:**
- Notification only fires when at least one device is ≤25% or unavailable
- If all batteries are healthy, no notification is sent — this is expected behavior

### False Alerts

**Symptoms:** Alert received but battery actually fine

**Possible Causes:**
- Sensor reporting temporary glitch
- Battery voltage fluctuation during heavy use
- Sensor initialization after restart

**Solutions:**
- Check actual battery level in States
- If persistent, replace battery preemptively

### Missed Alerts

**Symptoms:** Battery failed but no notification received

**Check:**
1. Battery sensor entity has `device_class: battery` attribute
2. Entity is not excluded — check both the `mobile_app` integration filter and the manual exclusion list in `/config/packages/batteryalerts.yaml`
3. Automation is enabled
4. Pushover service configured correctly

**Temporary Fix:**
- Run manual battery report script
- Check `input_text.battery_status` for current state

---

## Maintenance Schedule

### Daily (Automatic)
- Battery check and notification runs at 23:55
- Monitors all auto-discovered battery sensors continuously

### Weekly (Manual)
- Review Battery Alerts bucket for 25% warnings
- Order replacement batteries if needed

### Monthly (Manual)
- Test one smoke/CO detector manually
- Verify alert received in Pushover
- Run manual battery status report

### Annually (Manual)
- Review battery replacement schedule
- Replace aging batteries preemptively (3+ years old)
- Update battery inventory
- Test automation with manual trigger

---

## Integration with Other Systems

### Android Auto Display
- `input_text.battery_status` shows: Normal, Low, or Offline
- Updated nightly by the battery check automation
- Quick at-a-glance status while driving

### System Startup
- Verify `systemstartup.yaml` calls `script.update_battery_status` on restart
- Ensures status helper is current after system changes

### Safety Monitoring Package
- Smoke/CO detector batteries monitored via auto-discovery
- Integrated with smoke/CO offline monitoring

### Sump Pump Monitoring
- Sump sensor batteries monitored via auto-discovery
- Critical for backup pump failure detection
- Prevents basement flooding

---

## Related Documentation

- **[Device Inventory](device-inventory.md)** - Physical locations of all battery devices
- **[Pushover Integration](pushover-integration.md)** - Notification system details
- **[Safety Monitoring](safety-monitoring.md)** - Smoke/CO detector integration
- **[Sump Pump Monitoring](sumppump.yaml)** - Sump sensor integration

---

## Quick Reference

### Commands

**Manual Battery Report:**
```
Developer Tools -> Services
Service: script.battery_status_report
```

**Update Battery Status:**
```
Developer Tools -> Services
Service: script.update_battery_status
```

**Check Current Status:**
```
Developer Tools -> States
Entity: input_text.battery_status
```

### File Locations

**Package File:** `/config/packages/batteryalerts.yaml`
**Integration:** Settings -> Integrations -> Pushover
**Monitored Sensors:** All IoT entities with `device_class: battery`
**Excluded — Integration:** All `mobile_app` entities (phones, tablets) excluded automatically
**Excluded — Manual:** Non-IoT consumer devices (e.g. speakers); see `rejectattr('entity_id', 'in',` in `/config/packages/batteryalerts.yaml`
**Not Monitored:** Ecobee remote sensor (binary HomeKit reporting)

### Support

**Pushover Issues:** Check Settings -> Integrations -> Pushover
**Battery Sensor Issues:** Check device-specific integrations (YoLink, Ring)
**Automation Issues:** Check Developer Tools -> Automations -> Traces

---

**Document Status:** Current and Complete
**Last Updated:** April 2026
**System Status:** Fully operational — auto-discovery via device_class: battery
**Configuration:** Single-tier alerting (≤25% or unavailable → Battery Alerts at 23:55)
**Note:** Ecobee remote sensor (Master Bedroom) not monitored — replace CR2032 when sensor stops responding