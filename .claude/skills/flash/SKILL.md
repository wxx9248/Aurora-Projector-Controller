---
name: flash
description: Build and upload Aurora Projector firmware to the ESP32, over USB or OTA. User-triggered only (has side effects).
disable-model-invocation: true
---

Build and flash a firmware build to the ESP32-WROOM-32D.

`$ARGUMENTS` may name the build and/or target:
- Build: `standalone` (default) → `aurora-projector.yaml`; `ha` → `aurora-projector-ha.yaml`.
- Target: `usb` (default) → `--device /dev/ttyUSB0`; `ota`/`network` → no `--device` flag;
  or an explicit IP/hostname → `--device <value>`.

Steps:

1. Resolve the entrypoint and target from `$ARGUMENTS` (apply the defaults above if unspecified).
2. Validate first: `esphome config <entrypoint>.yaml`. Stop and report if invalid.
3. For a USB flash, confirm the board is present (`ls /dev/ttyUSB0`) and that **no
   serial log process is holding the port** (`pgrep -f "esphome logs"` — kill it if found,
   since it blocks the upload).
4. Run the flash (long-running; allow several minutes for the first compile):
   - USB: `esphome run <entrypoint>.yaml --device /dev/ttyUSB0`
   - OTA/network: `esphome run <entrypoint>.yaml` (or `--device <ip>`)
5. Report the result. On success, surface the device IP from the output. If the user
   wants logs afterward: USB works for any build, but network logs require the **HA**
   build (the standalone build has no API logging).

Note: `esphome run` is interactive when it can't determine the target — always pass an
explicit `--device` (USB) or rely on OTA discovery to keep it non-interactive.
