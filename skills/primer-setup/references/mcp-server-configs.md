# MCP Server Configurations Reference

Exact configurations and install commands for each MCP server and plugin used in the primer.

**MCP servers** are installed using `claude mcp add` CLI commands run in the terminal. These write to `~/.claude.json` and persist across all sessions automatically.

**Marketplace plugins** are installed using `/plugin install` slash commands typed directly inside Claude Code. The official Anthropic marketplace (`claude-plugins-official`) must be added first, then plugins are installed from it. These commands cannot be run via bash because they have interactive prompts (scope selection, trust confirmation).

---

## context7

**Type:** MCP server (CLI install)

**What it enables:** Live documentation lookup. Claude can fetch current docs for any library or framework instead of relying on training data. Eliminates outdated API references.

**Install command (run in terminal):**
```bash
claude mcp add --scope user context7 -- npx -y @upstash/context7-mcp@latest
```

This registers context7 as a user-scoped MCP server in `~/.claude.json`, which persists across all sessions and projects.

**Prerequisites:** Node.js and npm must be installed. npx (included with npm) downloads and runs the server automatically.

**Verification:** After restarting Claude, ask: "Use context7 to look up the Hono framework documentation." Claude should be able to fetch and reference live documentation.

**Common issues:**
- Server fails to start — Usually a Node.js/npm issue. Verify `npx` works by running `npx --version`.
- Server not available after install — Restart Claude Code for the new server to load.

---

## Cloudflare

**Type:** MCP server (CLI install)

**What it enables:** Direct interaction with over 2,500 Cloudflare API endpoints. Claude can manage Workers, D1 databases, R2 buckets, DNS, and other Cloudflare services. Authentication happens automatically via OAuth the first time a Cloudflare tool is used — no separate login step required.

**Install command (run in terminal):**
```bash
claude mcp add --transport http --scope user cloudflare https://mcp.cloudflare.com/mcp
```

**Prerequisites:** None. Wrangler authentication is not required for this MCP server — it uses its own OAuth flow.

**Verification:** After restarting Claude, ask: "Use the Cloudflare tools to search for Workers documentation." Claude should be able to access Cloudflare's API.

**Common issues:**
- OAuth prompt not appearing — Restart Claude Code and try again.
- Account selection — If multiple Cloudflare accounts exist, you may need to re-authorize with the correct account.

---

## Figma (optional)

**Type:** MCP server (CLI install)

**What it enables:** Design-to-code workflow. Claude can inspect Figma files, read component properties, extract design tokens, and generate code that matches designs.

**Install command (run in terminal):**
```bash
claude mcp add --transport http --scope user figma https://mcp.figma.com/mcp
```

After installing, the user will need to authenticate via `/mcp` inside Claude Code on next restart.

**Verification:** After restarting Claude, ask: "Use Figma to get metadata for this file" and provide a Figma file URL. Claude should be able to read the file's structure.

**Common issues:**
- Authentication needed — Run `/mcp` inside Claude Code and follow the Figma auth flow.
- Not configured — This is optional. Skip if the user does not use Figma.

---

## Playwright

**Type:** Marketplace plugin (from official Anthropic marketplace)

**Install:** User types this directly in Claude Code:
```
/plugin install playwright@claude-plugins-official
```
When prompted for scope, choose "Install for you (user scope)".

**What it enables:** Browser automation. Claude can navigate to URLs, take screenshots, click elements, fill forms, and inspect pages. Useful for testing web apps, debugging UI issues, and visual verification.

**Post-install:** Download browser binaries (this can take a few minutes):
```bash
npx playwright install
```

**Verification:** After restarting Claude, ask: "Use Playwright to navigate to https://example.com and take a screenshot."

**Common issues:**
- Browser not installed — Run `npx playwright install` to download browser binaries.
- Plugin installed but tools not available — Restart the Claude Code session.

---

## frontend-design

**Type:** Marketplace plugin (from official Anthropic marketplace)

**Install:** User types this directly in Claude Code:
```
/plugin install frontend-design@claude-plugins-official
```
When prompted for scope, choose "Install for you (user scope)".

**What it enables:** UI/UX patterns and best practices. Teaches Claude to build better-looking interfaces with stronger design sensibility.

**Verification:** The plugin loads automatically after restart. No additional configuration needed.

---

## superpowers

**Type:** Marketplace plugin (from official Anthropic marketplace)

**Install:** User types this directly in Claude Code:
```
/plugin install superpowers@claude-plugins-official
```
When prompted for scope, choose "Install for you (user scope)".

**What it enables:** Enhanced workflow skills including brainstorming, systematic debugging, test-driven development, writing plans, code review, and parallel agent dispatching. Makes Claude more disciplined and methodical.

**Verification:** The plugin loads automatically after restart.

---

## Official Marketplace Setup

The official Anthropic marketplace is NOT pre-configured on all installations. Before installing plugins, the user must add it:

```
/plugin marketplace add anthropics/claude-plugins-official
```

This is a one-time step. After the marketplace is added, plugins can be installed from it using `/plugin install <name>@claude-plugins-official`.

**Prerequisites:** Git must be configured to use HTTPS for GitHub (set up during Step 3 of primer-setup):
```bash
git config --global url."https://github.com/".insteadOf "git@github.com:"
```

---

## Configuration Notes

**MCP servers vs. plugins:** MCP servers (context7, Cloudflare, Figma) are installed using `claude mcp add` which writes to `~/.claude.json`. Plugins (Playwright, frontend-design, superpowers) are installed from the official marketplace using `/plugin install <name>@claude-plugins-official`.

**Scope options for MCP servers:**
- `--scope user` — Available in all projects for this user (recommended for primer setup)
- `--scope project` — Available only in the current project
- `--scope local` — Private to this user in this project only

**Managing MCP servers:**
```bash
claude mcp list          # List all configured servers
claude mcp get <name>    # Get details for a specific server
claude mcp remove <name> # Remove a server
```

**Restarting after changes:** After installing MCP servers or plugins, restart the Claude Code session for changes to take effect. Type `/exit` (or press Ctrl+C), then type `claude` to start a fresh session.
