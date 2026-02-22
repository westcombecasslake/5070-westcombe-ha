# Ring Integration

**Integration Type:** Video Doorbell & Security  
**Devices in System:** 1  
**Primary Use:** Front door monitoring  
**Status:** Active

---

## What is Ring?

Ring is a home security platform specializing in video doorbells and security cameras. The Ring Video Doorbell provides motion detection, live video streaming, two-way audio, and cloud recording of activity at the front door.

**Key Features:**
- HD video streaming (live and recorded)
- Motion detection with customizable zones
- Two-way audio communication
- Cloud video storage (subscription required)
- Mobile app notifications
- Integration with other smart home platforms

---

## Role in This System

**Ring Device Functions:**
- Front door video monitoring
- Motion detection
- Doorbell press detection
- Two-way audio communication
- Cloud video recording
- Mobile app notifications

**Home Assistant Integration (Limited):**
- **Premise lighting automation:** Motion at night triggers premise lights to turn on (`lightingpremise.yaml`)
- **Battery monitoring:** Low battery alerts via Pushover

**IMPORTANT:** Ring app handles all video, audio, doorbell, and notification features. HA only uses Ring for motion-triggered premise lighting and battery monitoring.

---

## Home Assistant Configuration

### Integration Setup

**Location:** Settings → Devices & Services → Integrations → Ring

Standard integration location - single location like YoLink and Pushover.

**Configuration Method:**
- Cloud-based integration
- Ring account authentication
- OAuth login through Ring service
- Real-time event notifications via webhook

### Hardware

**Ring Video Doorbell:**
- Model: Ring Video Doorbell 2
- Location: Front door
- Power: Battery powered
- Connection: WiFi to home network

### Setup Process

**Initial Setup:**
1. Ring doorbell installed and configured via Ring app
2. Ring account created (or existing account used)
3. Home Assistant Ring integration added
4. Login with Ring account credentials
5. Two-factor authentication (if enabled) - **Note:** 2FA uses owner's phone
6. Entities automatically created in HA

**Cloud Dependency:**
- Ring integration requires Ring cloud service
- Video streaming and recording through Ring servers
- Requires active internet connection
- HA receives events via Ring API

---

## Ring Features

### Video & Audio

**Live View:**
- On-demand video streaming
- Accessible via Ring app or HA
- HD video quality
- Night vision (infrared)

**Two-Way Audio:**
- Speak to visitors through doorbell
- Hear visitor responses
- Controlled via Ring app

**Video Recording:**
- Cloud storage of motion/doorbell events
- Requires Ring Protect subscription
- 60-day video history (varies by plan)
- Downloadable recordings

### Motion Detection

**Motion Zones:**
- Customizable detection areas
- Avoid false triggers (trees, street traffic, etc.)
- Adjustable sensitivity
- Configured in Ring app

**Motion Alerts:**
- Push notifications to phone
- Can be sent to HA for automation
- Customizable alert schedules
- Rich notifications with snapshot

**Motion Frequency:**
- Cooldown period between alerts
- Prevents notification spam
- Configurable in Ring app

### Doorbell Press

**Chime:**
- Internal mechanical/electronic chime (if wired)
- Ring Chime devices (separate purchase)
- App notifications

**Alerts:**
- Push notifications to all devices
- Available in HA as event
- Can trigger automations

---

## Integration with Home Assistant

### Entities Created

**Binary Sensors:**
- `binary_sensor.front_door_ding` - Doorbell button pressed
- `binary_sensor.front_door_motion` - Motion detected (less reliable, often unavailable)

**Event Entities:**
- `event.front_door_motion` - Motion detection events (RECOMMENDED FOR AUTOMATIONS)
  - Fires every time motion is detected
  - More reliable and captures all motion events
  - Used in premise lighting automation (`lightingpremise.yaml`)
  - May fire duplicate events (Ring sends redundant events)

