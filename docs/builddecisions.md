# Home Automation Build Decisions & Constraints

**Purpose:** Preserve critical "why" decisions to prevent re-introducing errors when modifying automations.  
**Last Updated:** February 2026

---

## 1. Insteon — Scene Commands Use Double-Send

**Decision:** Every `insteon.scene_on` / `insteon.scene_off` call is immediately repeated with a 3-second delay.

**Why:** Insteon scene broadcasts are fire-and-forget — no acknowledgment is returned. Network collisions, powerline noise, or RF interference can silently drop commands. The second send ensures activation even if the first fails.

**Timing:** 3 seconds is intentional — allows the first command's mesh broadcast and cleanup traffic to fully settle before the second attempt.

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

**⚠️ Do not remove double-sends to "clean up" code.** Insteon reliability depends on this pattern.

---

## 2. Insteon — Scene Numbers Are Decimal in YAML

**Decision:** Scene numbers in automation YAML use **decimal**, not hex.

**Why:** The HA Insteon integration requires decimal group numbers. Hex values (e.g., 0x15) will fail silently or trigger wrong scenes.

**Reference:**
- Scene 0x15 = Scene **21** (Premise Lights)
- Scene 0x1C = Scene **28** (HVAC Fans Off)
- Full hex-to-decimal chart: `insteon-scenes.md`

**⚠️ Never convert to hex in YAML — always use decimal.**

---

## 3. Insteon — N-Way Switching Is Independent of HA

**Decision:** Virtual multi-switch circuits are programmed directly into Insteon device link tables, not through HA automations.

**Why:** Physical switch-to-light control must survive HA or PLM failure. The Insteon mesh handles it at the hardware level. HA only adds scheduling, scene activation, and notifications on top.

**Consequence:** If you replace a device, its link table must be reprogrammed via the Insteon app/scene management. Removing it from HA does NOT remove the hardware links.

---

## 4. Premise Lighting — Photocell Is Primary, Sun Is Backup Only

**Decision:** Exterior lights trigger off the **photocell sensor** (`switch.photocell_micro_module`), not a sun-based schedule.

**Why:** Photocell responds to actual light conditions (clouds, shade, trees). Sun calculation assumes clear sky — too early in summer, may miss actual darkness. Seasonal delays are added after photocell trigger to further refine timing.

**Backup:** Sun-based automation fires 15 minutes after sunset *only if* photocell hasn't triggered yet. If backup fires, a **House Alert** (high priority) is sent — this indicates photocell maintenance is needed.

**⚠️ Do not switch to sun-only triggering.** The photocell + seasonal delay system was specifically built to solve seasonal variation problems.

---

## 5. Premise Lighting — Seasonal Delay Helpers

**Decision:** Four `input_number` helpers apply a per-month delay after photocell trigger before lights activate.

**Why:** Photocell sensitivity varies by season — it triggers earlier relative to actual darkness in summer than winter. Delays compensate so lights turn on when it's actually dark enough.

**Helpers:**
- `input_number.premise_summer_delay` — May, June, July
- `input_number.premise_shoulder_delay` — April, August
- `input_number.premise_spring_fall_delay` — March, September
- `input_number.premise_winter_delay` — October–February

**Biannual reminders** (Feb 1, Aug 1) prompt review and adjustment of these values.

---

## 6. Photocell — Daily Reset at 1:00 PM

**Decision:** Automation cycles the photocell module (off → on → off) every day at 1:00 PM.

**Why:** The analog photocell can drift or get confused — especially after overnight rain or heavy cloud cover. Daily reset clears this state and ensures reliable dusk detection.

**⚠️ Motion override automation has a guard window around 1 PM (12:58–1:01 PM) to prevent false motion triggers during the reset cycle.**

---

## 7. Notification Routing — Four Tiers, Never Mix

**Decision:** Four distinct Pushover services with hard routing rules.

