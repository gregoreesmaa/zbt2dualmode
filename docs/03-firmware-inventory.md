# 03 — Firmware inventory: everything you need to rebuild ZBT-2 firmware

Chip (corrected): **Silicon Labs EFR32MG24A420F1536IM40** — Series-2,
Cortex-M33, 256 KB RAM, 1536 KB flash. (ZBT-1/SkyConnect was EFR32MG21; do
not use MG21 manifests for the ZBT-2.) Source: `device:` line 2 of all four
`zbt2` manifests in `NabuCasa/silabs-firmware-builder`, plus the
[Nerivec/silabs-firmware-recovery](https://github.com/Nerivec/silabs-firmware-recovery)
chip table and Zigbee2MQTT's adapter list.

## Official build repo

<https://github.com/NabuCasa/silabs-firmware-builder> — manifests in
`manifests/nabucasa/zbt2/`:

| Manifest | Base project | What it builds |
|---|---|---|
| `zbt2_zigbee_ncp.yaml` | `src/zigbee_ncp` | EmberZNet Zigbee NCP (EZSP + XNCP extensions), factory default |
| `zbt2_openthread_rcp.yaml` | `src/openthread_rcp` | OpenThread RCP, **460800 baud** |
| `zbt2_router.yaml` | `src/zigbee_router` | Zigbee router + ZCL LED clusters |
| `zbt2_bootloader.yaml` | bootloader | Gecko bootloader, **115200 baud** |

Build environment: `simplicity_sdk: 2026.6.1`, GCC `14.2.1.20241119`,
Docker `ghcr.io/nabucasa/silabs-firmware-builder --manifest ... --output gbl`.
Proprietary steps use `slc` + Simplicity Commander. OpenThread RCP and the
host OTBR must share the same OpenThread commit (see
[openthread/ot-efr32](https://github.com/openthread/ot-efr32) `src/README.md`);
SiSDK-generated RCP is preferred over hand-rolled trees.

Board defines you must carry into any custom `.slcp` (from the manifests):
HFXO 39 MHz CTUNE 108; EUSART0 VCOM TX PA8 / RX PA7 / CTS PA5 / RTS PA0;
WS2812 on EUSART1 + I2C0 + pinhole button PC5; unique
`SL_APPLICATION_PRODUCT_ID` GUID (anti-cross-flash); `sdk_extensions`
(`nabucasa_hardware`, `xncp`, `led_effects`, watchdog) + `sdk_patches`.

## Flashing / recovery

- Flash: [NabuCasa/universal-silabs-flasher](https://github.com/NabuCasa/universal-silabs-flasher)
  — use `--profile zbt2`, e.g.
  `flash --profile zbt2 --firmware zbt2_openthread_rcp_*.gbl`.
  GBLs are checksum-validated XMODEM into the Gecko bootloader; `write-ieee`
  subcommand manages the EUI-64. (README-grounded; `flasher.py`
  `DEVICE_SPECIFIC_FLASHERS` registry not line-verified here.)
- Recovery: [Nerivec/silabs-firmware-recovery](https://github.com/Nerivec/silabs-firmware-recovery)
  — NVM3-clear defaults NCP 32768 / RCP 40960, plus APP-clear GBLs.
- Community path for the Thread role while stock USB-CDC bridge is alive:
  `universal-silabs-flasher --device /dev/serial/by-id/usb-Nabu_Casa_ZBT-2_*-if00 --bootloader-reset rts_dtr flash ...`
  (see `ha-connect-portable` community notes).

## Legacy / community builders (context, not ZBT-2 targets)

- [darkxst/silabs-firmware-builder](https://github.com/darkxst/silabs-firmware-builder):
  MG21-era per-device `firmware_builds/` dirs (`ncp-uart-hw` = pure Zigbee,
  `rcp-uart-802154` = RCP MultiPAN, `ot-rcp` = Thread-only). No `zbt2` dir —
  do not flash these onto a ZBT-2.
- [Nerivec/silabs-firmware-builder](https://github.com/Nerivec/silabs-firmware-builder)
  fork and firmware-recovery images fill the gap for newer parts.

## Firmware image types, plainly

- **Zigbee NCP**: full EmberZNet stack on-chip; host (ZHA/Z2M) speaks EZSP.
  Reflash to update; network size bounded by on-chip RAM.
- **OpenThread RCP**: chip does 802.15.4 PHY/MAC only; Thread runs on the
  host (otbr-agent via Spinel). Thread-only, single-PAN.
- **RCP-MultiPAN**: the concurrent vehicle (Zigbee `zigbeed` + Thread
  `otbr-agent` multiplexed through host `cpcd`). This is the rejected
  architecture — see `02-nabucasa-learnings.md`.
