# Ice Menu Bar (fork) — STATUS

Fork of [jordanbaird/Ice](https://github.com/jordanbaird/Ice), maintained at
[Leunammih/Ice_menubar](https://github.com/Leunammih/Ice_menubar). Goal: adapt/tweak for personal
use, starting from bug fixes.

## Done (2026-09-08)

- Installed app was stuck at 0.11.4 (1107) despite Homebrew cask `jordanbaird-ice` claiming 0.11.12
  was current — `auto_updates` casks are skipped by plain `brew upgrade`. Fixed with
  `brew reinstall --cask jordanbaird-ice`. Now running 0.11.12 (1117).
- Diagnosed the reported crash (`EXC_BREAKPOINT`/SIGTRAP, register dump shows `windowNumber`
  selector) as `CGWindowID(window.windowNumber)` — an unchecked `Int → UInt32` conversion that
  **traps** when `windowNumber` is out of range, which happens for menu bar item windows on
  macOS 26. Confirmed still present in 0.11.12 via a near-identical crash report on upstream
  issue [jordanbaird/Ice#977](https://github.com/jordanbaird/Ice/issues/977) (2026-09-02, same
  version). Fixed both call sites on branch `fix/windowid-overflow-crash`:
  - [Ice/MenuBar/ControlItem/ControlItem.swift](Ice/MenuBar/ControlItem/ControlItem.swift) (`windowID` getter)
  - [Ice/UI/IceBar/IceBar.swift](Ice/UI/IceBar/IceBar.swift) (frame-change publisher)
  Both now use `CGWindowID(exactly:)`, which returns `nil` instead of crashing.
- Working theory (not yet confirmed by a build/test): this crash is also the cause of the "can't
  close hidden items after clicking the dot" complaint. The hidden-section control item (the
  "dot") is designed to disappear once items are shown (`showSectionDividers` defaults to
  `false`) — the intended way back is clicking the main Ice icon, clicking elsewhere in the menu
  bar, or "smart rehide" (click into another app). The frame-change publisher that fires during
  that exact transition is the same one that was crashing, which would explain the app silently
  dying instead of collapsing, leaving items stuck open until Ice is relaunched.

## Open markers

🟦 **TASK · xcode1** — you, outside this session
Do: Install Xcode from the App Store (needs your Apple ID; ~10–15GB, will take a while)
Where: App Store app, search "Xcode"
Why: only Command Line Tools are installed on this Mac — can't build/run/verify the fix without
full Xcode
Then: `done xcode1`

🔶 **DECISION · pr1**
Once the fix is built and verified, should it also go upstream as a PR to jordanbaird/Ice?
  a) Yes — open a PR to jordanbaird/Ice after verifying locally
  b) No — keep it local-only in the Leunammih fork for now
→ Recommend: a — it's a minimal, well-scoped fix for a live, currently-open upstream bug
  (#977), low risk of rejection, and benefits from upstream's crash-report visibility
Answer: `go pr1` · `go pr1 b` · `hold pr1`

## Next step

Build isn't possible until Xcode is installed (`xcode1`). Once it is: build, reproduce the
crash/close bug on 0.11.12 first (to confirm baseline), then build this branch and confirm both
issues are resolved. Then decide `pr1`.
