# Idea 08 — Network-attached / remote radios

## Proposal summary

Decouple the 802.15.4 radios from the Home Assistant host: run the
Zigbee coordinator and the Thread RCP as network-attached devices and
reach them over TCP/serial-to-network proxies. Each radio still runs
one stock single-protocol image (Zigbee NCP via EZSP, OpenThread RCP
via Spinel); only the transport moves off USB. Examples: a ser2net /
socat forwarder on a Pi near the mesh, an ESPHome `stream_server`
serial bridge for portable setups, or a PoE-powered coordinator board
(SLZB-06 / TubeZB class) with wired Ethernet and a fixed IP.

## Why placement freedom matters for 2.4 GHz mesh quality

The HA host usually lives where the router and NAS live: a basement,
closet, or rack full of USB 3.0 noise, metal, and Wi-Fi overlap. Both
Zigbee and Thread are 2.4 GHz 802.15.4 meshes whose quality depends on
coordinator centrality, height, and distance from interferers. Moving
each radio to a central, elevated, low-noise spot shortens the average
first hop, reduces retransmissions, and lets Zigbee and Thread sit on
separate channels away from the site's Wi-Fi 1/6/11 plan — gains no
firmware rewrite can match on a badly placed stick.

## Transport options and failure modes

- **ser2net / socat TCP forwarder.** Proven for Zigbee (ZHA
  `socket://`, Zigbee2MQTT TCP coordinator). Failure modes: silent
  TCP stall looks like a dead stick; EZSP timeouts on jitter;
  reconnect must re-open the exact serial params (RCP: 460800 baud).
- **ESPHome `stream_server` bridge.** Portable Wi-Fi placement with
  stock ESPHome. Failure modes: Wi-Fi latency spikes break Spinel
  timing; DHCP moves need mDNS or static leases; unencrypted serial
  over the air unless tunneled.
- **PoE coordinator (wired Ethernet).** Best latency and power story;
  one cable for data + power. Failure modes: still a LAN dependency;
  switch reboot drops both meshes at once; OpenThread RCP over TCP
  to otbr-agent is fragile and largely outside the supported OTBR
  matrix, which expects a local serial device.
- **Common to all:** the network becomes a single point of failure
  for both meshes; encrypt or isolate the serial-over-LAN path.

## Effort estimate

Low firmware effort (stock NCP/RCP images, no custom build), medium
integration effort: one forwarder or PoE board per protocol, static
IPs, baud/flow-control matching, HA-side `socket://` or TCP config,
and reconnect testing under router/switch reboot. Hours, not weeks.

## Pros

- Optimal radio placement without relocating the HA host.
- Each radio keeps a single PAN on its own channel: no MultiPAN
  timeslicing, no `cpcd` multiplexer.
- Aligns with the two-radio recommendation; reuses stock images and
  the `zbt2` flasher profile.

## Cons

- Adds LAN/Wi-Fi as a failure domain for both meshes.
- Remote OpenThread RCP for OTBR is poorly supported over TCP;
  Zigbee-over-TCP is the mature half of this idea.
- Extra hardware, IPs, and credentials to manage; serial traffic
  needs network security attention.

## Verdict

Recommend for Zigbee-as-remote-coordinator where placement is poor;
treat remote Thread RCP as experimental until OTBR-over-TCP matures.
Where Thread must be reliable today, keep the Thread RCP local on USB
and remote only the Zigbee side — one placement win with no
unsupported transport.
