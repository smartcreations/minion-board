---
name: "session-board"
description: "ALWAYS use at the start and at the end of EVERY turn, in every thread: send the board go note as the very first tool call, and update this thread's card status as the very last step. Also before commands that may need permission."
---

# Minion Board (session board)

Minion Board skill, version 1.3.4. Keep this line: the update prompt reads it.

the user keeps one live board of all their Claude threads across Projects, Cowork and Claude Code. Each thread keeps its own card up to date. The board has two views: project columns, and an isometric "world" where each thread is a named little helper that walks between rooms by status (planning, workshop, their office, report desk, cafeteria, playground, triage). Tapping a card or helper opens its thread, so every card needs a `link` (or, for Claude Code sessions, a `ccSession`) whenever one can be found.

Four failures matter most:
1. A card that still says "waiting" after the user has answered and the thread is already working again. The very first thing you do in a turn is fix the card (see "When to write").
2. A card that says "working" while the thread is really waiting on them (for example on an "Allow once" permission prompt).
3. A card that asks for their attention when nothing is actually pending.
4. A thread that resolved itself (for example with `set_thread_resolved`) whose card isn't `done`: its helper never goes home.

Nothing is backfilled. A thread gets its card the first time the user takes a turn in it after this skill exists, including old threads they come back to. If this thread has no card yet, create it this turn.

## Where the board lives

- Artifact slug: `BOARD_SLUG`
- Board URL: BOARD_URL
- Collection: `sessions`. There is one document per thread: `sessions/<card-id>`. The `signals` collection holds small one-off notes the open board applies and deletes: the Claude Code hooks write there, and so does your go note (see "The go note"). Never edit or delete other notes there.
- Tools: `mcp__claude-artifacts__db_get`, `db_query`, `db_write`. If they're deferred, load them with ToolSearch in a single call (`select:mcp__claude-artifacts__db_get,mcp__claude-artifacts__db_query,mcp__claude-artifacts__db_write`).
- If those tools aren't available in this session, skip the board quietly. Mention it once at most, not every turn.

## Claude Code sessions with hooks

On the user's computer, Claude Code (terminal and the desktop Code tab) runs hooks that update the board on their own: working when they send a message, "Needs your OK" on a permission prompt, idle if a turn ends while still marked working, and done when the session ends. In those sessions:

- A context line at the start reads `Session board card id: cc-<session id> ... Store ccSession=<session id> on the card.` Use exactly that card id, and set `ccSession` to that session id and `cwd` to the working directory. The board reopens the session from `ccSession` (desktop app link or `claude --resume`).
- The hooks may have created the card already with a placeholder title like "my-app session". Replace it with a real title on your first write.
- Your own writes still matter: the hooks don't know the summary, `next`, or whether the result is `review`, `waiting`, `brainstorm` or `error`. Keep doing the end-of-turn write. You can skip the start-of-turn write and the permission flags, because the hooks cover both.

Cowork sessions and regular chats don't run hooks; everything below applies to them in full.

## Statuses

| status | board shows | use when |
|---|---|---|
| `error` | red, blinking siren, triage | Something broke and you can't get past it: a failed deploy or build, a crash, a tool or permission failure, data you can't reach. `next` says what failed and what you need. |
| `waiting` | bright pink "Needs you", their office | **Only** when you asked the user a direct question, or need something specific from them before you can go on: a decision, an approval, a key, a file. `next` says exactly what. |
| `review` | green "Ready to review" | **Only** when a discrete deliverable was handed over in this turn or the last one, and the user hasn't responded to it yet: a PR, a draft to approve, a file or document they asked for. `next` says what to look at. Never use it for a thread where they're iterating (making a change, testing it, giving feedback, repeating); that's `idle`. |
| `working` | orange, spinning gear, workshop | You're in the middle of a turn. |
| `brainstorm` | purple, thought bubble, planning room | The thread is in an exploratory back-and-forth: ideas, options, planning, nothing concrete in progress. |
| `idle` | grey, playground | **The default** for ongoing work with nothing specific pending: you've done what was asked, and the thread is waiting for the user's next test, feedback or idea. |
| `done` | collapsed / walks out the exit | When the user says the thread is finished ("done", "we're done here", "drop it"), or whenever you resolve the thread (for example with `set_thread_resolved`) because the work has ended. Delivering output alone never makes a thread done; resolving it does. The card comes back as soon as they write in the thread again. |

The board treats any turn that just finished (`idle`, `review`, `brainstorm`) as an unread report until the user has looked at it: the helper celebrates, walks to the report desk and reads out your `summary`. So always end a turn with a clear one-line `summary` of what you did or found.

