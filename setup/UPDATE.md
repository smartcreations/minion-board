# Update your Minion Board

Your board and skill are copies in your own Claude account, so they don't update by themselves. When a new release comes out, paste the prompt below into a **new Cowork session** (or Claude Code). Claude works out which version you have, merges the new release into your copies, and **keeps any changes you made**. Your cards and history stay: they live in the board's database, not in the page.

You approve the republish and click **Save** on the skill card. If the preference line changed, you paste the new one into Settings.

```text
Update my Minion Board to the latest release from https://github.com/smartcreations/minion-board, keeping my own changes.

1. Clone the repo with its tags: git clone https://github.com/smartcreations/minion-board && cd minion-board && git fetch --tags
   Read VERSION (the latest release) and CHANGELOG.md.
2. Find my board: list my artifacts and pick my Minion Board (or Session Board). Read its HTML and save it as mine.html.
   My version is in <meta name="minion-board-version">. If it has none, compare mine.html to board/minion-board.html at tags v1.0.0 and v1.1.0
   (git show v1.0.0:board/minion-board.html) and pick the closest. If I'm already on the latest version, say so and stop.
3. Merge the board, keeping my changes:
   - base.html = the release I'm on (git show v<my version>:board/minion-board.html), new.html = the latest.
   - If mine.html equals base.html, I never edited it: use new.html as is.
   - Otherwise run a three-way merge (cp mine.html merged.html; git merge-file merged.html base.html new.html). Resolve every conflict
     by keeping my customization and applying the release's intent from CHANGELOG.md's "Upgrade notes for Claude". Show me a short
     summary of what you kept of mine and what the release adds.
   - Republish merged.html onto my existing board artifact (same artifact, not a new one), from the file, after I confirm.
4. Merge the skill the same way: read my session-board skill, use git show v<my version>:skill/SKILL.md as the base and the latest
   skill/SKILL.md as new, merge, fill in my board id and URL wherever the template has BOARD_SLUG and BOARD_URL, and propose the
   result to me as an update to my session-board skill.
5. Preference: if any release between mine and the latest changed the preference line (see CHANGELOG.md), show me the new line from
   setup/START-HERE.md with my board id filled in, in a copy box, and tell me to replace my old line in Settings. Otherwise say no change.
6. Finish with one short list: old version → new version, what's new (from CHANGELOG.md), what you kept of mine, and anything I still need to do.
```

## For maintainers: making a release

1. Change the board, skill or setup files.
2. Bump `VERSION` (for example 1.2.0 → 1.3.0).
3. Add an entry at the top of `CHANGELOG.md`: what changed, which files, and **Upgrade notes for Claude** describing each change by intent ("add a Retire button to the helper popup that…"), not as line numbers. That's what lets Claude apply it to customized boards.
4. Make sure the board's `minion-board-version` meta tag and the skill's version line match `VERSION`.
5. Commit, tag `v<version>`, and push the tag (`git push origin v<version>`).
