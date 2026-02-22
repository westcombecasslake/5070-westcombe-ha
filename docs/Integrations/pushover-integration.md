# Pushover Integration

**Integration Type:** Notification Service  
**Devices in System:** 4 notification services  
**Primary Use:** Multi-tier alert and logging system  
**Status:** Active

---

## What is Pushover?

Pushover is a real-time notification service that delivers Home Assistant alerts to smartphones, tablets, and desktop computers. Unlike email or SMS, Pushover provides rich notifications with custom sounds, priorities, and delivery options optimized for home automation alerts.

**Key Features:**
- Real-time push notifications to iOS, Android, and desktop
- Custom notification priorities and sounds
- Rich notification content with formatting
- One-time purchase (no monthly fees)
- Reliable cloud-based delivery
- Multiple device support per account
- No rate limits for personal use
- **Apps (Buckets):** Organize notifications into separate streams for easy review

---

## Role in This System

**Primary Functions:**
- Four-tier notification system (battery alerts, house alerts, notices, operational logs)
- Safety system notifications (smoke/CO, water leaks, sump pump)
- Environmental monitoring alerts (humidity, temperature)
- Appliance status notifications (dryer completion, irrigation)
- System health and maintenance reminders
- Battery low warnings for all sensors

**Notification Services Configured:**
- **Battery Alerts** - Dedicated service for all battery-related warnings
- **House Alerts** (High priority) - Critical issues requiring immediate attention
- **House Notices** (Normal priority) - Non-urgent issues requiring awareness
- **Operational Log** (Low priority) - Silent daily summaries and routine events

---

## Pushover Apps Feature

### What Are Apps (Buckets)?

Pushover allows notifications to be organized into separate "apps" (buckets). Each Home Assistant notification service is configured as a separate Pushover app, creating distinct notification streams.

**Benefits:**
- **Organized Review:** View notifications by category (battery, alerts, notices, logs)
- **Temporal View:** Can also view all notifications chronologically across all apps
- **Selective Management:** Clear or manage notifications by app
- **Custom Settings:** Different sounds, icons, and priority per app
- **Easy Filtering:** Quickly find specific notification types

**In This System:**
Each of the four notification tiers is a separate Pushover app:
1. Battery Alerts app
2. House Alerts app
3. House Notices app
4. Operational Log app

**Reviewing Notifications:**
- **By App:** View all Battery Alerts, all House Alerts, etc.
- **Chronologically:** View all notifications across all apps in time order
- **Switch Views:** Toggle between app view and chronological view in Pushover app

---

## Home Assistant Configuration

### Integration Setup

**Location:** Settings → Devices & Services → Integrations → Pushover

Standard integration location - no special dual-location like Insteon.

**Configuration Method:**
- Cloud-based service
- API key authentication per app
- User key(s) for device targeting
- Multiple service configurations for priority levels

### Required Setup