Rules of thumb:
- When unsure between `review` and `idle`, choose `idle`. When unsure between `waiting` and `idle`, ask yourself whether you literally can't continue without them; if you can, it's `idle`.
- Optional suggestions ("want me to also fix X?") don't make a thread `waiting`. Put them in `summary`, not `next`, and set `idle` (or `review` if a deliverable also went out).
- When the user's message just acknowledges ("looks good", "thanks", "ok"), set `idle`, unless you resolve the thread in that turn, which means `done`.
- Older cards may still hold `approval`, `success` or `idea`. The board shows them as Needs you, Ready to review and Has an idea. Don't write those values any more.

### First card for an existing thread

When the thread already has history but no card, judge the status by how old the last exchange is. If the last delivery is more than a day old and the user hasn't reacted to it, use `idle`, not `review`. Use `waiting` only if your last message asked them something they never answered and the thread can't move without it.

## Permission prompts (sessions without hooks)

The app can pause a turn to ask the user "Allow once" before a command runs. You can't see that pause, and you can't write to the board while it's waiting, so you flag it in advance and let the board decide:

1. **Flag before every step that may prompt.** Right before a shell command (Bash), or any other tool call that has needed their permission in this thread, write to your card: `pendingApproval: true`, `pendingSince: <now>`, `pendingStep: "<a few words, e.g. the render command>"`, `updatedAt: <now>`, keeping `status: "working"`. Assume shell commands prompt unless the user has told you this thread runs them without asking. File edits through the Edit or Write tools and connector calls usually don't prompt; don't flag them unless you've seen otherwise.
2. **The board does the rest.** If the flag is still set 20 seconds later, the board shows "Needs your OK" in pink and their helper walks to their office. If they approve quickly, nothing visible happens.
3. **Clear it.** In your next board write after the step returns, set `pendingApproval: false` (or flag the next step with a fresh `pendingSince`). Always clear it at the end of the turn.
4. **Group steps.** Prefer one combined command over several small ones, so they approve once and you write once.
5. **Heartbeat.** While `working`, refresh `updatedAt` (and `progress`) at least every 3 minutes of activity. The board treats a `working` card with no update for 4 minutes as "Check on it" and moves it to their office.
6. **Blockers mid-turn.** If you hit anything mid-turn that needs them, switch to `waiting` right away instead of waiting for the end of the turn.

## Title: use the session's own name

The card `title` should match what the user sees in their session list, so the card, the helper and the app agree:

1. If this session has a name, use it exactly: one the user set (with `/rename`, `claude -n`, or by renaming the session in the Claude app or desktop app) or the title Claude generated for the session. Use it whenever you can see it in your context.
2. Otherwise write a distinctive 3 to 7 word title.
3. If the user renames the session, or says the session is now called something else, update `title` to the new name on your next write. Otherwise keep the title stable.

## Finding this thread's link

Work through these in order every time the card has no `link` (or `ccSession`) yet:

1. **Claude Code session id.** If a `Session board card id` context line is present, store `ccSession` and `cwd` from it (see above). That's all a Claude Code card needs.
2. **A full URL to this thread in your context.** Check system messages, attribution lines (for example a `Claude-Session:` line), environment details, and anything the user pasted. Known formats:
   - Cowork or claude.ai/code session: `https://claude.ai/code/session_<id>`
   - Thread inside a Project: `https://claude.ai/code/project/chan_<id>?thread=cmsg_<id>`
   - Regular chat: `https://claude.ai/chat/<uuid>`
3. **The IDs, without a URL.** If you can see this thread's own `chan_…` project ID and `cmsg_…` thread ID, build `https://claude.ai/code/project/<chan id>?thread=<cmsg id>`. If you can see only a `session_…` ID, build `https://claude.ai/code/session_<id>`.
4. **Nothing found.** Leave `link` out. Never guess or make one up. The board falls back to opening the project for cards without a link.

Always store the `https://claude.ai/…` form. The board converts it to an app link itself. Only use IDs that belong to this thread; don't copy another thread's IDs from an example or from someone else's card.

## Card id: tie it to the session

1. Claude Code with hooks: the `cc-<session id>` from the context line.
2. Otherwise, if you have this thread's session or thread ID, use it as the card id (for example `session_01ABC…` or `cmsg_01ABC…`).
3. Otherwise use a title-based id: `<project-slug>--<title-slug>`, lowercase, using only letters, digits and hyphens, for example `my-app--billing-page-refactor`.

Once the card exists, keep the same id for the rest of the thread. Earlier board writes in this conversation show which id you used.

When you have no earlier board write to look at (an old thread the user just reopened), look for an existing card before creating one: `db_get` the id you'd use, and also `db_query` `sessions` where `title` equals the title you'd use. Reuse a card that matches; otherwise create one. If a matching card has no `link` and you now have one, add it.

## Card fields

