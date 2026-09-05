# Corne 17 ZMK firmware

The existing 5-column keymap is shared by both the original two-part setup and
the optional HOLYIOT USB dongle setup.

## Safe bootloader test first

1. Build the workflow and download `corne_left_bootloader_test`.
2. Flash it only to the current left/central half as usual.
3. With the keyboard idle on the base layer, hold `Q + W + E + R + Space`
   together (positions 0, 1, 2, 3, and 32).
4. The left half should restart and its UF2 drive should appear. Do not proceed
   to the dongle migration until this succeeds.

The combo is restricted to layer 0 and requires one second of prior idle time
to make accidental activation unlikely. ZMK
always executes a reset behavior invoked by a combo on the split central. Thus,
after migration, the same combo enters the dongle bootloader rather than either
keyboard half.

## Migrate to the dongle

Changing split roles requires clearing old BLE bonds on all three devices.

1. Flash `settings_reset_nice_nano` to the left half, then to the right half.
2. Flash `settings_reset_holyiot_dongle` to the HOLYIOT dongle.
3. Flash `corne_left_peripheral` to the left half.
4. Flash `corne_right_peripheral` to the right half.
5. Flash `corne_holyiot_dongle_central` to the dongle.
6. Power-cycle the two halves and reconnect the dongle. The halves should bond
   automatically to the dongle. Remove the old host pairing if it remains.

The dongle image assumes the HOLYIOT nRF52840 has the already-tested MakerDiary
MDK/Adafruit-compatible UF2 bootloader: Nordic MBR at `0x00000000`, application
at `0x00001000`, settings at `0x000cc000`, and bootloader starting at
`0x000e0000`. Do not flash this image if the bootloader layout differs.

The original `corne_left_bootloader_test`, `corne_right_legacy`, and
`settings_reset_nice_nano` builds remain available as a rollback path.
