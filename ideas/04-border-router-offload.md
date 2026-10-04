# Idea 4 — Thread Border-Router Offload

Keep the ZBT-2 as a Zigbee-only radio and serve Matter-over-Thread through
a border router you already own: Apple TV / HomePod, Google Nest Hub,
or a second OpenThread Border Router (OTBR) stick on the Home Assistant host.

## Proposal summary

- Flash and run the ZBT-2 with Zigbee firmware only (ZHA or Zigbee2MQTT).
- Do not attempt MultiPAN on the ZBT-2; leave it on one PAN, one channel.
- Add Thread coverage with a separate border router:
  - Option A: vendor hub (Apple TV 4K gen 2+, HomePod / HomePod mini,
    Google Nest Hub v2 / Nest Wifi Pro, Echo gen 4+).
  - Option B: second 802.15.4 radio on the HA host running OTBR
    (e.g. another ZBT-1/ZBT-2 flashed OpenThread RCP, or any OTBR stick).
- Home Assistant talks Zigbee via the ZBT-2 and Matter via its Matter
  integration over LAN to whichever border router owns the Thread network.

## How Thread credentials sync into Home Assistant

- Thread networks are identified by an Active Operational Dataset
  (network name, PAN ID, channel, mesh-local prefix, keys).
- The vendor hub holds the authoritative dataset for its Thread network.
- Home Assistant learns it via the Thread integration's credential sync:
  the HA companion app (iOS/Android) imports the dataset from the phone's
  Thread stack, or the dataset is shared from the hub's ecosystem app.
- Once imported, HA's Thread panel lists the shared network and the
  Matter integration can commission Matter-over-Thread devices onto it.
- With Option B (local OTBR), HA hosts the dataset itself via the
  OpenThread Border Router add-on; no vendor cloud or phone sync needed.

## Cost / effort

- Option A: near-zero hardware cost if the hub already exists; effort is
  configuration only (commission hub, sync credentials, commission devices).
- Option B: cost of one extra radio (~US $30-40) plus OTBR add-on setup.
- No custom firmware, no reflashing the ZBT-2, no multiprotocol debugging.

## Pros

- Respects the official limitation: one radio, one protocol, fully supported.
- Most stable option: each radio runs a single MAC/PAN with no timeslicing.
- Reuses hardware many households already have (Apple TV, Nest Hub).
- Keeps Zigbee mesh untouched; Thread can use a different 2.4 GHz channel.

## Cons / limitations

- Dependence on a vendor hub: firmware updates, availability, and feature
  support (e.g. Thread 1.3 features) are controlled by Apple/Google/Amazon.
- Multi-fabric and multi-admin complexity: devices may need explicit sharing
  between ecosystems; some vendor hubs lag on Matter spec versions.
- Multi-network Thread topologies: a vendor-hub Thread network and an HA
  OTBR network are separate meshes unless datasets are merged deliberately.
- Debugging spans two systems (HA logs plus a closed vendor hub), and
  Thread topology tools on vendor hubs are limited.
- Matter-over-WiFi/Ethernet devices are unaffected, but every Thread device
  depends on the external border router being powered and reachable.

## Verdict

Best near-term answer when a usable border router already exists: keep the
ZBT-2 Zigbee-only and offload Thread. It avoids the MultiPAN failure domain
Nabu Casa abandoned, costs nothing extra, and works today. Choose a second
local OTBR stick instead if you want HA to own the full Thread dataset and
avoid vendor dependence. Either way, do not reflash the ZBT-2 for dual use.