| Service | Priority | Use For |
|---|---|---|
| `notify.house_alerts` | High (1) | Safety, immediate action required |
| `notify.house_notices` | Normal (0) | Awareness, act within 24–48 hrs |
| `notify.operational_log` | Silent (-2) | Routine confirmations, daily summaries |
| `notify.battery_alerts` | Normal (0) | 25% low battery warnings |

**Why:** Mixed priorities train the homeowner to ignore critical alerts. Battery warnings (30%) are isolated in `battery_alerts` so they don't pollute the alert stream. Critical battery alerts (15%) route to `house_alerts` because a dying smoke detector or leak sensor battery is a safety gap — it deserves the same urgency as an actual event.

**⚠️ Do not route 30% battery warnings to house_alerts or house_notices.**  
**⚠️ Do not route safety events to operational_log.**

---

## 8. Ecobee — Hybrid Control Model

**Decision:** Ecobee controls temperature/scheduling; HA controls only HVAC fan mode for ventilation.

**Why:** Ecobee is better at adaptive learning and HVAC scheduling. HA uses the fan-on capability (via HomeKit Device integration) only to drive bathroom and laundry ventilation when humidity thresholds are exceeded — functions Ecobee cannot perform on its own.

**HA does NOT set temperature setpoints or HVAC modes.**

**Fan control uses `device_id`, not `entity_id`:** The HomeKit Device integration does not expose a fan control entity for the Ecobee — it is only accessible at the device level. `device_id: c38113c2d02da790b689d4a3bdc4b5e1` is the only way to set fan mode from HA. Do not change to entity_id — there is no entity to use.

**`hvac_exhaust_boost` script is the single point of fan control**, called by:
- `masterbathhumidity.yaml`, `middlebathhumidity.yaml`, `eastbathhumidity.yaml` (after bathroom fan cycle)
- `laundry.yaml` (after dryer cooldown)
- `airquality.yaml` (CO2 ventilation, party mode) — highest priority

**Priority:** `airquality.yaml` wins via the `input_boolean.pm25_filtration_active` condition check — if PM2.5 filtration is active, the script skips returning fan to AUTO. `mode: restart` means the last caller resets the 20-minute timer.

---

## 9. Bathroom/Laundry Humidity — Relative Rise Detection

**Decision:** Shower/humidity detection uses relative rise from a baseline, not fixed thresholds.

**Why:** Fixed thresholds fail seasonally — 60% humidity in winter is a shower; 60% in summer may be baseline. The system captures baseline at automation start and detects the rise above it.

**Confirmation lock:** Baseline is locked at first trigger to prevent drift during the detection delay period.

**Each bathroom is a separate YAML package** with its own helpers to allow fully independent monitoring.

---

## 10. Basement Dehumidifier — Peak Hour Avoidance

**Decision:** Dehumidifier is blocked from running Mon–Fri 3–7 PM (utility peak rate hours).

**Why:** Cost savings. Basement humidity changes slowly enough that a 4-hour block doesn't meaningfully affect performance. On weekends, no restriction.

**Entity controlling outlet:** `switch.scheduled_power_save`

---

## 11. Fridge/Freezer Thresholds — Data-Driven, Not Estimated

**Decision:** Temperature alert thresholds were calibrated from actual sensor history, not food safety guidelines alone.

**Why:** Each appliance has its own normal operating range. Grocery loading creates legitimate temperature spikes that should not trigger alerts. Thresholds account for these patterns.

**⚠️ Do not "simplify" thresholds to round numbers without reviewing actual sensor data first.**

---

## 12. YoLink Battery — Single-Level Alert System

**Decision:** Battery alerts fire at 25%, routed to `notify.battery_alerts` (Normal priority).

**Why:** A single threshold at 25% provides enough lead time to order replacements without urgency. All battery alerts route to `battery_alerts` to keep the alert stream clean and prevent battery warnings from polluting `house_alerts`.

---

## 13. Ring Motion → Front Porch Lights (Post-Shutoff Only)

**Decision:** Ring doorbell motion triggers front porch lights only **after** the nightly premise lights shutoff.

