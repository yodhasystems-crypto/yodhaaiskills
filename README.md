# Yodha AI Skills

Free Claude Code plugins from [Yodha Systems](https://patlas.dev).

## Install (recommended)

This repo is also a [Claude Code plugin marketplace](https://code.claude.com/docs/en/plugin-marketplaces). In Claude Code:

```
/plugin marketplace add yodhasystems-crypto/yodhaaiskills
/plugin install patlas@yodhaaiskills
```

That's it — no manual file copying, no separate MCP setup. This installs the `/patlas:patent-scan` skill **and** connects the [Patlas](https://patlas.dev) MCP server in one step. The first time you use it, your browser will prompt you to sign in with GitHub — that provisions a free Patlas account automatically, no API key needed.

Run `/reload-plugins` if the install summary asks you to.

## Plugins

| Plugin | Skill | Description |
|---|---|---|
| [`patlas`](plugins/patlas) | `/patlas:patent-scan` | Scans a codebase and its docs for distinct technical mechanisms, then checks each candidate against prior art via the [Patlas](https://patlas.dev) MCP server's `search_prior_art`/`compare_claims` tools. |

## Not using Claude Code's plugin system?

Skills (`SKILL.md` slash commands) are Claude-Code-specific — other editors (Cursor, Windsurf, VS Code + Copilot Chat, etc.) don't have an equivalent mechanism. But the underlying capability is available to **any** MCP-compatible client: just connect your client to `https://patlas.dev/mcp` (sign in with GitHub, no API key needed) and describe what you want scanned in plain English — the connected assistant can drive the same `search_prior_art`/`compare_claims` tools directly. See [patlas.dev](https://patlas.dev) for client-specific connection instructions.

If you're on an older Claude Code version without plugin support, you can still copy the skill file manually:

```sh
mkdir -p .claude/skills/patent-scan
curl -o .claude/skills/patent-scan/SKILL.md \
  https://raw.githubusercontent.com/yodhasystems-crypto/yodhaaiskills/main/plugins/patlas/skills/patent-scan/SKILL.md
```

You'll also need to connect the Patlas MCP server yourself in that case — see [patlas.dev](https://patlas.dev).

## License

The skill files in this repository are licensed under Apache 2.0 — see [LICENSE](LICENSE). This license covers the skills themselves (the instructions Claude Code follows) only. It does not grant any rights to the Patlas MCP server, its API, or the underlying service some skills call into — that remains a separate, paid product governed by its own [Terms of Service](https://patlas.dev/terms).