**Pushover Account:**
- One-time purchase: ~$5 per platform (iOS, Android)
- Account at pushover.net
- **Account Email:** westcombecasslake@gmail.com (house email, not owner's personal email)
- User Key from account dashboard
- API Token/Key per application

**Home Assistant Configuration:**
Four separate Pushover notification services configured in `/config/configuration.yaml`:

```yaml
notify:
  - name: battery_alerts
    platform: pushover
    api_key: !secret pushover_battery_api_key
    user_key: !secret pushover_user_key
    
  - name: house_alerts
    platform: pushover
    api_key: !secret pushover_alerts_api_key
    user_key: !secret pushover_user_key
    priority: 1   # High priority
    
  - name: house_notices
    platform: pushover
    api_key: !secret pushover_notices_api_key
    user_key: !secret pushover_user_key
    priority: 0   # Normal priority
    
  - name: operational_log
    platform: pushover
    api_key: !secret pushover_log_api_key
    user_key: !secret pushover_user_key
    priority: -2  # Silent notification
```

---

## Four-Tier Notification System

### Battery Alerts

**Service:** `notify.battery_alerts`

**Characteristics:**
- **Priority:** Normal (0)
- **Sound:** Standard notification tone
- **Delivery:** Standard notification
- **Purpose:** 30% early warnings — plan battery replacement without urgency

**Two-Level Warning System:**

All YoLink battery devices send two levels of low battery notifications:

**First Warning (~30% battery) → `notify.battery_alerts`:**
- Priority: Normal
- Message: Early warning to order replacement batteries
- Time to replace: Several weeks to months

**Critical Warning (~15% battery) → `notify.house_alerts`:**
- Priority: High (same channel as smoke/CO/leak alerts)
- Message: Replace immediately
- Time to replace: Days to weeks
- Rationale: A dying smoke detector or leak sensor battery is a safety gap — treated with same urgency as an actual event

**Example Notifications:**
```
Title: 🔋 Low Battery Warning
Message: Device: Garage Motion Sensor
         Location: Garage wall facing out
         Battery Level: 28%
         
         Order replacement battery
```

```
Title: 🔋 CRITICAL Battery Alert
Message: Device: South Bedroom Smoke Detector
         Location: South Bedroom ceiling
         Battery Level: 12%
         
         REPLACE IMMEDIATELY
```

**Purpose:** Separate bucket for all battery notifications makes it easy to track which devices need attention without mixing with other alerts.

---

### House Alerts (High Priority)

**Service:** `notify.house_alerts`

**Characteristics:**
- **Priority:** 1 (High/Emergency)
- **Sound:** Loud alert tone (attention-getting)
- **Delivery:** Immediate notification with sound/vibration
- **Purpose:** Issues requiring prompt action

**Use Cases:**
- Smoke detected (any detector)
- Carbon monoxide detected (any detector)
- Water leak detected (any location)
- Sump pump failure during rain event
- Photocell sensor failure (backup activation)
- Motion detection at night (when away)
- Humidity critical low (pad replacement needed)

**Example Notifications:**
```
Title: 💧 WATER LEAK DETECTED
Message: Location: Hot Water Heater
         Time: 3:47 PM
         
         ACTION REQUIRED:
         1. Locate source
         2. Shut off water if needed
         3. Address leak
         4. Manually silence siren
```

```
Title: ⚠️ Sump Pump Alert
Message: No pump activity in 36 hours
         Despite 1.23" rainfall. Check pump!
         
         Sump Water Depth: HIGH!
         Floor: Dry
```

```
Title: Basement Smoke CO
Message: SMOKE DETECTED
         Check location immediately
```

```
Title: Front Motion at Night
Message: 9:47 PM
```

**Response Time:** Immediate (minutes)

---

### House Notices (Normal Priority)

**Service:** `notify.house_notices`

**Characteristics:**
- **Priority:** 0 (Normal)
- **Sound:** Default notification tone
- **Delivery:** Standard notification with sound
- **Purpose:** Issues requiring awareness but not urgency

**Use Cases:**
- Smoke/CO detector offline notifications
- Extended sump pump dry period (no rain)
- Humidity declining trend warning
- Maintenance reminders
- Device back online after offline period

**Example Notifications:**
```
Title: Smoke/CO Detector Offline
Message: South Bedroom Smoke Alarm
         Status: Offline (10+ min)
         Check detector and battery
```

```
Title: ℹ️ Sump Pump Notice
Message: No pump activity in 104 hours
         No recent rain - likely normal
         
         Sump Water Depth: Normal | Floor: Dry
```

```
Title: Humidity Pad Trend Warning
Message: 14-day average declining
         Pad may need replacement soon
         Current: 38% | Baseline: 42%
```

**Response Time:** Address within 24-48 hours

---

### Operational Log (Low Priority)

**Service:** `notify.operational_log`

**Characteristics:**
- **Priority:** -2 (Silent)
- **Sound:** None
- **Delivery:** No audible alert, notification appears in tray
- **Purpose:** Daily summaries, routine status updates, informational logs

**Use Cases:**
- Sump pump daily statistics (11:55 PM)
- Premise lights dusk trigger confirmation
- Irrigation system activation confirmation
- HVAC humidifier adjustments
- System startup completion
- Routine automation confirmations

**Example Notifications:**
```
Title: Sump Pump Daily
Message: Cycles 42 | Average Cycle 7.5 seconds
         Runtime 5.3 minutes | 71.4 Gallons
         Gallons this month: 1,247.8
         
         Sump Water Depth: Normal | Floor: Dry
```

```
Title: Premise Lights - Dusk Triggered
Message: Dusk triggered at 6:43 PM
         Season: Winter | Delay: 10 min
         Lights on at: 6:53 PM
         Actual dusk: 6:48 PM
```

**Review Frequency:** Daily or weekly batch review

---

## Notification Content Standards

### Information Always Included

Every notification follows a consistent format:

1. **Descriptive Title** - Clear event description
2. **Current Values** - Relevant sensor states
3. **Context** - Why it matters
4. **Timestamp** - When it occurred (if relevant)
5. **Related States** - Supporting data
6. **Action Required** - What to do (for alerts)

### Emoji Usage

Strategic use of emoji in titles for quick visual recognition:

- ⚠️ Warning/Alert
- 🔥 Fire/Smoke
- ☠️ Carbon Monoxide
- 💧 Water/Leak
- ✅ Normal/OK
- ℹ️ Information
- 🔋 Battery
- ✔ Resumed/Recovered

---

## Pushover App Configuration

### Per-Service Customization

The Pushover mobile app allows customization per notification priority level and per app:

**Battery Alerts:**
- Sound: Standard tone
- Vibration: Standard
- Quiet Hours: Optional
- Notification Display: Standard
- Review: Check daily or weekly

**House Alerts (Priority 1):**
- Sound: Loud/Siren (loudest available)
- Vibration: Persistent
- LED: Attention-getting color
- Quiet Hours: NONE (must wake for emergencies)
- Notification Display: Full screen
- Retry: Enable (repeat until acknowledged)
- Emergency Override: Enable

**House Notices (Priority 0):**
- Sound: Default or gentle tone
- Vibration: Standard
- LED: Optional
- Quiet Hours: Optional (10 PM - 7 AM)
- Notification Display: Standard

**Operational Log (Priority -2):**
- Sound: Silent
- Vibration: Off
- LED: Optional
- Quiet Hours: All night (review in morning)
- Notification Display: Minimize

### Device Targeting

Pushover supports multiple devices per account:
- Target specific devices per service
- Send Operational Log to phone only
- Send House Alerts to all devices (phone, tablet, watch, desktop)

---

## Integration with Home Assistant

### Service Calls in Automations

**Basic Notification:**
```yaml
- service: notify.house_alerts
  data:
    title: "Alert Title"
    message: "Alert message content"
```

**Battery Alert:**
```yaml
- service: notify.battery_alerts
  data:
    title: "🔋 Low Battery Warning"
    message: |
      Device: {{ device_name }}
      Battery Level: {{ battery_level }}%
      
      Order replacement battery
```

**Rich Notification with Template:**
```yaml
- service: notify.operational_log
  data:
    title: "Sump Pump Daily"
    message: |
      Cycles {{ states('sensor.sump_pump_daily_cycles') }}
      Gallons {{ states('sensor.sump_pump_daily_gallons') }}
```

**Conditional Notification:**
```yaml
- choose:
    - conditions:
        - condition: state
          entity_id: light.front_porch_override
          state: "off"
      sequence:
        - service: notify.house_alerts
          data:
            title: "Front Motion at Night"
            message: "{{ now().strftime('%I:%M %p') }}"
  default: []
```

### Used Throughout System

**Safety Systems:**
- `smokeco.yaml` - Detector alerts and offline notifications
- `basementleaksensors.yaml` - Water leak alerts
- `sumppump.yaml` - Failure alerts and daily summaries

**Environmental:**
- `hvachumiditypad.yaml` - Pad replacement and trend alerts
- `basementdehumidifiercontrol.yaml` - Humidity status
- `fridgefreezer.yaml` - Appliance temperature failures and recovery

**Lighting:**
- `lightingpremise.yaml` - Dusk trigger logs, photocell failure
- Motion detection alerts

**Utilities:**
- `irrigation.yaml` - System activation confirmations
- `garage.yaml` - Freezing temperature alerts

**Infrastructure:**
- `batteryalerts.yaml` - Low battery warnings (Battery Alerts service)
- `systemstartup.yaml` - Startup completion
- `systemhealth.yaml` - System health alerts

---

## Troubleshooting

**Not Receiving Notifications:**

1. **Check Pushover App:**
   - App installed and logged in?
   - Device registered in Pushover account?
   - Check app notification settings
   - Verify app has notification permissions (iOS/Android settings)

2. **Check HA Integration:**
   - Settings → Integrations → Pushover configured?
   - API keys (one per app) and User key correct?
   - Test notification via Developer Tools

3. **Check Pushover Service:**
   - Visit pushover.net and check service status
   - Monthly message quota not exceeded (unlikely for personal use)
   - Account subscription active

**Test Notification:**
```
Developer Tools → Services
Service: notify.house_alerts
Data: { "title": "Test", "message": "Test notification" }
Call Service
```

**Too Many Notifications:**

1. **Review by App:**
   - Check which app (bucket) has excessive notifications
   - Battery Alerts too frequent? Adjust thresholds
   - Operational Log cluttered? Reduce routine messages

2. **Adjust Priorities:**
   - Some House Alerts could be House Notices
   - Some House Notices could be Operational Log
   - Review notification triggers in automations

3. **Use Quiet Hours:**
   - Configure in Pushover app per service
   - Operational Log: Silence all night
   - House Notices: Optional 10 PM - 7 AM
   - House Alerts: Never silence (safety)
   - Battery Alerts: Optional quiet hours

4. **Batch Similar Events:**
   - Combine multiple sensor updates into single notification
   - Use daily summaries instead of real-time for non-critical info

**Missing Critical Notifications:**

1. **Check Priority Settings:**
   - Verify House Alerts use priority: 1
   - Verify no quiet hours on alerts
   - Test with emergency notification

2. **Check Automation:**
   - Review automation traces
   - Verify conditions being met
   - Check if automation disabled

3. **Pushover Emergency Override:**
   - Enable for critical life-safety notifications
   - Bypasses Do Not Disturb modes

---

## Costs and Licensing

**Purchase:**
- One-time purchase: ~$5 per platform
- iOS: Purchase in App Store
- Android: Purchase in Google Play
- Desktop: Free after mobile purchase

**No Recurring Fees:**
- No monthly subscription
- No per-message fees
- No rate limits for personal use
- Purchase once, use forever

**Family Sharing:**
- Separate accounts required per user
- Cannot share single account across family
- Each user needs own purchase

---

## Migration Notes for New Owner

**Account Ownership:**
- **Account email:** westcombecasslake@gmail.com (house email address, not owner's personal email)
- **Transfers with house:** Like YoLink, Ring, and Ecobee accounts
- Account credentials will be provided to new owner
- No need to create new Pushover account

**Account Transfer Process:**
1. Change account email to new owner's house email (if desired)
2. Update password
3. Verify all devices registered
4. Test notifications from all four services
5. Review and adjust app settings per preferences

**Alternative Notification Services:**
If new owner prefers different service:
- Home Assistant supports many notification platforms
- Can use mobile app notifications (free)
- Can use email notifications
- Can use Telegram, Discord, etc.
- Will need to update all automation YAML files

**Maintaining Four-Tier System:**
- The four-tier structure is independent of Pushover
- Can replicate with other notification services
- Separate services/apps for Battery, Alerts, Notices, Logs
- Maintains organizational benefits even with different platform

---

## Official Resources

**Pushover:**
- Website: https://pushover.net
- API Documentation: https://pushover.net/api
- FAQ: https://pushover.net/faq
- Support: support@pushover.net

**Home Assistant Integration:**
- Documentation: https://www.home-assistant.io/integrations/pushover/
- GitHub: https://github.com/home-assistant/core/tree/dev/homeassistant/components/pushover

---

## Important Notes

- **Four-tier system:** Battery Alerts, House Alerts, House Notices, Operational Log
- **Apps (Buckets):** Each tier is separate Pushover app for organized review
- **View options:** Review by app or chronologically across all apps
- **One-time purchase per platform** - no recurring fees
- **Critical for system operation** - all alerts depend on Pushover
- **Four-tier prevents alert fatigue** while ensuring critical alerts noticed
- **Two-level battery warnings** - advance notice for all YoLink devices
- **Configure quiet hours carefully** - never silence House Alerts
- **Test regularly** - verify notifications reaching phone
- **Requires internet** - cloud-based service
- **Reliable delivery** - industry-standard for home automation notifications
- **Account transfers with house** - like YoLink, Ring, Ecobee

---

**Last Updated:** February 2026  
**Integration Version:** Home Assistant Core (current)  
**Document Maintainer:** Homeowner
