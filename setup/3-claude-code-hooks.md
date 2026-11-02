# Optional: exact status for Claude Code

Paste everything below the line into a **new Claude Code session on your computer** (Code tab or terminal). It sets up the hooks once in `~/.claude/settings.json`, and they then apply to every Claude Code session on this computer, in both the terminal and the Code tab.

---

Set up Minion Board hooks for Claude Code on this computer. My board id (BOARD_SLUG) is: <paste your board id here>. Replace every BOARD_SLUG below with it. Work carefully and show me the diff before saving.

1. Run `claude mcp list` and find the MCP server that provides the Claude artifacts tools (it exposes a `db_write` tool; it's usually the claude.ai "Claude Artifacts" connector). Call its exact server name SERVER below. If no such server shows up, stop and tell me.
2. Back up `~/.claude/settings.json` to `~/.claude/settings.json.bak-session-board`.
3. Merge the `hooks` block below into `~/.claude/settings.json`, replacing SERVER with the real server name. Keep every existing setting and any hooks already there; only add these entries.
4. Validate the JSON, then start a throwaway test with `claude -p "say hi"` in any folder and confirm that a document appeared in the board's `signals` collection, or that a new card shows up on the Minion Board within a few seconds while the board is open.

```json
{
  "hooks": {
    "SessionStart": [
      { "hooks": [ { "type": "command",
        "command": "python3 -c \"import sys,json;d=json.load(sys.stdin);print('Session board card id: cc-'+d['session_id']+' (Claude Code session '+d['session_id']+', cwd '+d.get('cwd','')+'). Store ccSession='+d['session_id']+' on the card.')\"" } ] }
    ],
    "UserPromptSubmit": [
      { "hooks": [ { "type": "mcp_tool", "server": "SERVER", "tool": "db_write",
        "input": { "slug": "BOARD_SLUG",
          "ops": [ { "op": "set", "path": "signals/${session_id}-prompt",
            "data": { "session": "${session_id}", "event": "prompt", "cwd": "${cwd}" } } ] } } ] }
    ],
    "Notification": [
      { "matcher": "permission_prompt", "hooks": [ { "type": "mcp_tool", "server": "SERVER", "tool": "db_write",
        "input": { "slug": "BOARD_SLUG",
          "ops": [ { "op": "set", "path": "signals/${session_id}-perm",
            "data": { "session": "${session_id}", "event": "perm", "cwd": "${cwd}" } } ] } } ] }
    ],
    "Stop": [
      { "hooks": [ { "type": "mcp_tool", "server": "SERVER", "tool": "db_write",
        "input": { "slug": "BOARD_SLUG",
          "ops": [ { "op": "set", "path": "signals/${session_id}-stop",
            "data": { "session": "${session_id}", "event": "stop", "cwd": "${cwd}" } } ] } } ] }
    ],
    "SessionEnd": [
      { "hooks": [ { "type": "mcp_tool", "server": "SERVER", "tool": "db_write",
        "input": { "slug": "BOARD_SLUG",
          "ops": [ { "op": "set", "path": "signals/${session_id}-end",
            "data": { "session": "${session_id}", "event": "end", "cwd": "${cwd}" } } ] } } ] }
    ]
  }
}
```

## What each hook does

| Event | Board effect |
|---|---|
| SessionStart | Tells the session its card id (`cc-<session id>`), so the skill and the hooks update the same card. |
| UserPromptSubmit | Card switches to **Working** the moment you send a message. Creates the card if it doesn't exist yet. |
| Notification (permission_prompt) | Card turns pink, **Needs your OK**, and the helper walks to your office right away. |
| Stop | Safety net: if the turn ended and the card still says Working, it moves to Idle. |
| SessionEnd | Card goes to **Done** (it comes back as soon as you write in the session again). |

Notes:
- The board applies these events only while it's open somewhere (your computer or phone). Events from while it was closed are cleared, not replayed, so old events never move helpers around.
- Cowork and regular chats don't run hooks; they keep reporting through the session-board skill.
- To undo: restore `~/.claude/settings.json.bak-session-board`.
