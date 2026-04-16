# YoLink Integration

**Integration Type:** Wireless Sensor Network  
**Devices in System:** 25  
**Primary Use:** Safety sensors, environmental monitoring  
**Status:** Active

---

## What is YoLink?

YoLink is a wireless sensor platform using LoRa (Long Range) RF technology to create a reliable, long-range sensor network. YoLink devices communicate through a central hub that connects to Home Assistant, providing extensive coverage throughout the home with minimal infrastructure.

**Key Technology Features:**
- LoRa long-range RF communication (up to 1/4 mile line-of-sight)
- Low power consumption - battery devices last years
- Sub-GHz frequency (915 MHz in US) - better wall penetration than 2.4 GHz
- Hub-based architecture - all sensors connect through YoLink Hub
- Cloud integration with local control capabilities

---

## Role in This System

**Primary Functions:**
- Comprehensive 8-zone smoke and carbon monoxide detection
- Professional fire alarm monitoring (VirtualArm subscription)
- Environmental monitoring (temperature and humidity tracking)
- Sump pump power monitoring
- Laundry dryer power monitoring
- Whole-house alarm siren

**Device Breakdown:**
- **Smoke/CO Combo (YS7A01-UC):** 3 devices - Dual sensor units (one per floor)
- **Smoke Only (YS7A02-UC):** 5 devices - Smoke detection
- **Siren (YS7103-UC):** 1 device - Whole-house alarm
- **Temperature/Humidity Sensors (YS8003):** 6 devices - Environmental monitoring
- **Smart Plug with Power Monitoring (YS6602-UC):** 1 device - Dryer monitoring
- **Outdoor Sensors:** 2 devices - Sump pump and depth monitoring
- **Additional devices:** ~7 devices (leak sensors, door sensors, etc.)

---

## Home Assistant Configuration

### Integration Setup

**Location:** Settings → Devices & Services → Integrations → YoLink

Unlike Insteon, YoLink appears in a single location in Home Assistant settings - the standard integrations area.

**Configuration Method:**
- Cloud-based integration with local API fallback
- OAuth authentication through YoLink cloud service
- Real-time sensor updates via cloud push notifications
- Local network discovery for hub communication

### Hardware Required

**YoLink Hub:**
- Model: YS1604-UC (Hub 2)
- Serial Number: 78B6B7630E
- Connection: WiFi only (no Ethernet port)
- Power: USB power adapter
- Location: Not critical - LoRa range covers entire property from any location

**Network Flexibility:**
Due to hub login authentication, the YoLink hub can be placed on a separate LAN segment from Home Assistant without connectivity issues.

### Configuration Files

- Integration: Settings → Devices & Services → YoLink
- Device configuration stored in Home Assistant
- No local device JSON files (cloud-managed)

**Setup:**
1. YoLink Hub 2 connected to home network via WiFi
2. Hub registered with YoLink cloud service
3. Devices paired to hub via YoLink app
4. Home Assistant YoLink integration added via OAuth
5. All devices automatically discovered in HA
6. Hub can be on separate LAN segment from HA due to cloud authentication

---

## Professional Monitoring

### VirtualArm Fire Monitoring Service

**IMPORTANT:** This system includes professional fire alarm monitoring through YoLink/VirtualArm subscription.

**Service Features:**
- 24/7 professional monitoring of all smoke/CO detectors
- Automatic emergency dispatch when alarm triggers
- Redundant notification (monitoring service + local HA alerts)
- Monthly subscription service through YoLink

**How It Works:**
1. Smoke/CO detector triggers
2. YoLink cloud receives alarm signal
3. VirtualArm monitoring center notified
4. Monitoring center attempts contact
5. Emergency services dispatched if needed
6. Simultaneously, HA sends local notifications

**Subscription Management:**
- Managed through YoLink account
- VirtualArm console: https://console.virtualarm.com/login
- Product information: https://virtualarm.com/index.php/yolink-pro/
- Contact: YoLink support for service changes
- Critical: Maintain active subscription for monitoring

