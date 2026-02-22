# Home Assistant Device Inventory

Complete inventory of all devices integrated with Home Assistant, including physical location, entity IDs, and hardware details.

Last Updated: February 2026

---

## Inventory Summary

- **Total Devices:** 74
- **Insteon Devices:** 39
- **YoLink Devices:** 27
- **Matter/Thread Devices:** 4
- **Ring Devices:** 1 (Video Doorbell 2)
- **Ecobee:** 1
- **Physical Sensors:** 2

Note: This summary will lag the Google Sheet as devices are added or removed. The Sheet is the authoritative count.

---

## Device Inventory

**Complete device inventory available in Google Sheet:**

**[House IT Inventory & Documentation](https://docs.google.com/spreadsheets/d/1w4-gFy8rL0OqKrlzC7T9p2vFs5JBJYV2dyVfrrOA8DI)**  
Google Account: westcombecasslake@gmail.com (transfers with house)

The spreadsheet contains comprehensive details for all 74 devices including:
- Physical location (room and specific placement)
- Home Assistant entity IDs
- Device type and model numbers
- Serial numbers and firmware versions
- Integration type (Insteon, YoLink, Ecobee, Physical)
- Function and purpose
- Installation dates (smoke/CO detectors: February 2024)
- Special notes and cross-references

**Device Count Summary:**
- **Insteon Devices:** 39 (lighting, sensors, infrastructure)
- **YoLink Devices:** 27 (smoke/CO, temperature, humidity, power monitoring)
- **Matter/Thread Devices:** 4 (air quality sensors, bathroom fan switches)
- **Ring Devices:** 1 (Video Doorbell 2)
- **Ecobee:** 1 (smart thermostat with remote sensor)
- **Physical Sensors:** 2 (sump depth bobber, powerline monitor)
- **Total:** 74 devices

---

## Device Categories

### Lighting Control
- **Indoor Lighting:** 25 devices
- **Premise/Outdoor Lighting:** 12 devices
- **Seasonal Lights:** 7 devices

### Safety & Monitoring
- **Smoke/CO Detectors:** 8 YoLink sensors (installed 2/2024)
- **Leak Sensors:** 3 YoLink sensors + 1 YoLink siren
- **Sump Pump Monitoring:** 3 sensors (depth, power, leak)

### HVAC & Environmental
- **Ecobee Thermostat:** 1 main floor controller
- **Humidity Control:** 5 YoLink sensors (basement, laundry, master bath, middle bath, east bath)
- **Temperature Monitoring:** 6 YoLink sensors (appliances + rooms)

### Utilities
- **Irrigation:** 1 drip sprinkler control
- **Laundry:** 1 dryer power monitor
- **Garage:** 3 devices (lights, motion, noise filter)

### System Infrastructure
- **Insteon PLM:** 2413U (Powerline Modem)
- **YoLink Hub:** YS1604-UC with speaker

---

## Notes

- All Insteon PS01 switches are at firmware v.54
- YoLink smoke/CO sensors installed February 2024
- Sump pump uses physical float sensor connected to Insteon 2450
- Custom dusk sensor using Amazon lamp post sensor + Insteon micro module
- Powerline noise filter in garage isolates camper battery charger
