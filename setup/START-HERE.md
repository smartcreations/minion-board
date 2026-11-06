# Set up Minion Board with one prompt

1. Open a **new Cowork session** in the Claude app (or a Claude Code session). It needs to be able to download from GitHub.
2. Paste the prompt below and send it.
3. Approve what Claude shows you: the board, the skill save card, and any permission prompts.
4. Paste the one line it gives you into your Claude settings (and, ideally, each Project's instructions). That's the only step Claude can't do for you.

```text
Set up Minion Board for me from https://github.com/smartcreations/minion-board. Follow these steps in order, without rewriting any of the files:

1. Download the repo into your workspace: git clone --depth 1 https://github.com/smartcreations/minion-board
   (or download board/minion-board.html, skill/SKILL.md and setup/3-claude-code-hooks.md from raw.githubusercontent.com).
   If you can't reach GitHub, stop and ask me to attach those three files instead.
2. Publish board/minion-board.html, unchanged, as a new private artifact titled "Minion Board" (icon: board),
   publishing it from the file on disk. Then give it its database by declaring exactly these capabilities:
   {"db": {"rules": [{"path": "", "read": "view", "write": "owner"}]}, "user": {}}
3. Note the artifact's id (slug) and URL. In skill/SKILL.md, replace every BOARD_SLUG with the id and every
   BOARD_URL with the URL, then propose it to me as a skill named "session-board" so I can save it.
4. Following that skill, put this very thread on the board: send its go note, then write its card with
   status "idle" and the summary "Minion Board is set up".
5. Finish with a short checklist, nothing else:
   - the board link
   - "Save the session-board skill on the card above"
   - this line, in a copy box, with BOARD_SLUG replaced by my board id, to paste into Settings, in the personal preferences field:
     Session board, on EVERY turn, no exceptions (short replies and follow-ups included): 1. FIRST tool call: mcp__claude-artifacts__db_write (load it with ToolSearch first if it's deferred), slug BOARD_SLUG, one op: set "signals/<card-id>-go-<unix seconds>", data {card: <card-id>, event: "prompt", at: <now, ISO UTC>, title: <short thread title>, project: <project>}. 2. LAST tool call, before writing the reply: same tool and slug, one op: set "signals/<card-id>-end-<unix seconds>", data {card: <card-id>, event: "finish", at: <now, ISO UTC>, status: idle|review|waiting|brainstorm|error|done, summary: <one line>, next: <what you need from me, or null>}. 3. The reply's last line is "board: <status>", written only after step 2 succeeded; if it failed, "board: not updated". card-id: this thread's session or thread id (session_…, cmsg_…) if it appears anywhere in context, else <project-slug>--<title-slug>; keep the same id for the whole thread. No reads needed. The session-board skill has the status rules.
   - one tip: paste the same line into each Claude Project's instructions too, so it's in front of every thread twice.
   - one question: do I use Claude Code? If I say yes, give me the prompt from setup/3-claude-code-hooks.md
     with BOARD_SLUG already replaced by my board id, to paste into a Claude Code session on my computer.
```

## What happens next

- Why the preference line spells out both calls: a thread that has to remember a skill skips it now and then, but a line in your preferences is in front of every turn. Both calls are single write-only notes, so there's nothing to read or retry, and the visible "board:" line at the end shows you at a glance whether the thread did it. In Claude Code, the optional hooks make status exact no matter what.

- Every thread shows up on the board the next time you write in it. Nothing is backfilled: Claude can't list your past chats, so old threads join as you come back to them.
- Open the board in a pinned tab or on your phone. Switch to **World** to see the helpers.
- Want your own look? After setup, ask Claude to change the board: your logo on the billboard, other room names, another scenery.

## No GitHub access?

Some sessions can't download from GitHub (for example a plain chat). Download [`board/minion-board.html`](../board/minion-board.html), [`skill/SKILL.md`](../skill/SKILL.md) and [`setup/3-claude-code-hooks.md`](3-claude-code-hooks.md), attach them to the chat, and paste the same prompt. Claude uses the attached files instead.