---

## Device Types and Uses

### Safety Devices

**Smoke/CO Combo Units (YS7A01-UC):**
- Locations: One per floor (Basement, Main Floor, Upper Floor) - per fire code recommendations
- Dual photoelectric smoke + electrochemical CO sensors
- Battery powered (long life)
- Integrated with whole-house siren
- Professional monitoring enabled
- **Special Battery Note:** 10-year non-replaceable sensor head with 2 replaceable batteries in YoLink LoRa base (batteries last several years)

**Smoke-Only Units (YS7A02-UC):**
- Locations: South Bedroom, Den, Garage, Office, East Suite
- Photoelectric smoke detection
- Battery powered
- Integrated with whole-house siren
- Professional monitoring enabled
- **Special Battery Note:** 10-year non-replaceable sensor head with 2 replaceable batteries in YoLink LoRa base (batteries last several years)

**Siren (YS7103-UC):**
- Location: Middle bathroom
- 95dB alarm output
- Triggered by any smoke/CO detection
- Also used for water leak alarms (Scene 22)
- Battery backup + USB power

### Environmental Monitoring

**Temperature/Humidity Sensors (YS8003):**
- **Basement:** Humidity monitoring for dehumidifier and humidifier systems
- **Master Bathroom:** Fan automation trigger
- **Laundry Room:** Fan control and dryer monitoring
- **Additional locations:** Environmental tracking

**Features:**
- Temperature range: -4°F to 140°F
- Humidity range: 0-100% RH
- Update frequency: Configurable (typically every few minutes)
- Battery life: 2-3 years on single battery

### Power Monitoring

**Smart Plug (YS6602-UC):**
- **Dryer Monitor:** Tracks dryer power consumption for cycle completion

**Capabilities:**
- Real-time power consumption (watts)
- Energy usage tracking
- Remote on/off control
- Programmable schedules

### Sump Pump Monitoring

**YoLink Outdoor Sensors:**
- **Sump Pump Activity:** YoLink outdoor sensor monitors pump operation
- **Sump Depth:** YoLink outdoor sensor monitors water level
- Provides reliable monitoring in high-humidity environment
- Battery powered with long life in outdoor-rated enclosure

---

## LoRa Technology Benefits

**Long Range:**
- Up to 1/4 mile line-of-sight
- Excellent building penetration (sub-GHz frequency)
- Reliable communication through multiple walls/floors
- Hub centrally located covers entire property

**Low Power:**
- Battery devices last 2-5+ years
- No frequent battery replacements
- Low maintenance sensor network
- **All battery devices send 2 levels of low battery notifications to Pushover for phone alerts**

**Reliability:**
- Less susceptible to 2.4 GHz interference (WiFi, Bluetooth, microwaves)
- Robust protocol with error correction
- Automatic retransmission on failed packets
- Device-to-hub communication only (no mesh complexity)

---

## Key Features Used

### Smoke/CO Detection with Professional Monitoring

**8-Zone Coverage:**
All sleeping areas, living areas, and utility spaces covered per fire code requirements.

**Dual Notification:**
- Professional monitoring (VirtualArm) → Emergency dispatch
- Home Assistant alerts (Pushover) → Homeowner notification
- Local siren → Audible whole-house alarm

**Offline Monitoring:**
Home Assistant tracks all detector online status. If any detector goes offline for 10+ minutes, notification sent to address battery or connectivity issues.

### Environmental Automation Integration

**HVAC Humidifier Pad Monitoring:**
Basement humidity sensor tracks humidifier effectiveness and sends alerts when pad needs replacement.

**Dehumidifier Control:**
Basement humidity sensor triggers dehumidifier on/off with peak-hour avoidance.

**Bathroom/Laundry Fans:**
Humidity sensors trigger automatic fan operation to remove excess moisture.

### Appliance Monitoring

