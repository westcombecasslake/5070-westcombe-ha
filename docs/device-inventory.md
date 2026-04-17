# Home Assistant Device Inventory

Last Updated: April 2026

---

## How This Works

Most device inventory information lives in Home Assistant — room assignments, subsystems,
integrations, and entity IDs. But HA has no concept of physical placement, doesn't store
model names in a useful way, and keeps serial numbers in an internal hex format. Those
gaps live in a Google Sheet maintained by the home owner. This document describes how to
keep the two in sync using Claude Code to surface differences and generate an updated TSV
that gets pasted back into the Sheet.

    ┌─────────────────────┐           ┌─────────────────────────┐
    │   Home Assistant    │           │  TSV / Sheet            │
    │                     │           │  (Home Owner)           │
    ├─────────────────────┤           ├─────────────────────────┤
    │     Device Name     │───────────│     Model               │
    │     Location        │           │     SN/ID               │
    │     Subsystem       │           │     Notes               │
    │     Integration     │           └─────────────────────────┘                    │
    │     Entity ID(s)    │           
    └─────────────────────┘

Changes and deletions flow from HA outward. Claude Code validates HA against the TSV,
produces a diff table, and the owner updates the TSV before pushing to the Sheet.

    Home Assistant       Claude Code              TSV / Sheet
    (most fields)   ──►  validate HA vs TSV  ──►  open in VS Code
                         produce diff table       edit Model, SN, Notes
                                             ──►  Ctrl+A → paste special
                                                  (values only) → Sheet

**[House IT Inventory & Documentation](https://docs.google.com/spreadsheets/d/1w4-gFy8rL0OqKrlzC7T9p2vFs5JBJYV2dyVfrrOA8DI)**
Google Account: westcombecasslake@gmail.com (transfers with house)

The Sheet includes:
- A device list tab with all columns above
- A pivot table tab showing device counts by subsystem and integration

For any count or detail question, consult the Sheet or TSV — do not rely on counts in
this document.

---

## Reconciliation Process

After installation activity, device changes, or deletions, HA is the system of record
for most fields. Reconcile using Claude Code to validate HA against the TSV, then push
the result to the Sheet.

### Steps

1. **Validate HA against TSV**
   In Claude Code, run:
   > "Ask: validate HA device registry against TSV"

   Claude Code will compare HA's device registry to `device-inventory.tsv` and produce
   a diff table showing fields that differ for HA-owned fields, devices in HA missing
   from TSV, and devices in TSV missing from HA (separated — these are different cases).
   SN and Location Detail differences are expected and will not be flagged.

2. **Review and resolve**
   Decide whether each discrepancy requires a TSV update or investigation.

3. **Update the TSV**
   Open `device-inventory.tsv` in VS Code. Edit TSV-owned fields (Model, SN/ID, Notes)
   as needed. Ctrl+A, Copy.

4. **Push to Sheet**
   Open the Sheet → device list tab → select all → Paste Special → Values Only.

5. **Commit**
   Commit the updated TSV alongside any YAML changes that prompted the reconciliation.

---

## Stable Notes

- All Insteon PS01 switches are at firmware v.54
- YoLink smoke/CO sensors installed February 2024
- Sump pump uses physical float sensor (Letzgo SB2) connected to Insteon 2450 relay
- Custom dusk sensor: Amazon lamp post sensor driving Insteon micro module (2443-222)
- Powerline noise filters (Insteon 1626-10) in garage and living room isolate high-interference loads
- Ecobee thermostat is controlled exclusively through the Ecobee app and HomeKit -- HA only controls the fan via device_id