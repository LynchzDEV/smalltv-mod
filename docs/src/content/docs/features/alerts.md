---
title: Claude session alerts
description: The clock takes its whole screen with a mascot when a Claude Code session finishes or waits for your answer.
---

Separate from the usage meter: this fires on **events**, not percentages. When a Claude Code session finishes long work or stops to ask you something, the device shows a full-screen mascot overlay for a few seconds, then reverts to whatever it was displaying.

## Setup

The PC-side bridge is a single stdlib-only Python script, [`tools/voidzy-claude`](https://github.com/LynchzDEV/smalltv-mod/blob/main/tools/voidzy-claude). It wires itself into Claude Code's hook system and POSTs `/api/notify` on your device over the LAN. Nothing runs as a daemon — each hook event invokes it once.

1. Copy `tools/voidzy-claude` somewhere stable and make it executable:

   ```sh
   curl -o ~/bin/voidzy-claude https://raw.githubusercontent.com/LynchzDEV/smalltv-mod/main/tools/voidzy-claude
   chmod +x ~/bin/voidzy-claude
   ```

2. Point it at your device — either export `VOIDZY=<device-ip>` in the environment Claude Code runs under, or edit the `HOST = os.environ.get("VOIDZY", ...)` default near the top of the script (simplest, since hooks don't always inherit shell env). The mDNS fallback also works but adds a timeout per push.

3. Verify, then install:

   ```sh
   voidzy-claude selftest      # asserts trigger logic, never touches the clock
   voidzy-claude --install     # writes hook entries into ~/.claude/settings.json
   voidzy-claude --send-now    # real push: mascot should take the screen
   ```

`voidzy-claude --status` shows tracked sessions and what the clock was last told. `voidzy-claude --uninstall` removes exactly what `--install` added.

## Behavior knobs

All tunable via env vars, documented in the script's header: debounce window for folding simultaneous sessions, per-session cooldowns so a chatty conversation doesn't celebrate every turn, a minimum-work threshold so short replies don't count as "finished work", and an ignore list for background sessions you can't answer anyway.
