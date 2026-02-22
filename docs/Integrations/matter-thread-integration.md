# Matter/Thread Integration

**Integration Type:** Matter (application layer) / Thread (mesh network)  
**Devices in System:** 4  
**Primary Use:** Air quality sensing and fan/HVAC control  
**Status:** Active — expanding as technology matures

---

## What is Matter/Thread?

Matter is an open, royalty-free smart home connectivity standard developed by the Connectivity Standards Alliance (CSA). It defines how smart home devices communicate at the application layer, ensuring interoperability across ecosystems (Home Assistant, Apple Home, Google Home, Amazon Alexa) without cloud dependency.

Thread is the underlying IP-based mesh network protocol that Matter devices use to communicate. Unlike Zigbee or Z-Wave, Thread runs on IPv6 and integrates natively with standard networking infrastructure.

**Key Features:**
- Open standard — no proprietary hub or cloud required
- Local control with no cloud dependency
- Cross-ecosystem interoperability
- IPv6-based mesh networking (Thread)
- Self-healing mesh — devices route around failures
- Low power consumption suitable for battery-operated sensors
- Encrypted communication throughout

**Matter vs. Thread:**
- **Thread** is the radio/mesh protocol (how signals travel)
- **Matter** is the application standard (what the signals mean)
- A Thread Border Router bridges the Thread mesh to the IP home network, making devices visible to Home Assistant

---

## Role in This System

**Primary Functions:**
- Indoor air quality monitoring (CO2, PM2.5, TVOC) via IKEA Alpstuga sensors
- Fan control in second floor bathrooms via Inovelli fan switches
- HVAC humidity management via fan switch automations
- Thread mesh infrastructure — Inovelli switches deployed in part to extend reliable mesh coverage to the second floor

**Strategy:**
The system uses Thread-only devices. No WiFi-based Matter devices are deployed. Lighting on the Matter/Thread platform is intentionally deferred pending maturity of scene support in the ecosystem. Insteon remains the primary lighting protocol in the interim.

**Devices Connected:**

| Device | Location | Protocol | Function |
|--------|----------|----------|----------|
| IKEA Alpstuga sensor | Main living area (Floor 1) | Thread | CO2, PM2.5, TVOC monitoring |
| IKEA Alpstuga sensor | Master bedroom (Floor 2) | Thread | CO2, PM2.5, TVOC monitoring |
| Inovelli White fan switch | Middle bathroom (Floor 2) | Thread | Fan control + mesh node |
| Inovelli White fan switch | East suite bathroom (Floor 2) | Thread | Fan control + mesh node |

---

## Hardware

### Thread Border Router

**Silicon Labs ZBT-2 Adapter:**
- Installed in the HA Green server
- Located on the first floor, west side of the house
- Acts as the sole Thread Border Router — bridges the Thread mesh to the home IP network
- Configured in Home Assistant via the Thread integration
- The second floor Inovelli switches significantly extend mesh reach from this west first floor location

### Sensors

**IKEA Alpstuga Multi-Sensors (2):**
- Thread-based air quality sensors
- Capabilities: CO2 (ppm), PM2.5 (µg/m³), TVOC
- Battery or USB powered
- First deployed Thread devices on this system

### Fan Switches

**Inovelli White Matter/Thread Fan Switches (2):**
- Model: Inovelli White Series Matter/Thread fan switch
- Controls exhaust fan on/off
- Located in middle bathroom (Floor 2) and east suite bathroom (Floor 2)
- Selected for Thread support and to build out the mesh — placement extends coverage east to west across the second floor and the added elevation provides wider mesh reach
- Integrated with HVAC humidity management automations

---

## Home Assistant Configuration

### Integration Setup

**Thread Border Router:**
- **Location:** Settings → System → Hardware (ZBT-2 adapter configuration)
- Thread network automatically formed when ZBT-2 is configured
- Network credentials stored in HA

**Matter Integration:**
- **Location:** Settings → Devices & Services → Integrations → Matter (BETA)
- Requires the Matter server add-on (installed via Supervisor)
- Devices appear as standard HA entities once commissioned

### Add-ons Installed

The following Home Assistant add-ons are installed and must be running:

- **Matter Server** — installed via HA Supervisor (Settings → Add-ons); provides the Matter controller that manages device commissioning and communication
- **OpenThread Border Router** — installed via HA Supervisor; manages the ZBT-2 adapter as a Thread Border Router and maintains the Thread network

### Entities Created

**IKEA Alpstuga (per sensor):**
- `sensor.*_co2` — CO2 level (ppm)
- `sensor.*_pm25` — PM2.5 particulate matter (µg/m³)
- `sensor.*_tvoc` — Total volatile organic compounds
- `sensor.*_battery` — Battery level

**Inovelli Fan Switch (per switch):**
- `fan.*` — Fan entity (on/off); configured as standard single-speed exhaust fan switch — existing fans are single speed so variable speed control is not used
- Additional diagnostic entities (firmware, signal strength)

---

## Thread Mesh Network

Thread forms a self-healing IPv6 mesh where every powered device acts as a router, extending network coverage. Battery-powered devices (like the Alpstuga sensors) join as end devices and rely on nearby routers to relay their communications.

