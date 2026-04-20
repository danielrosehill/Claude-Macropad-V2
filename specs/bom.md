# Bill of Materials — DIY Claude Macropad

Three tiers. Pick one based on how much you want to build vs. buy.

All prices are approximate in USD (2026-Q2), shipped to Israel. Expect ±25% variance by vendor and whether you already have the tooling (soldering iron, 3D printer).

---

## Tier 1 — "Solder-free" DIY (≈ $45–70)

A hand-wired or pre-built QMK/VIA pad you flash and label yourself. No PCB design, no custom case.

### Parts

| # | Part | Qty | Est. unit | Notes |
|---|------|-----|-----------|-------|
| 1 | Pre-built QMK/VIA 3×3 or 3×4 macropad (Keychron Q0, YMDK Amag23, AliExpress generic) | 1 | $40–60 | Must support VIA in the browser. Confirm before buying. |
| 2 | Legended keycaps (MX-compatible, 1u) | 9–12 | $0.50–2 each | WASDKeyboards custom set, or blank + DIY labels. |
| 3 | Blank keycaps (fallback) | spare set | $8 | In case the custom set arrives wrong. |
| 4 | USB-C cable, braided, 1–1.5 m | 1 | $8 | If not included. |

### Tools
- None required beyond a keycap puller (usually included).

### Firmware
- VIA web configurator — no flashing required for most boards.
- Optional: QMK Raw HID for status-LED integration (requires toolchain setup — see Tier 2).

---

## Tier 2 — Custom firmware + 3D-printed case (≈ $60–110)

Stock PCB, your own case and keycaps, QMK compiled from source so the status LED integration works.

### Electronics

| # | Part | Qty | Est. unit | Notes |
|---|------|-----|-----------|-------|
| 1 | Amag23 / GH60 satellite / BM16A PCB (QMK-supported) | 1 | $25–40 | Must have at least 9 keys + 1 RGB-capable pin. |
| 2 | Kailh Choc or Cherry MX-style switches (Brown/Red) | 9–16 | $0.40–0.80 | Browns for tactile approve-key feel. |
| 3 | Keycaps, legended or blank | 9–16 | $0.50–2 | — |
| 4 | WS2812B addressable LED strip (1 m, 60 LEDs/m) | 1 | $5 | Cut to length for status bar. |
| 5 | USB-C breakout board (if PCB is micro-USB) | 0–1 | $3 | Optional. |
| 6 | Rubber feet, self-adhesive | 4 | $0.10 | — |
| 7 | M3 heat-set inserts + M3×6 screws | 8 | $5 (kit) | For the case. |

### Case
| # | Part | Qty | Est. unit | Notes |
|---|------|-----|-----------|-------|
| 8 | 3D-printed case (PLA or PETG) | 1 | ~$3 filament | Print-at-home or JLCPCB/Shapeways. |
| 9 | Clear acrylic diffuser strip (for underglow) | 1 | $2 | Or translucent filament. |

### Tools
- Soldering iron + solder (for LED strip leads and optional USB-C board).
- 3D printer or print service.
- Flush cutters.

### Firmware
- QMK Firmware (`qmk_firmware`) — custom keymap with `raw_hid_receive()` handler.
- Host-side helper: ~30-line Python script using `hidapi` to push state bytes from Claude Code hooks.

---

## Tier 3 — Fully custom PCB + ESP32 / RP2040 (≈ $90–180)

Your own PCB, your own firmware, full bidirectional comms, optional OLED. This is the "B3" path in `hardware-directions.md`.

### Electronics — core

