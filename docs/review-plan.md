# System Review Plan

**Purpose:** Track review progress across sessions. Doc file is the unit of review — each doc is reviewed alongside all YAML packages it covers, plus relevant integration docs.
**Started:** February 2026

---

## Subsystem Docs

| Doc | YAMLs Covered | Integration Docs | Status |
|-----|--------------|-----------------|--------|
| safety-monitoring.md | smokeco, sumppump, basementleaksensors, systemhealth, batteryalerts, systemstartup | yolink-integration.md | Complete -- February 2026 |
| environmental-control.md | hvachumiditypad, basementdehumidifier, masterbathhumidity, middlebathhumidity, eastbathhumidity, entryfan | homekit-device-integration.md, matter-thread-integration.md | Complete -- February 2026 |
| lighting-indoor.md | lightingindoor | insteon-integration.md, sun-integration.md | Complete -- February 2026 |
| lighting-premise.md | lightingpremise | insteon-integration.md, ring-integration.md, sun-integration.md | Complete -- February 2026 |
| air-quality-monitoring.md | airquality | matter-thread-integration.md | Complete -- February 2026 |
| battery-monitoring-system.md | batteryalerts | pushover-integration.md | Complete -- February 2026 |
| fridgefreezer.md | fridgefreezer | yolink-integration.md | Complete -- February 2026 |
| garage.md | garage | insteon-integration.md, yolink-integration.md | Pending |
| irrigation.md | irrigation | insteon-integration.md | Pending |
| laundry.md | laundry | yolink-integration.md | Complete -- February 2026 |
| washerdryernotify.md | washerdryernotify | yolink-integration.md | Pending |

---

## No Subsystem Doc

| Package | Notes | Status |
|---------|-------|--------|
| powerinfrastructure.yaml | No doc exists — decide during review if one needed | Pending |

---

## Integration Docs

Reviewed alongside relevant subsystem — not standalone unless noted.

| Doc | Review With | Status |
|-----|------------|--------|
| yolink-integration.md | safety-monitoring session | Pending |
| insteon-integration.md | lighting sessions | Pending |
| homekit-device-integration.md | environmental-control session | Pending |
| matter-thread-integration.md | environmental-control session | Pending |
| pushover-integration.md | battery-monitoring-system session | Complete -- February 2026 |
| ring-integration.md | lighting-premise session | Pending |
| sun-integration.md | lighting-premise session | Pending |
| backup-integration.md | Standalone (small) | Complete -- February 2026 |

---

## Project-Level Docs (review last)

| Doc | Status |
|-----|--------|
| README.md | Pending |
| apps.md | Pending |
| device-inventory.md | Pending |
| insteon-scenes.md | Complete -- February 2026 |
| transferring-ownership.md | Pending |

---

## scripts.yaml

| File | Status |
|------|--------|
| scripts.yaml | Complete -- February 2026 (reviewed with environmental-control) |

---

*Update status after each session. Commit review-plan.md with each session commit.*
