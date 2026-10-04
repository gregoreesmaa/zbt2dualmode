# Idea 07 — Matter bridge instead of a second radio network

## Proposal summary

Keep a single Zigbee network on the ZBT-2 (Zigbee NCP firmware, ZHA or
Zigbee2MQTT as owner) and expose its devices into Matter fabrics via a
Matter bridge — e.g. the Home Assistant Matter bridge or the Zigbee2MQTT
Matter bridge — so phones and ecosystems (Apple Home, Google Home, Alexa)
see Matter devices with no Thread radio needed.

## How it works

- The ZBT-2 stays on Zigbee only: one PAN, one channel, stock NCP image.
- A bridge application on the host joins selected Zigbee entities to a
  Matter fabric as bridged nodes/endpoints over Wi-Fi/Ethernet transport.
- Commissioning targets the bridge's Matter node, not each Zigbee device;
  the bridge translates Matter clusters to Zigbee clusters and back.
- No OpenThread RCP, no OTBR, no Thread dataset to manage for these devices.

## How bridging differs from native Matter-over-Thread

- Native Thread devices join a Thread mesh and speak Matter end to end;
  each device is individually commissioned and addressable on the fabric.
- Bridged devices never join Thread; the bridge is the only Matter node,
  and the Zigbee mesh remains the actual radio network behind it.
- Bridged devices inherit the bridge's availability: if the host or bridge
  app is down, all bridged endpoints go offline together.
- Feature surface is the intersection of the two cluster sets, not the full
  Matter device-type spec; vendor extensions on the Zigbee side may not cross.

## What bridges cleanly vs. poorly

- Clean: on/off lights, dimmable lights, smart plugs, basic switches,
  contact/occupancy/temperature sensors with standard clusters.
- Adequate with caveats: thermostats, color lights (gamut/transition mapping),
  multi-gang switches (endpoint composition varies by bridge).
- Poor: Zigbee scenes and bindings, Green Power devices, manufacturer-specific
  features (custom calibration, power monitoring quirks), locks and security
  devices where ecosystem certification or commissioning flows expect native
  Matter behavior.

## Effort estimate

- Software: no firmware work; enable the bridge integration, select entities,
  commission the bridge into each target fabric.
- Labor: under an hour for a small set of standard devices; longer for large
  networks or per-ecosystem testing of edge device types.
- Maintenance: track bridge mapping updates alongside HA/Zigbee2MQTT upgrades.

## Pros

- No second radio, no Thread channel planning, no MultiPAN timeslicing.
- Reuses the existing stable Zigbee mesh and its router coverage.
- One commissioning step exposes many devices to each ecosystem.
- Keeps the ZBT-2 on the vendor-supported single-protocol configuration.

## Cons

- Adds a translation layer: another failure domain and latency hop.
- Only standard clusters cross; exotic features stay in the Zigbee UI.
- Bridge availability gates all bridged devices at once.
- Some ecosystems display or automate bridged devices with reduced fidelity.

## Verdict

Best option when the goal is "Zigbee devices visible in Matter ecosystems"
rather than "run a Thread mesh." It sidesteps the ZBT-2 single-radio limit
entirely by moving Matter onto IP transport. Choose native Thread hardware
(a second radio or standalone border router) only when devices must be
genuinely Matter-certified endpoints or operate without the bridge host.
