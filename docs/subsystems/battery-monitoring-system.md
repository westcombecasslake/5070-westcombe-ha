# Battery Monitoring System

## Overview

The battery monitoring system tracks all 24 battery-powered devices in the home automation system and sends tiered alerts based on battery levels. The system uses a two-tier notification strategy that separates early warnings from critical alerts.

---

## System Architecture

### Components

**Package File:** `/config/packages/batteryalerts.yaml`

**Key Components:**
1. **Input Text Helper** - `input_text.battery_status` for Android Auto display
2. **Scripts** - Manual reporting and status updates
3. **Automations** - Automatic battery monitoring and alerting (3 automations)

**Dependencies:**
- Pushover integration (`notify.battery_alerts` and `notify.house_alerts`)
- All battery sensors from YoLink and Ring devices

---

## Two-Tier Alert System

### Alert Routing Strategy

```
+-------------------------------------------------------------+
|                    BATTERY MONITORING                        |
+-------------------------------------------------------------+
                            |
                            v
                    Battery Level?
                            |
            +---------------+---------------+
            |               |               |
            v               v               v
         > 30%          15-30%           < 15%
            |               |               |
            |               v               v
            |      +-----------------+  +-----------------+
            |      | Battery Alerts  |  |  House Alerts   |
            |      |   (Normal 0)    |  |   (High 1)      |
            |      +-----------------+  +-----------------+
            |               |               |
            |               v               v
            |      Low Battery Warning  CRITICAL Battery Alert
            |      Order replacement   REPLACE IMMEDIATELY
            |
            v
        No Alert
```

### 30% Low Battery Warning

**Service:** `notify.battery_alerts`
**Priority:** Normal (0)
**Trigger:** Battery drops below 30%
**Purpose:** Early warning to order replacement batteries
**Time to Act:** Several weeks to months

**Notification Example:**
```
Title: Low Battery Warning
Message: Device: Front Door
         Battery Level: 28%

         Order replacement battery
```

**Characteristics:**
- Standard notification tone
- Can use quiet hours
- Review at your convenience
- Gives advance planning time

### 15% Critical Battery Alert

**Service:** `notify.house_alerts` (SAME AS SMOKE/CO/SUMP FAILURES)
**Priority:** High (1)
**Trigger:** Battery drops below 15%
**Purpose:** Urgent warning - replace immediately
**Time to Act:** Days to weeks

**Notification Example:**
```
Title: CRITICAL Battery Alert
Message: Device: Front Door
         Battery Level: 12%

         REPLACE IMMEDIATELY
```

**Characteristics:**
- Loud siren tone (same as smoke detector alerts)
- Persistent vibration
- NO quiet hours - bypasses sleep mode
- Cannot be casually dismissed
- Treated as house emergency

**Rationale:**
A smoke detector at 12% battery is a safety issue that deserves the same urgency as actual smoke detection. Critical battery failures can compromise home safety and security systems.

---

## Monitored Devices (24 Total)

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

### 1. 30% Low Battery Warning
```yaml
ID: battery_low_30_percent
Mode: parallel (multiple devices can alert simultaneously)
Trigger: Battery < 30% for 5 minutes
Condition: Battery > 15% (avoids duplicate with critical alert)
Action:
  - Send notification to Battery Alerts
  - Update battery status helper
```

### 2. 15% Critical Battery Alert
```yaml
ID: battery_critical_15_percent
Mode: parallel
Trigger: Battery < 15% for 5 minutes
Action:
  - Send notification to House Alerts (HIGH PRIORITY)
  - Update battery status helper
```

### 3. Daily Battery Status Update
```yaml
ID: battery_status_daily_update
Trigger: 8:00 AM daily
Action: Update input_text.battery_status helper
Purpose: Keep Android Auto display current
```

### Design Patterns

**5-Minute Delay:**
- Prevents false alerts from temporary glitches
- Ensures stable low battery reading
- Reduces noise from sensor initialization

**Parallel Mode:**
- Multiple batteries can alert simultaneously
- Each device gets its own notification
- Important for tracking multiple low batteries

**Template Usage:**
- `{{ trigger.entity_id }}` - Which sensor triggered
- `{{ state_attr(trigger.entity_id, 'friendly_name') }}` - Device name
- `{{ states(trigger.entity_id) }}` - Current battery level

---

## Battery Status Helper

### Purpose
Provides at-a-glance battery system status for:
- Android Auto dashboard display
- Quick status checks
- Automation conditions

### States
- **Normal** - All batteries above 30%
- **Low** - One or more batteries below 30%
- **Critical** - One or more batteries below 15%

### Update Schedule
- Automatically updated by all battery automations
- Daily refresh at 8:00 AM
- Updated on system startup

---

## Testing & Verification

### Manual Testing

