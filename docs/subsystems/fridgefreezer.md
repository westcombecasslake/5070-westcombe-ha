# Fridge & Freezer Monitoring

**Last Updated:** February 2026

---

## Overview

The Fridge & Freezer Monitoring subsystem provides intelligent temperature and humidity monitoring for all 4 refrigeration appliances in the home. Using a hybrid detection approach, the system distinguishes between door-left-open events and compressor failures to provide actionable alerts with appropriate urgency levels.

All thresholds are data-driven, calibrated from actual sensor history to account for normal operating ranges, defrost cycles, and seasonal variations. The system avoids false positives from grocery loading while catching genuine failures within minutes.

---

## Monitored Appliances

### Kitchen Appliances (HVAC-Controlled Environment)

**Kitchen Fridge:**
- Normal operation: 20-36°F (runs cold)
- Humidity: 20-40% normal
- Warning threshold: >40°F for 30 minutes
- Critical threshold: >42°F (door) or >45°F (compressor)

**Kitchen Freezer:**
- Normal operation: -5 to 5°F
- Humidity: 62-80% normal
- Warning threshold: >22°F for 20 minutes
- Critical threshold: >23°F + >94% humidity for 15 min (door) or >25°F (compressor)

### Garage Appliances (Non-HVAC, Seasonal Variations)

**Garage Fridge:**
- Normal operation: 33-37°F (very stable)
- Humidity: 21-36% normal
- Warning threshold: >40°F for 30 minutes
- Critical threshold: >42°F (door) or >45°F (compressor)

**Garage Freezer:**
- Normal operation: -5 to 5°F
- Aggressive defrost cycles to 28°F (normal)
- Humidity: 53-97% (higher variance)
- Warning threshold: >18°F for 25 minutes (allows defrost)
- Critical threshold: >25°F (door) or >30°F (compressor)

---

## Hybrid Detection System

### How It Works

The system uses both **temperature** and **humidity** readings to distinguish between different failure modes:

**Door Left Open:**
- High temperature + High humidity
- Humid room air enters sealed compartment
- Faster temperature rise
- Different alert message and urgency

**Compressor Failure:**
- High temperature + Normal humidity
- Sealed compartment warms gradually
- Humidity stays relatively normal
- Different troubleshooting steps needed

### Why This Matters

**Actionable Alerts:**
- "Door open" → Walk to fridge and close door
- "Compressor failure" → Check power, call repair service, transfer food

**Reduced False Positives:**
- Grocery loading may raise temp temporarily but humidity stays normal
- Defrost cycles raise temp but have characteristic humidity patterns
- Genuine failures are unmistakable

---

## Alert Levels

### ⚠️ Warning (House Notices)

**Trigger:** Temperature elevated but not critical (20-30 minute delay)

**Purpose:** 
- Non-urgent awareness
- "Something might be developing"
- Time to investigate without panic

**Notification Service:** `notify.house_notices`

**Examples:**
- Kitchen Fridge >40°F for 30 minutes
- Garage Freezer >18°F for 25 minutes

### 🚨 Critical - Door Open (House Alerts)

**Trigger:** High temp + High humidity (10-15 minute delay)

**Detection Logic:**
- Temperature exceeds critical threshold
- AND humidity significantly elevated
- Sustained for alert duration

**Notification Service:** `notify.house_alerts` (Priority 1)

**Message Includes:**
- "Likely cause: Door left open"
- "ACTION REQUIRED: Close door immediately"

### 🚨 Critical - Compressor (House Alerts)

**Trigger:** Very high temperature (10-15 minute delay)

**Detection Logic:**
- Temperature exceeds higher critical threshold
- OR critical threshold without humidity spike
- Sustained for alert duration

**Notification Service:** `notify.house_alerts` (Priority 1)

**Message Includes:**
- "Likely cause: Compressor failure"
- "ACTION REQUIRED: Check immediately - food safety risk"

### ✅ Recovery (House Notices)

**Trigger:** Temperature returned to normal range (5 minute confirmation)

**Purpose:**
- Peace of mind
- Confirms issue resolved
- Documents recovery time

