# Washer & Dryer Completion Notifications

**Last Updated:** February 2026

---

## Overview

The Washer & Dryer Completion Notification subsystem sends Pushover house notices when laundry cycles complete. The system monitors power consumption of both appliances and sends notifications when cycles finish, allowing timely retrieval of laundry to prevent wrinkles and musty odors.

---

## What It Does

Monitors washer and dryer power consumption and sends notifications when cycles complete. Uses a 2-minute confirmation delay to prevent false alerts from brief pauses during cycles.

**Rationale:** Prompt notification when laundry cycles complete allows timely transfer from washer to dryer and prevents clothes from sitting in the dryer, reducing wrinkles and energy waste. The 2-minute delay ensures the appliance has actually completed its cycle rather than just pausing temporarily during operation.

### Hardware

**Washer Power Monitor:**
- Entity: `sensor.washer_monitor_current_power`
- Monitors: Washer power draw
- Thresholds: >10W = running, <5W = off

**Dryer Power Monitor:**
- Entity: `sensor.dryer_monitor_current_power`
- Monitors: Dryer power draw
- Thresholds: >200W = running, <2W = off

### Key Entities

**Sensors:**
- `sensor.washer_monitor_current_power` - Current washer power draw
- `sensor.dryer_monitor_current_power` - Current dryer power draw

**Helpers:**
- `input_boolean.washer_running` - Set true when washer exceeds 10W, cleared on completion
- `input_boolean.dryer_running` - Set true when dryer exceeds 200W, cleared on completion

**Notification Service:**
- `notify.house_notices` - Pushover normal priority notifications

**User Controls:**
- None (fully automatic)

### Automations

**Washer Running Flag:**
- `washer_running_set`
- Trigger: Power exceeds 10W
- Action: Sets `input_boolean.washer_running` on

**Washer Cycle Complete:**
- `washer_cycle_complete_notification`
- Trigger: Power drops below 5W for 2 minutes
- Condition: `input_boolean.washer_running` is on
- Action: Sends notification, clears running flag

**Dryer Running Flag:**
- `dryer_running_set`
- Trigger: Power exceeds 200W
- Action: Sets `input_boolean.dryer_running` on

**Dryer Cycle Complete:**
- `dryer_cycle_complete_notification`
- Trigger: Power drops below 2W for 2 minutes
- Condition: `input_boolean.dryer_running` is on
- Action: Sends notification, clears running flag

**YAML:** washerdryernotify.yaml

---

## User Controls

**Access:** None - fully automatic

**Manual Control:**
- Notifications cannot be disabled without removing/disabling the automations
- Pushover quiet hours can be configured in Pushover app settings

---

## Notifications

**Notification Service:** `notify.house_notices` (Pushover normal priority)

**Washer Complete:**
```
Title: 🧺 Washer Complete
Message: Washer cycle finished at [time]
```

**Dryer Complete:**
```
Title: 👕 Dryer Complete
Message: Dryer cycle finished at [time]
```

**Timing:** Notifications sent 2 minutes after power drops below threshold to confirm cycle completion

**Priority:** Normal (house_notices) - not urgent but requires timely attention

---

## Related Systems

**Laundry Fan Control:**
- Dryer fan control automation in `laundry.yaml` is independent
- Fan control and notifications operate separately
- Both use same dryer power sensor

**YoLink Integration:**
- Power monitors are YoLink smart plugs
- See: yolink-integration.md

**Pushover Notifications:**
- Uses house_notices tier (normal priority)
- See: pushover-integration.md

---

## Troubleshooting

**Not receiving notifications:**
- Verify `sensor.washer_monitor_current_power` is reporting correctly
- Verify `sensor.dryer_monitor_current_power` is reporting correctly
- Check Pushover `notify.house_notices` service is configured
- Test notification manually via Developer Tools

**False notifications (cycle not complete):**
- Increase the 2-minute delay in automation trigger
- Some cycles may have extended pauses that trigger notification
- Check power thresholds match actual appliance behavior

**Missing notifications (cycle completes but no alert):**
- Verify power stays below threshold for full 2 minutes
- Check if appliance has cooldown/wrinkle-prevent cycles that keep power above threshold
- Review automation condition - ensures appliance was actually running recently

**Delayed notifications:**
- Normal behavior - 2-minute confirmation delay built in
- Notification sent ~2 minutes after cycle actually ends
- Prevents false alerts from temporary power drops during cycles

**Testing:**
- Run short washer/dryer cycle
- Monitor power sensor in Developer Tools
- Verify power drops below threshold when cycle completes
- Notification should arrive 2 minutes later

---

## YAML Packages

- `/config/packages/washerdryernotify.yaml`

---

**Last Updated:** February 2026  
**Integration Version:** Home Assistant Core (current)  
**Document Maintainer:** Homeowner
