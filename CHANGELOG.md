# Changelog

Each release lists what changed and **upgrade notes written for Claude**: what to add or change, described by intent, so it can apply them even to a board or skill you've customized. The update prompt in [`setup/UPDATE.md`](setup/UPDATE.md) reads this file.

Your installed version:
- **Board:** `<meta name="minion-board-version" content="…">` in your board's HTML. Boards installed before 1.2.0 have no such line; treat them as 1.0.0 or 1.1.0 (see below).
- **Skill:** the line "Minion Board skill, version …" near the top of your session-board skill. Skills from before 1.2.0 don't have it.

Each release is a git tag (`v1.0.0`, `v1.1.0`, …), so the exact files of any version can be compared.

---

## 1.3.3

**Files:** `board/minion-board.html` (and the version line in `skill/SKILL.md`)

- The world view's project legend now lists only projects that have a helper on the map. Projects whose threads are all done or retired drop out of it, and come back as soon as one of their threads is active again. Project colors don't change.
- The assistant robot is drawn in the right front/back order wherever it actually is, not only in its booth.

**Upgrade notes for Claude**
- Board: in `sync()`, pass `drawLegend` only the projects that have at least one card whose `zoneKey(r)` isn't `"exit"` (skip `role: "assistant"` cards). Keep computing `projColor` from all projects.
- Board: build furniture draw items with `furnItem(b, t)`. For the `assistant` item, use the robot's live position (`botAt()`) for its depth and footprint box.
- Skill and preference: no change beyond the version line.

## 1.3.2

**Files:** `board/minion-board.html` (and the version line in `skill/SKILL.md`)

- Better front/back drawing order in the world view. Helpers no longer slip behind or through long furniture (benches, tables, desks, boards) or city buildings and cars when they stand near one end. Before, everything was sorted by the center of its footprint. Now the board compares footprints: when two things overlap on screen and one is fully behind the other, it's drawn first.

**Upgrade notes for Claude**
- Board: give each draw item a floor footprint `box: [x0, y0, x1, y1]` where it's known (furniture from `gx, gy, w, d`; helpers ±0.22 around their position; city buildings, shops, trees, cars) and replace the final `items.sort(by d)` with `depthOrder(items)`. That orders a pair only when their screen columns overlap and exactly one is fully behind the other along x or y. Otherwise it falls back to `d`, and it breaks cycles on `d`.
- Skill and preference: no change beyond the version line.

## 1.3.1

**Files:** `board/minion-board.html` (and the version line in `skill/SKILL.md`)

- Events have sounds when sound is on: a rumble for the earthquake, an original party tune for the Macarena, drums and chants for the revolution, and hooves, trumpets and crashes for the stampede.
- Your avatar and the robot react to every event with moves and a speech line. During the earthquake they shake and the robot glitches. They dance to the Macarena. In the revolution the robot joins the protest and the boss waves a white flag from behind the desk. In the stampede the boss jumps on his chair and the robot falls over, dizzy.
- The stampede is bigger: 12–18 larger animals that run faster and hit more cars.
- Reports panel: the ✕ now folds the panel on desktop (▾ opens it again, and a new report opens it). Before, it did nothing on desktop.
- Reports panel: "Mark all read" now saves in the board's data (a `seen` document) as well as the browser, so read reports stay read after the browser's storage is cleared and on your other devices.

**Upgrade notes for Claude**
- Board: in the events block, scale animals (`sz` per animal, drawn with translate/scale), use 12–18 animals at speed 8–10, a 24 s cap, a 1.7 knock radius, and call `evCrash()` on a hit.
- Board: add `evNoise`, `evSound(kind)` (called from `evStart`), `evCrash`, and the actor reactions: `evActor(who, t, fn)` wraps the boss and robot draw calls in the furniture list, `evActorExtras` draws their props, `evSpeech()` runs before the briefing dialogue in `draw`, with lines in `EV_LINES`.
- Board: in `markSeen`, also write `seen/all` as `{ids, at}`; subscribe to the `seen` collection and merge newer timestamps into the local read state.
- Board: in `drawReports`, add the `repMin` fold state, the `.reports.min` CSS rule, and make the close button fold or unfold the panel when the phone overlay isn't open.
- Skill and preference: no change beyond the version line.

## 1.3.0

**Files:** `board/minion-board.html` (and the version line in `skill/SKILL.md`)

- New 🎲 **Events** button in the world toolbar with four random little shows. Each one plays once (11–19 s), with randomized casts, paths and timings, then everyone goes back to their room:
  - 🌋 **Earthquake**: the map shakes, helpers panic and run around, dust flies, then things settle.
  - 💃 **Macarena**: helpers line up in the Workshop and dance in sync, with arm moves and music notes.
  - ✊ **Revolution**: helpers march into your office with angry faces and protest signs; the boss hides behind the desk.
  - 🐘 **Stampede**: a herd of elephants, rhinos, giraffes and zebras runs one lap around the city ring road (or the platform edge in other sceneries), knocks cars over (smoke, traffic jam), and leaves. Helpers run indoors.
