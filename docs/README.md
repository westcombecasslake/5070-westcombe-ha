# 5070 Westcombe Home Automation System

**Platform:** Home Assistant (HA Green)  
**Last Updated:** April 2026  
**Status:** Fully operational

---

## System Overview

A whole-home automation system built on Home Assistant, providing safety monitoring, environmental control, lighting automation, appliance monitoring, and system health tracking. The system emphasizes reliability, safety-first notification routing, and operation that survives partial failures (including HA itself going offline).

---

## Hardware

| Component | Model | Notes |
|---|---|---|
| HA Server | Home Assistant Green | Primary controller |
| Thread Border Router | Silicon Labs ZBT-2 | Installed in HA Green |
| Insteon PLM | 2413U | USB, address 70.8C.C1 |
| YoLink Hub | YS1604-UC (Hub 2) | WiFi, cloud-based |
| Ecobee Thermostat | Smart Thermostat | HomeKit Device integration |
| Ring Doorbell | Video Doorbell 2 | Cloud-based |

---

## Integrations

| Integration | Protocol | Purpose |
|---|---|---|
| Insteon | Powerline + RF mesh | All lighting control |
| YoLink | LoRa (915 MHz) | Safety sensors, environmental |
| Matter/Thread | Thread mesh | Air quality, bathroom fans |
| Ecobee (HomeKit Device) | WiFi/HomeKit | HVAC monitoring and fan control |
| Ring | WiFi/Cloud | Front door motion, battery monitoring |
| Pushover | Cloud | All notifications (4-tier system) |
| Sun | Built-in | Premise lighting backup failsafe |
| Backup | Built-in + Nabu Casa | System backups |

See `/docs/integrations/` for detailed documentation on each integration.

---

## Notification Architecture

Four-tier Pushover system -- never mix tiers:

| Service | Priority | Use For |
|---|---|---|
| `notify.house_alerts` | High (1) | Safety emergencies |
| `notify.house_notices` | Normal (0) | Awareness, act within 24-48 hrs |
| `notify.operational_log` | Silent (-2) | Daily summaries, routine confirmations |
| `notify.battery_alerts` | Normal (0) | 25% low battery warnings |

---

## Subsystems

| Subsystem | Package File | Summary |
|---|---|---|
| Smoke/CO Detection | `smokeco.yaml` | 8-zone YoLink detection, HVAC shutdown, offline monitoring |
| Leak Detection | `basementleaksensors.yaml` | 3 YoLink leak sensors, siren via YoLink alarm group |
| Sump Pump | `sumppump.yaml` | Cycle tracking, failure detection, daily summary |
| Lighting -- Indoor | `lightingindoor.yaml` | Seasonal, living room, master bedroom automations |
| Lighting -- Premise | `lightingpremise.yaml` | Photocell-triggered exterior lighting, Ring motion |
| Environmental | `hvachumiditypad.yaml` | Humidifier pad monitoring and trend alerting |
| Dehumidifier | `basementdehumidifiercontrol.yaml` | Humidity-based control with peak-hour avoidance |
| Bathroom Humidity | `masterbathhumidity.yaml`, `middlebathhumidity.yaml`, `eastbathhumidity.yaml` | Relative-rise shower detection, HVAC fan boost |
| Air Quality | `airquality.yaml` | CO2, PM2.5, TVOC via IKEA Alpstuga sensors |
| Fridge/Freezer | `fridgefreezer.yaml` | Temperature monitoring, failure detection |
| Laundry | `laundry.yaml` | Dryer cycle completion, HVAC fan boost |
| Washer/Dryer Notify | `washerdryernotify.yaml` | Washer and dryer cycle completion notifications |
| Garage | `garage.yaml` | Motion lighting, freeze alerts |
| Irrigation | `irrigation.yaml` | Seasonal schedule automation |
| Battery Monitoring | `batteryalerts.yaml` | Single-level battery alerts (25% threshold) |
| System Health | `systemhealth.yaml` | Integration status, daily health reports |
| System Startup | `systemstartup.yaml` | Post-restart status restoration |
| Power Infrastructure | `powerinfrastructure.yaml` | Preventive power cycling for devices requiring periodic resets |
| Entry Fan | `entryfan.yaml` | Entry bathroom fan control |

See `/docs/subsystems/` for detailed documentation on each subsystem.

---

## File Structure

```
/config/
  packages/          -- All automation YAML (one file per subsystem)
  docs/
    README.md        -- This file
    device-inventory.md
    insteon-scenes.md
    apps.md
    transferring-ownership.md
    Integrations/    -- Per-integration documentation
    subsystems/      -- Per-subsystem documentation
  insteon_devices.json
  insteon_scenes.json
  configuration.yaml
  secrets.yaml
```

---

## Key Design Decisions

Full rationale preserved in `builddecisions.md`. Critical items:

- **All Insteon scene commands use `service:` syntax** with decimal scene numbers (never hex)
- **All Insteon scene commands double-send** with 3-second delay -- fire-and-forget protocol, no acknowledgment
- **Photocell is primary for premise lights**, not sun schedule -- sun is backup failsafe only
- **Battery alerts split by tier** -- 30% to `battery_alerts`, 15% to `house_alerts` (safety gap)
- **Ecobee fan controlled via `device_id`** -- no fan entity exists through HomeKit Device integration
- **YoLink suppression** on smoke/CO offline alerts -- cloud outage handled by system health, not individual detectors

---

## Accounts

All service accounts use the house email: **westcombecasslake@gmail.com**

| Service | Account | Notes |
|---|---|---|
| Nabu Casa | westcombecasslake@gmail.com | HA Cloud + backups, $65/yr |
| YoLink | westcombecasslake@gmail.com | Hub + VirtualArm monitoring |
| Ring | westcombecasslake@gmail.com | Ring Solo plan |
| Ecobee | westcombecasslake@gmail.com | Thermostat + remote sensor |
| Pushover | westcombecasslake@gmail.com | One-time purchase, 4 apps |

See `transferring-ownership.md` for full account transfer procedures.

---

**Document Maintainer:** Homeowner