**Dryer Cycle Completion:**
Smart plug monitors dryer power consumption. When power drops after cycle, triggers laundry room fan to clear residual heat/humidity.

**Sump Pump Monitoring:**
YoLink outdoor sensors monitor pump activity and water depth, providing comprehensive failure detection and daily performance tracking.

---

## Accessing YoLink Settings

**In Home Assistant:**
Settings → Devices & Services → Integrations → YoLink
- View all 25 devices
- Check device status
- View entities (sensors, binary sensors)
- Battery levels
- Signal strength

**In YoLink App:**
- Device pairing and management
- Firmware updates
- Alert configuration
- VirtualArm monitoring service settings
- Hub status and diagnostics

**Most Common Tasks:**
- **View devices/status:** Use Home Assistant interface
- **Add new device:** Use YoLink app to pair, appears in HA automatically
- **Check battery levels:** HA Devices page or YoLink app
- **Update firmware:** YoLink app (automatic or manual)
- **Manage monitoring service:** YoLink account portal

---

## Troubleshooting

**Common Issues:**

**Device Offline:**
1. Check battery level (if battery-powered)
2. Check distance from hub (may be out of range)
3. Check hub status (power, network connection)
4. Try power cycling the hub
5. Check for physical obstructions (metal, concrete)

**Delayed Updates:**
1. Verify hub network connection
2. Check internet connectivity (cloud integration)
3. Restart YoLink integration in HA
4. Check YoLink cloud service status
5. Verify device update interval settings

**Detector False Alarms:**
1. Clean detector sensor chamber (dust, insects)
2. Check for nearby sources (cooking smoke, steam)
3. Verify proper placement (not near bathrooms, kitchens)
4. Replace if detector age >10 years
5. Test detector manually to verify function

**Hub Communication:**
1. Verify WiFi connection (signal strength)
2. Check router for hub IP assignment
3. Verify internet connectivity
4. Power cycle hub (unplug 30 seconds)
5. Check HA integration status
6. Hub can function on separate LAN segment

**Professional Monitoring Issues:**
1. Verify VirtualArm subscription active
2. Check YoLink account status
3. Test alarm to verify monitoring response
4. Contact YoLink support for service issues

---

## Device Management

**Adding New Devices:**
1. Open YoLink mobile app
2. Tap "Add Device"
3. Select device type
4. Follow pairing instructions (usually hold button)
5. Device appears in YoLink app
6. Restart YoLink integration in HA (if needed)
7. Device automatically appears in HA

**Removing Devices:**
1. Remove from YoLink app
2. Device automatically removed from HA
3. Factory reset device if reusing/selling

**Firmware Updates:**
- Managed through YoLink app
- Over-the-air (OTA) updates
- Devices update automatically when available
- Can manually check for updates in app

**Battery Replacement:**
- HA shows battery percentage for all devices
- Replace when battery drops below 15%
- **All battery devices send 2 levels of low battery notifications to Pushover**
- **Smoke/CO Detectors:** 10-year non-replaceable sensor head; YoLink LoRa base has 2 replaceable batteries lasting several years
- Temp/humidity sensors: Typically 2-3 years
- Siren: Has battery backup + USB power

---

## Migration Notes for New Owner

**Understanding the System:**
1. **YoLink Hub** - Central communication point for all sensors
2. **YoLink Cloud** - Manages devices and provides HA integration
3. **VirtualArm Monitoring** - Separate subscription service for fire monitoring
4. **Home Assistant** - Automation and notification platform

**If Replacing Home Assistant:**
1. YoLink devices continue to work (hub-based)
2. VirtualArm monitoring continues (independent service)
3. Install YoLink integration on new HA instance
4. OAuth authentication to YoLink cloud
5. All devices automatically rediscovered
6. Recreate automations from backups

**If Replacing YoLink Hub:**
1. All devices must be re-paired to new hub
2. VirtualArm monitoring subscription transfers to new hub
3. Contact YoLink support for hub replacement procedure
4. Reconnect HA integration after hub replacement

