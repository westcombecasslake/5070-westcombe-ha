# Irrigation

**Last Updated:** May 2026

---

## Overview

The Irrigation subsystem controls back patio drip irrigation with weather-aware scheduling that checks current weather state and forecast conditions before running. The system includes explicit enable/disable control, ET-based runtime calculation using temperature and humidity, manual override capability, and deadman safety protection to prevent uncontrolled operation.

All irrigation control requires the system enable latch to be ON, providing a clear safety mechanism to completely disable the system when it has been winterized or is undergoing maintenance. Runtime is automatically calculated each morning using forecasted high temperature and humidity to match plant water needs without manual schedule changes.

---

## Weather-Aware Scheduling

### What It Does

Calculates daily irrigation runtime at 6 AM based on forecasted high temperature and humidity, then runs the irrigation cycle at 7:45 AM if weather conditions are appropriate. Skips irrigation when the current weather state indicates active precipitation (rainy, pouring, lightning-rainy, or snowy-rainy), or when today's or tomorrow's daily forecast condition is in the same bad weather list.

**Rationale:** ET-based runtime uses both temperature and humidity to match actual plant water demand — hot dry days water more, cool humid days water less. This is tuned for a mixed drip system (container plants and an established birch tree), with a maximum of 40 minutes on the hottest driest days. Weather checks prevent unnecessary watering when rain is current or forecast. Morning timing (7:45 AM) minimizes evaporation while completing before the heat of day.

### ET Runtime Formula

Runtime is calculated from today's forecasted high temperature and humidity:

- `temp_factor = ((temp - 40) * 0.6) + 8`
- `humidity_modifier = 1.4 - (humidity / 100)`
- `runtime = temp_factor * humidity_modifier`, clamped between 10 and 40 minutes

Example outputs: 55°F / 60% RH ≈ 14 min | 70°F / 50% RH ≈ 23 min | 90°F / 30% RH ≈ 40 min

### Hardware

**Insteon I/O Outlet:**
- Model: Insteon i3 outlet
- Socket: Bottom socket
- Entity: `switch.patio_drip_sprinkler`
- Controls: 24VAC irrigation valve
- Location: Back patio drip irrigation zone

**Weather Data:**
- Entity: `weather.forecast_home_2`
- Provides: Forecasted high temperature, humidity, and daily condition
- Source: Met.no weather integration
- Note: Met.no does not provide precipitation probability (PoP) data

### Key Entities

**Sensors:**
- `weather.forecast_home_2` - Weather forecast data

**Switches:**
- `switch.patio_drip_sprinkler` - Irrigation valve control

**User Controls:**
- `input_boolean.irrigation_system_enabled` - Master enable/disable latch
- `input_number.irrigation_zone_runtime` - Calculated runtime (0-40 min)

### Automations

**Calculate Zone Runtime:**
- ID: `irrigation_calculate_runtime`
- `Irrigation - Calculate Zone Runtime`
- Trigger: 6:00 AM daily
- Condition: System enabled
- Logic: Fetches daily forecast, calculates runtime from forecasted high temp and humidity using ET formula, clamps result between 10 and 40 minutes
- Action: Updates `input_number.irrigation_zone_runtime`

**Automatic Morning Cycle:**
- ID: `irrigation_automatic_morning_cycle`
- `Irrigation - Automatic Morning Cycle`
- Trigger: 7:45 AM daily
- Condition: System enabled
- Logic: Fetches daily forecast; notifies and skips if current weather or today's/tomorrow's forecast condition is bad; otherwise runs irrigation for calculated runtime
- Action: Runs irrigation for calculated runtime, sends operational log notification

**YAML:** irrigation.yaml

---

## Manual Override & Safety

### What It Does

Provides a manual 30-minute irrigation run button for spot watering needs and enforces a deadman safety timer to automatically shut off irrigation after 45 minutes to prevent flooding from system failures.

**Rationale:** Manual button is useful for testing, spot watering, or running outside the normal schedule. Deadman timer is a critical safety feature — the maximum legitimate runtime is 40 minutes (ET cap) or 30 minutes (manual run). The deadman is set 5 minutes above that ceiling and should never fire during normal operation. It exists solely to catch stuck valves or automation failures.

### Hardware

