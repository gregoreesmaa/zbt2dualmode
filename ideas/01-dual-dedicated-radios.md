# Idea 01 — Dual dedicated radios (one stick per protocol)

## Proposal summary

Use two separate 802.15.4 radios: one for Zigbee, one for Thread.
Each radio runs a single-protocol firmware image owned by its own host
stack. No timeslicing, no shared serial multiplexer. This is the
configuration Nabu Casa currently recommends over single-radio MultiPAN.

## How it works

- **Stick A — Zigbee NCP.** A ZBT-2 (or equivalent EFR32MG24 stick)
  flashed with the Zigbee NCP image from
  NabuCasa/silabs-firmware-builder, driven by ZHA or Zigbee2MQTT.
- **Stick B — OpenThread RCP.** A second stick (ZBT-2, ZBT-1, or
  another EFR32MGxx OT-RCP stick) flashed with the OpenThread RCP
  image, driven by the OpenThread Border Router (OTBR) add-on.
- **Host stacks stay independent.** ZHA/Z2M owns the Zigbee serial
  port; OTBR owns the Thread serial port. No CPC daemon (`cpcd`)
  multiplexing both stacks over one UART.
- **Channel planning.** Keep Zigbee and Thread on non-overlapping
  802.15.4 channels (e.g. Zigbee 15, Thread 25), and keep both clear
  of the site's 2.4 GHz Wi-Fi channels (1/6/11). Re-survey channels
  if either network is relocated or a new AP is added.
- **Placement.** Separate the sticks by USB extension cable to reduce
  USB 3.0 noise and mutual 2.4 GHz desense; keep each on its own USB
  port with a stable serial path (`/dev/serial/by-id/...`).
- **Migration path.** Existing single-stick installs keep their
  Zigbee network as-is; add the second stick only for Thread, then
  commission Matter-over-Thread devices through OTBR.

## Cost / effort estimate

- **Hardware:** one additional 802.15.4 stick (roughly one ZBT-2-class
  unit) plus one or two USB extension cables.
- **Software:** no custom firmware work; both images are stock build
  targets in silabs-firmware-builder, flashed with
  universal-silabs-flasher (`zbt2` profile where applicable).
- **Labor:** under an hour for a fresh install (flash, attach, set
  channels, verify); longer if migrating an existing Zigbee network
  to a new channel or re-pairing Thread credentials.

## Pros

- Each radio runs one PAN on one channel: no MultiPAN timeslice loss.
- Removes the CPC multiplexor failure domain (version skew,
  "secondary unresponsive", socket failures).
- Matches vendor test matrix: stock NCP + stock RCP are the supported
  images, so updates and docs apply cleanly.
- Failures are isolatable: Zigbee and Thread logs point at separate
  hardware and stacks.

## Cons

- Costs an extra stick, USB port, and cable.
- Two 2.4 GHz transmitters colocated: still needs channel planning
  and physical separation.
- Two firmwares to keep updated instead of one.
- Not a single-stick product story; does not answer "one ZBT-2 for everything."

## Why it aligns with Nabu Casa guidance

- Nabu Casa withdrew the RCP-MultiPAN + Multiprotocol add-on path
  after finding single-radio operation "inconsistent" with device
  stability issues; current guidance is dedicated radios.
- The Silicon Labs Multiprotocol add-on is effectively deprecated and
  Zigbee2MQTT documents multiprotocol firmware as unsupported, both
  recommending one adapter per protocol.
- This idea implements exactly that recommendation with no new
  firmware surface to support.

## Open questions

- Which stick models are confirmed-good as the Thread RCP half for
  OTBR on HA OS, and at which firmware revisions?
- Recommended default channel pair for Zigbee + Thread given common
  Wi-Fi layouts in apartments vs. houses?
- Minimum physical separation / cable length for two colocated EFR32MG24 sticks?
- Backup and restore story: coordinated backup of Zigbee network and
  Thread dataset during host migration?
