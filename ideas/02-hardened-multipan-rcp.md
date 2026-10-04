# Idea 2 — Hardened single-radio MultiPAN RCP

## Proposal summary

Re-enter the deprecated single-radio multiprotocol configuration deliberately:
build an RCP-MultiPAN firmware for the ZBT-2's EFR32MG24 plus a pinned,
minimal host stack, and harden the exact failure points that made Nabu Casa
abandon it on the ZBT-1. One radio timeslices Zigbee and Thread; nothing about
the physics changes. The bet is that version discipline, a same-channel plan,
and measured soak testing can move it from "inconsistent" to "acceptable for
tinkerers" — not to supported-product quality.

## How RCP-MultiPAN + CPC multiplexing works

- Firmware is a bare 802.15.4 RCP: no Zigbee or Thread stack on-chip, only
  PHY/MAC with timesliced multitasking between two PANs.
- Both host stacks run concurrently: Zigbee via `zigbeed` and Thread via
  `otbr-agent`, each speaking Spinel frames.
- The CPC daemon (`cpcd`) multiplexes both Spinel streams over the single
  UART to the stick (socket endpoint such as
  `socket://core-silabs-multiprotocol:9999` in the old add-on).
- The radio alternates between the Zigbee PAN and the Thread PAN, so both
  networks share one 2.4 GHz front end, one transmit queue, and one channel
  scheduler.

## What would need hardening vs the deprecated setup

1. **Version pinning.** The old stack failed on CPC/EZSP/RCP skew between
   add-on, ZHA/Z2M, and OTBR. Pin one known-good triple (SiSDK, CPCd,
   OpenThread commit per `ot-efr32` guidance) and ship firmware plus host
   containers as a locked set. Any host update without the matching RCP
   build is an unsupported configuration.
2. **Same-channel plan.** Cross-channel timeslicing forces constant radio
   retunes and was the worst case for drops. Constrain both PANs to one
   shared 802.15.4 channel, accept the interference trade-off, and document
   channel selection explicitly.
3. **Soak testing with data.** Nabu Casa's verdict was empirical, so the
   rebuttal must be too: sustained two-PAN runs (days, loaded meshes on
   both sides), drop/route-flap counters, and publish the failure rate.
   No soak data means no claim of improvement.
4. **Strip the multiplexer failure modes.** Single `cpcd` version, health
   checks for "secondary unresponsive" states, and automatic recovery or
   clean failure instead of silent half-working radios.

## Effort / risk estimate

- Effort: medium. No new firmware architecture is needed; the RCP-MultiPAN
  pattern exists from the MG21 era and the ZBT-2 manifests give the board
  defines. Most work is build-system pinning, container plumbing for
  `cpcd` + `zigbeed` + `otbr-agent`, and test harness time.
- Risk: high. Every known failure mode (timeslice drops, CPC fragility,
  firmware/host skew, support load) returns by default. Success is bounded:
  best case is a fragile configuration made reproducible, not a robust one.

## Pros / cons

Pros: single stick, no extra hardware; reuses proven RCP-MultiPAN and CPC
design instead of a from-scratch rewrite; lowest firmware effort of the
dual-mode options; keeps stock flashing and recovery tooling.

Cons: inherits the abandoned architecture and its physics (one radio, two
PANs, shared airtime); permanent version-lock burden; same-channel operation
couples both meshes' interference fate; contradicts vendor, add-on, and
Zigbee2MQTT guidance, so future ecosystem updates may break it silently.

## Verdict

Not recommended as anything beyond a documented experiment. Nabu Casa tested
this shape on the ZBT-1, measured inconsistent operation and high support
load, and deprecated the Multiprotocol add-on rather than fixing it forward.
A hardened build is worth doing only to quantify the gap with soak data —
anyone who needs both protocols working should use two radios.
