# Hardware Directions

Two broad paths for the Claude Macropad. Not mutually exclusive — Direction A is the fastest route to value; Direction B is the more interesting build.

---

## Direction A — Off-the-shelf stock macropad

Buy a commodity programmable macropad and configure it in software. Zero hardware work.

### Candidates
- **Elgato Stream Deck** (6 / 15 / 32 keys) — per-key LCD icons, mature plugin ecosystem, unofficial Linux support via [`streamdeck-ui`](https://timothycrosley.github.io/streamdeck-ui/) or [`StreamController`](https://streamcontroller.github.io/).
- **Generic QMK/VIA macropad** (e.g. Keychron Q0, KeebMonkey, various AliExpress 3×4 pads) — remappable in the browser via VIA, keycodes only (no per-key screens).
- **Ploopy / Work Louder Micro Pad** — QMK, RGB per-key, slightly more hobbyist.

### Feature options (standard)
- Key remapping (layers, tap/hold, macros)
- Per-key RGB (most QMK pads)
- Per-key LCD icons (Stream Deck only)
- Rotary encoders (volume, scroll, undo/redo)

### Pros
- Works today. Plug in, configure, done.
- Stream Deck gives labelled buttons — big win for a pad with 10+ distinct Claude actions.
- Firmware is someone else's problem.

### Cons
- No native way to drive LEDs from host-side events without a custom daemon.
- Stream Deck on Linux is "community-maintained" — occasional breakage across KDE/Wayland updates.
- Form factor is whatever the vendor shipped.

---

## Direction B — Custom hardware (or heavily customized stock)

Build (or meaningfully customize) a pad designed for this specific workflow.

### Sub-options

**B1. Custom keycaps on a stock QMK pad** — cheapest form of "custom." Order legended caps (WASDKeyboards, MaxKeyboard, or DIY with a Cricut + blank caps) labelled with the Claude action, not the keycode.

**B2. Stream Deck with Claude-themed icons** — effectively a software-only "custom." Design a consistent icon set (approve ✓, stop ✋, new-session +, Claude logo pulse, etc.).

**B3. Fully custom PCB + case** — QMK-compatible MCU (RP2040, Pro Micro), addressable RGB (WS2812), optional small OLED or RGB strip for status, 3D-printed case. Full control over:
- Number and layout of keys (e.g. one big "Approve" key, small cluster for navigation)
- A dedicated **status LED zone** independent of the key backlighting
- Optional rotary encoder for scrolling Claude transcripts
- USB HID raw interface for bidirectional host comms (required for LED status — see below)

### Pros
- Exactly the form factor and affordances the workflow needs.
- Bidirectional USB HID means the host can *push* state to the pad (LEDs, OLED text) without polling.
- Fun.

### Cons
- Weeks of work vs. an afternoon of configuration.
- Firmware (QMK Raw HID or custom) has to be maintained.

---

## Status indicator — how does the pad know when Claude needs feedback?

Short answer: **Claude Code's hooks system gives you exactly the signal you need.** You do not need screen-scraping, OCR, or window-title parsing (though those are fallbacks).

### The native mechanism: Claude Code hooks

Claude Code fires shell commands at lifecycle events, configured in `~/.claude/settings.json` under `hooks`. The relevant ones:

| Hook event | Fires when | Use for |
|------------|------------|---------|
| `Notification` | Claude needs user attention — permission prompt, idle awaiting input, tool approval | **Set LED to "awaiting feedback"** |
| `Stop` | Claude finishes its turn and hands control back to the user | **Set LED to "awaiting feedback"** (idle-with-response) |
| `UserPromptSubmit` | User sends a new prompt | **Set LED to "working"** |
| `PreToolUse` / `PostToolUse` | Around tool calls | Optional — pulse LED during long operations |
| `SessionStart` / `SessionEnd` | Session boundaries | Set LED to "idle" on end |

Each hook runs an arbitrary shell command. That command is where you push state to the macropad.

### Transport to the pad

Three viable approaches, cheapest → nicest:

1. **File flag + polling firmware (hack)** — hook writes `echo working > /tmp/claude-state`; a daemon on the host reads it and... still needs to talk to the pad. Not actually simpler.
2. **USB HID Raw (QMK / custom firmware)** — hook invokes a small CLI (Python + `hidapi`, or Rust) that sends a single-byte state packet to the pad over HID. Pad firmware listens in `raw_hid_receive()` and drives the LEDs. Clean, fast, bidirectional.
3. **Serial over USB CDC** — simpler firmware if not using QMK. Hook does `echo 2 > /dev/ttyACM0`. Works but requires udev rules for stable device naming.

### Stream Deck-specific path

`streamdeck-ui` / `StreamController` expose per-key images over a Python API. A hook can call a helper script that swaps a key's icon (e.g. a pulsing red "Claude waiting" glyph) when `Notification` fires, and swaps it back on `UserPromptSubmit`. No firmware needed — the Stream Deck is effectively a tiny host-driven display.

### Fallbacks (if hooks don't cover a case)

- **Terminal bell / ANSI escape watching** — Claude Code emits a bell (`\a`) on notifications in some terminals. A wrapper (`script`, `tmux` pipe-pane, or a PTY shim) can watch for it.
- **Window title watching** — KWin can fire a D-Bus signal on window-title changes; Claude Code updates the terminal title with its state on some configurations.
- **Process state / CPU heuristic** — crude: if the `claude` process has been idle for >2s, assume awaiting input. Unreliable, don't use.

Hooks are the right answer. Everything else is a workaround for platforms that don't have them.

### Minimal proof-of-concept

```jsonc
// ~/.claude/settings.json
{
  "hooks": {
    "Notification": [
      { "hooks": [{ "type": "command", "command": "macropad-led awaiting" }] }
    ],
    "Stop": [
      { "hooks": [{ "type": "command", "command": "macropad-led awaiting" }] }
    ],
    "UserPromptSubmit": [
      { "hooks": [{ "type": "command", "command": "macropad-led working" }] }
    ],
    "SessionEnd": [
      { "hooks": [{ "type": "command", "command": "macropad-led idle" }] }
    ]
  }
}
```

`macropad-led` is a ~30-line Python script using `hidapi` (for custom QMK) or the `streamdeck` library (for Elgato).

---

## Recommendation

- **Ship Direction A first** with a Stream Deck or a VIA-remappable pad. Gets the 10 shortcuts live in a day.
- **Layer in hook-driven status** as soon as A works — this is the feature that turns the pad from "fancy keyboard" into "Claude companion."
- **Consider B3 only** if A reveals a form-factor need that no off-the-shelf pad satisfies (e.g. you really want one giant approve key and a status LED bar).
