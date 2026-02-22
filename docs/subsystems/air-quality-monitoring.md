# Air Quality Monitoring

**Last Updated:** February 2026

---

## Overview

The Air Quality Monitoring system provides comprehensive indoor air quality management using IKEA Alpstuga sensors to monitor CO2 and PM2.5 levels across two floors. The system automatically manages HVAC filtration and exhaust ventilation to maintain healthy indoor air quality while coordinating with existing bathroom and laundry ventilation automations.

**Key Features:**
- PM2.5 filtration takes absolute priority over all other HVAC fan automation
- CO2 management via targeted exhaust ventilation
- Party mode enhanced monitoring with periodic reminders
- Daily air quality summaries
- Sensor offline detection and recovery tracking
- Four-tier Pushover notification system

---

## Hardware

### Air Quality Sensors

**IKEA Alpstuga Multi-Sensors (2):**
- **Location 1:** Main living area (Floor 1)
  - Entity IDs: `sensor.living_area_co2`, `sensor.living_area_pm25`, `sensor.living_area_tvoc`
- **Location 2:** Master bedroom (Floor 2)
  - Entity IDs: `sensor.master_bedroom_co2`, `sensor.master_bedroom_pm25`, `sensor.master_bedroom_tvoc`

**Capabilities:**
- CO2 monitoring (ppm)
- PM2.5 particulate matter (Î¼g/mÂ³)
- TVOC (Total Volatile Organic Compounds)
- Temperature and humidity (if available)
- Battery powered or USB

### Ventilation Equipment

**HVAC System:**
- Ecobee thermostat with fan control
- HEPA 13 filter + activated carbon
- Controlled via `script.hvac_exhaust_boost` (20-minute cycle)
- Fan modes: AUTO (default) or ON (filtration)

**Exhaust Fans:**
- Master bathroom fan (`light.master_bath_fan`)
- Entry bathroom fan (`light.entry_bath_fan`)
- Laundry room fan (`light.laundry_fan_timer`)
- Kitchen hood (`switch.vent_hood_reset`) - can turn OFF only, not ON

---

## Air Quality Thresholds

### PM2.5 (Particulate Matter)

Based on EPA Air Quality Index:

| Level | Range (Î¼g/mÂ³) | Health Category | System Response |
|-------|---------------|-----------------|------------------|
| Good | 0-12 | Safe for all | Normal operation |
| Moderate | 12-35 | Acceptable | Monitor only |
| Unhealthy for Sensitive | 35-55 | At-risk groups affected | HVAC filtration + Notice |
| Unhealthy | 55-150 | Everyone affected | HVAC filtration + Alert |
| Very Unhealthy | 150-250 | Serious health effects | HVAC filtration + Emergency |
| Hazardous | 250+ | Health emergency | HVAC filtration + Emergency |

**System Triggers:**
- **35 Î¼g/mÂ³:** Activates HVAC filtration, sends House Notice
- **55 Î¼g/mÂ³:** Increases alert priority to House Alerts
- **150 Î¼g/mÂ³:** Emergency notification with Priority 2

### CO2 (Carbon Dioxide)

Based on ASHRAE and WHO guidelines:

| Level | Range (ppm) | Air Quality | System Response |
|-------|-------------|-------------|------------------|
| Excellent | <800 | Fresh air | None |
| Good | 800-1000 | Acceptable | None |
| Acceptable | 1000-1500 | Fair | HVAC exhaust boost + reminder |
| Poor | 1500-2000 | Stuffy/drowsy | Stronger ventilation + alert |
| Very Poor | >2000 | Health concerns | Emergency alert |

**System Triggers:**
- **1000 ppm:** HVAC exhaust boost + "open windows" notification
- **1200 ppm (party mode):** Periodic reminders every 15 minutes
- **2000 ppm:** Emergency alert to open windows immediately

---

## System Operation

### PM2.5 Filtration (Highest Priority)

**Activation (35 Î¼g/mÂ³ threshold):**
1. Sets `input_boolean.pm25_filtration_active` to ON
2. Records start time for duration tracking
3. Turns HVAC fan to ON mode (runs continuously)
4. Sends notification based on severity level
5. Increments daily event counter

**Continuation:**
- Runs every 10 minutes to keep HVAC fan ON
- Continues until BOTH sensors read <12 Î¼g/mÂ³ (good air quality)
- Minimum 15-minute runtime before deactivation allowed

**Deactivation:**
- Both sensors must be <12 Î¼g/mÂ³
- Must have run for at least 15 minutes
- Returns HVAC fan to AUTO mode
- Sends operational log notification with duration

