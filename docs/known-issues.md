# Known Issues — a-primer-skills

## MCP Server Persistence — RESOLVED

**Status:** Resolved
**Date:** 2026-02-26

**Root cause:** MCP servers were being written to `~/.claude/settings.json` — wrong file. The correct location is `~/.claude.json`, and the proper method is `claude mcp add` CLI commands.

**Fix applied:** Updated `references/mcp-server-configs.md` and `SKILL.md` Step 6 to use:
- `claude mcp add --scope user context7 -- npx -y @upstash/context7-mcp@latest`
- `claude mcp add --transport http --scope user cloudflare https://mcp.cloudflare.com/mcp`
- `claude mcp add --transport http --scope user figma https://mcp.figma.com/mcp`

These commands write to `~/.claude.json` and persist across all sessions automatically.

---

## Marketplace Plugin Persistence — RESOLVED

**Status:** Resolved
**Date:** 2026-02-26

Marketplace plugins installed during a `--plugin-dir` session didn't persist. This was resolved by switching to a regular `claude` session (no `--plugin-dir`) for all plugin installs.

---

## a-primer-skills Plugin Persistence — RESOLVED

**Status:** Resolved
**Date:** 2026-03-02

**Solution:** GitHub as custom marketplace (Option A). Users run:
```
/plugin marketplace add ammonhaggerty/a-primer-skills
/plugin install a-primer-skills@a-primer
```

This registers the GitHub repo as a marketplace source and persists the plugin across sessions. The guidebook (`00-tldr.md` and `03-setting-up.md`) has been updated to use this approach.