**Insteon I/O Outlet:**
- Entity: `switch.patio_drip_sprinkler`
- Same hardware as Weather-Aware Scheduling

### Key Entities

**User Controls:**
- `input_button.irrigation_manual_30min` - Triggers 30-minute manual run
- Maximum runtime enforced by deadman automation at 45 minutes

### Automations

**Manual 30 Minute Run:**
- ID: `irrigation_manual_30min_run`
- `Irrigation - Manual 30 Minute Run`
- Trigger: `input_button.irrigation_manual_30min` pressed
- Condition: System enabled
- Action: Runs irrigation for 30 minutes
- Useful for testing and spot watering

**Deadman Safety Off:**
- ID: `irrigation_deadman_safety_off`
- `Irrigation - Deadman Safety Off`
- Trigger: Valve on for 45 minutes continuously
- Action: Forces irrigation OFF, sends house alert
- Safety: Should never fire during normal operation — only on malfunction or stuck valve
- Always active whenever the valve is on

**YAML:** irrigation.yaml

---

## User Controls

**Access:** Settings → Helpers

**System Enable:**
- `input_boolean.irrigation_system_enabled` - Master ON/OFF control
  - OFF: Completely disables all irrigation (automatic and manual), silent with no notifications
  - ON: Enables weather-aware scheduling and manual control
  - Use during winterization or maintenance

**Runtime Settings:**
- `input_number.irrigation_zone_runtime` - Auto-calculated runtime (0-40 min)
  - Set by 6 AM ET calculation, can be manually adjusted if needed

**Manual Control:**
- `input_button.irrigation_manual_30min` - Triggers 30-minute manual run
  - Useful for testing or spot watering
  - Only works when system enabled
  - Deadman safety forces shutoff after 45 minutes regardless

---

## Notifications

**Pushover Operational Log** (`notify.operational_log`):
- Automatic cycle started: includes runtime, today's forecasted high, humidity, and forecast conditions
- Cycle skipped due to weather: includes reason, today's forecasted high, humidity, and forecast conditions
- System disabled: silent, no notification in either direction
- Priority: Low (informational)

**Pushover House Notices** (`notify.house_notices`):
- Irrigation manual runs
- Priority: Normal (awareness)

**Pushover House Alerts** (`notify.house_alerts`):
- Irrigation deadman safety shutoff triggered (indicates malfunction)
- Priority: High

---

## Related Systems

**Met.no Weather Integration:**
- Provides forecasted high temperature, humidity, and daily condition state
- Entity: `weather.forecast_home_2`
- Built-in HA integration, no separate documentation

**Insteon Controls:**
- I/O module controls 24VAC irrigation valve
- See: insteon-integration.md

---

## Troubleshooting

**Irrigation not running on schedule:**
- Verify `input_boolean.irrigation_system_enabled` is ON
- Check current weather state — skips on active precipitation (rainy/pouring/lightning-rainy/snowy-rainy)
- Check today's and tomorrow's forecast condition — skips if either is in the bad weather list
- Review `input_number.irrigation_zone_runtime` — should reflect today's ET calculation
- Verify `weather.forecast_home_2` is updating correctly

**Runtime seems too high or too low:**
- ET formula uses forecasted high temperature and humidity from `weather.forecast_home_2`
- Check that today's forecast values look reasonable in Developer Tools → Actions → weather.get_forecasts
- Manual adjustment of `input_number.irrigation_zone_runtime` is possible if needed
- Expected range: ~14 min (cool/humid) to 40 min (hot/dry)

**Manual button not working:**
- Verify `input_boolean.irrigation_system_enabled` is ON
- Check `switch.patio_drip_sprinkler` responds to manual commands
- Review automation trace for manual run automation

**System won't turn off:**
- Deadman timer forces off after 45 minutes
- If valve is physically stuck, check valve wiring and solenoid
- Can manually turn off via `switch.patio_drip_sprinkler`
- Disable system via `input_boolean.irrigation_system_enabled`

**Irrigation skipping too many days:**
- Skip triggers on active precipitation states or bad forecast conditions for today or tomorrow
- Check `weather.forecast_home_2` condition values when skip occurs
- Review notification in operational log for stated skip reason

---

## YAML Packages

- `/config/packages/irrigation.yaml`

---