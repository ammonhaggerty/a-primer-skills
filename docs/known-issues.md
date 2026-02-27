# Known Issues — a-primer-skills

## Critical: Plugin and MCP Server Persistence After Restart

**Status:** Unresolved
**Date:** 2026-02-26
**Discovered during:** First real-user test on fresh MacBook

### The Problem

After completing the full primer-setup flow, restarting Claude causes everything to disappear:

1. **a-primer-skills plugin** — not loaded (expected, since `--plugin-dir` is session-only)
2. **Marketplace plugins** (Cloudflare, Playwright, frontend-design) — not loaded despite being installed via `/plugin marketplace add` during the session
3. **MCP servers** (context7, Figma) — not loading despite configs still being present in `~/.claude/settings.json`

### What Was Done

1. User installed plugin: `npx -y degit ammonhaggerty/a-primer-skills ~/.claude/plugins/a-primer-skills`
2. Started Claude with: `claude --plugin-dir ~/.claude/plugins/a-primer-skills`
3. Ran primer-setup skill which:
   - Wrote context7 MCP config to `~/.claude/settings.json` under `mcpServers` key
   - Wrote Figma MCP config to `~/.claude/settings.json` under `mcpServers` key
   - Installed Cloudflare plugin via `unset CLAUDECODE && claude /plugin marketplace add anthropic/cloudflare`
   - Installed Playwright plugin via marketplace
   - Installed frontend-design plugin via marketplace
   - Installed superpowers plugin (failed — not found in marketplace, which is fine)
4. User exited (`/exit`) and restarted with just `claude`
5. **Nothing loaded** — no plugins, no MCP servers

### Key Observations

- `~/.claude/settings.json` still contains the context7 and Figma MCP configs — the data is persisted but Claude isn't reading it on restart
- Marketplace plugins were installed during a `--plugin-dir` session — unclear if the `--plugin-dir` flag affects how/where plugin registrations are stored
- The `--plugin-dir` flag is confirmed to be session-only (development/testing flag), not persistent

### Questions to Resolve

1. **Where should MCP servers be configured?** Is `~/.claude/settings.json` the correct file? Or should they be in `~/.claude.json`, `.mcp.json`, or another location? Is there a `claude mcp add` CLI command?

2. **Why didn't marketplace plugins persist?** They were installed with proper `/plugin marketplace add` commands. Does running inside a `--plugin-dir` session affect where these registrations are written?

3. **How to persistently install a local plugin?** `--plugin-dir` is session-only. Is there a `/plugin add /local/path` command? Do we need to publish to a marketplace? Can we create a local marketplace?

4. **What works on Windows?** The solution needs to be cross-platform.

### Potential Solutions to Investigate

**For the a-primer-skills plugin:**
- Option A: Always use `claude --plugin-dir ~/.claude/plugins/a-primer-skills` (clunky, users forget)
- Option B: Shell alias in `~/.zshrc`: `alias claude='claude --plugin-dir ~/.claude/plugins/a-primer-skills'` (Mac/Linux only, overrides claude globally)
- Option C: Publish as a real marketplace plugin (proper solution, requires marketplace setup)
- Option D: Find a permanent local plugin registration method

**For MCP servers:**
- Determine the correct config file location and format
- Consider using `claude mcp add` CLI command if it exists
- Test writing to different config files (`.mcp.json`, `~/.claude.json`)

**For marketplace plugins:**
- Test installing marketplace plugins WITHOUT the `--plugin-dir` flag to see if they persist normally
- The `--plugin-dir` flag may be poisoning the session in some way

### Impact on Guidebook

The setup flow in both the TL;DR (ch 00) and Setting Up (ch 03) currently tells users to:
1. `npx -y degit ammonhaggerty/a-primer-skills ~/.claude/plugins/a-primer-skills`
2. `claude --plugin-dir ~/.claude/plugins/a-primer-skills`
3. Run primer-setup skill
4. Restart Claude

Step 4 breaks everything. This needs to be resolved before the guide can be shared publicly.

### Files That Need Updating Once Resolved

- `guidebook/00-tldr.md` — TL;DR setup steps
- `guidebook/03-setting-up.md` — Full setup chapter
- `skills/primer-setup/SKILL.md` — Setup skill flow
- `skills/primer-setup/references/mcp-server-configs.md` — MCP config instructions
