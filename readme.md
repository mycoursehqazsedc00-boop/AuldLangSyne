# AuldLangSyne

A FuBar plugin that tracks your friends' and guildmates' class, race, level, and notes, and reports who's logged in.

## Client mods used

**None.** This update's only functional change is to a bundled shared library (`AceEvent-2.0`), which is pure internal Lua — no external client mod dependency, and nothing here requires one to be installed.

## Features

- Tracks class, race, level, PvP rank (with optional numeric rank), and guild membership for friends and guildmates, learned passively from login broadcasts, `/who` results, and tooltip hovers
- Color-codes names by class and/or gender (independently toggleable)
- Compact friends-list display mode, with an option to show levels in compact mode
- Shows a player's saved note in their tooltip, on logon, and when you `/who` them (each independently toggleable)
- Data management: clear stored data, dump stored data to chat, import notes from `CT_PlayerNotes`
- Backup: save and restore a snapshot of your friends list
- Minimap icon (standard FuBar plugin behavior), can be hidden
- `/who`-result and guild-roster note display integration

## New features in this update

**None.** This update is a performance fix and a data-completeness improvement to existing features — no new user-facing feature was added.

## What changed — performance

**The bundled `AceEvent-2.0` library's scheduler walked its entire timer registry on every single rendered frame, unconditionally** — even when nothing was due for seconds. This is the same class of bug found and fixed in a different addon's bundled copy of the same library; this one is an older, differently-structured revision (14125, manual `next()` iteration rather than `pairs()`+copy), so the fix was re-derived for this specific code shape.

- Tracks the earliest due time; `OnUpdate` skips the walk entirely until then.
- Recomputed after each walk and on every new scheduled event.
- Minor version bumped (14125 → 14126) so this patched copy wins `AceLibrary`'s version arbitration over the unpatched original.

**Why this matters beyond this one addon:** `AceLibrary` is a singleton system — whichever loaded copy of `AceEvent-2.0` has the highest version number becomes the *only* active instance for every addon using it, not just this one.

**Verified:** simulated 2 minutes of scheduled events (150 one-shots, two repeaters, a mid-run cancellation, a callback that schedules a new event). Identical event-firing set between original and patched; registry walk count cut from 7,200 to 362 — a 95% reduction.

## What changed — data completeness

Two small additions that close a gap the addon's own source flags with a comment: *"Right now offline friends that have never been seen by the addon look out of place, for they are not getting munged. Fix?"*

1. **`ScanTooltip` now also captures class and level** (`UnitClass`, `UnitLevel`) — both free native calls it was already positioned to make alongside the race/PvP-rank/guild info it already captured. A single tooltip hover is now enough to fully decorate a friend's row from then on.
2. **Optional backfill from ClassicAPI's `C_PlayerCache`**, if installed and opted into: a persistent, realm-shared player-name cache that can supply class/race/sex for a friend even if *this character* has never personally interacted with them. Fully optional — detected at runtime, silent no-op without it. Never overwrites data already known. Does not and cannot backfill level (not part of the name-cache protocol), so this alone won't fully "munge" a friend with zero other data — combined with the tooltip fix above, one hover covers the rest.

**Verified:** 5 scenarios against a mock cache — no ClassicAPI present, player not cached, an already-complete entry (confirmed not overwritten), a partial entry (confirmed only gaps filled), and no prior entry at all, the real target case (confirmed created and filled, with level correctly left nil).

## Checked and found clean (full regression pass)

Every other `OnUpdate` across the addon and its 10 bundled libraries (`AceAddon`, `AceConsole`, `AceDB`, `AceDebug`, `AceHook`, `AceLibrary`, `AceLocale`, `AceOO`, `Babble-Class`, `Deformat`, `Dewdrop`, `FuBarPlugin`, `Tablet`) — the addon's own display functions are bounded by small Blizzard UI constants and gated behind the Friends panel being open; library timers are self-terminating or gated behind a specific UI interaction (menu open, tooltip shown, drag in progress). This addon doesn't do anything during combat or raiding at all.

## Verification

- Both changed files (`AuldLangSyne.lua`, `lib/AceEvent/AceEvent-2.0.lua`) syntax-checked (`luac5.1 -p`) — clean.
- Scheduler fix and data-backfill fix both verified by simulation, see above.
- Patch reapplies to a clean copy of the originals and reproduces the shipped files exactly, byte for byte.
- Not tested in a live client.

## Files changed

```
AuldLangSyne.lua
lib/AceEvent/AceEvent-2.0.lua
```
