# Ideated Shortcuts

Brainstormed macropad commands for Claude Code workflow on Ubuntu/KDE.

## Commands

| # | Shortcut | Type | Action |
|---|----------|------|--------|
| 1 | Launch Claude at home level | Konsole command | `konsole -e bash -c "cd ~ && claude"` |
| 2 | Open this folder in Claude | Dolphin integration | Launch Konsole + `claude` at currently-focused Dolphin directory |
| 3 | Open a new Claude instance here | Text injection | Types command to spawn a new Claude session at cwd |
| 4 | Approved | Text injection | Types `approved` (quick confirmation reply) |
| 5 | Stop! | Text injection | Types `stop!` (interrupt/halt instruction) |
| 6 | Copy Claude | Konsole split | Duplicate current Claude session in split-screen at same cwd |
| 7 | Open this level in file manager | Shell command | `dolphin .` at Konsole's cwd |
| 8 | Commit and push all changes | Text injection | Types `/commit-and-push` (or equivalent git message) |
| 9 | Spawn Konsole here | Text injection | Types command to open a new Konsole window at the current working directory |
| 10 | Open a repo in Claude | Runner / picker | Launches a fuzzy-finder rooted at `~/repos/github`; arrow keys navigate matches, Enter opens the chosen repo in Claude inside a new Konsole window |

## Shortcut #10 — Open-Repo Runner (detail)

**Flow:**
1. Hit macropad key → runner popup appears (e.g., `rofi`, `fzf` in a floating Konsole, or `krunner` plugin).
2. Popup is pre-scoped to `~/repos/github/*` (one level deep — each subdir = a repo).
3. User types; fuzzy matcher filters repo names live.
4. Arrow up/down navigates the result list.
5. Enter → launches a new Konsole window, `cd`s into the selected repo, runs `claude`.

**Implementation sketch:**
```bash
#!/usr/bin/env bash
repo=$(find ~/repos/github -mindepth 1 -maxdepth 1 -type d -printf '%f\n' \
  | fzf --prompt="Open repo in Claude > ")
[ -n "$repo" ] && konsole --workdir "$HOME/repos/github/$repo" -e bash -c "claude; exec bash"
```

Alternative: a `rofi -dmenu` frontend for a nicer floating popup without needing a host terminal.

## Notes

- **Text injection** = macropad sends the literal keystrokes into the active window (Konsole running Claude).
- **Konsole commands** = macropad launches a new terminal window/session with a preset command.
- **Dolphin integration** = relies on the currently active Dolphin window's path (may need `qdbus` or a service-menu helper).
- **Split-screen duplication** = Konsole's built-in split view, with the new pane inheriting cwd and auto-running `claude`.

## Suggested Additional Shortcuts (for consideration)

### Claude Code session control
| Shortcut | Type | Action |
|----------|------|--------|
| Escape / Cancel | Keystroke | Sends `Esc` to interrupt Claude mid-generation (distinct from "Stop!" text injection) |
| Toggle plan / auto-accept mode | Keystroke | Sends `Shift+Tab` |
| Clear context | Text injection | Types `/clear` |
| Compact context | Text injection | Types `/compact` |
| Resume last session | Konsole command | `konsole -e bash -c "claude --resume"` |
| Yes / accept prompt | Keystroke | Sends Enter (or arrow + Enter) to confirm permission dialogs |
| No / deny prompt | Keystroke | Sends the deny sequence on permission dialogs |

### Git / repo flow
| Shortcut | Type | Action |
|----------|------|--------|
| Status + diff peek | Shell command | `git status && git diff --stat` in focused terminal |
| New branch from HEAD | Shell command | Prompt for name, run `git checkout -b <name>` |
| Open repo on GitHub | Shell command | `gh browse` at cwd |
| Session handover | Text injection | Types `/dev-tools:session-handover` |

### KDE / desktop
| Shortcut | Type | Action |
|----------|------|--------|
| New Konsole tab here | Konsole D-Bus | Open new tab in current Konsole at same cwd (complements split #6) |
| Screenshot region → clipboard | Shell command | `spectacle -rbc` — grab a region, copy to clipboard for pasting into Claude |
| Paste clipboard into Claude | Text injection | Types current clipboard contents into active Konsole |
| Toggle Do Not Disturb | D-Bus | `qdbus` call to silence KDE notifications while Claude runs |
| Focus existing Claude window | KWin script | Cycle/raise Konsole windows running `claude` |

### Workspace
| Shortcut | Type | Action |
|----------|------|--------|
| Open `~/.claude/` in Dolphin | Shell command | `dolphin ~/.claude/` — fast access to settings/skills/memory |
| Tail latest agent log | Konsole command | `tail -f` newest file in `Agent-Logs/` |

## Ambitious / Stretch Features

- **LED status indicator** — macropad LED reflects Claude's state so the user can glance over and know whether input is needed.
  - **Idle / ready** — off or dim white
  - **Claude working** — pulsing blue/amber
  - **Awaiting feedback (permission prompt, question, plan approval)** — solid red or blinking green
  - **Error / crashed session** — solid red
  - **Implementation sketch:** hook into Claude Code's stop/notification hooks (`settings.json` hooks: `Stop`, `Notification`, `UserPromptSubmit`) to send the macropad a state update over USB HID / serial. QMK/ZMK firmware on the pad listens and drives the per-key RGB.

## Open Questions

- How to detect the active Dolphin window's path for shortcut #2 — likely via `qdbus6 org.kde.dolphin-*` or a Dolphin service menu.
- Split-screen duplication (#6) — Konsole supports split view via Ctrl+( ; may need a D-Bus call to open a new session at the same cwd.
- Should "Approved" and "Stop!" auto-press Enter, or just type the text and let the user confirm?
