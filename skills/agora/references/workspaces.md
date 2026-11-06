# Workspaces and human takeover

## Lifecycle

A workspace is a persistent virtual computer: one X display, one Chromium, a named browser profile, apps and cursors. States: `created → active → idle → frozen → suspended → destroyed`.

| Tool | Use |
| --- | --- |
| `workspace_list` | All workspaces with state, resources, viewers, cursors |
| `workspace_info` | One workspace: display, browser, usage, locks |
| `workspace_create(name, width, height, profile, start, layout)` | New workspace. Free until used. `profile` reuses a named browser profile (shared logins); default is a dedicated one |
| `workspace_op(op)` | `start`, `suspend` (checkpoint and free everything), `resume` (restore tabs to their owners), `freeze` / `thaw` (stop processes, keep RAM), `checkpoint` (`snapshot: true` also snapshots the profile), `fork` (copy-on-write profile, `name`), `resize`, `layout` (`tile`, `max`, `float`), `browser-restart` |
| `system_info` | Daemon memory and CPU per workspace |

`suspend`, `freeze` and `fork` need the `workspace` permission. Do not suspend, freeze or restart a workspace other cursors or the user are using unless asked. Deleting workspaces is admin-only through the HTTP API: ask the user first.

Use a separate workspace when you need a clean profile, different screen size or heavy parallel work; otherwise stay in your own.

## Human takeover

1. Detect: tool results include `CHALLENGE DETECTED: {...}` for CAPTCHAs and bot checks; `browser_action(action: challenge)` checks explicitly. Logins, MFA codes, payment confirmations and approvals also need a human.
2. `human_request(reason)`: be precise ("Solve the hCaptcha on the acme.com/login tab, then return control"). Your status becomes `needs_human`; the browser and session stay alive.
3. Tell the user where to act: the dashboard, the Paseo Agora panel, or a viewer URL from `agora open <workspace> --control`.
4. `events_wait(types: "control.returned", wait_ms: 30000)` in a loop until it arrives. Pass `since` with the last seq to avoid re-reading events.
5. Continue with the same `target`; the page state is whatever the human left.

Never use CAPTCHA-solving services or brute force, and never ask the user to paste passwords or MFA codes into chat: they type them directly in the viewer.

## Events

`events_wait(types, cursor, since, wait_ms)` long-polls the event bus. Types accept a `*` suffix. Useful: `control.returned`, `human.requested`, `cursor.ended`, `cursor.status`, `process.exited`, `browser.tab.opened`, `workspace.state`.
