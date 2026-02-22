# HomeKit Device Integration

**Integration Type:** Apple HomeKit Bridge  
**Devices in System:** 2  
**Primary Use:** Ecobee smart thermostat integration  
**Status:** Active

---

## What is HomeKit Device Integration?

The HomeKit Device integration allows Home Assistant to connect to devices that support Apple's HomeKit protocol. This integration acts as a HomeKit controller, enabling HA to control and monitor HomeKit-compatible devices without requiring an Apple device or Apple Home app.

**Key Features:**
- Direct local control of HomeKit devices
- No Apple device or iCloud account required
- Automatic device discovery
- Secure encrypted communication
- Real-time status updates
- Native Home Assistant entities

---

## Role in This System

**Primary Functions:**
- Main floor climate control via Ecobee thermostat
- Temperature and humidity monitoring
- HVAC fan control for bathroom and laundry room ventilation
- Occupancy detection via remote sensor
- Integration with Home Assistant automations

**Devices Connected:**
- **Ecobee Smart Thermostat:** Main floor HVAC control
- **Ecobee Remote Sensor:** Occupancy and temperature sensing (Master bedroom)

---

## Home Assistant Configuration

### Integration Setup

**Location:** Settings → Devices & Services → Integrations → HomeKit Device

Standard integration location - single location like YoLink and Pushover.

**Configuration Method:**
- Local network discovery
- HomeKit pairing code authentication
- Automatic entity creation
- Real-time status polling

### Hardware

**Ecobee Smart Thermostat:**
- Model: Ecobee Smart Thermostat
- Serial Number: 522679989982
- Connection: WiFi to home network
- Controls: Main floor HVAC system
- Location: Main floor hallway

**Ecobee Remote Sensor:**
- Type: Room sensor with occupancy detection
- Connection: Wireless to thermostat
- Monitors: Temperature and occupancy
- Location: Master bedroom

### Setup Process

**Initial Pairing:**
1. Ecobee thermostat connected to WiFi
2. HomeKit enabled on Ecobee (via Settings menu)
3. HomeKit pairing code generated on thermostat display
4. Home Assistant discovers thermostat on local network
5. Enter pairing code in HA to complete setup
6. All entities automatically created

**No Cloud Dependency:**
- Communication is local between HA and Ecobee
- No Apple iCloud account needed
- No Apple device required
- Works entirely on local network

---

## Ecobee Thermostat Features

### Climate Control

**HVAC Modes:**
- Heat
- Cool  
- Auto (heat/cool)
- Off
- Aux heat (if equipped)

**Fan Control:**
- Auto (runs with heating/cooling)
- On (continuous fan)
- Circulate (periodic circulation)
- **HA Control Enabled:** Home Assistant can control fan via script through HomeKit integration

**Temperature Control:**
- Heating setpoint
- Cooling setpoint
- Current temperature
- Humidity level

### Smart Features

**Occupancy Sensing:**
- Built-in motion sensor
- Remote sensor occupancy detection (Master bedroom)
- Smart home/away mode
- Can trigger HA automations based on occupancy

**Weather Integration:**
- Ecobee has built-in weather data
- Displays outdoor temperature and forecast
- Not used by HA (HA uses Met.no for weather)

**Demand Response:**
- Ecobee has built-in demand response feature
- Configured with 3-7 PM weekday window (DTE peak hours)
- **Not enrolled in actual program** - just configured with window
- Can automatically adjust temperature during peak hours
- Independent of Home Assistant

### Remote Sensor

**Capabilities:**
- Temperature monitoring
- Occupancy detection (motion-based)
- Wireless connection to thermostat
- Battery powered
- Location: Master bedroom

**Integration:**
- Ecobee uses sensor for temperature averaging
- Can prioritize rooms with occupied sensors
- Occupancy status available in HA as entity
- Can trigger automations (though not currently used for this)

---

## Integration with Home Assistant

### Entities Created

