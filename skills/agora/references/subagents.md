# Subagents and swarms

## Spawning

```json
cursor_spawn {
  "name": "pricing-acme",
  "goal": "Open https://acme.com/pricing and return {plans:[{name, monthly_usd}]}.",
  "launch": {"adapter": "claude", "model": "claude-haiku-5-5", "effort": "low"},
  "permissions": ["browser"]
}
```

Returns the child cursor id. Agora starts the agent process with its own scoped token, the agora MCP server and a lean system prompt that already teaches the cheap workflow.

`launch` fields:

| Field | Values |
| --- | --- |
| `adapter` | `claude` (Claude Code headless), `codex` (`codex exec`), `command` (any program; gets `AGORA_URL`, `AGORA_TOKEN`, `AGORA_CURSOR`, `AGORA_WORKSPACE`, `AGORA_GOAL`) |
| `prompt` | Overrides `goal` as the agent prompt |
| `model` | Model id for the adapter |
| `effort` | `low`, `medium`, `high` (Claude) |
| `tools` | Tool groups: `browser` (default), `browserx` (hover, drag, upload, tabs, wait, emulate, audit), `desktop`, `spawn`, `workspace`, `process`. Groups also follow `permissions`. |
| `profile` | `lean` (default: only agora tools, short descriptions, no project settings) or `full` (normal agent with agora tools added) |
| `command` | Argv for `adapter: command` |

Other fields: `workspace` (default yours), `permissions` (subset of `desktop, browser, process, spawn, message, workspace`; can only shrink), `max_children`, `worktree: {repo, branch?, base?}` to give a coding child its own git worktree, directory and port range (`PORT`, `AGORA_PORT_BASE`).

Without `launch`, you get a token to hand to an agent you start yourself.

## Writing goals

- Self-contained: the child does not see your conversation. Include URLs, credentials source (never the secret itself unless the user gave it for this), constraints and the exact result shape.
- Ask for a structured result (JSON shape) so `cursor_wait` output is directly usable.
- Cheapest model and lowest effort that can do it; only add `browserx`, `desktop` or `spawn` when needed. Every extra tool group costs the child context.

## Swarm pattern

1. Spawn all independent children in one turn.
2. One `cursor_wait(ids: [...])` (all by default; `all: false` returns on the first finished; `timeout_ms` default 10 min).
3. Merge results. On timeout, `cursor_list(parent: <you>, active: true)` to see who is still running, then wait again or `cursor_control(op: cancel)`.

Children share the workspace browser and login profile but each has its own tabs and input seat, so they do not collide. For heavy parallel browsing, put groups of children in separate workspaces.

## While children run

- `cursor_get(id)` for status, progress and result; `cursor_list(parent, active)` for an overview.
- `cursor_message(to, body)` sends JSON to a child's inbox; children read with `cursor_inbox(since)`.
- `cursor_control(id, op)`: `cancel` (cascades to descendants), `pause`, `resume`, `transfer`.
- `events_wait(types: "cursor.ended,cursor.status", cursor: <id>)` to react to a child needing a human.

## When you are the cursor

- `cursor_status(status, detail)`: `running | thinking | executing | waiting | blocked | needs_human`.
- `cursor_report(message, percent)` for progress your parent sees.
- `cursor_complete(result)` with concise JSON, or `cursor_fail(error)`. Completing releases your input seat, locks and owned tabs.
