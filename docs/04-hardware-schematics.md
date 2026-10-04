# 04 — Hardware: SoC, bridge wiring, and why one radio blocks dual-mode

## What the ZBT-2 is (verified)

- **Radio SoC**: EFR32MG24A420F1536IM40 (Series-2, Cortex-M33 @ 78 MHz,
  256 KB RAM / 1536 KB flash, +10 dBm class, external antenna ~+4.16 dBi).
  Sources: NabuCasa firmware manifests, Nerivec recovery chip table,
  SmartHomeScene teardown/review, Zigbee2MQTT adapter list, cnx-software specs.
- **USB bridge**: ESP32-S3 acting as **USB-to-serial only** (its Wi-Fi/BT are
  disabled in the official role). Sources: Nabu Casa support article
  ("MG24 … and ESP32-S3 (USB-serial bridge)"), cnx-software, community
  `ha-connect-portable` notes.
- **Host link**: 460800 baud 8-N-1 to the MG24 application firmware;
  bootloader at 115200 baud.
- **Community pin map** (from `ha-connect-portable` ZBT-2 notes — community,
  not an official schematic; verify against your board before driving pins):

  | ESP32-S3 GPIO | MG24-side | Function |
  |---|---|---|
  | 14 (UART RX) | UART TX | host → radio |
  | 13 (UART TX) | UART RX | radio → host |
  | 4 | RESETn | radio reset, active-low open-drain |
  | 10 | PA6 | bootloader trigger, active-low |

  Manifest board defines add: EUSART0 VCOM PA8/PA7, CTS PA5, RTS PA0;
  HFXO 39 MHz CTUNE 108; WS2812 LED on EUSART1; I2C0; pinhole button PC5.
- **SWD pads** for the MG24 are exposed on the PCB (review photos) for
  debug/flash recovery.

No official Nabu Casa schematic PDF was found. The actionable hardware refs
are the teardown photos, the community pin tables above, the firmware
manifests' `c_defines`, and Silicon Labs' EFR32MG24 reference manuals /
radio-board schematics (e.g. BRD4188B RF section) for the RF front end.

## Why a single radio blocks "true" simultaneous dual-mode

Both MG21 and MG24 expose **one 2.4 GHz DSSS-OQPSK 802.15.4 transceiver**
(datasheet §4.1.5.1.2): one frequency/channel and one PHY frame at a time.
"Multiprotocol" is software sharing, not two radios. (Contrast: IKEA Dirigera
uses two separate EFR32MG21 modules for dedicated Zigbee + Thread.)

Consequences for a dual-stack rewrite:

1. **Channel/PAN/key split.** Zigbee and Thread are separate PANs with
   separate PAN IDs, network keys, and (usually) separate channels and extended
   PAN / Thread network names. Default MultiPAN RCP requires **all PANs on
   the same channel**; independent channels need "Concurrent Listening"
   (Series-2-only, GSDK 4.2+ experimental), which retunes and time-shares
   receive windows. Either way Rx frames demux by PAN ID through one shared
   CSMA-CA channel access — one Tx/Rx at a time.
2. **Timeslicing costs.** Dynamic Multiprotocol uses the RAIL scheduler to
   slice the radio (the Zigbee+BLE reference uses ~1 ms-scale slots); even
   same-channel concurrent mode arbitrates every frame. Different-channel
   operation drops/sacrifices receive windows → added latency, missed beacons
   and polls, battery-end-device and router strain, throughput loss. Community
   and vendor threads report Zigbee broadcast/beacon starvation and "extremely
   unreliable" Thread under MultiPAN load.
3. **Broken always-on assumption.** Both stacks assume always-on Rx on their
   channel/PAN/key (Zigbee mains routers, broadcasts, Touchlink/beacons;
   Thread MLE/CSL/data-poll SEDs). Sharing breaks that assumption; forcing a
   shared channel couples your Zigbee channel plan (commonly 15/20/25) to your
   Thread channel plan and interference environment.
4. **RAM/flash wall.** MultiPAN guidance targets ~1024 KB-flash-class parts,
   which is part of why MG21-class designs moved to MG24 — and the MG24 is
   still one radio.

Net: the silicon can *construct* a dual-stack image, but the air interface
cannot serve two always-on networks without compromise. That is the physical
half of Nabu Casa's "theoretically possible, doesn't work well."
