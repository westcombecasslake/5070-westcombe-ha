# Irrigation

**Last Updated:** February 2026

---

## Overview

The Irrigation subsystem controls back patio drip irrigation with weather-aware scheduling that checks current weather state before running. The system includes explicit enable/disable control, temperature-based runtime adjustment, manual override capability, and deadman safety protection to prevent uncontrolled operation.

All irrigation control requires the system enable latch to be ON, providing a clear safety mechanism to completely disable the system when it's been winterized for the year or maintenance. Runtime is automatically adjusted based on outdoor temperature to match plant water needs without manual schedule changes.

---

## Weather-Aware Scheduling

### What It Does

Automatically calculates daily irrigation runtime based on outdoor temperature, then runs the irrigation cycle at 7:45 AM if weather conditions are appropriate. Skips irrigation when the weather state indicates active precipitation (rainy, pouring, lightning-rainy, or snowy-rainy), or when the Met.no hourly forecast shows maximum precipitation probability (PoP) over the next 24 hours at or above the configured threshold.

**Rationale:** Temperature-based runtime adjustment matches water needs without manual intervention (25 minutes below 80°F, 45 minutes at 80°F+). Weather checks prevent unnecessary watering after rain, saving water and preventing overwatering. Morning timing (7:45 AM) minimizes evaporation while completing before heat of day.

### Hardware

**Insteon I/O Outlet:**
- Model: Insteon i3 outlet
- Socket: Bottom socket
- Entity: `switch.patio_drip_sprinkler`
- Controls: 24VAC irrigation valve
- Location: Back patio drip irrigation zone

**Weather Data:**
- Entity: `weather.forecast_home_2`
- Provides: Temperature and precipitation probability
- Source: Met.no weather integration

### Key Entities

**Sensors:**
- `weather.forecast_home_2` - Weather forecast data

**Switches:**
- `switch.patio_drip_sprinkler` - Irrigation valve control

**User Controls:**
- `input_boolean.irrigation_system_enabled` - Master enable/disable latch
- `input_number.irrigation_zone_runtime` - Calculated runtime (0-60 min)
- `input_number.irrigation_pop_threshold` - Precipitation probability threshold (0-100%)

### Automations

**Calculate Zone Runtime:**
- ID: `irrigation_calculate_runtime`
- `Irrigation - Calculate Zone Runtime`
- Trigger: 6:00 AM daily
- Condition: System enabled
- Logic: Sets runtime to 25 min if temp <80°F, 45 min if ≥80°F
- Action: Updates `input_number.irrigation_zone_runtime`

**Automatic Morning Cycle:**
- ID: `irrigation_automatic_morning_cycle`
- `Irrigation - Automatic Morning Cycle`
- Trigger: 7:45 AM daily
- Condition: System enabled AND weather state not rainy/pouring/lightning-rainy/snowy-rainy AND forecast max PoP < irrigation_pop_threshold
- Action: Runs irrigation for calculated runtime

**YAML:** irrigation.yaml

---

## Manual Override & Safety

### What It Does

Provides manual 30-minute irrigation run button for spot watering needs and enforces deadman safety timer to automatically shut off irrigation after maximum runtime to prevent flooding from system failures.

**Rationale:** Manual button useful for testing, spot watering, or running outside normal schedule. Deadman timer is critical safety feature - prevents uncontrolled operation if automation fails, valve sticks, or system error occurs. Maximum runtime prevents water waste and potential property damage.

### Hardware

**Insteon I/O Outlet:**
- Entity: `switch.patio_drip_sprinkler`
- Same hardware as Weather-Aware Scheduling

### Key Entities

**User Controls:**
- `input_button.irrigation_manual_30min` - Triggers 30-minute manual run
- Maximum runtime enforced by deadman automation (check YAML for limit)

### Automations

**Manual 30 Minute Run:**
- ID: `irrigation_manual_30min_run`
- `Irrigation - Manual 30 Minute Run`
- Trigger: `input_button.irrigation_manual_30min` pressed
- Condition: System enabled
- Action: Runs irrigation for 30 minutes, activates deadman timer
- Useful for testing and spot watering

**Deadman Safety Off:**
- ID: `irrigation_deadman_safety_off`
- `Irrigation - Deadman Safety Off`
- Trigger: Valve on for 60 minutes continuously
- Action: Forces irrigation OFF, sends house alert
- Safety: Prevents flooding from stuck valves or system failures
- Cannot be disabled - always active when system enabled

**YAML:** irrigation.yaml

---

## User Controls

**Access:** Settings → Helpers

**System Enable:**
- `input_boolean.irrigation_system_enabled` - Master ON/OFF control
  - OFF: Completely disables all irrigation (automatic and manual)
  - ON: Enables weather-aware scheduling and manual control
  - Use during travel or maintenance

**Runtime Settings:**
- `input_number.irrigation_zone_runtime` - Auto-calculated runtime (0-60 min)
  - Set by temperature automation, can be manually adjusted
- `input_number.irrigation_pop_threshold` - Precipitation probability threshold (0-100%)
  - Skip fires if Met.no max hourly PoP over next 24h meets or exceeds this value
  - Default: 40% (per NOAA/extension service guidance)

**Manual Control:**
- `input_button.irrigation_manual_30min` - Triggers 30-minute manual run
  - Useful for testing or spot watering
  - Only works when system enabled
  - Deadman safety forces shutoff after 60 minutes regardless

---

## Notifications

**Pushover Operational Log** (`notify.operational_log`):
- Irrigation automatic cycle runs
- Includes runtime, temperature, and weather decision inputs
- Priority: Low (informational)

**Pushover House Notices** (`notify.house_notices`):
- Irrigation manual runs
- Priority: Normal (awareness)

**Pushover House Alerts** (`notify.house_alerts`):
- Irrigation deadman safety shutoff triggered
- Priority: High (indicates possible system malfunction)

---

## Related Systems

**Met.no Weather Integration:**
- Provides temperature and precipitation state data
- Entity: `weather.forecast_home_2`
- Built-in HA integration, no separate documentation

**Insteon Controls:**
- I/O module controls 24VAC irrigation valve
- See: insteon-integration.md

---

## Troubleshooting

**Irrigation not running on schedule:**
- Verify `input_boolean.irrigation_system_enabled` is ON
- Check recent weather state — skips on active precipitation states (rainy/pouring/lightning-rainy/snowy-rainy) OR if Met.no forecast max PoP over next 24h >= irrigation_pop_threshold
- Review `input_number.irrigation_zone_runtime` - should be 25 or 45 min
- Check `weather.forecast_home_2` is updating correctly

**Runtime always 25 minutes even in hot weather:**
- Verify outdoor temperature reading from `weather.forecast_home_2`
- Check if temperature attribute is available (80°F threshold)
- Manual adjustment of `input_number.irrigation_zone_runtime` possible

**Manual button not working:**
- Verify `input_boolean.irrigation_system_enabled` is ON
- Check `switch.patio_drip_sprinkler` responds to manual commands
- Review automation trace for manual run automation

**System won't turn off:**
- Deadman timer should force off after maximum runtime
- If valve physically stuck, check valve wiring and solenoid
- Can manually turn off via `switch.patio_drip_sprinkler`
- Disable system via `input_boolean.irrigation_system_enabled`

**Irrigation skipping too many days:**
- Skip triggers on active precipitation states OR forecast PoP >= threshold — check both
- Raise `input_number.irrigation_pop_threshold` if PoP skips are too aggressive
- Check `weather.forecast_home_2` state when skip occurs
- Review weather forecast accuracy for location

---

## YAML Packages

- `/config/packages/irrigation.yaml`

---
