# CLAUDE.md - Home Assistant Project Instructions

This file is automatically loaded by Claude Code at the start of every session.
It defines how Claude must behave when working on this system.

---

## First Step — Always Read This Before Anything Else

Before reviewing, editing, or proposing any changes:
1. Read `/homeassistant/docs/builddecisions.md` — it is the authoritative record of why things are built the way they are
2. Read the relevant package YAML in `/homeassistant/packages/` alongside its doc in `/homeassistant/docs/subsystems/`
3. Read `scripts.yaml` if the work touches shared scripts

---

## Path Mapping

The HA config directory is the same physical folder, but different add-ons mount it at different paths:

| Tool | Path |
|------|------|
| Claude Code (this add-on) | `/homeassistant/` |
| VS Code Server | `/config/` |
| HA Core docs and forums | `/config/` |

When docs, comments, or the user reference `/config/...`, translate to `/homeassistant/...` -- they are the same location.

- Packages: `/homeassistant/packages/`
- Docs: `/homeassistant/docs/`
- Subsystem docs: `/homeassistant/docs/subsystems/`
- Integrations docs: `/homeassistant/docs/Integrations/`

---

## Working Style Rules

### Ask First, Change Second
- Review all relevant files before proposing anything
- Ask about anything ambiguous before touching code
- Present proposed changes with rationale before implementing
- Confirm before editing any production file

### Never "Clean Up" Without Being Asked
- Do not remove logic that looks redundant — it probably has a non-obvious reason
- Do not simplify, refactor, or reorganize unless explicitly instructed
- Do not add comments, docstrings, or formatting changes beyond what is asked

### No Silent Changes
- Flag issues and wait for direction — do not silently fix things
- If something looks wrong, describe it and ask before changing it

---

## Hard Rules — Never Violate

### Insteon
- Every `insteon.scene_on` / `insteon.scene_off` call must be immediately repeated with a 3-second delay (double-send). Do not remove this.
- Scene numbers in YAML are always decimal, never hex.
- N-way switch links are in device hardware, not HA automations. Do not try to replicate them in YAML.

### Notification Routing
Four tiers — never mix:

| Service | Priority | Use For |
|---|---|---|
| `notify.house_alerts` | High (1) | Safety events, 15% critical battery |
| `notify.house_notices` | Normal (0) | Awareness items |
| `notify.operational_log` | Silent (-2) | Routine confirmations, daily summaries |
| `notify.battery_alerts` | Normal (0) | 30% low battery warnings only |

Do not route 30% battery warnings to `house_alerts`.
Do not route safety events to `operational_log`.

### Premise Lighting
- Photocell is primary trigger — not sun/schedule. Do not switch to sun-only triggering.
- Seasonal delay helpers exist for a reason. Do not remove or simplify them.
- Photocell daily reset at 1 PM is intentional. Guard window (12:58-1:01 PM) in motion automation is intentional.

### Ecobee / HVAC Fan
- HA does NOT set temperature or HVAC mode — Ecobee owns that.
- Fan control uses `device_id: c38113c2d02da790b689d4a3bdc4b5e1` only. There is no entity to use.
- `hvac_exhaust_boost` script is the single point of fan control. Do not bypass it.

### Fridge/Freezer Thresholds
- Do not round or simplify thresholds. They were calibrated from actual sensor history.

---

## Automation ID Rules

- Every automation must have a unique `id:` field
- Convention: `subsystem_descriptive_action` in snake_case
- During any review: flag missing or non-conforming IDs — do not silently add or change them
- Never change an existing ID without explicit confirmation — it deletes run history

---

## YAML Formatting

- Comments use plain ASCII only — no em dashes, smart quotes, arrows, unicode symbols
- Separator lines: `# --------------------------------------------------`
- Emoji in notification message strings (values) is fine — restriction applies to comments only
- Scan for and flag non-ASCII characters in comments when editing any YAML file

---

## Architecture Rules

- Each subsystem is a self-contained package in `/homeassistant/packages/`
- Do not split automations, helpers, or scripts across files for the same subsystem
- Before any Insteon changes: note that `insteon_devices.json` and `insteon_scenes.json` should be backed up

---

## Review Order (Full System Reviews)

1. Safety (smoke/CO, leaks, sump)
2. Lighting — indoor
3. Lighting — premise/exterior
4. Environmental (humidity, dehumidifier, air quality)
5. Appliance monitoring (fridge/freezer, laundry)
6. Garage, irrigation
7. Power/infrastructure, battery, system health
8. Project-level docs last (README, device-inventory, apps, transferring-ownership)

---

## Adding New Decisions

When a new build decision is made during a session:
- Technical/architectural decisions with "why" rationale -> add to `builddecisions.md`
- Rules about how Claude should behave -> add to this file (`CLAUDE.md`)
- Some decisions belong in both
- Always update the relevant subsystem doc in `/homeassistant/docs/subsystems/` if the decision affects that package

---

*Updated: February 2026*