**Safety Timeout:**
- 4-hour maximum continuous filtration
- Prevents excessive HVAC runtime if sensor malfunction
- Sends alert if timeout occurs

**Priority Override:**
- PM2.5 filtration takes precedence over bathroom/laundry automations
- Other systems must check `input_boolean.pm25_filtration_active` before returning HVAC fan to AUTO
- Prevents competing HVAC fan control

### CO2 Ventilation

**Living Area (Floor 1):**
- **Trigger:** 1000 ppm for 5 minutes
- **Action:** Runs `script.hvac_exhaust_boost` (20-minute cycle)
- **Notification:** "Open Windows" with outdoor temperature
- **Condition:** Only when PM2.5 filtration is OFF and kitchen hood is ON

**Master Bedroom (Floor 2):**
- **Trigger:** 1000 ppm for 5 minutes during nighttime (9 PM - 7 AM)
- **Action:** Turns on master bathroom fan for 20 minutes
- **Notification:** Bedroom-specific alert
- **Condition:** Only when PM2.5 filtration is OFF

**Critical Alert:**
- **Trigger:** 2000 ppm for 5 minutes
- **Action:** High-priority alert to open windows immediately
- **Priority:** 1 (House Alerts)

### Party Mode Integration

**Activation:**
1. Runs HVAC exhaust boost immediately
2. Turns on entry bathroom fan for additional exhaust
3. Sends activation notification

**Periodic Monitoring:**
- Checks CO2 every 15 minutes during party
- If >1200 ppm, runs another exhaust boost and sends reminder
- Helps manage elevated occupancy air quality

**Deactivation:**
1. Waits 30 minutes after party mode turns off
2. Turns off kitchen hood (if it was left on)
3. Runs final exhaust boost
4. Sends summary with final air quality readings

---

## Notifications

### House Alerts (Priority 1-2)

**Emergency - Priority 2:**
- PM2.5 >150 Î¼g/mÂ³ (Very Unhealthy)
- Title: "âš ï¸ AIR QUALITY EMERGENCY"
- Sound: Siren
- Action required: Close windows, stay indoors

**High Priority - Priority 1:**
- PM2.5 55-150 Î¼g/mÂ³ (Unhealthy)
- CO2 >2000 ppm (Critical)
- Title: "âš ï¸ AIR QUALITY ALERT" or "âš ï¸ CO2 CRITICAL"
- Sound: Loud alert
- Action required: Immediate ventilation

### House Notices (Priority 0)

**Standard Alerts:**
- PM2.5 35-55 Î¼g/mÂ³ (Unhealthy for sensitive groups)
- CO2 1000-2000 ppm (Open windows recommended)
- Party mode periodic checks
- Filtration timeout alerts
- Sensor offline alerts

**Examples:**
- "ðŸ’¨ Air Quality Alert" - Elevated PM2.5
- "ðŸªŸ Open Windows" - High CO2 with outdoor temp
- "ðŸŽ‰ Party Air Quality Check" - Periodic party monitoring

### Operational Log (Priority -2)

**Silent Notifications:**
- Return to normal air quality
- Daily summaries at 7:00 AM
- Sensor recovery
- Party mode deactivation summary

**Daily Summary Includes:**
- Color-coded emoji status for each floor (ðŸŸ¢ good, ðŸŸ¡ elevated, ðŸ”´ poor)
- Max CO2 and PM2.5 readings with trend vs 7-day average
- CO2 and PM2.5 split onto separate lines for readability
- PM2.5 filtration events, total runtime, and trend comparison
- Contextual status message explaining readings

**Daily Summary Format Example:**
```
ðŸ“Š Daily Air Quality

ðŸŸ¢ Floor 1 CO2: 987 ppm max (â†“ 78 ppm vs avg)
           PM2.5: 11 Î¼g/mÂ³ max (â†’ stable)
ðŸŸ¢ Floor 2 CO2: 1,089 ppm max (â†‘ 45 ppm vs avg)
           PM2.5: 9 Î¼g/mÂ³ max (â†“ better)

PM2.5 filtration: 0 events, 0 min runtime
Trend: -2 events vs 7-day avg (better)

âœ“ Excellent air quality all day
```

**Trend Indicators:**
- CO2: â†‘/â†“ shown if difference > 100 ppm, â†’ if stable
- PM2.5: â†‘ worse if +10 Î¼g/mÂ³, â†“ better if -10 Î¼g/mÂ³, â†’ stable otherwise