**Why:** During normal evening hours, premise lights are already on — no need for motion override. The motion trigger is a security feature for the late-night window when lights are off.

**Uses `event.front_door_motion`** (event entity, not binary sensor) for reliable detection without state-change timing issues.

---

## 14. Sump Pump — Daily Summary vs Real-Time Alerts

**Decision:** Normal sump pump operation sends only a daily summary (11:55 PM, operational log). Alerts fire only on failure conditions.

**Why:** Sump pump cycles many times per day in rain. Real-time notifications would be noise. Failure detection (no activity despite rainfall, floor sensor wet) routes to house_alerts.

---

## 15. Files and Architecture

**Package approach:** Each subsystem is a self-contained YAML package in `/config/packages/`.  
**No splitting automations across files** — each subsystem's automations, helpers, and scripts stay together.  
**Insteon device/scene backups:** `/config/insteon_devices.json` and `/config/insteon_scenes.json` — back up before any Insteon changes.

---

---

## 16. Automation IDs — All Automations Must Have Explicit IDs

**Decision:** Every automation must have a unique `id:` field.

**Why:** HA uses IDs to track automation state, history, and traces. Missing IDs cause automations to lose their run history on restart, break automation traces in the UI, and can cause duplicate execution issues. Previously, some automations were missing IDs and this caused production problems.

**Standard convention:** `subsystem_descriptive_action` in snake_case. Examples:
- `power_vent_hood_daily_reset`
- `system_health_check_daily`
- `garage_lights_motion_on`

**Before changing any automation ID:** Verify the ID is not referenced anywhere else (scripts, other automations, dashboards). Changing an ID is effectively deleting and recreating the automation — history is lost.

**Review checklist rule:** During every subsystem review —
- Missing ID → flag, recommend adding using standard convention
- ID present but not following convention → flag, recommend correction
- Never silently add, change, or remove IDs without explicit confirmation

---

## 17. YAML Formatting — Comments Use Plain ASCII Only

**Decision:** YAML comment lines use only plain ASCII characters. Separator lines use dashes (`---` or `# ---`).

**Why:** Non-ASCII characters in YAML comments (em dashes, smart quotes, unicode symbols, box-drawing characters) can cause parse errors in some HA versions or editors, and are invisible corruption risks when copying between systems.

**Rules:**
- Separators: `# --------------------------------------------------` (dashes only)
- No em dashes (`—`), curly quotes (`""`), arrows (`→`), or other unicode in comments
- Emoji in notification message strings (values) are fine — only comments are restricted

**⚠️ When editing YAML, scan for and replace any unusual formatting characters in comments with plain ASCII equivalents.**

---

## 18. Working Style — Ask First, Change Second

**Decision:** Before making any code or documentation changes, ask clarifying questions to fully understand intent, scope, and side effects.

**Why:** Several production errors were introduced by making "obvious" fixes without understanding the full context — including removing double-sends, changing automation IDs, and simplifying logic that had non-obvious dependencies.

**Standard practice:**
- Review the relevant YAML and docs first
- Ask questions about anything ambiguous before proposing changes
- Present proposed changes with rationale and alternatives before implementing
- Confirm before touching anything that touches prod

---

## 19. Review Order — Subsystems First, Project-Level Docs Last

**Decision:** When doing a full system review, review individual subsystem files first, then update project-level overview/architecture documents to reflect what was found.

**Why:** Project-level docs (README, architecture, device inventory) describe the system as a whole — they should be updated to reflect the actual verified state of subsystems, not the other way around.

**Order:**
1. Safety (smoke/CO, leaks, sump)
2. Lighting — indoor
3. Lighting — premise/exterior
4. Environmental (humidity, dehumidifier, air quality)
5. Appliance monitoring (fridge/freezer, laundry)
6. Garage, irrigation
7. Power/infrastructure, battery, system health
8. Project-level MD files last (README, device-inventory, apps, transferring-ownership, etc.)

---

*This document should be updated whenever a significant design decision is made or changed.*