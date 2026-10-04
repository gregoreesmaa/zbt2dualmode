# zbt2dualmode — Can the Home Assistant Connect ZBT-2 do Zigbee + Thread at once?

Research notes on why the [Home Assistant Connect ZBT-2](https://www.home-assistant.io/connect/zbt-2/)
cannot run Zigbee and Thread (Matter-over-Thread) simultaneously, what the
Nabu Casa team learned from the multiprotocol experiment, and what you would
need if you wanted to rewrite the firmware for true dual-mode.

> Status: community research, not affiliated with Nabu Casa / Home Assistant.
> Quotes below are sourced; see [docs/SOURCES.md](docs/SOURCES.md).
> Where a claim is inferred rather than directly quoted, it is labeled as such.

## TL;DR

- The ZBT-2 runs **one protocol at a time**: Zigbee **or** Thread. Running both
  on the same stick ("multiprotocol" / "MultiPAN") is **not supported** and
  Nabu Casa says they **do not plan to implement it**.
- Official reason (ZBT-2 FAQ, quoted by multiple outlets): multiprotocol is
  "theoretically possible with the hardware" but "doesn't work well" — the
  team "thoroughly tested multiprotocol with [the] ZBT-1 adapter and found its
  operation to be inconsistent, often causing device stability issues."
- The follow-through across the ecosystem matches: the Silicon Labs
  Multiprotocol add-on is effectively **deprecated**, Zigbee2MQTT documents
  "multiprotocol firmware is not supported," and the guidance is **one radio
  per protocol** (e.g. two sticks, or Yellow-internal-Zigbee + USB-Thread).
- Hardware is a single-radio part: **Silicon Labs EFR32MG24A420F1536IM40**
  (2.4 GHz 802.15.4) behind an **ESP32-S3 USB-serial bridge**. One radio means
  Zigbee and Thread would have to **timeslice** one 2.4 GHz front end, with
  separate PANs/channels/keys — the failure domain Nabu Casa hit on the ZBT-1.
- Firmware surface for a rewrite is public: [NabuCasa/silabs-firmware-builder](https://github.com/NabuCasa/silabs-firmware-builder)
  (ZBT-2 Zigbee NCP / OpenThread RCP / router targets), flashing via
  [universal-silabs-flasher](https://github.com/NabuCasa/universal-silabs-flasher)
  (`zbt2` profile), community manifests/pin maps for the MG24 + ESP32-S3
  bridge. No official schematic was found; the actionable hardware refs are
  teardowns, community pin tables, and the EFR32MG24 reference manuals.

## Layout

- `docs/01-official-limitation.md` — what the ZBT-2 page actually says
- `docs/02-nabucasa-learnings.md` — multiprotocol postmortem, failure modes
- `docs/03-firmware-inventory.md` — repos, manifests, image types, tooling
- `docs/04-hardware-schematics.md` — SoC, bridge wiring, radio constraints
- `docs/05-dual-mode-rewrite-plan.md` — what a true dual-mode rewrite needs
- `docs/SOURCES.md` — every primary source consulted

## Recommendation if you need both today

Use **two radios**: one flashed Zigbee NCP, one flashed OpenThread RCP — or
one ZBT-2 plus an existing Thread border router. Do not ship a product on
single-radio MultiPAN; that is the configuration Nabu Casa abandoned.