**Sensors:**
- `sensor.front_door_last_motion` - Timestamp of last motion
- `sensor.front_door_last_ding` - Timestamp of last doorbell press
- `sensor.front_door_battery` - Battery level
- `sensor.front_door_wifi_signal` - WiFi signal strength

**Camera:**
- `camera.front_door` - Live video stream entity
- Can display in HA dashboard
- Can capture snapshots

**Switches:**
- `switch.front_door_siren` - Enable/disable siren (if equipped)

---

### Motion Detection: Binary Sensor vs Event Entity

Ring provides TWO motion detection entities with significantly different behavior:

**1. `binary_sensor.front_door_motion` - State-based (NOT RECOMMENDED)**
- Maintains "on" or "off" state
- Often goes "unavailable"
- Misses many motion events
- Less reliable overall
- Example activity: 1 event on Feb 15, 0 events on Feb 14

**2. `event.front_door_motion` - Event-based (RECOMMENDED)**
- Fires as an event every time motion is detected
- Much more reliable and comprehensive
- Captures all motion events
- Example activity: 9 events on Feb 15, 11 events on Feb 14
- **Used in `lightingpremise.yaml` automation**
- May fire duplicate events (Ring behavior), but automation handles this with `mode: restart`

**For all automations:** Always use `event.front_door_motion` for motion triggers.

---

### Battery Monitoring via Pushover

**Low Battery Notifications:**
- HA monitors `sensor.front_door_battery` level
- Low battery alerts sent to Pushover (Battery Alerts service)
- Integrated with system-wide battery monitoring (`batteryalerts.yaml`)
- Notifications follow same two-level warning system as YoLink devices

**Notification Levels:**
- Early warning when battery drops to moderate level
- Critical warning when battery drops to low level
- Allows time to plan battery charging before device fails

**Purpose:**
Ensures Ring doorbell battery is charged before it goes offline, maintaining front door security monitoring.

---

### Used in Automations

**IMPORTANT:** While Ring provides many features (motion detection, doorbell alerts, video, etc.), the **ONLY Home Assistant automation** using Ring is:

**Premise Lighting (`lightingpremise.yaml`):**
- Ring motion detection (`event.front_door_motion`) triggers premise lights to turn on
- Only active at night when photocell is active (dark outside)
- Only triggers after premise lights have shut off for the day
- Provides security lighting when motion detected at front door
- Uses `mode: restart` to extend timer on continued motion
- Lights stay on 8-14 minutes (randomized) after LAST motion event

**All Other Ring Features:**
The following Ring capabilities are handled by the Ring app, NOT Home Assistant:
- Doorbell press notifications → Ring app
- Motion alerts → Ring app
- Video viewing → Ring app
- Two-way audio → Ring app
- Video recording → Ring cloud service

**Battery Monitoring via HA:**
- HA monitors `sensor.front_door_battery` level
- Low battery alerts sent to Pushover (Battery Alerts service)
- Integrated with system-wide battery monitoring (`batteryalerts.yaml`)

**Summary:**
Ring integration in HA is minimal - premise lighting automation + battery monitoring. Ring app handles all other functionality.

---

## Ring App vs Home Assistant

### What Ring App Handles

**Primary Control:**
- Live video viewing
- Two-way audio communication
- Motion zone configuration
- Alert schedule settings
- Video history review
- Device settings and configuration
- Sharing access with family members

**Ring App is Primary Interface:**
Ring app provides full-featured control and is the recommended primary interface for Ring devices.

### What Home Assistant Provides

**Actual HA Integration:**
- **Motion-triggered premise lighting:** Ring motion at night turns on premise lights (only active HA automation)
- **Battery level monitoring:** Alerts via Pushover when battery low
- Dashboard display of battery level and WiFi signal

**HA Does NOT:**
- Handle doorbell press notifications (Ring app does this)
- Send motion alerts (Ring app does this)
- Display video (Ring app required)
- Enable two-way audio (Ring app required)
- Store or review video (Ring cloud service)

