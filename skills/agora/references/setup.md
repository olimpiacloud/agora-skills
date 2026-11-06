# Setup

## Is Agora running?

```bash
agora api GET /v1/health
```

`{"ok":true,"version":"..."}` means the daemon is up. Other outcomes:

- `agora: command not found`: Agora is not installed. Tell the user:
  ```bash
  curl -fsSL https://agora.olimpia.dev/install.sh | sh
  ```
  Docs: https://agora.olimpia.dev. It needs Linux with Xvfb and Chrome or Chromium. Do not run the installer yourself unless the user asks: it installs system packages and a service.
- Connection refused: the binary exists but the daemon is down. On systemd hosts, `systemctl status agora`. Ask before starting or restarting a shared service; other agents may depend on it.
- `401`: no valid token. See Tokens below.

## Tokens

- `agora token` prints the admin token (needs read access to the Agora home, e.g. `/var/lib/agora/admin.token`).
- Without `AGORA_TOKEN`, `agora api` and `agora mcp` read `admin.token` from `$AGORA_HOME`, `/var/lib/agora` or `~/.agora`.
- Cursor tokens are scoped: permissions only shrink when delegated. Agents launched by Agora get their own cursor token automatically.
- Never print a token into chat or logs; pass it through env.

| Variable | Meaning |
| --- | --- |
| `AGORA_URL` | Daemon URL, default `http://127.0.0.1:7788` |
| `AGORA_TOKEN` | Admin or cursor token |
| `AGORA_WORKSPACE` | Default workspace when the token is not bound to one |
| `AGORA_TOOLS` | Comma-separated tool groups to expose (`browser,browserx,desktop,spawn,workspace,process`); `self` is always on |
| `AGORA_LEAN` | `1` shortens tool descriptions to save context |

## MCP server

`agora mcp` is a stdio MCP server.

```bash
claude mcp add agora -- agora mcp
codex mcp add agora -- agora mcp
```

With an explicit token or a remote daemon behind a tunnel:

```bash
claude mcp add agora -e AGORA_URL=http://127.0.0.1:7788 -e AGORA_TOKEN=<token> -- agora mcp
codex mcp add agora --env AGORA_URL=http://127.0.0.1:7788 --env AGORA_TOKEN=<token> -- agora mcp
```

Cursor, VS Code, Windsurf and other JSON configs:

```json
{ "mcpServers": { "agora": { "command": "agora", "args": ["mcp"], "env": { "AGORA_TOKEN": "<token>" } } } }
```

Restart the agent session after adding it so the tools load.

## Seeing the desktop

- Dashboard: `http://127.0.0.1:7788` (paste the admin token).
- `agora open <workspace>` prints a viewer URL with a short-lived token; add `--control` to allow taking control. Give that URL to the user for human takeover.
- The daemon refuses to listen on non-loopback addresses. For remote access, put it behind an authenticated tunnel (e.g. Cloudflare Tunnel + Access), never an open port.

## HTTP API from the shell

`agora api METHOD PATH [JSON]` calls the API with the same auth:

```bash
agora api GET /v1/workspaces
agora api POST /v1/workspaces/<ws>/tabs '{"url":"https://example.com","observe":true}'
agora api POST /v1/workspaces/<ws>/tabs/<target> '{"action":"snapshot"}'
agora api GET /v1/system
```
