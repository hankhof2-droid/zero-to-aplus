# Claude Desktop (Linux Beta) — Blank White Screen Fix

**System:** Linux Mint Cinnamon · Gigabyte X870 board · AMD integrated graphics + NVIDIA card (driver 595)
**Date resolved:** August 1, 2026

## Problem

Claude Desktop opened to a blank white window every time. The window frame and
min/max/close buttons rendered normally, but the content area was completely white.

## Root Cause

A **corrupted app profile** (the settings/login data stored in `~/.config/Claude`).
The app launched, but its web content failed to load because of the bad profile data.

## What Didn't Fix It (but was worth ruling out)

1. **Clearing the render caches:**
```bash
   rm -rf ~/.config/Claude/Cache ~/.config/Claude/"Code Cache" ~/.config/Claude/GPUCache
```
2. **Disabling GPU acceleration** (a common Electron/NVIDIA issue on Linux):
```bash
   claude-desktop --disable-gpu --no-sandbox --enable-logging
```

## Key Lesson Learned

Electron apps are **single-instance**. If a copy is already running in the background,
launching again with new flags does nothing — the new launch just hands off to the
existing (broken) instance and the flags are silently ignored. A command that returns
instantly to the prompt with no output is the tell.

Always kill all instances before testing launch flags:

```bash
pkill -f claude-desktop     # kill all instances
pgrep -af claude            # verify nothing survived (no output = clean)
```

## The Fix

1. Kill all running instances:
```bash
   pkill -f claude-desktop
```
2. Rename the settings folder to force a factory-fresh profile (nothing is deleted —
   reversible by renaming back):
```bash
   mv ~/.config/Claude ~/.config/Claude.backup
```
3. Launch Claude — it starts fresh, shows the login screen, works normally after
   signing back in.

## Cleanup (after confirming everything works for a few days)

```bash
rm -rf ~/.config/Claude.backup
```

## Handy Commands Reference

| Command | What it does |
|---|---|
| `grep Exec /usr/share/applications/*laude*.desktop` | Find the launch command behind a menu shortcut |
| `pkill -f <name>` | Kill all processes matching a name |
| `pgrep -af <name>` | List matching processes (verify a kill worked) |
| `mv <folder> <folder>.backup` | Safely "remove" a config folder without deleting it |
| `--disable-gpu` | Launch an Electron app with GPU rendering off |
| `--enable-logging` | Make an Electron app print errors to the terminal |

## General Troubleshooting Pattern (reusable for any misbehaving Linux app)

1. Kill all instances of the app.
2. Clear its caches.
3. Launch from the terminal with logging on and watch for errors.
4. Rule out GPU rendering (`--disable-gpu`).
5. Test with a fresh profile (rename the config folder).
6. If all else fails: reinstall the latest version.
