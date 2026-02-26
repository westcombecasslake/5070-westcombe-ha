# Insteon Integration

**Integration Type:** Home Automation Protocol  
**Devices in System:** 39  
**Primary Use:** Lighting control and scenes  
**Status:** Active

---

## What is Insteon?

Insteon is a dual-band (powerline and RF) home automation protocol that enables device-to-device communication without requiring a central hub for basic operation. Each Insteon device acts as a repeater, creating a reliable mesh network throughout the home.

**Key Technology Features:**
- Simultaneous powerline (electrical wiring) and 915 MHz RF communication
- Self-healing mesh network - devices repeat signals for reliability
- Scene-based control - multiple devices respond to single command
- No cloud dependency - operates locally on your powerline
- **Device-to-device links operate independently** - switches control lights even without HA or PLM

---

## Role in This System

**Primary Functions:**
- All interior and exterior lighting control
- Multi-switch virtual circuits (n-way switching without physical wiring)
- Scene coordination (15 programmed Insteon scenes for room/area control)
- Motion-activated lighting
- Seasonal/holiday decoration automation

**Device Breakdown:**
- **Switches (PS01, 2442-222):** ~35 devices - Standard on/off wall switches
- **Micro Modules (2443-222):** 4 devices
  - 2 dimmers
  - 2 on/off relays
- **Lamp Modules (2457D2, 2856D2):** ~6 devices - Plug-in dimmable control for lamps
- **Appliance Modules (2635-222):** ~2 devices - Outlet-based on/off control
- **Wireless Remotes (2342-2):** 1 device

---

## Home Assistant Configuration

### Special Integration Structure

**IMPORTANT:** Insteon appears in TWO locations in Home Assistant settings:

**Location 1: Settings → Devices & Services → Integrations**
- Shows alongside other integrations (YoLink, Pushover, etc.)
- Standard integration card showing device count
- Access device/entity management

