# Spec: ESP32 Standard

The baseline DIY build. A competent Claude companion macropad with keys, RGB, and a status indicator — nothing theatrical.

## Goals

- 10–12 programmable keys for the shortcuts defined in [`../ideated-shortcuts.md`](../ideated-shortcuts.md).
- Bidirectional USB HID so the host can push Claude-state updates to onboard LEDs.
- Buildable in a weekend. Off-the-shelf parts only.

## Bill of materials

| Part | Notes |
|------|-------|
| ESP32-S3 dev board (e.g. Seeed XIAO ESP32-S3, Lolin S3 Mini) | Native USB, enough GPIO, cheap |
| 12× mechanical key switches (MX or Choc low-profile) | Whatever feels good |
| 12× keycaps (blank or legended — see Direction B1 in `hardware-directions.md`) | Legended is worth the extra cost |
| WS2812B per-key RGB (12 LEDs) | Or a single RGB strip mounted behind keys |
| 1× dedicated status RGB LED (WS2812 or a beefy 5mm RGB) | Independent of key backlighting |
| 1N4148 diodes (if using a matrix) | 12 keys can also go direct-pin on ESP32-S3 |
| USB-C breakout | Direct to board if the dev board has USB-C |
| 3D-printed case | Or a cheap aluminum project box |

**Approx cost:** $35–60 depending on switches.

## Firmware

Two viable stacks:

1. **ZMK on ESP32** — emerging support, BLE + USB HID, good for macropads. RGB + Raw HID possible.
2. **ESP-IDF + TinyUSB HID** — full control, custom Raw HID channel. More work, more flexibility. Recommended for this build since we want bidirectional host comms.

Firmware responsibilities:
- Standard keyboard HID interface — each key emits a configurable keycode or macro.
- Raw HID interface — listens for 1-byte state packets from the host (`0=idle`, `1=working`, `2=awaiting`, `3=error`) and drives the status LED accordingly.
- RGB key backlighting — static or layer-indicator.

## Host integration

- `macropad-led` CLI on the host (Python + `hidapi`, ~30 lines) sends state bytes over Raw HID.
- Claude Code hooks in `~/.claude/settings.json` invoke `macropad-led` on `Notification` / `Stop` / `UserPromptSubmit` / `SessionEnd`. See [`../hardware-directions.md`](../hardware-directions.md) for the hook config.

## Status LED states

| State | Color | Pattern |
|-------|-------|---------|
| Idle | Dim white | Steady |
| Working | Blue | Slow pulse (1 Hz) |
| Awaiting feedback | Amber | Fast pulse (2 Hz) |
| Error | Red | Steady |

## Shortcut mapping

Defer to the 10 shortcuts in `../ideated-shortcuts.md`. Extra keys can be loaded from the "Suggested Additional Shortcuts" section.

## Stretch (still in scope for "standard")

- Single rotary encoder for scrolling / volume.
- Small OLED (128×32) displaying current Claude state in text.

---
*Authors: Claude (initial draft)*
