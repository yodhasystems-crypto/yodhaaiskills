# Yodha AI Skills

Free [Claude Code Skills](https://docs.claude.com/en/docs/claude-code/skills) from [Yodha Systems](https://patlas.dev). Each skill lives in its own top-level folder as a `SKILL.md`.

## Using a skill

Copy the skill's folder into your project's `.claude/skills/` directory:

```sh
mkdir -p .claude/skills
curl -o .claude/skills/patent-scan/SKILL.md \
  --create-dirs \
  https://raw.githubusercontent.com/yodhasystems-crypto/yodhaaiskills/main/patent-scan/SKILL.md
```

Claude Code picks up any `SKILL.md` under `.claude/skills/<name>/` automatically — no restart or registration step needed.

## Skills

| Skill | Description |
|---|---|
| [`patent-scan`](patent-scan/SKILL.md) | Scans a codebase and its docs for distinct technical mechanisms, then checks each candidate against prior art via the [Patlas](https://patlas.dev) MCP server's `search_prior_art`/`compare_claims` tools. Requires a connected Patlas MCP server. |

## License

Apache 2.0 — see [LICENSE](LICENSE).