**Why Minimal Integration?**
Ring's native app and cloud service handle most features better than HA could. HA integration is limited to what adds value: lighting automation triggered by motion detection and battery monitoring integrated with other system sensors.

---

## Ring Protect Subscription

**Cloud Storage:**
Ring Protect subscription required for video recording and history.

**Subscription Tiers:**
- **Basic:** Single device, 180-day video history
- **Plus:** All devices at one location, extended features
- **Pro:** Multiple locations, professional monitoring option

**Without Subscription:**
- Live view still available
- Motion/doorbell alerts still work
- No video history/recording
- No snapshot capture in alerts

**Current Status:**
Ring Solo Plan (active)

---

## Accessing Ring Settings

**Ring Mobile App:**
- iOS and Android
- Live video viewing
- Motion zone configuration
- Alert schedules
- Video history
- Device settings
- Sharing and permissions

**Ring Web Portal:**
- https://ring.com
- Desktop access to cameras
- Video history review
- Account management
- Device settings

**Home Assistant:**
- Settings → Devices & Services → Ring
- View entities
- Check device status
- Battery level monitoring
- WiFi signal strength
- Cannot configure device settings

---

## Troubleshooting

**Ring Offline in HA:**

1. **Check Ring Service:**
   - Is Ring working in Ring app?
   - Check Ring service status online
   - Verify internet connectivity

2. **Check HA Integration:**
   - Settings → Integrations → Ring
   - Integration status OK?
   - Try reloading integration

3. **Re-authenticate:**
   - Remove and re-add integration
   - Login with Ring credentials
   - Complete two-factor auth if enabled

**Poor Video Quality:**

1. **Check WiFi Signal:**
   - View signal strength in Ring app
   - Move router closer or add WiFi extender
   - Check for interference

2. **Bandwidth:**
   - Verify adequate internet upload speed
   - Other devices consuming bandwidth?
   - Contact ISP if persistent issues

**Motion Detection Issues:**

1. **False Alerts:**
   - Adjust motion zones in Ring app
   - Reduce sensitivity
   - Exclude problem areas (trees, street, etc.)

2. **Missed Motion:**
   - Increase sensitivity in Ring app
   - Adjust motion zones to cover desired area
   - Check camera angle and position

3. **Missing Events in HA:**
   - Use `event.front_door_motion` instead of `binary_sensor.front_door_motion`
   - Event entity captures all motion events
   - Binary sensor often misses events or goes unavailable

**Battery Issues:**

1. **Short Battery Life:**
   - Cold weather reduces battery life significantly
   - High motion activity drains battery faster
   - Frequent live viewing drains battery
   - Consider hardwiring if frequent charging needed

2. **Not Receiving Battery Alerts:**
   - Check HA battery monitoring automation
   - Verify Pushover integration working
   - Test notification manually via Developer Tools
   - Check `sensor.front_door_battery` updating

**Doorbell Not Triggering:**

1. **Check Power:**
   - Battery charged?
   - Check battery level in Ring app or HA
   - Remove and charge battery if low

2. **Check Button:**
   - Test button physically
   - Listen for mechanical click
   - Contact Ring support if button defective

---

## Battery Management

**Battery Powered Operation:**
Ring Video Doorbell operates on rechargeable battery.

**Battery Life Factors:**
- Motion frequency (high activity = shorter life)
- Live view usage (streaming drains battery)
- Temperature (cold reduces capacity)
- WiFi signal strength (weak signal = more power)
- Typical life: 1-6 months depending on usage

**Charging Process:**
1. Remove doorbell from mount (press release tab)
2. Bring inside to charge
3. Use included micro-USB cable
4. Charge until LED indicates full (several hours)
5. Reinstall in mount

**Home Assistant Monitoring:**
- Battery level continuously monitored via `sensor.front_door_battery`
- Pushover alerts sent at two warning levels (via Battery Alerts service)
- Early warning allows planned charging
- Critical warning indicates immediate attention needed

