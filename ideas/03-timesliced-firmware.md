# Idea 3 — Time-sliced firmware with network suspend/resume

## Proposal summary

Run one stack at a time on the single MG24 radio: alternate between a
Zigbee NCP image (or task) and an OpenThread RCP image (or task), saving
network state on each switch and restoring it on return. Unlike RAIL
Dynamic Multiprotocol (millisecond-scale frame arbitration, same channel),
this slices at seconds scale: only one PAN is ever "on the air," the other
is fully suspended. Goal: one ZBT-2 serves both a Zigbee network and a
Thread network without concurrent-listening hardware support.

## How alternating operation with save/restore would work

- Two stack contexts live in the MG24 flash (1536 KB) with statically
  partitioned RAM/NVM; only one owns the 802.15.4 transceiver at a time.
- Suspend = leave quiet air: finish or abort the current transaction,
  persist MAC state (channel, PAN ID, short/extended address, frame
  counters), network keys, and neighbor/child/source-route tables to NVM,
  then halt the stack tick.
- Resume = retune the radio to the other network's channel, reload keys
  and counters, re-enable Rx, re-announce presence (Zigbee rejoin-with-
  trust-center or orphan-scan avoidance via cached nwk state; Thread MLE
  attach resume from stored dataset + parent/leader info).
- Host side needs a coordinator: the ESP32-S3 bridge is USB-serial only
  (460800 baud 8-N-1 to the MG24), so slice scheduling must live in MG24
  firmware plus host drivers (ZHA/zigpy on one side, OTBR/border router on
  the other) that tolerate a port that goes deaf for whole dwell periods.
- Frame counters must never roll back across a restore, or encrypted
  peers drop frames as replays; NVM writes per switch add wear and latency.

## Duty-cycle design

- Symmetric baseline: e.g. 5 s Zigbee / 5 s Thread with ~50-200 ms guard
  time for state save, channel retune, and Rx settle. 50% duty per PAN.
- Asymmetric option: weight the busier network (e.g. 70/30) or adapt on
  pending traffic, but each switch still pays the full guard cost.
- Shorter slices cut latency but raise switch overhead and NVM churn;
  longer slices improve per-dwell throughput but starve the parked PAN.

## What breaks

- Sleepy end devices (SEDs): Zigbee end-device polls and Thread SED/CSL
  data polls arriving during the parked window go unanswered. Past the
  child-supervision / poll-timeout interval the parent drops the child,
  forcing rejoin/reattach storms on every cycle.
- Broadcasts: Zigbee broadcasts, route requests, and Thread MLE
  advertisements sent while the network is parked are lost; retries flood
  the next dwell and starve unicast traffic.
- Router re-routes: Zigbee routers and Thread routers see the coordinator/
  leader vanish each cycle, age out links, trigger route repair and leader
  re-election churn, and may blacklist a flapping neighbor.
- Beacons and commissioning: Touchlink/beacon requests and Thread
  discovery responses only work when their slice is active; joins pair
  only half the time and users see intermittent discovery.

## Effort / risk estimate

- Effort: high. Custom dual-image bootloader/state-store, verified
  save/restore of two closed stacks, host-driver tolerance work. Weeks to
  months for an experienced 802.15.4 firmware engineer.
- Risk: very high. Save/restore bugs corrupt keys, counters, or tables
  and brick both networks at once; NVM wear and 256 KB RAM partitioning
  leave little margin; debugging needs 802.15.4 sniffers on two channels.

## Pros / cons

Pros: no new hardware; each slice runs a near-stock stack on its own
channel/PAN; simpler air behavior than same-channel frame arbitration.
Cons: each network is deaf half the time; latency floor equals the parked
interval; SED/broadcast/router churn as above; doubles host-integration work.

## Verdict

Lab demo: yes — a useful experiment to prove suspend/resume and measure
poll-timeout and re-route behavior under controlled dwell times.
Home use: no. For daily reliability use two radios (one Zigbee NCP, one
Thread RCP), per the project recommendation; time-slicing replays the
instability Nabu Casa cited when it dropped multiprotocol support.
