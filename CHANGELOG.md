# Changelog

Each release lists what changed and **upgrade notes written for Claude**: what to add or change, described by intent, so it can apply them even to a board or skill you've customized. The update prompt in [`setup/UPDATE.md`](setup/UPDATE.md) reads this file.

Your installed version:
- **Board:** `<meta name="minion-board-version" content="…">` in your board's HTML. Boards installed before 1.2.0 have no such line; treat them as 1.0.0 or 1.1.0 (see below).
- **Skill:** the line "Minion Board skill, version …" near the top of your session-board skill. Skills from before 1.2.0 don't have it.

Each release is a git tag (`v1.0.0`, `v1.1.0`, …), so the exact files of any version can be compared.

---

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