**Emoji Logic:**
- ðŸŸ¢ Green: CO2 < 1200 ppm AND PM2.5 < 35 Î¼g/mÂ³
- ðŸŸ¡ Yellow: CO2 1200-2000 ppm OR PM2.5 35-55 Î¼g/mÂ³
- ðŸ”´ Red: CO2 > 2000 ppm OR PM2.5 > 55 Î¼g/mÂ³

---

## Integration with Existing Systems

### Bathroom Fan Automation

**Master Bathroom (`masterbathhumidity.yaml`):**
- Checks `input_boolean.pm25_filtration_active` before returning HVAC fan to AUTO
- If PM2.5 filtration is active, skips HVAC fan control
- Allows bathroom fan to run independently during PM2.5 events
- **Implemented:** This check runs through `script.hvac_exhaust_boost` (scripts.yaml), which gates HVAC fan control on PM2.5 filtration state

### Laundry Fan Automation

**Laundry Room (`laundry.yaml`):**
- Checks `input_boolean.pm25_filtration_active` before returning HVAC fan to AUTO
- If PM2.5 filtration is active, skips HVAC fan control
- Laundry exhaust fan operates independently during PM2.5 events
- **Implemented:** This check runs through `script.hvac_exhaust_boost` (scripts.yaml), which gates HVAC fan control on PM2.5 filtration state

### Kitchen Hood

**Limitations:**
- Can turn OFF via automation (`switch.vent_hood_reset`)
- Cannot turn ON via automation (manual control only)

**Integration Logic:**
- CO2 ventilation pauses if kitchen hood is detected OFF (to avoid fighting it)
- Party mode deactivation turns hood OFF after 30-minute delay
- Daily reset automation (5:00 AM) maintains hood operation

---

## Dashboard & Monitoring

### Key Entities

**Sensors:**
- `sensor.living_area_co2` - Living area CO2 (ppm)
- `sensor.living_area_pm25` - Living area PM2.5 (Î¼g/mÂ³)
- `sensor.master_bedroom_co2` - Master bedroom CO2 (ppm)
- `sensor.master_bedroom_pm25` - Master bedroom PM2.5 (Î¼g/mÂ³)

**Status Indicators:**
- `input_boolean.pm25_filtration_active` - PM2.5 filtration running
- `input_boolean.party_mode` - Party mode active

**Daily Statistics:**
- `input_number.living_area_co2_max_today` - Daily max CO2 (Floor 1)
- `input_number.living_area_pm25_max_today` - Daily max PM2.5 (Floor 1)
- `input_number.master_bedroom_co2_max_today` - Daily max CO2 (Floor 2)
- `input_number.master_bedroom_pm25_max_today` - Daily max PM2.5 (Floor 2)
- `input_number.pm25_filter_events_today` - Filtration event count
- `input_number.pm25_filtration_runtime_today` - Total filtration runtime (minutes)

**7-Day Trend Tracking:**
- `input_number.pm25_filter_events_7day_avg` - Filtration events 7-day average
- `input_number.living_area_co2_7day_avg` - Floor 1 CO2 7-day average
- `input_number.living_area_pm25_7day_avg` - Floor 1 PM2.5 7-day average
- `input_number.master_bedroom_co2_7day_avg` - Floor 2 CO2 7-day average
- `input_number.master_bedroom_pm25_7day_avg` - Floor 2 PM2.5 7-day average

### Suggested Dashboard Cards

**Current Air Quality:**
```yaml
type: entities
title: Air Quality Status
entities:
  - sensor.living_area_co2
  - sensor.living_area_pm25
  - sensor.master_bedroom_co2
  - sensor.master_bedroom_pm25
  - input_boolean.pm25_filtration_active
```

**Daily Statistics:**
```yaml
type: entities
title: Air Quality Today
entities:
  - input_number.living_area_co2_max_today
  - input_number.living_area_pm25_max_today
  - input_number.master_bedroom_co2_max_today
  - input_number.master_bedroom_pm25_max_today
  - type: divider
  - input_number.pm25_filter_events_today
  - input_number.pm25_filtration_runtime_today
  - type: divider
  - input_number.pm25_filter_events_7day_avg
  - input_number.living_area_co2_7day_avg
  - input_number.living_area_pm25_7day_avg
  - input_number.master_bedroom_co2_7day_avg
  - input_number.master_bedroom_pm25_7day_avg
```

---

## Troubleshooting

### PM2.5 Filtration Keeps Running

**Symptom:** HVAC fan won't return to AUTO mode

**Check:**
1. Current PM2.5 readings - are both sensors <12 Î¼g/mÂ³?
2. Filtration duration - has it been running for 15+ minutes?
3. Sensor status - are sensors online and reporting?

