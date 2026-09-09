# Ice Menu Bar (fork) — STATUS

Fork of [jordanbaird/Ice](https://github.com/jordanbaird/Ice), maintained at
[Leunammih/Ice_menubar](https://github.com/Leunammih/Ice_menubar). Goal: adapt/tweak for personal
use, starting from bug fixes.

## Done

- **2026-09-08** — Installed app was stuck at 0.11.4 (1107) despite Homebrew cask
  `jordanbaird-ice` claiming 0.11.12 was current — `auto_updates` casks are skipped by plain
  `brew upgrade`. Fixed with `brew reinstall --cask jordanbaird-ice`. Now running 0.11.12 (1117).
- **2026-09-08** — Diagnosed the reported crash (`EXC_BREAKPOINT`/SIGTRAP, register dump shows
  `windowNumber` selector) as `CGWindowID(window.windowNumber)` — an unchecked `Int → UInt32`
  conversion that **traps** when `windowNumber` is out of range, which happens for menu bar item
  windows on macOS 26. Confirmed still present in 0.11.12 via a near-identical crash report on
  upstream issue [jordanbaird/Ice#977](https://github.com/jordanbaird/Ice/issues/977)
  (2026-09-02, same version). Fixed both call sites on branch `fix/windowid-overflow-crash`:
  - [Ice/MenuBar/ControlItem/ControlItem.swift](Ice/MenuBar/ControlItem/ControlItem.swift) (`windowID` getter)
  - [Ice/UI/IceBar/IceBar.swift](Ice/UI/IceBar/IceBar.swift) (frame-change publisher)
  Both now use `CGWindowID(exactly:)`, which returns `nil` instead of crashing.
- **2026-09-08/09** — Xcode 26.6 installed and `xcode-select` switched to it. Built the branch
  (Debug config): first attempt failed to compile (`WindowInfo(windowID:)` is itself failable, so
  chaining `.map` after `.flatMap` produced a nested `WindowInfo??` instead of flattening) — fixed
  by using `.flatMap` for both steps. Second build succeeded clean.
- **2026-09-08/10** — Ran the built app for ~27 hours of normal use (launched Sep 8 21:07, exited
  Sep 10 00:46) with **no crash** and no new `~/Library/Logs/DiagnosticReports/` entries for Ice —
  versus 8 crash reports on 0.11.12 on Sep 8 alone before this fix. The exit at 00:46 was not a
  crash (no diagnostic report, no reboot in between).
- **2026-09-10** — Pushed branch to the fork and opened
  [jordanbaird/Ice#989](https://github.com/jordanbaird/Ice/pull/989) against upstream, referencing
  #977 (and related #580, #786, #867). Relaunched the fixed debug build so there's a working menu
  bar in the meantime.

## Still unconfirmed

The "can't close hidden items after clicking the dot" complaint was a working theory, not
independently confirmed: the hidden-section control item (the "dot") is designed to disappear once
items are shown (`showSectionDividers` defaults to `false`) — the intended way back is clicking the
main Ice icon, clicking elsewhere in the menu bar, or "smart rehide" (click into another app). The
frame-change publisher that fires during that exact transition is the same one that was crashing,
which would explain the app silently dying instead of collapsing. Not yet verified against your own
click-testing.

## Open markers

🟦 **TASK · testclose1** — you
Do: With the fixed debug build running now, open hidden items (click the dot) and try to close them
— click the dot again, click elsewhere in the menu bar, or click into another app window
Why: confirms whether the "can't close" symptom is actually resolved, not just theorized
Then: tell me what happened (or `done testclose1` if it worked)

## Next step

Waiting on `testclose1`. Once confirmed, decide whether to also install this fixed build as the
daily driver in `/Applications/Ice.app` (replacing the Homebrew-managed 0.11.12) until upstream
merges #989, or keep running the debug build from `.build/`.
