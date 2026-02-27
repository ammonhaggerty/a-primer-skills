# Known Issues — a-primer-skills

## MCP Server Persistence — RESOLVED

**Status:** Resolved
**Date:** 2026-02-26
**Root cause:** MCP servers were being written to `~/.claude/settings.json` — wrong file. The correct location is `~/.claude.json`, and the proper method is `claude mcp add` CLI commands.

**Fix applied:** Updated `references/mcp-server-configs.md` and `SKILL.md` Step 5 to use:
- `claude mcp add --scope user context7 -- npx -y @upstash/context7-mcp@latest`
- `claude mcp add --transport http --scope user figma https://mcp.figma.com/mcp`

These commands write to `~/.claude.json` and persist across all sessions automatically.

---

## Marketplace Plugin Persistence — NEEDS TESTING

**Status:** Likely resolved (needs verification)
**Date:** 2026-02-26

Marketplace plugins (Cloudflare, Playwright, frontend-design) were installed during a `--plugin-dir` session and didn't persist after restart. The `--plugin-dir` flag may have affected how plugin registrations were stored.

**Hypothesis:** The `--plugin-dir` session may have interfered with where `enabledPlugins` entries were written. Plugins should persist normally when installed from a regular `claude` session (no `--plugin-dir`).

**To verify:** On the test machine, install marketplace plugins from a clean `claude` session (no `--plugin-dir`), restart, and confirm they load.

---

## a-primer-skills Plugin Persistence — IN PROGRESS

**Status:** Investigating options
**Date:** 2026-02-26

The `--plugin-dir` flag is session-only (development/testing). The a-primer-skills plugin disappears on restart.

### Options

**Option A: GitHub as custom marketplace (preferred)**
Users would run:
```bash
/plugin marketplace add ammonhaggerty/a-primer-skills
/plugin install a-primer-skills@ammonhaggerty/a-primer-skills
```
This registers the GitHub repo as a marketplace source and persists the plugin across sessions. Cross-platform. No publishing required.

**Needs testing:** Does this actually work with a single-plugin repo? The repo has `.claude-plugin/plugin.json` but no `marketplace.json`.

**Option B: Submit to official marketplace**
Submit at https://platform.claude.com/plugins/submit. Proper long-term solution. Users would install with:
```bash
/plugin marketplace add anthropic/a-primer-skills
```
(or whatever the marketplace name ends up being)

**Option C: extraKnownMarketplaces in settings**
The setup skill could write to `~/.claude/settings.json`:
```json
{
  "extraKnownMarketplaces": {
    "a-primer": {
      "source": {
        "source": "github",
        "repo": "ammonhaggerty/a-primer-skills"
      }
    }
  },
  "enabledPlugins": {
    "a-primer-skills@a-primer": true
  }
}
```
This is automated (no user interaction) but relies on settings.json structure staying stable.

**Option D: Keep `--plugin-dir` with user guidance**
Tell users to always start with `claude --plugin-dir ~/.claude/plugins/a-primer-skills`. Clunky, users forget, but works immediately.

### Impact on Guidebook

Once resolved, update:
- `guidebook/00-tldr.md` — TL;DR setup steps
- `guidebook/03-setting-up.md` — Full setup chapter

Current instructions still use `--plugin-dir` approach. Do NOT update the guidebook until the solution is tested and confirmed working.
