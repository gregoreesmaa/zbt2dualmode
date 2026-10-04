# Sources

Primary sources consulted (fetch/search, Oct 2026). Prefer these over
second-hand summaries.

## Official / Nabu Casa

- ZBT-2 product + FAQ — <https://www.home-assistant.io/connect/zbt-2/>
- About Home Assistant Connect ZBT-2 (support) — <https://support.nabucasa.com/hc/en-us/articles/31313065259421-About-Home-Assistant-Connect-ZBT-2>
- silabs-firmware-builder (manifests `manifests/nabucasa/zbt2/`, Docker, SDK) — <https://github.com/NabuCasa/silabs-firmware-builder>
- universal-silabs-flasher (`--profile zbt2`) — <https://github.com/NabuCasa/universal-silabs-flasher>
- SkyConnect/ZBT-1 multiprotocol notice (staff summary in community thread) — <https://community.home-assistant.io/t/home-assistant-skyconnect-home-assistant-connect-zbt-1-official-zigbee-and-or-thread-usb-radio-dongle-from-nabu-casa/433594?page=10>
- ZBT-2 launch discussion (incl. MG24 / Zigbee 4.0 / Suzi notes) — <https://community.home-assistant.io/t/the-best-gets-better-home-assistant-connect-zbt-2/953028?page=4>

## Firmware / tooling

- darkxst/silabs-firmware-builder (MG21-era `ncp-uart-hw` / `rcp-uart-802154` / `ot-rcp`) — <https://github.com/darkxst/silabs-firmware-builder>
- Nerivec/silabs-firmware-builder fork — <https://github.com/Nerivec/silabs-firmware-builder>
- Nerivec/silabs-firmware-recovery (chip table, NVM3/APP GBLs) — <https://github.com/Nerivec/silabs-firmware-recovery>
- ot-efr32 OpenThread version-match notes — <https://github.com/openthread/ot-efr32>
- Gecko SDK — <https://github.com/SiliconLabs/gecko_sdk>
- Zigbee2MQTT EmberZNet adapter docs (incl. "Multiprotocol firmware is not supported") — <https://www.zigbee2mqtt.io/guide/adapters/emberznet.html>
- Community ZBT-2 Thread + Zigbee portable notes (pin maps, flash commands) — <https://github.com/silenthooligan/code-sharing> (`ha-connect-portable/zbt-2-thread`, `zbt-2-zigbee`)

## Hardware / silicon

- EFR32MG24 reference / BRD4188B radio-board manual (RF section) — via Silicon Labs docs (`manuals.plus` mirror consulted; re-verify against docs.silabs.com)
- cnx-software ZBT-2 launch specs — <https://www.cnx-software.com/2025/11/20/home-assistant-connect-zbt-2-zigbee-thread-usb-adapter/>
- SmartHomeScene ZBT-2 teardown/review + coordinator table — <https://smarthomescene.com/reviews/home-assistant-zbt-2-zigbee-and-thread-coordinator-review/>
- XDA ZBT-2 piece (FAQ quotes) — <https://www.xda-developers.com/switched-home-assistant-connect-zbt-2-zigbee/>
- Hackster ZBT-2 launch note — <https://www.hackster.io/news/home-assistant-launches-the-bigger-faster-connect-zbt-2-zigbee-thread-dongle-d46ed0c4136f>

## Unresolved / not directly verified

- Measured MultiPAN degradation metrics beyond the qualitative ZBT-1 statement.
- Any supported/validated custom dual-protocol firmware path on ZBT-2.
- Full official schematic/netlist (relies on teardowns + manifests).
- AN1333 PDF + concurrent-multiprotocol blog bodies (403 at research time).
- `flasher.py` ZBT-2 device-class body (README-grounded only).
