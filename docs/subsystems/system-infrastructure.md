# System Infrastructure

**Last Updated:** April 2026

---

## Overview

Three packages manage system-level health, startup, and preventive maintenance:

- **systemstartup.yaml** — Startup sequence: notifications, status refresh, and occupancy restore on every HA restart
- **systemhealth.yaml** — Integration offline detection and recovery alerts for five key integrations
- **powerinfrastructure.yaml** — Preventive power cycling for devices that require periodic resets

These packages have no user-facing controls. They operate silently in the background and surface alerts only when something needs attention.

---

## System Startup

### What It Does

On every Home Assistant restart, runs a startup sequence that:

1. Waits 30 seconds for integrations to initialize
2. Sends a restart notification (timestamp) to House Alerts
3. Refreshes script-based status helpers (battery, smoke/CO online, smoke/CO alert status)
4. Resets all integration status helpers to Online — offline detection automations in systemhealth.yaml will re-fire if any integration fails to come back
5. Waits an additional 30 seconds for Matter/Thread devices to fully reconnect
6. Turns off IKEA Alpstuga air quality monitor screens (they reset to ON after power loss)
7. Restores occupancy booleans based on elapsed time since occupancy was last detected

### Occupancy Boolean Restore

The `house_unoccupied` and `long_term_away` booleans in garage.yaml rely on `for:` timers that do not survive restarts. On startup, the sequence calculates how long `binary_sensor.main_floor_occupancy` has been continuously off and restores the correct state:

| Condition | Action |
|-----------|--------|
| Occupancy currently ON | Clear both booleans (someone is home) |
| Off for 8-23 hours | Set `house_unoccupied` only |
| Off for 24+ hours | Set both `house_unoccupied` and `long_term_away` |

### Alpstuga Screen Handling

IKEA Alpstuga air quality monitor screens reset to ON after any power interruption. The startup sequence turns them off after allowing Matter/Thread to reconnect. This is intentional — screens are not normally displayed during operation.

### Key Entities

- `input_boolean.house_unoccupied` — restored based on occupancy elapsed time
- `input_boolean.long_term_away` — restored based on occupancy elapsed time
- `binary_sensor.main_floor_occupancy` — source for elapsed time calculation
- `switch.dinning_air_quality_screen` — turned off on startup
- `switch.master_bedroom_air_quality_screen` — turned off on startup

### Automations

**System Startup Notification & Status Refresh:**
- ID: `system_startup_notification`
- Alias: `System - Startup Notification & Status Refresh`
- Trigger: `homeassistant` start event
- Action: See sequence above

**YAML:** systemstartup.yaml

---

## Integration Health Monitoring

### What It Does

Monitors five integrations via proxy entities. If a proxy entity goes unavailable for the configured duration, an alert is sent and the integration's status helper is set to "Offline". When the proxy recovers and stays available for 2 minutes, the helper is reset to "Online".

A combined `integrations_status` helper reflects the overall state — "Normal" when all five are Online, "Problem" if any are offline. Template sensors mirror all six helpers for display in Android Auto.

### Monitored Integrations

| Integration | Proxy Entity | Offline Threshold | Notification |
|-------------|-------------|-------------------|--------------|
| Insteon | `light.entry_bath_fan`, `light.master_bath_fan` | 10 min | House Alerts |
| YoLink | `sensor.basement_smoke_co_battery`, `sensor.master_bath_temp_humidity_sensor_battery`, `sensor.laundry_temp_humidity_sensor_battery` | 10 min | House Alerts |
| Ecobee | `sensor.main_floor_current_temperature` | 10 min | House Alerts |
| Ring | `sensor.front_door_battery` | 15 min | House Alerts |
| Matter/Thread | `sensor.master_bedroom_outlet_thread_router_power_top` | 10 min | House Alerts |

Ring uses 15 minutes (vs 10) because it is WiFi-connected and less critical to immediate safety response.

Matter/Thread uses an Eve Energy outlet (Thread router) as a proxy for the Thread network health.

### Helpers

