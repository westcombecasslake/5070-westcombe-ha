# Safety & Monitoring Systems

**Document Version:** 1.0  
**Last Updated:** February 2026

---

## Overview

The safety and monitoring subsystems provide 24/7 vigilance over life-safety systems, water intrusion, and critical home infrastructure. These systems prioritize reliability through redundant sensing, intelligent alerting, and comprehensive status tracking.

---

## Smoke & Carbon Monoxide Detection

**Package:** `smokeco.yaml`  
**Devices:** 8 YoLink sensors (installed February 2024)  
**Priority:** CRITICAL

### Hardware Configuration

**Complete device information available in Google Sheet:**

**[House IT Inventory & Documentation](https://docs.google.com/spreadsheets/d/1w4-gFy8rL0OqKrlzC7T9p2vFs5JBJYV2dyVfrrOA8DI)**  
Google Account: westcombecasslake@gmail.com (transfers with house)

**8 YoLink Smoke/CO Detectors (Installed February 2024):**

**Combo Units (Smoke + CO):**
- Basement - YS7A01-UC (SN:FBC38CFBE9)
- Living Area - YS7A01-UC (SN:87806D927D)
- Master Bedroom - YS7A01-UC (SN:F88A4B3835)

**Smoke Only:**
- Dan's Bedroom - YS7A02-UC (SN:CD87276F48)
- Den - YS7A02-UC (SN:F076C89F79)
- Garage - YS7A02-UC (SN:A03FCE857A)
- Office - YS7A02-UC (SN:F462D4B547)
- Sarah's Bedroom - YS7A02-UC (SN:E958B19405)

**Additional:**
- **YoLink Siren** (YS7103-UC, SN:281B4FEB47) in middle bathroom - whole house alert

### Coverage Strategy

**3 Smoke/CO Combo Units:**
- Basement (furnace/utilities area)
- Living Area (central location)
- Master Bedroom (sleeping area)

**5 Smoke-Only Units:**
- All other bedrooms
- Den/office spaces
- Garage

**Rationale:** CO detection concentrated near fuel-burning appliances and primary sleeping areas. All sleeping areas have smoke detection per code requirements.

### Monitoring Features

#### Real-Time Detection
```yaml
Trigger: binary_sensor.*_smoke → "on"
Alert: House Alerts (HIGH PRIORITY)
Action: Immediate notification with location
```

```yaml
Trigger: binary_sensor.*_carbon_monoxide → "on"
Alert: House Alerts (HIGH PRIORITY)  
Action: EVACUATE message with location
```

#### Offline Detection
```yaml
Trigger: Any sensor → "unavailable" for 10 minutes
Alert: House Notices (NORMAL PRIORITY)
Action: Notification to check detector and battery
```

**Recovery:** Automatic status update when sensor returns online after 2 minutes.

#### Status Tracking

Two helper entities provide system-wide status:

**`input_text.smoke_co_online_status`**
- ✅ Normal - All sensors online
- ⚠️ Offline - One or more sensors unavailable

**`input_text.smoke_co_alert_status`**
- ✅ Normal - No detections
- 🔥 Smoke Detected - Active smoke alarm
- ☠️ CO Detected - Active CO alarm

**Use Cases:**
- Dashboard quick-glance status
- Android Auto display integration
- Condition checks in other automations

### Notification Examples

**Smoke Detection:**
```
Title: Basement Smoke CO
Message: SMOKE DETECTED
         Check location immediately
```

**CO Detection:**
```
Title: Living Area Smoke CO  
Message: CARBON MONOXIDE DETECTED
         EVACUATE and check immediately
```

**Offline Notification:**
```
Title: Garage Smoke Sensor
Message: Status: Offline (10+ min)
         Check detector and battery
```

### Maintenance

**Monthly:** 
- Test all detectors manually
- Verify alert notifications received

**Annually:**
- Replace batteries in all units
- Clean sensor chambers per manufacturer specs

**Replacement Schedule:**
- YoLink sensors: 10-year lifespan
- Installed: February 2024
- Replace: February 2034

---

## Sump Pump Monitoring

**Package:** `sumppump.yaml`  
**Priority:** HIGH

### System Overview

Comprehensive monitoring with adaptive failure detection based on rainfall correlation and historical pump patterns. Tracks daily runtime, cycle counts, gallons pumped, and monthly totals.

### Hardware Configuration

**Primary Pump Monitor:**
- **Functional Devices RIB relay** (21AJ46) - Monitors pump hot lead
- **YoLink door sensor** wired to relay - Binary state reporting
- Entity: `binary_sensor.sump_pump_running`

**Depth Monitoring:**
- **Letzgo SB2 bobber switch** - Mechanical float
- **Insteon 2450 I/O module** (70.7C.5F) - Interface to HA
- Entity: `binary_sensor.sump_pump_depth_alert_door`
- States: `on` = Normal depth | `off` = HIGH WATER ALERT

**Floor Moisture:**
- **YoLink leak sensor** - Floor moisture detection
- Entity: `binary_sensor.sump_leak_sensor_moisture`
- States: `off` = Dry | `on` = WET ALERT

### Calculated Metrics

**Daily Statistics:**
- **Daily Cycles** - Count of pump activations (history_stats)
- **Daily Gallons** - Cycles × 1.7 gal/cycle (based on sump geometry)

**Weekly Average:**
- **7-Day Average Cycles** - Rolling average for comparison

**Monthly Tracking:**
- **Monthly Gallons** - Cumulative total, resets on 1st of month

### Automation Logic

#### Daily Summary (11:55 PM)
```yaml
Notification: Operational Log
Content:
  - Cycle count
  - Gallons pumped today
  - Month-to-date gallons (calculated inline)
  - Current depth status
  - Floor moisture status
```

**Timing:** Runs at 11:55 PM to capture complete day before midnight reset.

#### Failure Detection - Rain Event

```yaml
Trigger: Template evaluation every 5 minutes
Condition: 
  - Hours since last cycle > expected max idle
  - Rainfall in last 24 hrs > 0.1"
  - Rain is still occurring
Alert: House Alerts (HIGH PRIORITY)
```

**Expected Max Idle Logic:**
```
Rain > 0.5" → Max idle: 12 hours
Rain > 0.1" → Max idle: 48 hours  
Rain < 0.1" → Max idle: 72 hours
```

**Notification:**
```
Title: ⚠️ Sump Pump Alert
Message: No pump activity in [X] hours
         Despite [X.XX]" rainfall. Check pump!
         
         Sump Water Depth: Normal/HIGH!
         Floor: Dry/WET!
```

#### Activity Drop Detection

```yaml
Trigger: Daily at 6:00 PM
Condition:
  - Today's cycles < 50% of 7-day average
  - 7-day average > 5 cycles
  - Rainfall > 0.5" in last 24 hours
Alert: House Alerts (HIGH PRIORITY)
```

**Notification:**
```
Title: ⚠️ Sump Pump Alert
Message: Only [X] cycles today
         (vs 7-day avg of [X.X] cycles)
         Rained [X.XX]" in last 24hrs. Check pump!
```

**Purpose:** Detects partial pump failure where pump still runs but less frequently than expected given rainfall.

#### Extended Dry Period Notice

```yaml
Trigger: No pump activity for 96+ hours
Condition: No recent rain (< 0.1" in 24 hrs)
Alert: House Notices (NORMAL PRIORITY)
```

**Notification:**
```
Title: ℹ️ Sump Pump Notice
Message: No pump activity in [X] hours
         No recent rain - likely normal
```

**Purpose:** Informational awareness during dry spells. Not an alert since no pump activity is expected.

#### Pump Resumed Notice

```yaml
Trigger: Pump cycles after 24+ hour idle
Alert: Operational Log
```

**Purpose:** Confirms pump operation resumed after extended idle period.

### Configuration Parameters

**User-Adjustable via Settings → Helpers:**

**`input_number.sump_pump_gallons_per_cycle`**
- Default: 1.7 gallons
- Based on: Sump pit geometry (18" diameter × 8" drawdown)
- Adjust if: Pit dimensions differ

**`input_number.sump_pump_monthly_gallons`**
- Automatically maintained
- Resets: First of each month at midnight
- Manual reset: Via Settings → Helpers if needed

### Monitoring Best Practices

**Normal Operation Indicators:**
- Cycles vary with rainfall
- More cycles during/after rain
- Fewer cycles during dry periods
- Consistent cycle duration (Â±1-2 seconds)

**Warning Signs:**
- Continuous cycling (>10 cycles/hour sustained)
- No cycles during heavy rain
- Drastically reduced cycle count vs. historical
- Depth bobber remains in HIGH position
- Floor moisture detected

**Monthly Review:**
- Compare monthly gallons to historical data
- Seasonal variation is normal (spring > summer)
- Sudden doubling/halving may indicate issues

---

## Water Leak Detection

**Package:** `basementleaksensors.yaml`  
**Priority:** HIGH

### Hardware Configuration

**Leak Sensors (YoLink):**
- **Sump Area** - `binary_sensor.sump_leak_sensor_moisture`
- **Hot Water Heater** - `binary_sensor.hot_water_heater_leak_sensor_moisture`
- **Well Room** - `binary_sensor.well_leak_sensor_moisture`

**Siren:**
- YoLink Siren (YS7103-UC) in middle bathroom
- Triggered natively by YoLink alarm group when any leak sensor fires
- No HA automation required for siren activation
- Siren is silenced manually via YoLink app

**Note:** All Insteon leak sensors (2852-222) and Insteon siren (2868-222) have been removed. System is YoLink only.

### Automation Behavior

#### Leak Detection

Trigger: Any leak sensor turns "on"
Actions:
1. Update `input_text.leak_sensor_status` to "Leak Detected" (Android Auto display)
2. Send EMERGENCY alert to House Alerts

**Notification:**
```
Title: EMERGENCY - Water Leak Detected
Message: Location: [Hot Water Heater / Sump Pump Area / Well Area]
         Status: MOISTURE DETECTED
         Action: CHECK IMMEDIATELY
```

#### Leak Cleared

Trigger: Sensor returns to "off" for 2 minutes
Action: Script checks all 3 sensors - resets status helper to Normal only if all clear.
No Pushover notification on clear - status visible via helper and daily summary.

### Sensor Placement Strategy

**Sump Area:**
- Near sump pump behind furnace
- Detects: Sump overflow, pump failure backup

**Hot Water Heater:**
- Utility room by water heater
- Detects: Tank leaks, relief valve discharge, pipe failures

**Well Room:**
- Near well pressure tank
- Detects: Tank leaks, pipe failures, check valve issues

**Coverage:** All major water sources in basement with potential for significant leaks.

### Integration with Sump Monitoring

Sump leak sensor also contributes to sump pump monitoring:
- Included in daily summary notifications
- Provides redundant floor-level detection
- Confirms water reached floor level (critical failure indicator)

### Testing & Maintenance

**Monthly:**
- Test each sensor with small amount of water
- Verify siren activates (YoLink alarm group)
- Confirm Pushover notification received

**Annually:**
- Replace sensor batteries
- Clean sensor probes
- Verify sensor sensitivity

**Sensor Battery Monitoring:**
- See `batteryalerts.yaml` for low battery notifications
- YoLink devices report battery percentage to HA

## Battery Status Monitoring

**Package:** `batteryalerts.yaml`  
**Priority:** MEDIUM

### Monitored Devices

All battery-powered sensors report their battery status to Home Assistant:

**YoLink Devices:**
- 8 smoke/CO detectors
- 6 temp/humidity sensors
- 3 leak sensors (sump, hot water heater, well room)
- Outdoor sensors (sump pump activity and depth)

**Note:** Insteon leak sensors (2852-222) have been removed. All leak sensing is YoLink only.

### Alert Strategy

```yaml
Trigger: battery_level attribute < 15%
Alert: House Notices (NORMAL PRIORITY)
Frequency: Daily until replaced
```

**Notification:**
```
Title: 🔋 Low Battery Alert
Message: Device: [Device Name]
         Location: [Physical Location]
         Battery Level: [X]%
         
         Replace battery soon
```

### Battery Replacement Schedule

**Proactive Approach:**
- Replace at 15% (not at failure)
- Keep spare batteries on hand
- Group replacements by device type

**Battery Types Used:**
- See device-inventory.md or YoLink app for battery type per device

**Estimated Life:**
- Insteon sensors: 2-3 years
- YoLink smoke/CO: 5+ years
- YoLink temp/humidity: 2-3 years

### Dashboard Integration

Battery status entities can be added to dashboards for at-a-glance monitoring:
- Filter entities with `battery_level` attribute
- Sort by percentage (lowest first)
- Color-code: Green >50%, Yellow 25-50%, Red <25%

---

## System Health Monitoring

**Package:** `systemhealth.yaml`  
**Priority:** MEDIUM

### Monitored Parameters

**Home Assistant Core:**
- System uptime
- Restart events
- Integration status
- Database size

**Hub Status:**
- YoLink Hub connectivity
- Ecobee API status

**Network:**
- Local network connectivity
- Internet connectivity
- DNS resolution

### Startup Monitoring

```yaml
Entity: binary_sensor.ha_startup_pending
States: 
  - "on" = System starting up
  - "off" = Fully operational
```

**Startup Sequence:**
```
1. Home Assistant starts
2. Core integrations load
3. Packages initialize
4. Startup automation runs
5. Status set to operational
6. Notification sent
```

### Alert Conditions

**Integration Failures:**
```yaml
Trigger: Integration becomes unavailable
Alert: House Notices (NORMAL PRIORITY)
Action: Notify which integration failed
```

**Unusual Restarts:**
```yaml
Trigger: HA restart without user initiation
Alert: House Notices (NORMAL PRIORITY)
Action: Log restart reason if available
```

**Database Growth:**
```yaml
Trigger: Database size exceeds threshold
Alert: House Notices (NORMAL PRIORITY)
Action: Notify to review recorder configuration
```

### Recorder Configuration

Optimized for performance and relevant history:

**Included Domains:**
- All safety sensors (smoke, CO, leak)
- All monitored switches and sensors
- Automation states for debugging

**Excluded:**
- High-frequency updates (< 1 second)
- Derived sensors (calculated from other sensors)
- Temporary/working variables

**Retention:**
- Default: 10 days
- Critical events: 90 days
- Consider external long-term logging for analytics

---

## Operational Procedures

### Daily Monitoring

**Morning Routine:**
1. Check notification summary (Operational Log)
2. Review any alerts from overnight
3. Verify system status helpers (smoke/CO status)

**Evening Routine:**
1. Review sump pump daily summary (11:55 PM)
2. Check any anomalies or warnings
3. Verify premise lighting activated at dusk

### Weekly Monitoring

**Review:**
- Sump pump 7-day cycle trends
- Battery levels for all devices
- Any recurring warnings/notices
- System uptime and restarts

**Actions:**
- Address any low battery alerts
- Investigate repeated sensor offline events
- Review automation traces for failed automations

### Monthly Maintenance

**Required:**
- Test all smoke/CO detectors manually
- Test all leak sensors with water
- Review sump pump monthly gallon totals
- Compare to historical monthly averages

**Optional:**
- Review automation effectiveness
- Adjust seasonal parameters (lighting delays, HVAC thresholds)
- Update documentation for any changes

### Seasonal Maintenance

**Spring (March-April):**
- Enable irrigation system
- Adjust premise lighting delays
- Review HVAC humidifier settings
- Prepare for increased sump pump activity

**Summer (May-June):**
- Monitor irrigation system performance
- Verify basement dehumidifier peak-hour avoidance
- Increase premise lighting delays (longer dusk)

**Fall (September-October):**
- Adjust premise lighting delays (shorter dusk)
- Prepare for heating season
- Review HVAC humidifier threshold

**Winter (November-February):**
- Disable irrigation system
- Monitor for freezing temperatures (garage alerts)
- Humidifier typically most active
- Photocell maintenance reminder (Feb 1)

---

## Troubleshooting

### Sump Pump False Alerts

**Symptom:** Alert for no pump activity but no recent rain

**Check:**
1. Weather integration data - verify rainfall reading
2. Pump sensor state - ensure detecting cycles
3. Historical data - compare to normal patterns

**Solution:**
- If weather data incorrect: Verify Met.no integration
- If sensor not detecting: Check RIB relay and wiring
- If pattern changed: May be normal seasonal variation

### Smoke/CO Detector Offline

**Symptom:** Detector shows unavailable

**Check:**
1. Device battery level
2. YoLink Hub status
3. Physical detector status LED
4. Distance from YoLink Hub

**Solution:**
- Replace battery if low
- Power cycle YoLink Hub
- Relocate detector if signal strength poor
- Check for physical obstruction/interference

### Leak Sensor False Alarms

**Symptom:** Leak sensor triggers but no water present

**Check:**
1. Sensor probe condition (corrosion, dirt)
2. Sensor placement (puddle from humidity?)
3. Battery level

**Solution:**
- Clean sensor probes with isopropyl alcohol
- Relocate if condensation area
- Replace battery
- Test sensor with small water amount

### Premise Lights Not Activating

**Symptom:** Photocell triggers but lights don't turn on

**Check:**
1. Photocell automation trace
2. Time window condition (3:45-7:30 PM)
3. Insteon scene 21 functionality
4. Individual device states

**Solution:**
- Verify photocell daily reset ran (1:00 PM)
- Test scene 21 manually
- Check if backup automation ran (15 min after sunset)
- Verify individual switches respond to commands

---

## Related Documentation

- **[Main README](README.md)** - System overview
- **[Device Inventory](device-inventory.md)** - Hardware details

---

**Document Maintained By:** Homeowner  
**Review Schedule:** Quarterly or after safety system changes