**Climate Entity:**
- `climate.ecobee` - Main thermostat control
  - Current temperature
  - Current humidity
  - Target temperature (heat/cool)
  - HVAC mode
  - Fan mode
  - HVAC action (heating/cooling/idle)

**Sensor Entities:**
- `sensor.ecobee_temperature` - Current temperature
- `sensor.ecobee_humidity` - Current humidity
- `sensor.ecobee_occupancy` - Occupancy status
- Remote sensor temperature and occupancy entities (Master bedroom)

**Fan Control:**
- HVAC fan controllable via script through HomeKit integration
- Used for bathroom and laundry room ventilation

**Additional Entities:**
- Weather information (if enabled)
- Comfort settings
- System status

### Used in Automations

**HVAC Fan Control:**
- **Master Bath Humidity (`masterbathhumidity.yaml`):** HA controls HVAC fan via script to remove humidity
- **Laundry Room Fan (`laundry.yaml`):** HA controls HVAC fan via script during dryer operation to clear heat/humidity
- Fan control enabled through HomeKit device integration

**HVAC Humidifier Pad Monitoring (`hvachumiditypad.yaml`):**
- Monitors Ecobee humidity sensor readings
- Tracks 7-day and 14-day rolling averages
- Detects humidifier pad degradation
- Alerts when replacement needed
- Does NOT control humidifier on/off (humidifier not wired to Ecobee)

**Potential Uses (Not Currently Implemented):**
- Occupancy-based temperature setbacks
- Away mode automation
- Temperature-based humidifier control (future enhancement)

---

## HVAC Fan Control via Scripts

### How It Works

**Script-Based Control:**
Home Assistant controls the Ecobee HVAC fan through scripts that interact with the HomeKit device integration.

**Use Cases:**
1. **Master Bathroom Ventilation:**
   - Humidity sensor detects high humidity (shower/bath)
   - HA script turns HVAC fan ON
   - Fan runs until humidity drops to normal
   - HA script turns HVAC fan back to AUTO

2. **Laundry Room Ventilation:**
   - Dryer power consumption detected (cycle running)
   - Temperature rises in laundry room
   - HA script turns HVAC fan ON
   - Fan runs to clear heat and humidity
   - HA script returns fan to AUTO when complete

**Benefits:**
- Uses existing HVAC ductwork for ventilation
- No separate exhaust fans needed in these rooms
- Centralized air distribution
- Controlled humidity removal

**Script Configuration:**
Scripts defined in `/config/scripts.yaml` that call climate services to control fan mode via HomeKit device integration.

---

## Ecobee vs Home Assistant Control

### What Ecobee Handles

**Built-in Features:**
- Temperature control and scheduling
- Smart recovery (pre-heating/cooling)
- Adaptive learning
- Weather display
- Demand response window (3-7 PM weekdays configured)
- Default fan circulation schedules
- Humidity display

**Ecobee App:**
- Remote control when away from home
- Schedule programming
- Comfort settings (Home, Away, Sleep)
- System alerts and reminders
- Firmware updates
- Accessory management

### What Home Assistant Monitors and Controls

**Monitoring:**
- Temperature and humidity readings for automations
- Humidifier effectiveness tracking
- System status monitoring
- Occupancy status (Master bedroom remote sensor)

**Active Control:**
- **HVAC Fan:** HA controls fan mode via scripts for bathroom and laundry ventilation
- Fan mode changes: AUTO ↔ ON based on humidity and temperature conditions

**HA Does NOT Control:**
- Temperature setpoints (handled by Ecobee)
- HVAC modes (heat/cool/auto - user controls via Ecobee)
- Ecobee scheduling
- Demand response

### Why This Hybrid Approach?

**Ecobee is better at:**
- HVAC scheduling (adaptive learning, smart recovery)
- Direct hardware control
- Maintaining comfort settings
- Temperature control decisions

