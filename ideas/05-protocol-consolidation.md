# Idea 5 — Protocol Consolidation (migrate to one protocol)

## Proposal summary

Eliminate the two-protocol problem instead of solving it: migrate the whole
network to a single protocol so one ZBT-2 radio is enough. In practice that
means either (a) replace Zigbee end devices with Matter-over-Thread equivalents
and run the ZBT-2 as a dedicated OpenThread RCP / border router, or (b) keep
Zigbee only and use Matter-over-WiFi/Ethernet devices (no second 802.15.4
radio needed) for anything new. No MultiPAN firmware, no timeslicing, no host
multiplexer — the single-radio constraint in `docs/04-hardware-schematics.md`
stops mattering because only one 802.15.4 PAN remains.

## Migration mechanics

1. Inventory the Zigbee network (coordinator backup, device list, router vs.
   end-device roles, quirks/custom clusters in use).
2. Pick the target: Thread for battery sensors/locks, Zigbee for a large
   sunk-cost install, Matter-over-WiFi for mains-powered devices.
3. Run a transition period with two radios (ZBT-2 on the new protocol plus a
   second stick or existing border router on the old one), as recommended in
   `docs/05-dual-mode-rewrite-plan.md` Option C.
4. Re-pair devices onto the target network in router-first order to build mesh
   backbone before adding sleepy end devices; keep old automations pointed at
   abstraction (e.g. HA entities) so re-pairing does not rewrite automations.
5. Bridge only where unavoidable: a Zigbee2MQTT / Home Assistant instance can
   expose both networks to Matter fabric during transition, then the old PAN
   is decommissioned and the spare radio becomes cold backup.

## What cannot migrate

- Zigbee devices with no Matter equivalent (niche sensors, Tuya-custom-cluster
  gear, some green-power switches) — these stay Zigbee or get replaced with
  functional equivalents, not 1:1 ports.
- Installed/inaccessible devices (in-wall relays, hardwired switches) where
  physical re-pairing cost exceeds the benefit.
- Custom Zigbee bindings, scenes, and touchlink setups with no Matter
  counterpart; expect to rebuild automations, not import them.
- Thread radio requirements do not vanish: Matter-over-Thread still needs a
  border router (the ZBT-2 in RCP mode, or Apple TV / Nest Hub / second stick).

## Cost, effort, and timeline reality

- Cost is per-device replacement ($15–60 each) plus labor; a 40-device Zigbee
  network consolidates at roughly $600–2000 in hardware alone.
- Effort is dominated by physical access: factory-reset, re-pair, rename,
  re-test per device; budget 10–20 min per device plus mesh-stabilization days.
- Realistic timeline: small networks (<15 devices) over a weekend; medium
  networks (15–50) over 2–4 weeks of phased migration; large installs only pay
  off at natural replacement churn, not as a project.
- Cheapest variant is consolidation toward Zigbee (change buying policy, keep
  installed base) — near-zero cost, but it caps Matter adoption.

## Pros

- Only structurally reliable fix on single-radio hardware: one PAN, no
  timeslice contention, none of the failure modes in `02-nabucasa-learnings`.
- Uses fully supported firmware paths (Zigbee NCP or OpenThread RCP) and
  standard tooling (`universal-silabs-flasher --profile zbt2`).
- Simplifies support permanently: one channel plan, one mesh to debug, one
  backup/restore story; aligns with Nabu Casa's "one radio per protocol" stance.

## Cons

- Highest upfront cost and e-waste of any option; discards working hardware.
- No 1:1 device parity; some Zigbee devices simply have no Matter equivalent.
- Migration is disruptive (re-pairing, downtime, automation rebuilds) and slow
  for large networks; mixed households may re-diverge to two protocols anyway.

## Verdict

Protocol consolidation is the long-term structural fix: it is the only option
that removes the root cause (two PANs, one radio) rather than managing it.
Recommend it as standing buying policy (one protocol for new devices) plus
opportunistic migration at replacement churn — not as a forced rip-and-replace
unless the device count is small enough that re-pairing costs less than a
second radio and its ongoing support burden.
