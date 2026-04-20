# Spec: ESP32 OTT (Over-The-Top)

Standard build plus deliberately ridiculous companion-device features. Tier 1 OTT — haptics, buzzer, strobe, mic.

## Delta from standard

Inherits everything in [`esp32-standard.md`](./esp32-standard.md) and adds the following.

## Additions

| Feature | Part | Purpose | Author |
|---------|------|---------|--------|
| Haptic feedback | Linear resonant actuator (LRA) + DRV2605L driver, or small ERM coin motor | Vibrate when Claude needs feedback; tactile tick on key press | Daniel |
| Buzzer / small speaker | Piezo buzzer (cheap) or I2S MAX98357A + 1W speaker (nicer tones) | Audible notification when Claude is awaiting input | Daniel |
| Strobe light | 1W–3W high-intensity RGB LED with MOSFET driver, or a COB strip | Comically bright "pay attention" flash on notifications | Daniel |
| Gooseneck / omni mic + PTT button | INMP441 I2S MEMS mic on a gooseneck, dedicated PTT key | Hold-to-talk dictation; pad streams mic audio as USB audio class device or triggers host-side dictation via keybind | Daniel |
| Claude figurine / badge | 3D-printed mini modelled on the Lobe `claudecode-color` icon (see `../assets/claudecode-color.png`) | Cosmetic. Optional servo mount for animated nods (advanced build). | Daniel |

## Behavior additions

### Notification escalation
When Claude awaits feedback, escalate if ignored:

| Elapsed | Action |
|---------|--------|
| 0s | Status LED pulses amber (standard) |
| 5s | Soft haptic tick |
| 15s | Buzzer chirp |
| 30s | Strobe flash + louder buzzer |
| 60s | All-of-the-above on a loop until acknowledged |

Press any key to acknowledge and reset to steady pulse.

### PTT mic modes
Configurable per-key:
- **Push-to-talk** — hold key, mic streams while held, releases on key-up.
- **Toggle** — tap to start, tap to stop. LED indicates recording state.
- **Tap-to-transcribe** — single tap records until silence detected (VAD), then stops.

Base OTT build ships the mic as a **USB Audio Class device** — the OS sees it as a standard input. Claude Code integration is then whatever dictation tool the user prefers (nerd-dictation, Dragon via Wine, a custom Whisper skill, etc.). On-device transcription is in [`esp32-ott-advanced.md`](./esp32-ott-advanced.md).

## TTS voice personalities

Buzzer upgraded to I2S speaker unlocks spoken notifications. Three selectable voice profiles:

| Profile | Voice | Use case | Author |
|---------|-------|----------|--------|
| **The Butler** | Posh English gentleman, measured and polite | Default. "Ahem. Sir, Claude requests your attention." | Daniel |
| **The Alarm** | Shrill, high-pitched, slightly panicked | For errors or when escalation hits maximum. "CLAUDE IS WAITING! CLAUDE IS WAITING!" | Daniel |
| **The Villain** | Deep, menacing, James Earl Jones-adjacent | For long-ignored prompts. "You have been avoiding me. Claude still requires your input." | Daniel |

Implementation options:
- Pre-render a finite set of phrases offline using a cloud TTS (ElevenLabs, OpenAI, Azure) with three voice selections → store as MP3/WAV on an SD card or in ESP32 flash → play via I2S on trigger.
- Or generate at runtime on the host and stream PCM to the pad over USB — simpler, requires host to be awake.

Pre-rendered is the right call: fixed phrase set, no host dependency, fast playback.

## Optional chaos mode (opt-in only)

Disabled by default. Toggle via a hidden key-combo.

| Feature | Author |
|---------|--------|
| Solenoid desk-knock on notification | Claude |
| Relay trigger for a desk fan during long generations | Claude |
| MQTT publish of state (for Home Assistant-driven theatrics — bulbs, Sonos announcements) | Claude |
| IR blaster to flash room lights red on error | Claude |

## Bill of materials delta

| Part | Approx cost |
|------|-------------|
| DRV2605L + LRA | $8 |
| MAX98357A + 1W 8Ω speaker | $6 |
| 3W RGB LED + MOSFET | $4 |
| INMP441 I2S MEMS mic + gooseneck | $10–20 |
| Dedicated PTT keycap (oversized, distinctive) | $3 |
| 3D print filament for figurine | — |
| SD card module (for TTS phrase storage) | $3 |

**Approx total over standard:** +$35–50.

---
*Authors: Daniel (haptics, buzzer, strobe, mic+PTT, figurine, TTS personalities), Claude (escalation ladder, chaos mode add-ons, implementation notes)*
