# Spec: ESP32 OTT-Advanced

OTT build plus genuinely ambitious additions — onboard ML, wake word, animated figurine. For the version that goes from "novelty desk toy" to "standalone AI dictation device."

## Delta from OTT

Inherits everything in [`esp32-ott.md`](./esp32-ott.md) and adds the following.

## Additions

| Feature | Part | Purpose | Author |
|---------|------|---------|--------|
| Onboard speech-to-text | NPU module (Coral Edge TPU via USB, Hailo-8L M.2, or Rockchip RK3566 co-processor) running a Whisper tiny/base model | Pad transcribes mic audio locally and emits text as USB HID keystrokes — works in any app, no host software needed | Daniel |
| Wake-word detection | openWakeWord or Porcupine KWS running on ESP32-S3 or the NPU | "Hey Claude" starts recording; no PTT required | Claude |
| Servo-animated Claude figurine | SG90 micro servo + 3D-printed mount | Figurine nods when working, shakes head on error, points when awaiting feedback | Claude |
| OLED "status face" | 128×64 SSD1306 I2C OLED | Emoticon reflects state: ⏳ working, ❓ awaiting, ✅ done, 💀 error | Claude |
| Rotary encoder | EC11 with push | Scroll Claude transcript; dial recent prompts to resend; push to select | Claude |
| Capacitive touch strip | TTP229 or similar | Swipe to switch between running Claude sessions (each Konsole window = a "channel") | Claude |
| Foot pedal passthrough | 3.5mm TRS jack wired to a GPIO | Stomp-to-approve when hands are on the keyboard | Claude |

## Architecture

The ESP32-S3 alone cannot run Whisper at real-time. Two viable topologies:

### Topology A — ESP32-S3 + external NPU
ESP32-S3 handles keys / RGB / HID / I2S mic. An external USB-attached Coral or Hailo module runs Whisper. The pad exposes itself to the host as a composite USB device (keyboard + audio + storage), but internally one device acts as the host for the NPU.

Complex. Probably requires a Raspberry Pi Zero 2 W or CM4 as the main board instead of ESP32 — at which point the "ESP32" branding is wrong. Rename to **Pi-OTT-Advanced** if going this route.

### Topology B — ESP32-S3 with tinyML only
Skip full Whisper. Run:
- Wake-word (small footprint, runs fine on ESP32-S3)
- Command-word recognition (50–100 word vocab) trained on the Claude shortcut phrases ("approve," "stop," "clear," "compact," etc.)

Not general dictation, but *is* hands-free control over the defined shortcut set. Stays on the ESP32, stays cheap, stays honest to the "ESP32" name.

**Recommendation:** ship Topology B as OTT-Advanced on ESP32. Make Topology A its own spec (`pi-full-dictation.md`) if anyone actually wants it.

## Behavior additions

### Wake-word + command flow (Topology B)
1. ESP32 listens continuously for "Hey Claude" (low-power KWS loop).
2. On wake, status LED goes blue-spin; pad listens for a known command word.
3. On command recognized, pad emits the corresponding HID keystroke sequence (same as pressing the physical key).
4. Unknown command → buzzer chirp + return to idle.

### Figurine animations
Driven by the same Claude-state packets from Raw HID:

| State | Animation |
|-------|-----------|
| Idle | Still, slight idle sway every 30s |
| Working | Slow nod, looping |
| Awaiting feedback | Turn head toward user, hold |
| Error | Slow head-shake |

### OLED status face
Same state inputs, rendered as large emoticons. Optional: show the current prompt's first line in small text under the face.

## Bill of materials delta

| Part | Approx cost |
|------|-------------|
| SG90 servo + mount | $3 |
| SSD1306 OLED | $4 |
| EC11 rotary encoder + knob | $3 |
| TTP229 touch strip | $4 |
| 3.5mm TRS jack for foot pedal | $1 |
| (Topology B) Extra flash for KWS model | — |
| (Topology A) Coral USB Accelerator | $60 |

**Topology B total over OTT:** +$15.
**Topology A total over OTT:** +$75 minimum, plus Pi board swap.

---
*Authors: Daniel (onboard Whisper/NPU concept, PTT + mic integration from OTT), Claude (wake word, figurine animation, OLED, rotary, touch strip, foot pedal, topology analysis)*