**Solutions:**
- Wait for air to clear naturally (may take hours if source is present)
- Check for air quality source (cooking, fireplace, outdoor smoke)
- If stuck after 4 hours, system auto-deactivates with alert
- Manually toggle `input_boolean.pm25_filtration_active` if sensors stuck

### CO2 Alerts Too Frequent

**Symptom:** Constant "open windows" notifications

**Check:**
1. Occupancy - high occupancy naturally elevates CO2
2. Ventilation - are windows closed during occupied periods?
3. Party mode - is it enabled when not needed?

**Solutions:**
- Open windows periodically during high occupancy
- Use party mode during gatherings for enhanced monitoring
- Adjust threshold in automation if desired (currently 1000 ppm)
- Consider HVAC fresh air intake if available

### Sensor Offline

**Symptom:** Sensor shows "unavailable"

**Check:**
1. Sensor battery level (if battery powered)
2. USB power connection (if USB powered)
3. Thread network connectivity
4. Distance from coordinator/hub

**Solutions:**
- Replace batteries or reconnect USB power
- Move sensor closer to hub/coordinator
- Check network connectivity
- Restart sensor or integration if needed

### Bathroom/Laundry Not Controlling HVAC Fan

**Symptom:** HVAC fan not returning to AUTO after bathroom/laundry cycle

**Check:**
1. Is `input_boolean.pm25_filtration_active` currently ON?
2. Review automation traces for bathroom/laundry automations

**Solutions:**
- This is expected behavior - PM2.5 filtration takes priority
- HVAC fan will return to AUTO when air quality improves
- Bathroom/laundry exhaust fans still operate independently

---

## Maintenance

### Weekly
- Review air quality trends in daily summaries
- Check sensor online status

### Monthly
- Test sensors by creating temporary air quality event (cooking, etc.)
- Verify notifications are received
- Review PM2.5 filtration event frequency

### Quarterly
- Clean IKEA Alpstuga sensor housings
- Verify sensor accuracy against reference device (if available)
- Review and adjust thresholds based on seasonal patterns

### Annually
- Replace HVAC HEPA 13 and carbon filters
- Verify sensor battery levels (if battery powered)
- Review automation effectiveness and adjust as needed

---

## Sensor Specifications

**IKEA Alpstuga:**
- CO2 Range: 400-5000 ppm
- PM2.5 Range: 0-500 Î¼g/mÂ³
- Update Frequency: ~60 seconds
- Accuracy: Â±50 ppm (CO2), Â±10% (PM2.5)
- Power: Battery or USB
- Communication: Thread (requires Thread border router, e.g., HomePod mini or Apple TV 4K)

---

## Related Documentation

- **[Pushover Integration](pushover-integration.md)** - Notification system details
- **[Master Bath Humidity](masterbathhumidity.yaml)** - Bathroom fan coordination
- **[Laundry](laundry.yaml)** - Laundry fan coordination
- **[HomeKit Device Integration](homekit-device-integration.md)** - HVAC fan control
- **[Scripts](scripts.yaml)** - HVAC exhaust boost script

---

## Migration Notes for New Owner

**Understanding the System:**
1. **IKEA Alpstuga Sensors** - Use Thread protocol; require a Thread border router (e.g., HomePod mini or Apple TV 4K)
2. **PM2.5 Priority** - Always takes precedence over other HVAC fan automation
3. **Pushover Notifications** - Four-tier system provides appropriate alert levels
4. **Party Mode** - Manual toggle for enhanced monitoring during gatherings

**If Replacing Sensors:**
1. Any air quality sensor can be used with entity ID updates in automation
2. Update entity IDs in `airquality.yaml`
3. Ensure new sensors provide CO2 (ppm) and PM2.5 (Î¼g/mÂ³) readings
4. Test thresholds with new sensor accuracy specifications

**If Replacing Home Assistant:**
1. Restore `airquality.yaml` package
2. Restore helper entities (input_boolean, input_number)
3. Verify HVAC fan script still works
4. Verify Pushover notification services configured
5. Test PM2.5 and CO2 automations

**Alternative Approaches:**
- Could use separate air purifiers instead of HVAC filtration
- Could add HRV/ERV for automatic fresh air exchange
- Could integrate with Ecobee ventilator (requires HVAC modification)
- Could add outdoor air quality monitoring for wildfire smoke detection

---

**Last Updated:** February 2026  
**Package File:** `airquality.yaml`  
**Document Maintainer:** Homeowner