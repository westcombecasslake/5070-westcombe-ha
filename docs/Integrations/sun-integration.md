# Sun Integration

**Integration Type:** Astronomical Data  
**Devices in System:** 1 service  
**Primary Use:** Backup failsafe and informational display  
**Status:** Active

---

## What is Sun Integration?

The Sun integration provides astronomical data including sunrise, sunset, solar noon, and sun position (elevation/azimuth). This built-in Home Assistant integration uses your location coordinates to calculate accurate sun timing throughout the year.

**Key Features:**
- Precise sunrise/sunset times
- Solar elevation and azimuth
- Dawn/dusk calculations
- Automatic seasonal adjustments
- No external service required
- Updates every minute

---

## Role in This System

**Functions:**
- **Backup failsafe** for premise lighting if photocell sensor fails
- **Informational display** in notifications

**How It's Used:**

**Backup Automation (`lightingpremise.yaml`):**
- Triggers 15 minutes after sunset if photocell fails to activate lights
- Escalated alert sent to Pushover (House Alerts) indicating photocell maintenance needed
- Ensures lights turn on even if photocell malfunctions

**Informational:**
- Shows "Actual dusk" time in operational log notifications for comparison
- Helps verify photocell+delay timing is appropriate

**NOT Primary Trigger:**
- Premise lights are primarily triggered by **photocell sensor** + **seasonal delay helpers**
- Sun integration is backup/failsafe only

---

## Home Assistant Configuration

### Integration Setup

**Location:** Settings → Devices & Services → Integrations → Sun

**Configuration:**
- Automatically configured during HA setup
- Uses Home Assistant location (latitude/longitude)
- No API keys or authentication required
- Built-in core integration

**Entity Created:**
- `sun.sun` - Main sun entity with all attributes

---

## Sun Entity Attributes

**State:**
- `above_horizon` or `below_horizon`

**Key Attributes:**
- `next_rising` - Next sunrise time
- `next_setting` - Next sunset time
- `elevation` - Sun elevation angle (-90° to 90°)
- `azimuth` - Sun compass direction (0° to 360°)
- `rising` - Boolean, is sun rising?

**Additional Times:**
- `next_dawn` - Civil dawn
- `next_dusk` - Civil dusk
- `next_noon` - Solar noon
- `next_midnight` - Solar midnight

---

## Integration with Home Assistant

### Used in Automations

**Premise Lighting Backup (`lightingpremise.yaml`):**
- Automation ID: `premise_lights_dusk_backup`
- Triggers 15 minutes after sunset
- Conditions:
  - Photocell module still OFF (should be ON by now)
  - Premise lights still OFF (photocell automation didn't run)
  - Sun below horizon (actually dark)
- Action: Turn on lights + send alert about photocell failure

**Premise Lighting Notification:**
- Sun's "next_dusk" attribute displayed in operational log
- Compares actual dusk time to photocell trigger time
- Format: "Actual dusk: [time]"
- Helps homeowner verify seasonal delay settings appropriate

**Why Photocell Instead of Sun?**
- Photocell responds to actual light conditions (clouds, trees, buildings)
- Sun calculation is theoretical (clear sky)
- Photocell provides more appropriate trigger for lighting
- Sun serves as reliable backup if photocell fails

---

## Premise Lighting System Overview

**Primary System:**
1. Photocell sensor detects dusk (actual light conditions)
2. Seasonal delay helper adds appropriate wait time (varies by month)
3. Premise lights turn on via Insteon scene

**Backup System:**
1. If photocell fails (sun below horizon but lights still off)
2. Sun integration triggers 15 min after sunset
3. Lights turn on + maintenance alert sent

**Why This Approach:**
- Photocell: Responds to real-world conditions
- Seasonal delays: Compensate for photocell sensitivity variations
- Sun backup: Ensures lights always come on even if photocell breaks

---

## Accessing Sun Data

**In Home Assistant:**
- Developer Tools → States → `sun.sun`
- View current state and all attributes
- See next sunrise/sunset times
- Check solar elevation/azimuth

**Dashboard Display:**
- Can add sun card to dashboard
- Shows sunrise/sunset times
- Displays sun position graphically

---

## Troubleshooting

**Backup Automation Triggering Frequently:**

1. **Indicates Photocell Problem:**
   - Photocell sensor needs maintenance
   - Check sensor for obstructions
   - Verify photocell module power
   - Review biannual maintenance reminders

2. **Check Automation:**
   - Developer Tools → Automations → Traces
   - View `premise_lights_dusk_backup` execution
   - Verify why photocell wasn't ON

**Inaccurate Sun Times:**

1. **Check Location:**
   - Settings → System → General
   - Verify latitude/longitude correct
   - Ensure time zone set properly (America/Detroit)

2. **Restart Integration:**
   - Rare, but reload integration if times seem wrong

**Backup Not Triggering:**

1. **Verify Automation Enabled:**
   - Settings → Automations
   - Check `premise_lights_dusk_backup` enabled

2. **Check Conditions:**
   - Automation only runs if photocell OFF
   - Automation only runs if lights OFF
   - Automation only runs if sun below horizon

---

## Migration Notes

**No Migration Needed:**
- Built-in integration
- Automatically configured on any HA instance
- Uses HA location settings
- No account or API keys

**If Replacing Home Assistant:**
1. Sun integration automatically configured
2. Recreate backup automation from YAML backup
3. Verify location settings correct
4. Test backup automation manually

---

## Official Resources

**Home Assistant Integration:**
- Documentation: https://www.home-assistant.io/integrations/sun/
- GitHub: https://github.com/home-assistant/core/tree/dev/homeassistant/components/sun

---

## Important Notes

- **Built-in integration** - no setup required
- **Backup failsafe only** - NOT primary trigger for premise lighting
- **Primary trigger:** Photocell sensor + seasonal delay helpers
- **Backup trigger:** Sun sunset + 15 minutes (if photocell fails)
- **Informational display:** Shows actual dusk time in notifications
- **Automatic seasonal adjustment** - no manual schedule changes needed
- **Location-dependent** - accuracy depends on correct HA location settings
- **Updates every minute** - always current
- **No external dependencies** - calculated locally
- **Used in:** `lightingpremise.yaml` for backup failsafe + informational display

---

**Last Updated:** February 2026  
**Integration Version:** Home Assistant Core (current)  
**Document Maintainer:** Homeowner