**Notification Service:** `notify.house_notices`

---

## Hardware

### YoLink Temperature/Humidity Sensors

**Kitchen Fridge Sensor:**
- Entity: `sensor.kitchen_fridge_temp_temperature`
- Humidity: `sensor.kitchen_fridge_temp_humidity`
- Battery: `sensor.kitchen_fridge_temp_battery`
- Location: Inside kitchen refrigerator

**Kitchen Freezer Sensor:**
- Entity: `sensor.kitchen_freezer_temp_temperature`
- Humidity: `sensor.kitchen_freezer_temp_humidity`
- Battery: `sensor.kitchen_freezer_temp_battery`
- Location: Inside kitchen freezer

**Garage Fridge Sensor:**
- Entity: `sensor.garage_fridge_temp_temperature`
- Humidity: `sensor.garage_fridge_temp_humidity`
- Battery: `sensor.garage_fridge_temp_battery`
- Location: Inside garage refrigerator

**Garage Freezer Sensor:**
- Entity: `sensor.garage_freezer_temp_temperature`
- Humidity: `sensor.garage_freezer_temp_humidity`
- Battery: `sensor.garage_freezer_temp_battery`
- Location: Inside garage freezer

**Technology:** YoLink LoRa wireless sensors
- Battery life: 2-3 years
- Update frequency: Every few minutes
- Temperature range: -4°F to 140°F
- Humidity range: 0-100% RH

---

## Key Entities

### Temperature Sensors (YoLink)
- `sensor.kitchen_fridge_temp_temperature`
- `sensor.kitchen_freezer_temp_temperature`
- `sensor.garage_fridge_temp_temperature`
- `sensor.garage_freezer_temp_temperature`

### Humidity Sensors (YoLink)
- `sensor.kitchen_fridge_temp_humidity`
- `sensor.kitchen_freezer_temp_humidity`
- `sensor.garage_fridge_temp_humidity`
- `sensor.garage_freezer_temp_humidity`

### Alert State Tracking (created by this package)
- `input_boolean.kitchen_fridge_alert_active`
- `input_boolean.kitchen_freezer_alert_active`
- `input_boolean.garage_fridge_alert_active`
- `input_boolean.garage_freezer_alert_active`

**Purpose:** Prevents duplicate alerts and enables recovery notifications

---

## Automations

### Per-Appliance Automation Set (4 automations × 4 appliances = 16 total)

**Kitchen Fridge Automations:**
1. `kitchen_fridge_warning` - Temperature warning (>40°F, 30 min)
2. `kitchen_fridge_critical_door` - Door open alert (>42°F + >65% humidity, 15 min)
3. `kitchen_fridge_critical_compressor` - Compressor failure (>45°F, 15 min)
4. `kitchen_fridge_recovery` - Temperature recovered (<38°F, 5 min)

Alert state helper: `input_boolean.kitchen_fridge_alert_active`

**Kitchen Freezer Automations:**
1. `kitchen_freezer_warning` - Temperature warning (>22°F, 20 min)
2. `kitchen_freezer_critical_door` - Door open alert (>23°F + >94% humidity, 15 min)
3. `kitchen_freezer_critical_compressor` - Compressor failure (>25°F, 10 min)
4. `kitchen_freezer_recovery` - Temperature recovered (<5°F, 5 min)

Alert state helper: `input_boolean.kitchen_freezer_alert_active`

**Garage Fridge Automations:**
1. `garage_fridge_warning` - Temperature warning (>40°F, 30 min)
2. `garage_fridge_critical_door` - Door open alert (>42°F + >45% humidity, 15 min)
3. `garage_fridge_critical_compressor` - Compressor failure (>45°F, 15 min)
4. `garage_fridge_recovery` - Temperature recovered (<38°F, 5 min)

Alert state helper: `input_boolean.garage_fridge_alert_active`

**Garage Freezer Automations:**
1. `garage_freezer_warning` - Temperature warning (>18°F, 25 min)
2. `garage_freezer_critical_door` - Door open alert (>25°F + >92% humidity, 12 min)
3. `garage_freezer_critical_compressor` - Compressor failure (>30°F, 12 min)
4. `garage_freezer_recovery` - Temperature recovered (<5°F, 5 min)