**Home Assistant is better at:**
- Cross-system integration (humidity sensors + fan control)
- Complex multi-condition automations (dryer + temperature + fan)
- Real-time response to non-thermostat sensors
- Data logging and trend analysis
- Notifications and alerts

**Result:** Ecobee handles climate control, HA uses fan for targeted ventilation based on other sensors.

---

## Ecobee Features Not Wired/Configured

### Humidifier Control

**Status:** Humidifier exists but is NOT wired to Ecobee

**Current Setup:**
- Whole-house humidifier installed on HVAC
- Powered independently (not controlled by Ecobee)
- Runs whenever furnace fan runs
- Manual humidity adjustment on humidifier unit

**Why Not Wired to Ecobee:**
- Would allow Ecobee to control humidifier on/off
- Could enable outdoor temperature-based control
- Aspiration for future but not current implementation

**Current Monitoring:**
Home Assistant monitors humidifier effectiveness via humidity sensor, but cannot control it.

### Ventilator Control

**Status:** No ventilator/HRV/ERV in system

**Ecobee Support:**
If ventilator existed, Ecobee could control fresh air intake based on schedules or CO2 levels.

---

## Accessing Ecobee Settings

**On Thermostat:**
- Touch screen interface
- Menu → Settings
- Temperature schedules
- Comfort settings (Home, Away, Sleep)
- System settings
- Accessory management

**Ecobee Mobile App:**
- iOS and Android app
- Remote temperature control
- Schedule management
- System monitoring
- Alerts and reminders
- Usage reports

**Home Assistant:**
- Settings → Devices & Services → HomeKit Device → Ecobee
- View all entities
- Check current status
- Monitor temperature/humidity
- View occupancy
- Control fan mode via scripts

---

## Troubleshooting

**Ecobee Offline in HA:**

1. **Check Network:**
   - Is Ecobee connected to WiFi? (check on thermostat)
   - Can you control via Ecobee app? (verifies cloud connection)
   - Is HA on same network/VLAN?

2. **Check HomeKit Integration:**
   - Settings → Integrations → HomeKit Device
   - Integration status OK?
   - Try reloading integration

3. **Re-pair if Needed:**
   - Remove integration
   - Regenerate HomeKit code on Ecobee
   - Re-add integration with new code

**Fan Control Not Working:**

1. **Check Script:**
   - Verify scripts in `/config/scripts.yaml`
   - Test script manually via Developer Tools
   - Check script trace for errors

2. **Check HomeKit Connection:**
   - Integration online?
   - Fan mode entity available?
   - Try controlling fan manually in HA

3. **Check Automation:**
   - Verify automation calling script correctly
   - Check automation traces
   - Verify conditions being met

**Inaccurate Temperature Readings:**

1. **Check Sensor Placement:**
   - Is thermostat in good location? (not in direct sun, drafts, etc.)
   - Remote sensor placed well in master bedroom?

2. **Calibration:**
   - Ecobee allows temperature calibration
   - Menu → Settings → Installation Settings → Thresholds

3. **Sensor Priority:**
   - Check which sensors Ecobee is using for temperature averaging
   - Menu → Settings → Installation Settings → Reminders & Alerts

**HVAC Not Responding to Temperature:**

1. **Ecobee Controls Temperature:**
   - HA only controls fan mode
   - Check Ecobee directly for temperature issues
   - Verify HVAC system power
   - Check for Ecobee alerts/error codes

2. **Common Ecobee Issues:**
   - C-wire power issue
   - HVAC breaker tripped
   - Filter reminder lockout
   - System safety lockout

---

## Demand Response Configuration

**Ecobee Built-in Feature:**
The Ecobee has a native demand response feature configured but **not enrolled** in an actual utility program.

**Configuration:**
- Window: Monday-Friday, 3-7 PM (DTE Energy peak hours)
- Configured in Ecobee settings
- Can automatically adjust temperature during peak hours
- Independent of Home Assistant