**Mesh Roles:**
- **Border Router:** ZBT-2 adapter in HA Green — bridges Thread mesh to home IP network
- **Routers (Full Thread Devices):** Inovelli fan switches — always-powered, extend the mesh
- **End Devices (Minimal Thread Devices):** IKEA Alpstuga sensors — battery powered, connect through nearest router

**Coverage Rationale:**
The ZBT-2 is located on the first floor, west side. The Inovelli switches in the middle bathroom and east suite bathroom extend mesh coverage east to west across the second floor. The added elevation of the second floor also provides wider mesh reach relative to the first floor border router. This placement was a deliberate consideration in device selection and location.

**Thread-Only Deployment:**
Only Thread-capable devices are added to this system. No WiFi-based Matter devices are used, keeping the network topology simple and fully local.

---

## Lighting Strategy

Matter/Thread lighting (switches, dimmers, bulbs) is intentionally **not deployed** at this time. The ecosystem's support for scenes — equivalent to the Insteon scene functionality used throughout the house — is not yet mature enough to meet the system's requirements.

**Current State:**
- Insteon is the primary lighting protocol for all interior and exterior lighting
- Insteon scenes handle multi-device coordinated lighting

**Future Path:**
- When Matter scene support matures sufficiently, Thread-based lighting switches are a natural migration candidate
- The existing Thread mesh (Alpstuga sensors + Inovelli switches) provides infrastructure that lighting devices would join
- No action required until the technology is ready

---

## Commissioning a New Device

New Matter/Thread devices are commissioned using the Home Assistant mobile app with the device's QR code or setup code.

**Steps:**
1. Open the **Home Assistant mobile app** on your phone
2. Navigate to **Settings → Devices & Services**
3. Tap **Add Integration** → **Matter**
4. Select **Add Matter device**
5. Use the phone camera to **scan the QR code** on the device (or enter the 11-digit setup code manually)
6. HA will commission the device onto the Matter/Thread network
7. The device joins the Thread mesh automatically via the ZBT-2 border router
8. Entities are created automatically — verify in **Settings → Devices & Services → Matter**

**Notes:**
- Phone must be on the same local network as HA during commissioning
- The Matter server add-on must be running
- Some devices require a factory reset before commissioning if previously paired elsewhere
- Commissioning is a one-time process; devices reconnect automatically after power cycles

---

## Troubleshooting

### Device Won't Commission
- Verify the Matter Server add-on is running (Supervisor → Add-ons)
- Ensure phone and HA are on the same network
- Factory reset the device and retry
- Check that the ZBT-2 adapter is recognized in Settings → System → Hardware

### Device Drops Off Mesh
- Thread is self-healing — wait a few minutes for re-routing
- Check device power (Inovelli switches require neutral wire and stable power)
- Verify ZBT-2 is operational
- Check Thread network topology in **Settings → System → Thread** (if available)
- Restart the Matter Server add-on if devices remain unavailable

### Alpstuga Sensor Unavailable
- Check battery level (if battery powered) or USB connection
- Sensor may be out of mesh range — powered Thread devices (Inovelli switches) nearby improve this
- Restart the sensor by removing/restoring power

### Thread Network Status
- **Settings → System → Thread** shows connected devices and border router status
- Matter device diagnostics available in device detail pages

---

## Maintenance

### Periodic
- Check Matter device availability in HA dashboard
- Verify Alpstuga sensors are reporting (see Air Quality Monitoring documentation)
- Monitor Inovelli switch firmware updates via HA device pages

### After HA Updates
- Verify Matter Server add-on compatibility with new HA version before updating
- Matter integration is still marked BETA — review release notes for breaking changes

---

## Migration Notes for New Owner

**Key Points:**
- Matter/Thread is fully local — no cloud accounts, subscriptions, or external services required
- The ZBT-2 adapter is the only Thread Border Router; it must remain connected to the HA Green server
- Devices commission once and reconnect automatically
- The IKEA Alpstuga sensors feed the Air Quality Monitoring system — see that documentation for automation details
- The Inovelli fan switches are integrated with HVAC humidity automations on the second floor

**If Replacing Home Assistant:**
1. Reinstall the Matter Server add-on
2. Re-commission Matter devices (each device requires a factory reset and re-pairing)
3. Alpstuga sensor entity IDs must match those in `airquality.yaml`
4. Inovelli switch entity IDs must match humidity automation configurations

**If Adding New Devices:**
- Any Matter-certified Thread device can join the existing mesh
- Commission via HA mobile app as described above
- WiFi-based Matter devices can also be added but are not the current system strategy

**Expanding Lighting (Future):**
- When Matter scene support is sufficient, lighting switches/dimmers can be added
- Existing Thread mesh provides the infrastructure
- Insteon system can be decommissioned progressively as lighting migrates

---

## Related Documentation

- **[Air Quality Monitoring](air-quality-monitoring.md)** — IKEA Alpstuga sensor automations
- **[Insteon Integration](insteon-integration.md)** — Current primary lighting protocol
- **[Lighting Indoor](lighting-indoor.md)** — Indoor lighting automations
- **[Lighting Premise](lighting-premise.md)** — Exterior lighting automations
- **[Device Inventory](device-inventory.md)** — Full device list
- **[Environmental Control](environmental-control.md)** — Humidity and HVAC management

---

**Last Updated:** February 2026  
**Document Maintainer:** Homeowner