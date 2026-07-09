# Aurora Projector Controller

[ESPHome](https://esphome.io/) firmware for controlling projector hardware with
an **ESP32-C3 Super Mini**. Drives an RGB LED and a motor — controllable from a
browser **or** Home Assistant.

A single shared hardware definition (`common.yaml`) is reused by two build
variants via ESPHome [`packages:`](https://esphome.io/components/packages/), so
the hardware is defined once and never drifts between builds.

## Features

- 🎨 **RGB LED** — full color + brightness via PWM (LEDC), exposed as one light
- ⚙️ **Motor** — on/off switch (easily extended to PWM speed control)
- 💡 **Status LED** — onboard LED indicates WiFi state at a glance
- 🌐 **Two builds** — standalone web UI, or encrypted Home Assistant API
- 🔄 **OTA updates** — flash once over USB, then update wirelessly

## Hardware

Board: **ESP32-C3 Super Mini** (`esp32-c3-devkitm-1`).

| Function          | GPIO      | Notes |
|-------------------|-----------|-------|
| RGB LED — R/G/B   | 0 / 1 / 3 | PWM via transistors (low-side switching) |
| Motor             | 4         | Transistor driver; **add a flyback diode** across the motor |
| Onboard status LED| 8         | Auto-driven WiFi indicator (active-low; strapping pin — fine as output) |

## Repository layout

| File | Purpose |
|------|---------|
| `common.yaml` | Shared config: board, WiFi, OTA, all hardware + status LED. **Not flashed directly.** |
| `aurora-projector.yaml` | **Standalone** build — browser control (`web_server`), no HA. |
| `aurora-projector-ha.yaml` | **Home Assistant** build — encrypted native API. |
| `secrets.example.yaml` | Template for secrets. Copy to `secrets.yaml` and fill in. |
| `secrets.yaml` | Your real WiFi creds / keys — **git-ignored, never committed.** |

Edit hardware/behavior **once** in `common.yaml` and both builds inherit it.

## Setup

1. Install ESPHome (e.g. `pacman -S esphome`, `pip install esphome`, or Docker).
2. Create your secrets file:
   ```sh
   cp secrets.example.yaml secrets.yaml
   ```
   Fill in your WiFi SSID/password. Generate the keys it references:
   ```sh
   openssl rand -base64 32   # api_encryption_key
   openssl rand -hex 16      # ota_password
   ```
3. Plug the board in via USB (native USB; appears as `/dev/ttyACM0`).
4. First flash over USB — pick the build you want:
   ```sh
   esphome run aurora-projector.yaml       # standalone (browser)
   esphome run aurora-projector-ha.yaml    # Home Assistant
   ```
5. After the first flash, updates go over the air — `esphome run` auto-detects
   the device on the network.

## Usage

**Standalone:** after it joins WiFi, open `http://<device-ip>/` for the control
page (the device IP is printed during `esphome run` / `esphome logs`).

**Home Assistant:** HA auto-discovers the device (Settings → Devices &
Services). When prompted, paste the `api_encryption_key` from `secrets.yaml`.

### Useful commands

```sh
esphome config <file>.yaml    # validate
esphome compile <file>.yaml   # build only
esphome run <file>.yaml       # build + upload (USB or OTA)
esphome logs <file>.yaml      # stream device logs
```

## Status LED legend (onboard GPIO8)

| LED | Meaning |
|-----|---------|
| Solid on | WiFi connected (normal) |
| Slow blink (1 Hz) | Connecting / lost connection |
| Fast blink (5 Hz) | Fallback AP mode (router unreachable; hotspot active) |