| Helper | Entity | Purpose |
|--------|--------|---------|
| Insteon Status | `input_text.insteon_status` | Online / Offline |
| YoLink Status | `input_text.yolink_status` | Online / Offline |
| Ecobee Status | `input_text.ecobee_status` | Online / Offline |
| Ring Status | `input_text.ring_status` | Online / Offline |
| Matter/Thread Status | `input_text.matter_status` | Online / Offline |
| Integrations Status | `input_text.integrations_status` | Normal / Problem |

Template sensor mirrors exist for each of the above for Android Auto display.

### Automations

Each integration has a paired offline/online automation:

| ID | Alias |
|----|-------|
| `system_health_insteon_offline` | System Health - Insteon System Offline |
| `system_health_insteon_online` | System Health - Insteon System Online |
| `system_health_yolink_offline` | System Health - YoLink System Offline |
| `system_health_yolink_online` | System Health - YoLink System Online |
| `system_health_ecobee_offline` | System Health - Ecobee Offline |
| `system_health_ecobee_online` | System Health - Ecobee Online |
| `system_health_ring_offline` | System Health - Ring Doorbell Offline |
| `system_health_ring_online` | System Health - Ring Doorbell Online |
| `system_health_matter_offline` | System Health - Matter/Thread Offline |
| `system_health_matter_online` | System Health - Matter/Thread Online |
| `system_health_integrations_status_update` | System Health - Update Overall Integrations Status |

The status update automation triggers whenever any individual status helper changes and recalculates the combined `integrations_status`.

**YAML:** systemhealth.yaml

---

## Power Infrastructure

### What It Does

Performs a daily preventive power cycle on the kitchen vent hood at 5:00 AM. Turns the switch off, waits 15 seconds, then turns it back on. Prevents electronics lock-up that requires manual intervention.

**Rationale:** The kitchen vent hood electronics occasionally lock up and become unresponsive. A brief daily power cycle at a low-traffic time prevents this without any user impact.

### Key Entities

- `switch.vent_hood_reset` — Insteon outlet controlling vent hood power

### Automations

**Kitchen Vent Hood Daily Reset:**
- ID: `power_vent_hood_daily_reset`
- Alias: `Powerline - Kitchen Vent Hood Daily Reset`
- Trigger: Daily at 5:00 AM
- Action: Turn off → wait 15 seconds → turn on

**YAML:** powerinfrastructure.yaml

---

## Notifications

| Alert | Service | Condition |
|-------|---------|-----------|
| HA Restarted | `house_alerts` | Every restart |
| [Integration] Offline | `house_alerts` | Proxy unavailable for threshold duration |

Recovery is silent — no notification is sent when an integration comes back online, the status helper is simply reset.

---

## Troubleshooting

**Restart notification not received:**
- Check `system_startup_notification` automation is enabled
- Verify Pushover House Alerts service is functioning
- Note: notification fires 30 seconds after start, not immediately

**Integration showing Offline after restart:**
- Expected if the integration failed to reinitialize — check the integration itself
- If integration is actually online, check the proxy entity is reporting correctly
- Manual fix: set the `input_text.[integration]_status` to "Online" via Developer Tools

**Alpstuga screens turning on after restart:**
- Expected behavior — they reset to ON after any power interruption
- The startup sequence turns them off; if they remain on, check Matter/Thread connectivity
- Startup sequence waits 60 seconds total before issuing the screen-off command

**Vent hood unresponsive:**
- Check `switch.vent_hood_reset` entity state in Developer Tools
- Manual power cycle: turn switch off, wait 15 seconds, turn back on
- If switch itself is unavailable, check Insteon PLM connection

---

## Related Documentation

- **[YoLink Integration](../Integrations/yolink-integration.md)** — YoLink hub and sensor details
- **[Insteon Integration](../Integrations/insteon-integration.md)** — Insteon PLM and device details
- **[HomeKit Device Integration](../Integrations/homekit-device-integration.md)** — Matter/Thread details
- **[Garage Systems](garage.md)** — Occupancy booleans and away mode
- **[Safety Monitoring](safety-monitoring.md)** — Smoke/CO status helpers refreshed on startup
- **[Battery Monitoring](battery-monitoring-system.md)** — Battery status helper refreshed on startup

---

## YAML Packages

- `/config/packages/systemstartup.yaml`
- `/config/packages/systemhealth.yaml`
- `/config/packages/powerinfrastructure.yaml`