**Important:**
- System has the capability configured
- Not currently enrolled in DTE demand response program
- Would need enrollment to receive actual events
- Configuration allows easy enrollment if desired

**Home Assistant Role:**
- None - this is Ecobee native functionality
- HA basement dehumidifier separately avoids same peak hours

---

## Device Management

**Firmware Updates:**
- Automatic via Ecobee cloud
- Updates download and install overnight
- Check: Menu → Settings → About → Software version

**Adding Remote Sensors:**
1. Purchase additional Ecobee room sensors
2. Menu → Settings → Installation Settings → Manage Accessories
3. Add Accessory → Room Sensor
4. Follow pairing instructions
5. Sensor appears in Ecobee and HA automatically

**Replacing Thermostat:**
1. Take photo of existing wire connections
2. Turn off HVAC power at breaker
3. Remove old thermostat
4. Install new Ecobee per installation guide
5. Restore power
6. Complete Ecobee setup wizard
7. Enable HomeKit in settings
8. Re-pair with Home Assistant
9. Recreate fan control scripts if needed

---

## Migration Notes for New Owner

**Ecobee Account:**
- **Account email:** westcombecasslake@gmail.com (house email)
- **Transfers with house:** Like YoLink, Ring, Pushover accounts
- Account credentials provided to new owner
- All schedules and settings preserved in Ecobee account

**Home Assistant Integration:**
1. Ecobee continues to function independently
2. Ecobee app continues to work (cloud account)
3. HA integration continues to work (local HomeKit)
4. Fan control scripts continue to work
5. No reconfiguration needed unless HA is replaced

**If Replacing Home Assistant:**
1. Ecobee remains fully functional
2. Ecobee app access unchanged
3. Re-add HomeKit Device integration on new HA
4. Enter pairing code from Ecobee
5. All entities recreated automatically
6. Recreate fan control scripts from backup
7. Update automations to call scripts

**If Replacing Ecobee:**
1. Install new thermostat
2. Transfer/recreate schedules
3. Enable HomeKit on new device
4. Re-pair with Home Assistant
5. Verify fan control scripts work with new device
6. Verify humidity monitoring automation still works

**Alternative Control Options:**
- Ecobee app (iOS/Android) - Primary control method
- Ecobee web portal - Desktop access
- HomeKit integration in HA (current)
- Ecobee native HA integration (alternative to HomeKit method)
- Apple Home app (if Apple devices present)

---

## Official Resources

**Home Assistant Integration:**
- **HomeKit Device integration officially supported**
- Documentation: https://www.home-assistant.io/integrations/homekit_controller/
- GitHub: https://github.com/home-assistant/core/tree/dev/homeassistant/components/homekit_controller

**Ecobee Resources:**
- Website: https://www.ecobee.com
- Support: https://support.ecobee.com
- Installation guides: https://support.ecobee.com/s/articles/Installing-your-ecobee
- User manual: Available in support section
- Mobile app: iOS and Android

**HomeKit Protocol:**
- Apple HomeKit is local, encrypted protocol
- No Apple hardware required for HA integration
- Works over local WiFi network

---

## Important Notes

- **Ecobee handles climate control** - HA monitors and controls fan only
- **HA controls HVAC fan via scripts** for bathroom and laundry ventilation
- **Fan control enabled** through HomeKit device integration
- **Remote sensor in master bedroom** provides occupancy and temperature
- **Demand response configured** but not enrolled in utility program
- **Humidifier NOT wired to Ecobee** - cannot be controlled remotely
- **HA monitors humidity** for humidifier pad effectiveness tracking
- **Account transfers with house** - westcombecasslake@gmail.com
- **Local control** - no cloud dependency for HA integration
- **Ecobee app** remains primary interface for temperature control
- **Potential for expansion** - outdoor temp-based humidifier control (future)

---

**Last Updated:** February 2026  
**Integration Version:** Home Assistant Core (current)  
**Document Maintainer:** Homeowner