**Best Practices:**
- Charge before winter (cold reduces battery life)
- Keep spare charged battery for quick swap
- Monitor battery level in HA dashboard
- Respond to Pushover battery alerts promptly

---

## Device Management

**Firmware Updates:**
- Automatic via Ring cloud
- Updates applied overnight
- Check in Ring app: Device Settings → Device Health → Firmware

**Cleaning:**
- Wipe lens with soft, dry cloth
- Do not use cleaners or water
- Keep lens clear for best video quality

**Replacing Device:**
1. Remove old device from mount
2. Install new Ring doorbell per instructions
3. Add to Ring account via Ring app
4. HA integration automatically discovers new device
5. Update device name if needed
6. Verify battery monitoring automation working

---

## Migration Notes for New Owner

**Ring Account:**
- **Account email:** westcombecasslake@gmail.com (house email)
- **Transfers with house:** Like YoLink, Pushover, Ecobee accounts
- Account credentials provided to new owner
- Video history transfers with account

**Ring Protect Subscription:**
- Subscription tied to Ring account
- Transfer subscription to new owner or cancel
- New owner can purchase own subscription
- Subscription details in Ring account settings

**Home Assistant Integration:**
1. Ring doorbell continues to function via Ring app
2. HA integration continues if Ring credentials unchanged
3. Battery monitoring continues with Pushover alerts
4. New owner can update Ring account details in HA

**If Replacing Home Assistant:**
1. Ring remains fully functional via Ring app
2. Re-add Ring integration on new HA
3. Login with Ring account credentials
4. All entities recreated automatically
5. Recreate battery monitoring automation from backup

**If Replacing Ring Device:**
1. Remove old device from Ring account
2. Install new device
3. Add to Ring account via Ring app
4. HA integration discovers new device automatically
5. May need to reload integration
6. Verify battery sensor entity name matches automation

**Alternative Options:**
- Continue using Ring (recommended)
- Replace with different doorbell brand
- HA supports many video doorbell integrations

---

## Official Resources

**Home Assistant Integration:**
- **Ring integration officially supported**
- Documentation: https://www.home-assistant.io/integrations/ring/
- GitHub: https://github.com/home-assistant/core/tree/dev/homeassistant/components/ring

**Ring Resources:**
- Website: https://ring.com
- Support: https://support.ring.com
- Community: https://community.ring.com
- Mobile app: iOS and Android
- Video doorbell setup: https://support.ring.com/hc/en-us/categories/360002090291-Doorbells

---

## Important Notes

- **Ring app is primary interface** - handles video, audio, doorbell, notifications
- **HA integration is minimal** - only premise lighting automation + battery monitoring
- **Single HA automation:** Motion at night triggers premise lights (`lightingpremise.yaml`)
- **Event entity recommended:** Use `event.front_door_motion` for all motion automations
- **Binary sensor unreliable:** `binary_sensor.front_door_motion` often unavailable, misses events
- **Cloud-based service** - requires internet connection
- **Ring Protect subscription** - required for video recording/history (handled by Ring app)
- **Account transfers with house** - westcombecasslake@gmail.com
- **Battery powered** - requires periodic charging (1-6 months typical)
- **Battery monitoring via HA** - Pushover alerts sent at two warning levels
- **Motion detection** - triggers both Ring app alerts AND HA premise lighting
- **Duplicate events normal** - Ring sends redundant motion events, automation handles with `mode: restart`
- **Two-way audio** - Ring app only
- **Cold weather** - significantly reduces battery life
- **WiFi signal critical** - ensure strong signal at front door
- **Privacy considerations** - video recording active when motion detected (Ring cloud)
- **HA does NOT handle:** doorbell notifications, video, audio, motion alerts (all Ring app)

---

**Last Updated:** February 2026  
**Integration Version:** Home Assistant Core (current)  
**Document Maintainer:** Homeowner