Alert state helper: `input_boolean.garage_freezer_alert_active`

**YAML:** fridgefreezer.yaml

---

## Threshold Calibration

### Data Source

All thresholds were calibrated from 2 days of actual sensor history (Feb 13-14, 2026):
- Kitchen Fridge: 2,114 temperature samples
- Kitchen Freezer: 718 temperature samples
- Garage Fridge: 53 temperature samples
- Garage Freezer: 1,203 temperature samples

### Calibration Methodology

**Normal Operating Range:**
- Calculated from median and standard deviation
- Excludes door-open events and defrost cycles
- Accounts for seasonal variations (garage)

**Warning Thresholds:**
- Set above 95th percentile of normal operation
- Allows for grocery loading temperature spikes
- Longer delay (20-30 min) reduces false positives

**Critical Thresholds:**
- Set well above any normal operating condition
- Separate thresholds for door (with humidity) vs compressor
- Shorter delay (10-15 min) for faster response

**Humidity Thresholds:**
- Based on observed door-open event data
- Kitchen freezer door-open event showed 96% humidity spike
- Normal operation: 62-80% (freezers), 20-40% (fridges)

### Threshold Adjustments

**Kitchen Freezer Door-Open (Feb 15, 2026):**
- **Issue:** False positive during aggressive defrost cycle (23.9°F + 90.9% humidity)
- **Original:** >20°F + >90% humidity for 10 minutes
- **Adjusted:** >23°F + >94% humidity for 15 minutes
- **Rationale:** Tightened detection to split difference between defrost (23.9°F, 90.9%) and actual door-open (24.6°F, 96%), while maintaining ability to catch genuine door-left-open events

**Kitchen Freezer Warning (Feb 16, 2026):**
- **Issue:** Warning triggering on every automatic defrost cycle (2-3x daily)
- **Analysis:** Defrost cycles every 12-13 hours, consistently peaking at 22-24°F for 20-30 minutes (normal operation)
- **Original:** >15°F for 20 minutes
- **Adjusted:** >22°F for 20 minutes
- **Rationale:** Normal defrost peaks at 22-24°F. Warning threshold set at 22°F to only alert when temperature exceeds normal defrost range, while still providing early warning before critical threshold (25°F compressor failure)

---

## User Controls

**Access:** No user controls - fully automatic monitoring

**Alert State Helpers:**
- Alert state booleans are managed automatically
- Manual reset possible via Developer Tools if needed
- Not intended for user interaction

**Threshold Tuning:**
- Thresholds can be adjusted in YAML if needed
- Current values are data-driven and tested
- Changes should be based on observed patterns

---

## Notifications

### House Notices (Normal Priority)

**Warning Alerts:**
- Temperature elevated but not critical
- Standard notification tone
- Review when convenient

**Recovery Alerts:**
- Temperature returned to normal
- Informational only
- Confirms issue resolved

### House Alerts (High Priority - Priority 1)

**Critical Alerts:**
- Door left open (temp + humidity)
- Compressor failure (very high temp)
- Loud alert tone
- Immediate action required
- No quiet hours

**Configuration:**
- Service: `notify.house_alerts`
- Priority: 1 (Emergency)
- Same urgency as smoke/CO/leak alerts
- Ensures immediate awareness

---

## Related Systems

### YoLink Integration
- All sensors are YoLink devices
- Connected via YoLink Hub
- See: yolink-integration.md

### Battery Monitoring
- Sensor batteries monitored separately
- Two-level battery alerts (30% and 15%)
- See: battery-monitoring-system.md

### Pushover Notifications
- House Notices for warnings/recovery
- House Alerts for critical issues
- See: pushover-integration.md

---

## Troubleshooting

### False Positive Warnings

**Grocery Loading:**
- Temperature may spike >40°F temporarily
- Should recover within 20-30 minutes
- If warnings persist, check door seal or compressor

