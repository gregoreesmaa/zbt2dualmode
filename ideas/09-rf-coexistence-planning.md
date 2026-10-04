# Idea 09 — RF coexistence engineering (channel, power, and placement plan)

## Proposal summary

Make two side-by-side 2.4 GHz 802.15.4 networks — Zigbee and Thread —
reliable through disciplined RF planning instead of firmware. Applies
whether the radios are two sticks (the recommended setup) or any future
dual/single-radio arrangement: pick 802.15.4 channels that sit in the gaps
of the site's Wi-Fi 1/6/11 plan, set transmit power to the minimum that
closes the link, separate the antennas from each other and from noise
sources, and verify with energy scans before and after.

## Background facts

- 802.15.4 channels 11–26 are 2 MHz wide on 5 MHz spacing starting at
  2405 MHz; Wi-Fi 1/6/11 are ~22 MHz wide at 2412/2437/2462 MHz.
- The quietest 15.4 slots against a 1/6/11 Wi-Fi plan are 15 (2425 MHz,
  between Wi-Fi 1 and 6), 20 (2450 MHz, between 6 and 11), and 25/26
  (2475/2480 MHz, above 11). Channel 26 has regional and legacy-device
  restrictions, so treat 25 as the prime slot and 26 as backup only.
- The ZBT-2 radio (EFR32MG24, +10 dBm class, ~+4.16 dBi antenna) easily
  overpowers battery end devices on the return path if run at maximum;
  symmetric links matter more than raw range.

## Default channel plan

- Pin site Wi-Fi to channels 1 and 6; keep 2.4 GHz Wi-Fi off channel 11
  where possible, or run any mandatory ch-11 AP at low power and far away.
- Zigbee on 25 (cleanest slot; Zigbee carries broadcasts and many mains
  routers, so it gets priority). Thread on 15.
- Fallback if Wi-Fi 11 is unavoidable: Zigbee 25, Thread 20. Never put
  Zigbee and Thread on the same channel unless a single-radio build forces
  it — co-channel PANs share one CSMA domain and degrade together.
- Record the plan (Wi-Fi channels, Zigbee channel/PAN ID, Thread channel
  and dataset) with the network backups so a rebuild reproduces it.

## Power and placement rules

- Start each coordinator/border-router radio at 5–9 dBm; raise only if
  edge routers show persistent low RSSI after placement is fixed.
- Put each stick on a 1–2 m USB 2.0 extension cable, away from USB 3.0
  ports/drives, metal cases, and the Wi-Fi AP (at least 1 m, ideally more).
- Separate the two stick antennas by 30 cm or more, oriented parallel;
  keep both clear of microwaves, baby monitors, and 2.4 GHz cameras.

## How to verify with scans

1. Baseline: run a Wi-Fi analyzer (phone/laptop) plus an 802.15.4 energy
   scan (Zigbee2MQTT/ZHA scan, OTBR `scan-energy` / Network Manager scan)
   and note the per-channel RSSI floor.
2. Deploy the plan, then re-scan: target channels should read at least
   10–15 dB below adjacent Wi-Fi lobes during busy hours.
3. Soak-test 24–48 h: watch Zigbee LQI/route flaps and Thread MLE
   retransmit/child-table churn in normal traffic; re-scan once under load.

## Effort estimate

No firmware or hardware work. About 1–2 hours: survey, set channels, place
sticks, verify. Channel migration on an established Zigbee network can add
re-pairing time for stubborn devices.

## Pros

- Cheap, reversible, and multiplies the reliability of every other idea.
- Uses only stock scans and settings; no custom firmware to maintain.
- Diagnosable: scans give an objective before/after record.

## Cons

- Cannot fix single-radio timeslicing loss; only reduces the ambient
  interference component of it.
- Shared buildings limit control over neighbours' Wi-Fi; plan degrades to
  best-effort on crowded spectrum.
- Channel moves can force re-pairing of picky end devices.

## Verdict

Adopt regardless of architecture choice. Channel, power, and placement
discipline is the lowest-cost reliability win available and a precondition
for judging whether any dual-radio or future dual-mode setup actually works.