**Test Battery Alerts (30% Warning):**
```
Developer Tools -> Services
Service: notify.battery_alerts
Data:
  title: "Test: Low Battery Warning"
  message: "Testing 30% alert routing"
```

**Test House Alerts (15% Critical):**
```
Developer Tools -> Services
Service: notify.house_alerts
Data:
  title: "Test: CRITICAL Battery"
  message: "Testing 15% alert routing - should be LOUD"
```

### Verification Checklist
1. Check Pushover app - two separate buckets
2. Confirm 30% test in **Battery Alerts** bucket
3. Confirm 15% test in **House Alerts** bucket
4. Verify House Alerts has loud/persistent sound
5. Verify no quiet hours on House Alerts

### Automation Traces
```
Developer Tools -> Automations
Find: "Battery - 30% Low Warning"
View Traces -> Check last trigger details
```

---

## Battery Replacement Guide

### When to Replace

**30% Warning Received:**
- Order replacement batteries
- No immediate action required
- Plan replacement within 4-8 weeks

**15% Critical Alert Received:**
- Replace within 3-7 days
- Device may fail soon
- Prioritize safety-critical devices

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
6. Verify recovery notification (battery > 50%)

**Safety-Critical Devices (Smoke/CO):**
1. Replace battery
2. Press test button to verify
3. Confirm sensor LED flash
4. Monitor for recovery notification
5. Document replacement date

---

## Troubleshooting

### No Notifications Received

**Check Automation Status:**
```
Settings -> Automations
Search: "Battery"
Verify all 3 automations are enabled
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

### False Alerts

**Symptoms:** Alert received but battery actually fine

**Possible Causes:**
- Sensor reporting temporary glitch
- Battery voltage fluctuation during heavy use
- Sensor initialization after restart

**Solutions:**
- Wait 5 minutes - automation has built-in delay
- Check actual battery level in States
- If persistent, replace battery preemptively

### Missed Alerts

**Symptoms:** Battery failed but no notification received

**Check:**
1. Battery sensor entity appears in the trigger entity list of the `battery_low_30_percent` automation in batteryalerts.yaml
2. Automation is enabled
3. Pushover service configured correctly
4. Battery dropped below threshold gradually (5-min delay)

**Temporary Fix:**
- Run manual battery report script
- Enable daily status updates

---

## Maintenance Schedule

### Daily (Automatic)
- Battery status helper updates at 8:00 AM
- Monitors all 24 devices continuously

### Weekly (Manual)
- Review Battery Alerts bucket for 30% warnings
- Order replacement batteries if needed

### Monthly (Manual)
- Test one smoke/CO detector manually
- Verify alert received in Pushover
- Check battery status report

### Annually (Manual)
- Review battery replacement schedule
- Replace aging batteries preemptively (3+ years old)
- Update battery inventory
- Test all 3 automations with manual triggers

---

## Integration with Other Systems

### Android Auto Display
- `input_text.battery_status` shows: Normal, Low, or Critical
- Updates automatically via automations
- Quick at-a-glance status while driving

### System Startup
- Battery status updated on Home Assistant restart
- Ensures status helper is current after system changes
- Called by `systemstartup.yaml`

### Safety Monitoring Package
- Smoke/CO detector batteries monitored
- Critical alerts ensure life safety devices operational
- Integrated with smoke/CO offline monitoring

### Sump Pump Monitoring
- Sump sensor batteries monitored
- Critical for backup pump failure detection
- Prevents basement flooding

---

## Why This Configuration Works

### Proactive + Insistent = Effective

**30% Warning (Proactive):**
- Gives weeks of advance notice
- Time to order batteries in bulk
- Reduce emergency late-night battery runs
- Review at your convenience

**15% Critical (Insistent):**
- Cannot be ignored or forgotten
- Same priority as smoke detector alerts
- Wakes you up if needed
- Multiple critical batteries = immediate action

### Organized by Urgency

**Battery Alerts Bucket:**
- All routine battery maintenance
- 30% warnings
- Replacement confirmations
- Review daily or weekly

**House Alerts Bucket:**
- Only critical emergencies
- 15% critical batteries alongside smoke/CO/leaks
- Review immediately
- Cannot be silenced

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
**Monitored Sensors:** 24 entities ending in `_battery`
**Not Monitored:** Ecobee remote sensor (binary HomeKit reporting)

### Support

**Pushover Issues:** Check Settings -> Integrations -> Pushover
**Battery Sensor Issues:** Check device-specific integrations (YoLink, Ring)
**Automation Issues:** Check Developer Tools -> Automations -> Traces

---

**Document Status:** Current and Complete
**Last Updated:** February 2026
**System Status:** Fully operational with 24 monitored devices
**Configuration:** Two-tier alerting (30% -> Battery Alerts, 15% -> House Alerts)
**Note:** Ecobee remote sensor (Master Bedroom) not monitored - replace CR2032 when sensor stops responding