- Real status always wins: a helper whose thread changes state leaves the event right away.
- With reduced motion, the screen does not shake; everything else still plays.
- The code is one self-contained block (`/* city events */`) plus a few one-line hooks, so it is easy to remove or extend.

**Upgrade notes for Claude**
- Board: add the events block from `board/minion-board.html` (between the `city events` comments) and its hooks: the 🎲 button and `#evMenu` panel with its CSS; `evTick(dt)` in `update`; `evHelper` before the brief check in the per-helper loop; event speed in the walk step; skip `arrive` and zone routing while `m.ev` is set; call `evRelease(m)` where a helper's target zone changes; event bubble override at the top of `bubbleSpec` plus the `angry` glyph and fill; `evDrawExtras` in the helper drawing; the boss hiding check in `drawBoss`; the lane jam in the car loop; `evItems` in the draw list; the shake offset in the main draw transform.
- If you renamed rooms, adjust `EV_ZONES`, `EV_INDOOR` and the Workshop and your-office coordinates in the events block.
- Skill and preference: no change beyond the version line.

## 1.2.2

**Files:** `board/minion-board.html` (and the version line in `skill/SKILL.md`)

- The report popup (Open thread / Dismiss) shows the project name with its color above the helper's name.
- The map toolbar buttons and the scenery menu use a dark background for contrast, and the menu text is vertically centered.

**Upgrade notes for Claude**
- Board: in the report choice popup, add a first line with a small square in the project's color and the project name.
- Board: give `.zoom button` and `.zoom select` a dark translucent background (about rgba(20,24,34,.68), darker on hover) and a 28px line height so their text is centered.
- Skill and preference: no change beyond the version line.

## 1.2.1

**Files:** `skill/SKILL.md` (and the version stamp in `board/minion-board.html`)

- A thread that resolves itself because the work has ended now marks its card `done`, so its helper goes home instead of staying idle.
- The end-of-turn write retries once on `version_mismatch` (the board may have applied the go note in between).

**Upgrade notes for Claude**
- Skill, statuses table: the `done` row becomes "When the user says the thread is finished, or whenever you resolve the thread (for example with `set_thread_resolved`) because the work has ended. Delivering output alone never makes a thread done; resolving it does."
- Skill, "When to write" step 3: if the thread was resolved this turn, the final status is `done` (never `idle` or `review`) and `next` is removed. In the card fields table, `next` is also removed for `done`.
- Skill, rules of thumb: an acknowledgement ("thanks", "ok") means `idle`, unless the thread is resolved in that turn, which means `done`.
- Skill: add a fourth "failure that matters": a resolved thread whose card isn't `done`.
- Skill, "The go note": add that the end-of-turn `merge` may return `version_mismatch` because the board applied the go note after the `db_get`; read the card again and retry once.
- Board: only the version stamp changes to 1.2.1. Preference: no change.

## 1.2.0

**Files:** `board/minion-board.html`, `skill/SKILL.md`, `setup/UPDATE.md` (new), `CHANGELOG.md` (new)

- Versioning: the board carries a `minion-board-version` meta tag and the skill a version line, so updates know where you are.
- New update prompt that merges a release into your board and skill while keeping your own changes.
- The board's page title is now "Minion Board".

**Upgrade notes for Claude**
- Board: add `<meta name="minion-board-version" content="1.2.0">` in the `<head>`. If the page title is still the default "Session Board", it may become "Minion Board"; keep any title the user picked.
- Skill: add the line "Minion Board skill, version 1.2.0. Keep this line: the update prompt reads it." under the main heading.
- Preference: no change.

## 1.1.1

**Files:** `skill/SKILL.md`, `setup/START-HERE.md`

- The skill's description is now directive ("ALWAYS use at the start and at the end of EVERY turn…"), which gets it picked up more reliably.
- The preference line now spells out the exact first call (the go note) instead of just "use the skill", and can also be pasted into each Project's instructions.

**Upgrade notes for Claude**
- Skill: replace only the `description` with: "ALWAYS use at the start and at the end of EVERY turn, in every thread: send the board go note as the very first tool call, and update this thread's card status as the very last step. Also before commands that may need permission." Keep the rest of the user's skill.
- Preference: show the user the new line from `setup/START-HERE.md`, with their board id filled in, and tell them to replace their old line in Settings (and add it to each Project's instructions).

## 1.1.0

**Files:** `board/minion-board.html`

- Idle or brainstorming cards with no update for 7 days leave the world on their own and move to the Done list. They return when their thread writes again.
- Every helper popup has a "Retire" button for threads that were deleted or finished.

**Upgrade notes for Claude**
- Board: add a stale check (status idle or brainstorm, no unread report, `updatedAt` older than 7 days) and treat such cards like `done`: in the world they walk to the exit, in the Board view they're listed with Done. Add a "Thread deleted or finished? Retire" bar under the helper popup that sets the card to `done` with `changedBy: "board"`.
- Skill and preference: no change.

## 1.0.0

First public release: Board and World views, statuses, the go note, report desk with Open/Dismiss, three sceneries (floating, city, academy), pixel bubbles, full screen, Claude Code hooks, and the one-prompt setup.
