# 01 — The official limitation: what the ZBT-2 page says

Source: <https://www.home-assistant.io/connect/zbt-2/> (FAQ "Why do I have to
choose between Zigbee … Why not both?"), corroborated by the
[Nabu Casa support article](https://support.nabucasa.com/hc/en-us/articles/31313065259421-About-Home-Assistant-Connect-ZBT-2)
and independent coverage (XDA, Hackster, cnx-software, resellers).

## The FAQ position (paraphrased fromsnippet quotes, wording consistent across outlets)

> "Connect ZBT-2 cannot do both Zigbee and Thread simultaneously. This
> technology is often called 'multiprotocol' or 'MultiPAN'. Though it is
> theoretically possible with the hardware within Connect ZBT-2, in our
> experience, this functionality doesn't work well, and we don't plan to
> implement it. We previously thoroughly tested multiprotocol with our Connect
> ZBT-1 adapter and found its operation to be inconsistent, often causing
> device stability issues."

What this tells you:

1. **Single-radio, single-firmware at a time.** You flash the stick for
   Zigbee *or* Thread and the host stack (ZHA/Zigbee2MQTT *or* OpenThread
   Border Router) owns it. Switching protocols = reflash + re-pair/migrate.
2. **"Theoretically possible" ≠ supported.** Nabu Casa explicitly scopes the
   ZBT-2 out of MultiPAN despite the MG24 being capable of RCP-MultiPAN builds.
3. **Evidence cited is the ZBT-1 generation**, not a ZBT-2-specific benchmark.
   No public ZBT-2 MultiPAN test report was found — the decision is inherited
   from ZBT-1/SkyConnect testing. Treat "ZBT-2 can't" as *policy from prior
   learning*, not a published ZBT-2 measurement.
4. **Matter enters via Thread.** "Matter over Thread" needs a Thread network;
   Matter-over-WiFi does not need the stick. So "Zigbee + Matter at once" in
   practice means "Zigbee + Thread RCP at once" on one radio — the unsupported
   combo.

## Adjacent official statements found

- Support article: "supports either the Zigbee or Thread protocol", "running
  Zigbee and Thread at the same time on the same adapter is not supported",
  no Bluetooth / no Z-Wave.
- Reseller/coverage echo: "does not support running Zigbee and Thread
  simultaneously … real-world performance has proven unreliable."
- Positioning vs ZBT-1: ~4× faster communication, 460,800 bps host baud
  (vs ZBT-1's lower rate), Zigbee 4.0-ready by software update; still one
  protocol at a time.

## What the page does *not* give you

- No channel/PAN/timeslice numbers, no packet-loss traces, no CPC-host
  architecture diagram. For the *why*, see `02-nabucasa-learnings.md`.
- No schematic or MG24-vs-ESP32-S3 firmware split documentation. See
  `03-firmware-inventory.md` and `04-hardware-schematics.md`.
