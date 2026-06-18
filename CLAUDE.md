# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

ESPHome firmware (YAML, no compiled source) for an **ESP32-WROOM-32D** projector
controller. There is no package manager, test suite, or lint step — the toolchain
is the `esphome` CLI (installed system-wide, e.g. `pacman -S esphome`).

## Architecture

One shared hardware definition is reused by two build variants via ESPHome
[`packages:`](https://esphome.io/components/packages/):

- `common.yaml` — **all** shared config (board, WiFi, OTA, every hardware entity,
  status LED). **Not flashed directly.**
- `aurora-projector.yaml` — standalone build: includes `common.yaml` + `web_server`.
- `aurora-projector-ha.yaml` — Home Assistant build: includes `common.yaml` + encrypted `api`.

**Edit hardware/behavior in `common.yaml`, never in an entrypoint** — otherwise the
two builds drift. Entrypoints should only carry what differs between builds.

## Commands

```sh
esphome config <entrypoint>.yaml                  # validate (use this, not a linter)
esphome run <entrypoint>.yaml --device /dev/ttyUSB0   # build + flash over USB
esphome run <entrypoint>.yaml                     # build + flash over the network (OTA)
esphome logs <entrypoint>.yaml --device /dev/ttyUSB0  # serial logs
```

Replace `<entrypoint>` with `aurora-projector` (standalone) or `aurora-projector-ha` (HA).

## Gotchas

- **The standalone build has no network logging.** It uses `web_server`, not the
  API, so `esphome logs … --device <ip>` fails with "No remote or local logging
  method configured". To read logs over the network, flash the **HA build** (it has
  `api`); otherwise use USB serial. Logging always works over USB.
- **A running serial log holds `/dev/ttyUSB0`** — stop it before flashing over USB.
- The GPIO2 (onboard LED) strapping-pin warning from `esphome config`/`run` is
  benign; the LED works fine as an output.

## Secrets

`secrets.yaml` is git-ignored and must never be committed. `secrets.example.yaml`
is the committed template — `cp secrets.example.yaml secrets.yaml` and fill in.
Generate keys with `openssl rand -base64 32` (API key) and `openssl rand -hex 16` (OTA).

## Git workflow

Work on a feature branch and open a PR via `gh` — do not commit directly to `master`.
