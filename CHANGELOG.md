# Changelog

Each release lists what changed and **upgrade notes written for Claude**: what to add or change, described by intent, so it can apply them even to a board or skill you've customized. The update prompt in [`setup/UPDATE.md`](setup/UPDATE.md) reads this file.

Your installed version:
- **Board:** `<meta name="minion-board-version" content="…">` in your board's HTML. Boards installed before 1.2.0 have no such line; treat them as 1.0.0 or 1.1.0 (see below).
- **Skill:** the line "Minion Board skill, version …" near the top of your session-board skill. Skills from before 1.2.0 don't have it.

Each release is a git tag (`v1.0.0`, `v1.1.0`, …), so the exact files of any version can be compared.

---

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
