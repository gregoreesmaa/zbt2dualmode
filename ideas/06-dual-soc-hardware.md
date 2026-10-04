# Idea 06 — Single-enclosure dual-SoC hardware

## Proposal summary

Ship one USB product containing two EFR32MG24 radios behind an internal
USB hub: radio A flashed as Zigbee NCP, radio B flashed as OpenThread
RCP. Each radio presents its own USB serial port to the host, so ZHA /
Zigbee2MQTT and OTBR attach independently with stock firmware. This is
the IKEA Dirigera pattern (two separate EFR32MG21 modules for dedicated
Zigbee + Thread) repackaged as a single USB device.

## Board-level design sketch

- **USB uplink:** one USB-C connector into a USB 2.0 hub chip (e.g.
  FE2.1-class 4-port or equivalent), exposing two downstream CDC ports,
  one per radio path.
- **Radio path x2:** EFR32MG24A420F1536IM40 module per protocol, each
  with its own 39 MHz crystal, decoupling, and SWD pads for recovery.
  Either direct native USB (if module pins allow) or the proven
  ESP32-S3 / CP210x-class USB-serial bridge per leg, reusing the
  existing ZBT-2 UART pin map (460800 baud app, 115200 baud bootloader).
- **Antenna isolation:** two 2.4 GHz antennas with orthogonal
  orientation and maximum PCB separation (opposite ends, ground-plane
  slot or shield wall between front ends); independent channels assumed
  (e.g. Zigbee 15, Thread 25), since filtering cannot fix co-channel
  operation.
- **Power:** single 5 V USB feed with per-radio LDO and bulk
  capacitance sized for simultaneous Tx peaks; hub must budget ~500 mA
  total; USB 3.x host ports preferred for current headroom.

## Cost / effort estimate

- **BOM delta vs one ZBT-2:** second MG24 module, hub chip, second
  antenna and RF section, larger PCB and enclosure; roughly 1.6-2x the
  single-stick hardware cost before tooling.
- **Firmware:** near zero new work — both images are existing stock
  targets in NabuCasa/silabs-firmware-builder, flashed per-leg with
  universal-silabs-flasher.
- **Effort:** new PCB, RF layout and certification (FCC/CE/RED per
  intentional radiator), USB hub bring-up, per-port serial-path UX;
  weeks of layout plus certification lead time, not a firmware patch.

## Pros

- No MultiPAN timeslicing: each PAN keeps its own channel, PAN ID,
  keys, and always-on Rx, dodging the ZBT-1 failure mode.
- One enclosure, one USB port, one SKU vs two sticks and two cables.
- Stock host stacks and update story: ZHA/Z2M plus OTBR attach to two
  stable `/dev/serial/by-id` paths behind the hub.
- Failures isolatable per radio; either leg reflashable independently.

## Cons (vs two separate sticks)

- Higher upfront cost and lead time: new board, enclosure, RF
  certification; two sticks already exist and need no hardware work.
- Colocated antennas in one shell desense more than two sticks on
  extension cables; isolation is layout-constrained.
- Shared single point of failure: hub chip or USB uplink down takes
  both networks out; two sticks survive one stick failing.
- Less placement flexibility: two sticks can be separated by meters,
  this cannot; still consumes two logical ports worth of power.

## Verdict

Technically sound and the only honest "one product, both protocols"
answer: it concedes the single-radio fight and buys a second radio.
Recommend only as a new-product bet where the single-SKU UX justifies
board and certification cost; for existing installs, two separate
sticks deliver the same air-interface outcome today at lower cost.
