# 05 — Dual-mode rewrite plan: what "true dual-mode" would actually take

## The bar (from 01–04)

You must beat, with measurements, the configuration Nabu Casa rejected:
sustained two-PAN reliability on one radio, no fragile host multiplexer (or a
provably robust one), pinned host-stack versions, and a support story. If you
cannot publish 72-hour+ two-network soak data that matches dedicated radios,
you have reproduced the problem, not the fix.

## Option A — Single-radio MultiPAN RCP (the rejected path, revisited)

- Build an RCP-MultiPAN image from SiSDK/GSDK for the MG24 (`rcp-uart-802154`
  class), run Zigbee (`zigbeed`) + Thread (`otbr-agent`) on the host behind
  `cpcd`, all PANs on one channel to start.
- You inherit every failure mode in `02-nabucasa-learnings.md` (CPC coupling,
  version skew, beacon starvation). Only attempt this as an instrumented
  experiment with RF logs, drop counters, and SED-poll latency histograms —
  and expect to confirm Nabu Casa's result.
- Open questions left unresolved by this research: no published MultiPAN
  degradation metrics; no validated custom dual-protocol path on ZBT-2.

## Option B — Time-sliced NCP with network suspend/resume (fragile)

- Alternate full Zigbee-NCP and Thread-RCP images (or a dual-MAC app) with
  network-state save/restore across switches. Devices on the parked network
  see an offline coordinator; SEDs time out, routers re-route, broadcasts are
  lost. Only viable for lab demos, never for a home.

## Option C — Recommended: two radios (the supported architecture)

- Flash one stick Zigbee NCP, one stick OpenThread RCP (both via
  `universal-silabs-flasher --profile zbt2`), or pair one ZBT-2 with an
  existing Thread border router / Yellow-internal radio. This is Nabu Casa's
  stated recommendation ("dedicate a device to each protocol") and
  Zigbee2MQTT's documented position.

## Minimum lab setup for any rewrite attempt

1. Second radio for control comparison (dedicated Zigbee vs dedicated Thread
   baseline).
2. Build chain: `ghcr.io/nabucasa/silabs-firmware-builder` container,
   Simplicity SDK + `slc` + Simplicity Commander, manifest
   `zbt2_zigbee_ncp.yaml` / `zbt2_openthread_rcp.yaml` as starting points,
   unique `SL_APPLICATION_PRODUCT_ID` for any fork.
3. Flash/recovery: `universal-silabs-flasher --profile zbt2`; Nerivec
   recovery GBLs; SWD as last resort; record the EUI-64 before wiping.
4. RF plan: fixed Zigbee vs Thread channels, spectrum scan, same-channel and
   split-channel runs, SED + router + broadcast mix, 72 h soak each.
5. Host pinning: lock ZHA/Z2M, OTBR, CPCd, EZSP/Spinel versions per run.

## Risks to state upfront

- Bricking / EUI loss without recovery images on hand; cross-flashing MG21
  builds onto MG24 (check `device:` line first).
- Burning the single-radio compromise into a "product" others depend on.
- SiLabs docs gaps hit here: AN1333 / concurrent-multiprotocol blog bodies
  returned 403 during research; channel/CSMA-CA claims above rest on
  docs.silabs.com mirrors and community quotes — re-verify against the
  GSDK sources you build with.