**Important Service Continuity:**
- **VirtualArm monitoring:** Transfer subscription to new owner or cancel
- **YoLink account:** Transfer ownership or create new account
- **Device pairing:** All sensors must be re-paired if changing accounts

**Alternative Monitoring Options:**
- YoLink/VirtualArm (current) - Professional monitoring
- Local-only monitoring - HA notifications without professional service
- Other professional services - Some may integrate with YoLink devices

---

## Integration with Home Assistant

### Entity Types Created

**Binary Sensors:**
- `binary_sensor.*_smoke` - Smoke detection state
- `binary_sensor.*_carbon_monoxide` - CO detection state
- `binary_sensor.*_motion` - Motion detection (if applicable)
- `binary_sensor.*_door` - Door/contact sensors

**Sensors:**
- `sensor.*_temperature` - Temperature readings
- `sensor.*_humidity` - Humidity readings
- `sensor.*_battery` - Battery percentage
- `sensor.*_power` - Power consumption (smart plugs)

**Switches:**
- `switch.*` - Smart plug on/off control
- `switch.*_siren` - Siren control

**Device Attributes:**
- Battery level
- Signal strength (RSSI)
- Last update time
- Firmware version

### Used in Automations

**Safety Monitoring (`smokeco.yaml`):**
- All smoke/CO binary sensors for detection
- Offline monitoring for all detectors
- Status helper entities for dashboard

**Environmental Control:**
- `basementdehumidifiercontrol.yaml` - Basement humidity sensor
- `hvachumiditypad.yaml` - Basement humidity sensor
- `masterbathhumidity.yaml` - Master bath humidity sensor
- `laundry.yaml` - Laundry room temp/humidity sensors

**Appliance Monitoring:**
- `laundry.yaml` - Dryer smart plug power sensor
- `sumppump.yaml` - YoLink outdoor sensors for pump activity and depth
- `fridgefreezer.yaml` - All 4 fridge/freezer temp/humidity sensors

---

## Maintenance

**Monthly:**
- Test all smoke/CO detectors manually
- Review battery levels in HA
- Verify all devices online
- Check VirtualArm subscription status

**10-Year Replacement:**
- Smoke/CO detectors have 10-year sensor head lifespan (non-replaceable)
- YoLink LoRa base batteries replaceable (last several years)
- All detectors installed: March 2024
- Replace sensor heads by: March 2034
- Mark calendar for replacement reminders

---

## Official Resources

**Home Assistant Integration:**
- **Officially supported HA integration**
- Documentation: https://www.home-assistant.io/integrations/yolink/
- GitHub: https://github.com/home-assistant/core/tree/dev/homeassistant/components/yolink

**YoLink Resources:**
- Support: https://www.yosmart.com/support
- Device manuals: Available in YoLink app and website
- YoLink mobile app: iOS and Android
- VirtualArm monitoring: Managed through YoLink account

**Professional Monitoring:**
- VirtualArm service information: Via YoLink support
- Emergency contact procedures: In YoLink account settings
- Service billing: Managed through YoLink account

---

## Important Notes

- **Professional monitoring requires active VirtualArm subscription**
- **All battery devices send 2-level low battery notifications to Pushover**
- **Smoke/CO detectors:** 10-year non-replaceable sensor heads; YoLink LoRa base has replaceable batteries (several years life)
- Battery life: 2-5+ years depending on device type
- Hub requires WiFi connection (no Ethernet port on Hub 2)
- Hub can be on separate LAN segment from Home Assistant
- LoRa range: Excellent building penetration, up to 1/4 mile
- All devices require YoLink hub - no direct HA connection
- Cloud integration - requires internet for full functionality
- Local control available for some functions via hub
- Smoke/CO sensor head replacement: March 2034 (10-year schedule)

---

**Last Updated:** February 2026  
**Integration Version:** Home Assistant Core (current)  
**Document Maintainer:** Homeowner