| # | Part | Qty | Est. unit | Notes |
|---|------|-----|-----------|-------|
| 1 | **MCU** — Seeed XIAO ESP32-S3 **or** Raspberry Pi Pico (RP2040) | 1 | $5–8 | ESP32 if you want Wi-Fi/BLE status push. RP2040 if you want the simplest QMK path. |
| 2 | Custom PCB (JLCPCB, 2-layer, 100×80 mm, black soldermask) | 5 (min order) | $2 each + shipping | Effective cost ~$25 with DHL. |
| 3 | Kailh hotswap sockets (MX) | 9–12 | $0.15 | Lets you swap switches without desoldering. |
| 4 | Cherry MX / Kailh switches | 9–12 | $0.40–0.80 | Pick per key: Brown for approve, Red for fast-repeat, Clicky for "stop". |
| 5 | Keycaps (MX, 1u, legended or blank + DIY labels) | 9–12 | $0.50–2 | — |
| 6 | 1N4148 diodes (through-hole or SMD SOD-123) | 9–12 | $0.03 | One per key for matrix. |
| 7 | WS2812B LEDs (SMD 5050, reverse-mount for underglow) | 10–20 | $0.10 | Underglow + dedicated status zone. |
| 8 | 0.91" 128×32 OLED (I²C, SSD1306) | 0–1 | $3 | Optional. Shows current Claude state as text. |
| 9 | EC11 rotary encoder (with push-switch) | 0–1 | $1 | Optional. Scroll transcript / volume. |
| 10 | USB-C breakout (if MCU lacks native) | 0–1 | $3 | XIAO already has USB-C. |
| 11 | Decoupling caps (100 nF ×4, 10 µF ×2) | ~6 | $0.05 | — |
| 12 | Pull-up resistors (4.7 kΩ ×2 for I²C) | 2 | $0.05 | If adding OLED. |

### Case + mounting
| # | Part | Qty | Est. unit | Notes |
|---|------|-----|-----------|-------|
| 13 | 3D-printed two-piece case (top plate + base) | 1 | ~$5 filament | FDM PLA fine. Resin for crisper edges. |
| 14 | Clear/frosted acrylic or PETG light-pipe strip | 1 | $2 | Underglow diffuser. |
| 15 | M3 heat-set inserts | 8 | $0.25 | — |
| 16 | M3×6 / M3×8 socket-head screws | 8 | $0.10 | — |
| 17 | 3M VHB tape OR silicone feet | — | $3 | Non-slip base. |

### Tools
- Soldering iron (temp-controlled, 350°C for hotswap sockets).
- Solder paste + hot air gun (nice-to-have, not required).
- Multimeter.
- 3D printer or print service.
- Flush cutters, tweezers.
- Computer for flashing (ESP32 → esptool / `idf.py`; RP2040 → UF2 drag-drop or QMK).

### Firmware options
- **QMK** (RP2040 only in this list) — mature, Raw HID works out of the box.
- **ZMK** (supports RP2040 + some ESP32 variants) — BLE-friendly, newer.
- **Custom ESP-IDF / Arduino** (ESP32-S3) — for the "push status over MQTT/HTTP" path. Trades off complexity for "Claude state visible on every device in the house."

---

## Optional accessories (any tier)

| Part | Est. | Why |
|------|------|-----|
| Coiled/braided USB-C cable (1 m) | $15–25 | Aesthetic; matches the banner vibe. |
| Acrylic tented wedge / wrist rest | $10 | 8° tilt reads better for a one-handed pad. |
| Magnetic cable connector | $8 | So you can yank it off the desk without pulling the pad. |
| Spare hotswap sockets (×20) | $3 | You will break one. |

---

## Ordering notes (Israel-specific)

- **PCBs**: JLCPCB → DHL to IL is ~4–7 days, ~$25 shipping for a small board. PCBWay comparable.
- **Switches/keycaps**: KBDfans, Drop, or local IL keyboard communities (check the mechkeys groups). AliExpress standard-shipping adds 3–5 weeks; Choice/DHL faster.
- **MCUs**: Seeed XIAO is easiest from AliExpress official store. Adafruit / Pimoroni via EU distributors if you want faster delivery.
- **3D printing**: Local Jerusalem/Tel Aviv makerspaces will print a case in PLA for ₪30–80.
- **VAT/customs**: Orders under ~$75 usually clear without extra fees; above that, expect 17% VAT at pickup.

---

## Reference builds

- **Cheapest working macropad** (Tier 1): Amag23 + blank caps + VIA → $45, one afternoon.
- **Recommended DIY** (Tier 2): Amag23 + QMK from source + WS2812 status strip + printed case → $75, one weekend.
- **"Dream build"** (Tier 3): XIAO ESP32-S3 + custom PCB + OLED + encoder + underglow → $150, two weekends.
