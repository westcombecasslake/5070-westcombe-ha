# Insteon Scenes and Groups

Master reference for all Insteon scene and group IDs configured in the system.

Last Updated: February 2026

---

## Scene Overview

Insteon scenes allow multiple devices to be controlled simultaneously with a single command. Each scene has both a hexadecimal and decimal ID for reference in different contexts.

**Total Scenes:** 13

---

## Scene Definitions

**Complete scene information available in Google Sheet:**

**[House IT Inventory & Documentation](https://docs.google.com/spreadsheets/d/1w4-gFy8rL0OqKrlzC7T9p2vFs5JBJYV2dyVfrrOA8DI)**  
Google Account: westcombecasslake@gmail.com (transfers with house)

The spreadsheet contains:
- Complete scene list with hex and decimal IDs
- Device assignments for each scene
- Scene purposes and usage
- Cross-references to automations

**Quick Scene Reference:**
- Scene 17 (0x11) - Back Patio Lights
- Scene 18 (0x12) - TV Flood
- Scene 19 (0x13) - Kitchen Lights
- Scene 20 (0x14) - Garage Lights
- Scene 21 (0x15) - Premise Lights (Master exterior scene)
- Scene 23 (0x17) - Seasonal Lights
- Scene 24 (0x18) - Living Room Lamps
- Scene 26 (0x1A) - Dining Room Table
- Scene 27 (0x1B) - Patio Gazebo Lights
- Scene 28 (0x1C) - HVAC Fans Off on Siren
- Scene 29 (0x1D) - Garage Front Porch Override
- Scene 30 (0x1E) - Master Bedroom Lights
- Scene 31 (0x1F) - Living Room Lamps - Fade up

---

## Scene Details

### Lighting Scenes

#### Back Patio Lights (0x11)
- **Devices:** `switch.back_patio_living_room`, `switch.back_patio_kitchen`
- **Purpose:** Synchronized control of all back patio lighting
- **Automation:** Dusk sensor triggers

#### TV Flood (0x12)
- **Devices:** `switch.living_room_over_tv_flood_control`, `switch.living_room_over_tv_flood_remote`
- **Purpose:** Synchronize TV area flood lighting between control switch and remote
- **Use Case:** Evening entertainment lighting

#### Kitchen Lights (0x13)
- **Devices:** `switch.kitchen_lights_back`, `switch.kitchen_lights_middle`, `switch.kitchen_lights_sink`
- **Purpose:** Master kitchen lighting control (3-gang coordination)
- **Pattern:** All lights on/off together, individual control available

#### Garage Lights (0x14)
- **Devices:** `light.garage_lights_garage`, `switch.garage_lights_mudroom`
- **Purpose:** 3-way lighting between garage and mudroom
- **Sensor:** `binary_sensor.garage_motion` triggers scene

#### Premise Lights (0x15)
- **Purpose:** Master exterior lighting scene - all outdoor lights
- **Includes:** Front porch, back floods, gazebo, landscape
- **Trigger:** Dusk sensor at sunset

#### Seasonal Lights (0x17)
- **Indoor:** Christmas tree, hearth lights, dining room seasonal, TV stand
- **Outdoor:** Front door outlets, entry closet landscape
- **Pattern:** All seasonal decorations controlled together

#### Living Room Lamps (0x18)
- **Devices:** `switch.living_room_lamp_kitchen_side`, `switch.living_room_lamp_back_corner`
- **Purpose:** Coordinated lamp control
- **Related:** Scene 0x1F provides fade-up variant

#### Dining Room Table (0x1A)
- **Purpose:** Dining area overhead lighting
- **Integration:** Part of evening lighting routines

#### Patio Gazebo Lights (0x1B)
- **Devices:** `switch.gazebo_lights_kitchen`, `switch.gazebo_wireless_remote`, `switch.gazebo_led_power`
- **Purpose:** Complete gazebo lighting control
- **Features:** Wireless remote + hardwired switches

#### Master Bedroom Lights (0x1E)
- **Devices:** All master bedroom and bath switches
- **Purpose:** Coordinated bedroom lighting scene
- **Pattern:** Single-button all-off capability

#### Living Room Lamps - Fade up (0x1F)
- **Same devices as 0x18**
- **Behavior:** Gradual ramp-up over 2 minutes
- **Trigger:** 30 minutes before sunset
- **Purpose:** Smooth transition to evening lighting

---

### Safety & Alert Scenes

#### HVAC Fans Off on Siren (0x1C)
- **Trigger:** Smoke/CO siren activation (`switch.smoke_co_siren`)
- **Action:** Shut off all HVAC fans
  - Master bath fan
  - Laundry fan
  - Entry bath fan
- **Purpose:** Prevent smoke distribution through HVAC system
- **Safety Note:** Critical life-safety automation

---

### Override Scenes

#### Garage Front Porch Override (0x1D)
- **Control:** `switch.front_porch_override`
- **Purpose:** Manual override of automated porch outlet schedules
- **Use Case:** Keep seasonal lights on during parties/events
- **Behavior:** Bypasses normal timer automation

---

## Scene Usage in Automations

### Referenced in YAML Files

**lightingpremise.yaml:**
- Premise Lights (0x15) - Sunset activation
- Living Room Lamps - Fade up (0x1F) - Pre-dusk ramp

**smokeco.yaml:**
- HVAC Fans Off on Siren (0x1C) - Smoke distribution prevention

**lightingindoor.yaml:**
- Seasonal Lights (0x17) - Holiday decoration control
- Living Room Lamps (0x18) - Evening routines

---

## Scene ID Reference Guide

### Hex to Decimal Quick Reference
- 0x11 = 17
- 0x12 = 18
- 0x13 = 19
- 0x14 = 20
- 0x15 = 21
- 0x17 = 23
- 0x18 = 24
- 0x1A = 26
- 0x1B = 27
- 0x1C = 28
- 0x1D = 29
- 0x1E = 30
- 0x1F = 31

### Scene Numbering Strategy
- **0x11-0x15:** Primary functional lighting groups
- **0x17-0x1B:** Secondary/seasonal lighting
- **0x1C:** Safety automation
- **0x1D:** Manual override
- **0x1E-0x1F:** Specialized lighting scenes

---

## Programming Notes

- Scenes persist in Insteon device link tables
- Backup stored in `/mnt/project/insteon_scenes.json`
- Test scenes individually after any device replacement
