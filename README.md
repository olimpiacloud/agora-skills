<h1 align="center">Agora for agents</h1>

<p align="center">
  The official skill and plugin to use <a href="https://agora.olimpia.dev">Agora</a> from Claude Code, Codex, Cursor and any agent that speaks MCP.
</p>

---

[Agora](https://agora.olimpia.dev) by [Olimpia](https://olimpia.dev) is an agent-native desktop runtime: one daemon gives every agent a real Linux computer with Chromium, its own pointer, keyboard and tabs, subagents with scoped tokens, and live human takeover for CAPTCHAs and logins. This repository gives your agent two things:

- **The Agora MCP server** (`agora mcp`, stdio): browser tools with semantic refs, desktop tools for any GUI app, subagent and swarm tools, workspaces and events.
- **The `agora` skill**: how to use those tools well and cheaply. Action tools already return what is visible, refs beat coordinates, screenshots only when visuals matter, swarms with `cursor_spawn` + `cursor_wait`, and handing CAPTCHAs to a human.

## Requirements

A running Agora daemon on the machine where the agent runs:

```bash
curl -fsSL https://agora.olimpia.dev/install.sh | sh
agora api GET /v1/health
```

See the [docs](https://agora.olimpia.dev) for details.

## Install

**Any agent with skills support** (Claude Code, Codex, Cursor, OpenCode, Gemini CLI and others)

```bash
bunx skills add olimpiacloud/agora-skills
```

Remove it with `bunx skills remove agora`.

Then add the MCP server to your client:

| Client | Command or config |
| --- | --- |
| Claude Code | `claude mcp add agora -- agora mcp` |
| Codex | `codex mcp add agora -- agora mcp` |
| Cursor, VS Code, Windsurf | `{"mcpServers": {"agora": {"command": "agora", "args": ["mcp"]}}}` |

`agora mcp` reads `AGORA_URL` (default `http://127.0.0.1:7788`) and `AGORA_TOKEN` (default: the local admin token). Pass them with `-e` in Claude Code or `--env` in Codex when needed.

**Claude Code plugin** (skill and MCP server together)

```
/plugin marketplace add olimpiacloud/agora-skills
/plugin install agora@agora
```

## Try it

> Open our staging site, log in with the test account and check the signup flow on mobile.

> Compare the pricing pages of these five competitors in parallel and give me a table.

> Run a UI audit of http://127.0.0.1:3000 and fix what it finds.

> Open GIMP on the Agora desktop and export this file as PNG.

## What's inside

```
.claude-plugin/        plugin and marketplace manifests
.mcp.json              the local Agora MCP server (agora mcp)
skills/agora/
  SKILL.md             what Agora is, setup check, cheap workflows, swarms, takeover, rules
  references/
    setup.md           installing, tokens, env vars, MCP config per client, viewer, HTTP API
    browser.md         reading and acting on pages, waiting, recipes, desktop apps
    subagents.md       cursor_spawn launch options, goals, swarm pattern, reporting
    workspaces.md      workspace lifecycle, human takeover, events
```

## Safety

The daemon listens on 127.0.0.1 only; expose it remotely only through an authenticated tunnel. Cursor tokens are scoped and permissions only shrink when delegated. The skill tells agents not to take destructive or irreversible actions unless asked, never to handle secrets they were not given, and to hand CAPTCHAs, logins and MFA to a human instead of bypassing them.

## Links

- Agora: https://agora.olimpia.dev
- Olimpia: https://olimpia.dev
- Contact: hola@olimpia.dev

## License

MIT
