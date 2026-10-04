# 02 — Nabu Casa learnings: why multiprotocol was abandoned

## Short version

Nabu Casa shipped a multiprotocol (Zigbee + Thread on one radio) story around
the ZBT-1 / SkyConnect era via RCP-MultiPAN firmware plus the Silicon Labs
Multiprotocol add-on (CPC daemon multiplexing both stacks over one serial
link). They then walked it back: operation was "inconsistent," support load
was high, and the current guidance is **dedicated radios**.

## Evidence trail

- **SkyConnect community thread** (Nabu Casa staff summary): "while Silicon
  Labs' multiprotocol works, it comes with technical limitations. These
  limitations mean users will not have the best experience compared to using
  dedicated Zigbee and Thread radios."
- **Silicon Labs Multiprotocol add-on**: community and maintainer language now
  describes it as "essentially deprecated"; migration advice is to move off it
  unless you actively use multiprotocol. HA OS / add-on issue trackers carry
  CPC-version-conflict and "secondary unresponsive" reports.
- **Forks of silabs-firmware-builder** carry the same warning verbatim: "RCP
  MultiPAN in multiprotocol mode is no longer available / no longer
  recommended. Running multi-protocol with multiple active networks on a
  single radio adapter has proven unstable when using Zigbee and Thread
  simultaneously. It also increases software component complexity."
- **Zigbee2MQTT EmberZNet docs**: "Multiprotocol firmware is not supported.
  The recommended alternative to establish multiple networks is to use one
  adapter per protocol."

## Failure modes reported (support / tracker level)

These are *reported symptoms*, not a Nabu Casa benchmark table:

1. **Device instability under MultiPAN** — drops / flaky routing attributed to
   one radio timeslicing two 802.15.4 networks (separate PAN IDs, often
   separate channels/keys).
2. **CPC multiplexor fragility** — the `cpcd`/`zigbeed`-style host daemon that
   shares one UART between Zigbee and Thread stacks adds version coupling
   (CPCd version conflicts), socket/serial failure modes ("secondary
   unresponsive", socket://core-silabs-multiprotocol:9999 issues), and a
   second moving part to debug.
3. **Firmware/host skew** — EZSP / CPC / RCP versions must line up across
   add-on, ZHA/Z2M, and OTBR; mismatches surface as flashing or commissioning
   failures (e.g. "multiprotocol firmware detected, reflash to Zigbee-only").
4. **Support economics** — one flaky-but-plausible configuration generates a
   disproportionate share of "my devices drop" tickets. Killing the combo
   removes a whole class of tickets. This is inference from the deprecation
   pattern, labeled as such.

## What this means for a "true dual-mode" rewrite

You are not fixing a missing feature; you are re-entering a configuration the
vendor measured and rejected. Your rewrite must beat, with data, each point
above: sustained two-PAN reliability, no fragile host multiplexer (or a
provably robust one), version-pinned host stacks, and a support story. See
`05-dual-mode-rewrite-plan.md` for the bar.