**Location 2: Settings → Insteon** (dedicated menu item)
- Special integration-specific settings page
- Unique to Insteon (most integrations don't have this)
- Access to:
  - PLM configuration and status
  - Device linking and management tools
  - Scene programming interface
  - Advanced Insteon-specific settings
  - Device link table management

**Why Two Locations?**
The Insteon integration is more complex than typical integrations due to its device-to-device mesh protocol. The dedicated settings page provides tools for managing the Insteon network that don't fit the standard HA integration model.

### Hardware Required

- Insteon 2413U Powerline Modem (PLM)
- USB connection to Home Assistant server
- PLM Address: 70.8C.C1

### Configuration Files

- Integration: Settings → Devices & Services → Insteon
- Device registry: `/config/insteon_devices.json`
- Scene registry: `/config/insteon_scenes.json`

**Setup:**
1. PLM connected via USB to Home Assistant Green
2. Integration auto-discovered and configured
3. Devices linked to PLM via Insteon protocol
4. Scenes programmed and stored in device link tables

---

## Scene Control - IMPORTANT

### Insteon Scenes vs. Home Assistant Scenes

**This system uses NATIVE INSTEON SCENES only - NOT Home Assistant scenes.**

**Insteon Scenes:**
- Programmed directly into Insteon device link tables
- Stored in each device's memory
- Triggered via `insteon.scene_on` / `insteon.scene_off` service calls in YAML
- Survive Home Assistant restarts, updates, or failures
- Work even if HA is offline (devices can trigger scenes directly)
- Example YAML:
  ```yaml
  action:
    - service: insteon.scene_on
      data:
        group: 21  # Scene 21 = Premise Lights
  ```

**Home Assistant Scenes:**
- NOT USED in this system
- Would be found in Settings → Automations & Scenes → Scenes tab
- Stored in HA configuration, not device memory
- Require HA to be running

**Why Insteon Scenes?**
- More reliable (stored in device hardware)
- Faster execution (simultaneous activation via mesh broadcast)
- Scene continues to work if HA fails
- Devices can trigger scenes directly (motion sensor → lights)

### Critical: Use action: Not service: for Insteon Calls

**⚠️ Insteon scene commands must use `action:` syntax — not `service:`.**

```yaml
# CORRECT
- action: insteon.scene_on
  data:
    group: 21

# WRONG -- silently fails; scenes do not execute
- service: insteon.scene_on
  data:
    group: 21
```

Confirmed in production (February 2026): `service:` syntax does not execute Insteon scenes. The `action:` keyword is the correct syntax for Insteon calls in this system. Do not change Insteon calls to `service:` when editing or reviewing YAML.

### This System's 13 Insteon Scenes

**Scene Programming:**
All scenes were programmed using the Insteon integration's scene management tools (Settings → Insteon → Scenes).

**Scene List (Decimal numbers used in HA):**
- Scene 17: Back Patio Lights
- Scene 18: TV Flood synchronization
- Scene 19: Kitchen Lights group
- Scene 20: Garage Lights
- Scene 21: **Premise Lights** (all exterior)
- Scene 23: Seasonal decoration lights
- Scene 24: Living Room Lamps
- Scene 26: Dining Room Table
- Scene 27: Patio Gazebo Lights
- Scene 28: HVAC Fans Off (safety)
- Scene 29: Front Porch Override
- Scene 30: Master Bedroom Lights
- Scene 31: Living Room Lamps Fade-up

**How Scenes Are Called:**
In automation YAML files, scenes are activated using **decimal** scene numbers:
```yaml
- service: insteon.scene_on
  data:
    group: 21  # Decimal - Premise Lights
```
Or:
```yaml
- service: insteon.scene_off
  data:
    group: 21
```

Full scene documentation with members and purposes: `insteon-scenes.md`

**Scene Benefits:**
- Single command activates all scene members simultaneously
- No sequential delays
- Mesh network repeats scene commands for reliability
- Guaranteed state synchronization across all devices

### Scene Reliability - Double-Send Pattern

**Implementation:** All scene commands in automations use double-send with 3-second delay

**Rationale:** Insteon scene broadcasts are fire-and-forget with no acknowledgment. Network collisions, powerline noise, or RF interference can cause dropped commands. Second command ensures scene activation even if first command fails.

**Pattern:**
```yaml
- service: insteon.scene_on
  data:
    group: 21
- delay: "00:00:03"
- service: insteon.scene_on
  data:
    group: 21
```

**Timing:** 3-second delay allows first command's broadcast + cleanup messages to fully complete before second attempt. Based on Insteon mesh network settling time.

---

## Key Features Used

**Virtual N-Way Switching:**
Multiple physical switches control the same lights without additional wiring. Example: Kitchen has 3-gang switch - each switch can control all kitchen lights independently.

**IMPORTANT:** These virtual n-way circuits are programmed directly into the Insteon devices and operate independently of Home Assistant and the PLM. The switches communicate directly with the lights via the Insteon mesh network. Even if HA is removed or fails, all switch-to-light control continues to function normally. HA and the PLM are only needed for automation, scheduling, and scene control via software.

---

## System Backup and Migration

**Before Making Changes:**
1. Backup `/config/insteon_devices.json`
2. Backup `/config/insteon_scenes.json`
3. Backup all YAML automation files (they contain scene calls)
4. Note PLM address for reference

**If Moving to New HA Instance:**
1. Export device and scene JSON files
2. Backup all YAML automation files (they contain scene calls)
3. Install Insteon integration on new HA instance
4. Connect PLM via USB
5. Restore device and scene JSON files
6. Restore automation YAML files
7. All Insteon scenes remain in device memory (survive HA changes)

**If Replacing PLM:**
1. New PLM must be linked to all 39 devices manually
2. **Insteon scenes stored in device link tables survive PLM replacement**
3. Device-to-device links survive PLM replacement
4. HA integration automatically discovers new PLM
5. May need to re-configure scenes in HA (Settings → Insteon → Scenes)

**Alternative Control Options:**
- Insteon Hub (2245-222) - Cloud-based, no longer supported by manufacturer
- Universal Devices ISY controller - Local control, powerful but complex
- Home Assistant (current) - Recommended, actively maintained integration

---

## Finding Scene References in YAML

Since this system uses Insteon scenes called from automations, you'll find scene calls throughout the YAML files:

**Example locations:**
- `lightingpremise.yaml` - Scene 21 (Premise Lights), Scene 29 (Front Porch Override)
- `lightingindoor.yaml` - Scene 23 (Seasonal), Scene 24 (Lamps), Scene 30 (Master bedroom), Scene 31 (Lamp fade-up)
- `garage.yaml` - Scene 20 (Garage lights)

**To find all scene calls:**
```bash
grep -r "insteon.scene" /config/packages/
```

This shows every automation that triggers an Insteon scene.

---

## Official Resources

**Home Assistant Integration:**
- Documentation: https://www.home-assistant.io/integrations/insteon/
- GitHub: https://github.com/home-assistant/core/tree/dev/homeassistant/components/insteon
- Integration-specific settings: Settings → Insteon (in HA)

**Insteon Protocol:**
- Note: Insteon brand discontinued, but protocol still functional
- Devices widely available on secondary market
- No cloud services required for operation

**Device Manuals:**
- Archived at: https://www.insteon.com (if still available)
- Alternative: Search model numbers online for PDF manuals

---

## Important Notes

- **Insteon is a discontinued protocol** - manufacturer Smartlabs ceased operations
- Existing devices continue to function indefinitely (local control)
- No new devices being manufactured (stock up on spares if needed)
- Home Assistant integration remains actively maintained
- Large secondary market for used Insteon devices
- **This system uses NATIVE Insteon scenes, not HA scenes** - scenes survive HA failures
- Insteon has unique two-location presence in HA settings (normal integration + dedicated menu)

---

**Last Updated:** February 2026  
**Integration Version:** Home Assistant Core (current)  
**Document Maintainer:** Homeowner