**Garage Seasonal Variations:**
- Garage ambient temperature affects recovery time
- Summer: Longer recovery, more warnings possible
- Winter: Faster recovery, fewer warnings
- Thresholds account for this but extreme temps may trigger

**Defrost Cycles:**
- Freezers have automatic defrost cycles
- Garage freezer defrost is more aggressive (to 28°F)
- Warning thresholds set above normal defrost peaks
- If warnings during every defrost, thresholds may need adjustment

### Missing Critical Alerts

**Check Sensor Status:**
- Verify YoLink sensors are online
- Check battery levels (>15%)
- Confirm sensors reporting current data

**Check Alert State:**
- View `input_boolean.*_alert_active` states
- If stuck "on", alert won't re-trigger
- Manual reset via Developer Tools

**Check Automation Status:**
- Settings → Automations → Search "fridge" or "freezer"
- Verify all 20 automations are enabled
- Check automation traces for errors

### Actual Failures

**Door Left Open:**
1. Close door immediately
2. Check door seal and alignment
3. Verify nothing blocking door closure
4. Monitor temperature recovery

**Compressor Failure:**
1. Check appliance is plugged in
2. Verify circuit breaker not tripped
3. Listen for compressor running
4. Call appliance repair service
5. Transfer food to backup fridge/freezer if needed
6. Document time of failure for food safety

**Temperature Not Recovering:**
- Even with door closed, temp stays high
- Likely compressor failure
- DO NOT wait - transfer food immediately
- Critical for food safety within 2-4 hours

### Threshold Adjustment

**If Too Sensitive (too many warnings):**
1. Review notification history in Pushover
2. Note temperature patterns when warnings occur
3. Increase warning threshold by 2-3°F
4. Increase delay time by 5-10 minutes
5. Test for 48 hours and re-evaluate

**If Not Sensitive Enough (missing issues):**
1. Review actual failure events
2. Note temperature/humidity at failure
3. Decrease critical threshold by 2-3°F
4. Decrease delay time by 2-5 minutes
5. Test carefully to avoid false positives

**Editing Thresholds:**
- Edit `/config/packages/fridgefreezer.yaml`
- Modify `above:` values in numeric_state triggers
- Modify `minutes:` values in for: statements
- Restart Home Assistant to apply changes
- Monitor closely for 24-48 hours

---

## YAML Packages

- `/config/packages/fridgefreezer.yaml`

---

## Maintenance

### Regular Checks

**Monthly:**
- Review notification history in Pushover
- Check for pattern of warnings (may indicate developing issue)
- Verify all 4 sensors are online and reporting

**Quarterly:**
- Test one sensor by briefly opening door >15 minutes
- Verify critical alert received
- Confirm recovery notification after closing door

**Annually:**
- Review threshold accuracy based on year of data
- Adjust for appliance aging or replacement
- Check sensor battery levels proactively

### Sensor Replacement

**When Replacing Fridge/Freezer:**
1. Move YoLink sensor to new appliance
2. Monitor temperatures for 24-48 hours
3. Verify normal operating range
4. Adjust thresholds if needed for new appliance

**When Replacing Sensor:**
1. Pair new YoLink sensor to hub
2. Verify entity IDs match (or update YAML)
3. Calibrate by monitoring 48 hours of data
4. Test with door-open simulation

---

## Important Notes

- **Food Safety:** Critical alerts require immediate action
- **Data-Driven:** Thresholds based on actual sensor history, not estimates
- **Hybrid Detection:** Temp + humidity distinguishes failure types
- **No False Positives:** Grocery loading spikes ignored by design
- **Recovery Confirmation:** Always notified when temperature normalizes
- **Four Appliances:** Each monitored independently with tailored thresholds
- **16 Automations:** 4 per appliance (warn, door, compressor, recovery) plus 4 alert state helpers
- **YoLink Platform:** Requires YoLink Hub and active integration
- **Battery Separate:** Battery monitoring handled by batteryalerts.yaml
- **Non-HVAC Garage:** Garage thresholds account for seasonal variations

---

**Last Updated:** February 2026  
**Subsystem:** Environmental Monitoring  
**Document Maintainer:** Homeowner