| field | meaning |
|---|---|
| `project` | Short project name as the user would say it: "Website", "Side project", "Home admin", "Claude setup". Use the Claude Project's name when there is one, spelled the same way every time: the board makes one column and one helper color per exact name. For Claude Code, replace the hooks' folder-name placeholder with the real project name. |
| `title` | The session's own name if it has one (see above), else a distinctive 3 to 7 word title. |
| `status` | One of the statuses above. |
| `next` | One short line shown on the card, only for `error`, `waiting` and `review`: what failed, what you need from them, or what to look at. Remove it (`{"__delete__": true}`) for `idle`, `brainstorm`, `working` and `done`. |
| `summary` | One line on what this turn did or found, including any optional suggestion. The board reads it out at the report desk. |
| `progress` | Only while `working`: a short marker such as "step 3 of 6" or "running tests". Delete it when the turn ends. |
| `pendingApproval`, `pendingSince`, `pendingStep` | The permission-prompt flag (see above). `pendingSince` is an ISO time (UTC). |
| `surface` | `chat`, `cowork` or `code`. |
| `link` | This thread's URL (see above). Never guessed. |
| `ccSession`, `cwd` | Claude Code only: the session id and working directory from the context line. |
| `startedAt` | ISO time when the current `working` stretch began. |
| `updatedAt` | ISO time (UTC) of this write. Always set it. |
| `changedBy` | Always `"claude"`. |

## When to write

1. **First tool call of every turn: the "go" note.** Before reading files, searching, thinking it through, or anything else, send the go note described in "The go note" below. It's a single write with no read first, so it lands in seconds. Do this on every turn, short or long, and above all when the user is answering a `waiting` or `review` card: on the board, that's the moment their helper gets its orders and leaves their office. Only skip it for a pure one-line reply with no tool calls at all, or in Claude Code sessions with hooks.
2. **During the turn.** In sessions without hooks, follow "Permission prompts" above. In all sessions, heartbeat at least every 3 minutes of activity and switch to `waiting` the moment you're blocked.
3. **Last tool call of every turn.** Create the card if it doesn't exist; otherwise update it. Set the final status by the rules above (if you resolved the thread this turn, for example with `set_thread_resolved`, the final status is `done`, never `idle` or `review`, and `next` is removed), set or remove `next` to match, write a clear one-line `summary`, update `updatedAt`, delete `progress`, and set `pendingApproval: false`. Then give your reply.

## The go note

The go note flips the card to Working with one write and no read: it creates a small new document in `signals`, so it never needs `ifVersion`. The open board turns it into the card update (and creates the card if it doesn't exist yet), then deletes the note.

```json
{"slug": "BOARD_SLUG", "ops": [{"op": "set", "path": "signals/<card-id>-go-<unix seconds>", "data": {"card": "<card-id>", "event": "prompt", "at": "<now, ISO UTC>", "progress": "on it", "project": "<project>", "title": "<title>", "link": "<this thread's https://claude.ai/... link, if known>", "surface": "chat|cowork|code"}}]}
```

- `card`, `event` and `at` are required. Add `project`, `title`, `link` and `surface` when you know them; the board only uses them if the card doesn't exist yet.
- Use the same card id rules as below. The `<unix seconds>` suffix keeps each note new, so `set` never collides.
- If `db_write` is deferred, load it with ToolSearch in that same first step, then send the note right away.
- The open board can apply your go note after your `db_get`, so the end-of-turn `merge` may come back `version_mismatch`. Read the card again and retry once.
- The go note doesn't replace the end-of-turn write. Do the end-of-turn write with `db_get` and `merge` as usual. If the card doesn't exist yet at the end of the turn (the board wasn't open to create it), create it then with `set`.

## How to write

- Existing cards need `ifVersion`. Run `db_get` on `sessions/<card-id>` first: if the card exists, use `op: "merge"` with its version; if it doesn't, use `op: "set"` with no `ifVersion`. The version from your last write in this thread is also valid if nothing else changed the card since. In Claude Code sessions with hooks, the hooks also write the card, so always `db_get` first there.
- If a write is refused as out of date, read the card again and retry once.
- Write only your own thread's card. Never edit or delete other threads' cards.
- Keep it light: one small write each time, no extra commentary in the reply about updating the board.

## Example writes

First call of a turn, the user just answered a question (the go note, no read first):

```json
{"slug": "BOARD_SLUG", "ops": [{"op": "set", "path": "signals/my-app--billing-page-refactor-go-1793642400", "data": {"card": "my-app--billing-page-refactor", "event": "prompt", "at": "2026-11-02T18:00:00Z", "progress": "applying their answer"}}]}
```

End of turn in a Claude Code session with hooks:

```json
{"slug": "BOARD_SLUG", "ops": [{"op": "merge", "path": "sessions/cc-6f1c2a9e-1b2c-4d3e-9f00-123456789abc", "ifVersion": 3, "data": {"project": "My app", "title": "billing-page-refactor", "status": "review", "summary": "Billing page migrated; PR #212 open against dev", "next": "Review PR #212", "ccSession": "6f1c2a9e-1b2c-4d3e-9f00-123456789abc", "cwd": "/Users/me/code/my-app", "progress": {"__delete__": true}, "pendingApproval": false, "updatedAt": "2026-11-02T18:05:00Z", "changedBy": "claude"}}]}
```