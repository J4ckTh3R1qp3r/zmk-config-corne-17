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

The HOLYIOT board definition explicitly uses its built-in 32.768 kHz crystal.
The dongle is configured for exactly three simultaneous BLE links (two split
halves and at most one host), while retaining five host profiles. Its two split
connections use a 7.5 ms interval with zero peripheral latency. The pinned ZMK
revision also includes the Zephyr controller fix for split-central prepare
pipeline lockups.

## Test the corrected dongle image

1. Enter the dongle UF2 bootloader with `Q + W + E + R + Space`. If the split
   links are too unreliable for the combo, hold the dongle's boot button while
   plugging it in.
2. Copy only `corne_holyiot_dongle_central.uf2` to the UF2 volume. Do not flash
   either half for this first test.
3. Power-cycle the dongle and both halves. Existing bonds should be retained.
4. Type continuously with both halves for at least two minutes, including fast
   alternating left/right keys. Then switch either half off and on and confirm
   that it reconnects and resumes typing.

If a half does not reconnect, run the settings-reset images on all three
devices once and repeat the migration sequence above. A reset is not needed
merely to update this dongle image.

The original `corne_left_bootloader_test`, `corne_right_legacy`, and
`settings_reset_nice_nano` builds remain available as a rollback path.
