# MCP Server Configurations Reference

Exact configurations and install commands for each MCP server and plugin used in the primer.

**Important:** MCP servers are installed using `claude mcp add` CLI commands, which persist across sessions. Plugins are installed using `/plugin marketplace add` commands from within Claude Code. Both approaches write to the correct config files automatically.

---

## context7

**Type:** MCP server (CLI install)

**What it enables:** Live documentation lookup. Claude can fetch current docs for any library or framework instead of relying on training data. Eliminates outdated API references.

**Install command (run in terminal, outside Claude):**
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

**Type:** Marketplace plugin

**Install command (from within Claude Code):**
```
/plugin marketplace add anthropic/cloudflare
```

When running inside an existing Claude Code session (e.g., during primer-setup), use the nested session workaround:
```bash
unset CLAUDECODE && claude /plugin marketplace add anthropic/cloudflare
```

**What it enables:** Direct interaction with Cloudflare services. Claude can search Cloudflare documentation and help manage Workers, D1 databases, KV namespaces, R2 buckets, and other Cloudflare resources.

**Prerequisites:** Wrangler must be installed and authenticated.
```bash
npm install -g wrangler
wrangler login
```

The `wrangler login` command opens a browser for Cloudflare OAuth authentication. Complete the flow and return to the terminal.

**Verification:** After restarting Claude, ask: "Use the Cloudflare tools to list my Workers." Claude should be able to query the Cloudflare account.

**Common issues:**
- "Not authenticated" — Run `wrangler login` to re-authenticate.
- Plugin not working after install — Restart the Claude Code session.
- Account selection — If multiple Cloudflare accounts exist, may need to set the active account via the Cloudflare MCP tools.

---

## Playwright

**Type:** Marketplace plugin

**Install command (from within Claude Code):**
```
/plugin marketplace add anthropic/playwright
```

When running inside an existing Claude Code session, use the nested session workaround:
```bash
unset CLAUDECODE && claude /plugin marketplace add anthropic/playwright
```

**What it enables:** Browser automation. Claude can navigate to URLs, take screenshots, click elements, fill forms, and inspect pages. Useful for testing web apps, debugging UI issues, and visual verification.

**Post-install:** Download browser binaries (this can take a few minutes):
```bash
npx playwright install
```

**Verification:** After restarting Claude, ask: "Use Playwright to navigate to https://example.com and take a screenshot." Claude should open a browser, navigate, and return a screenshot.

**Common issues:**
- Browser not installed — Run `npx playwright install` to download browser binaries.
- Plugin installed but tools not available — Restart the Claude Code session.

---

## Figma (optional)

**Type:** MCP server (CLI install)

**What it enables:** Design-to-code workflow. Claude can inspect Figma files, read component properties, extract design tokens, and generate code that matches designs.

**Install command (run in terminal, outside Claude):**
```bash
claude mcp add --transport http --scope user figma https://mcp.figma.com/mcp
```

After installing, authenticate by running `/mcp` inside Claude Code and following the Figma authentication flow. Alternatively, if the user already has a Figma personal access token, they can set it during install:
```bash
claude mcp add --transport http --scope user --header "Authorization: Bearer FIGMA_TOKEN" figma https://mcp.figma.com/mcp
```

**To generate a Figma personal access token (if needed):**
1. Open Figma and go to account settings.
2. Scroll to "Personal access tokens."
3. Generate a new token with appropriate scopes.
4. Use the token in the install command above.

**Verification:** After restarting Claude, ask: "Use Figma to get metadata for this file" and provide a Figma file URL. Claude should be able to read the file's structure.

**Common issues:**
- Token expired — Generate a new personal access token in Figma settings.
- "Unauthorized" errors — Token may lack the required scopes. Regenerate with full read access.
- Not configured — This is optional. Skip if the user does not use Figma.

---

## frontend-design

**Type:** Marketplace plugin

**Install command (from within Claude Code):**
```
/plugin marketplace add anthropic/frontend-design
```

When running inside an existing Claude Code session, use the nested session workaround:
```bash
unset CLAUDECODE && claude /plugin marketplace add anthropic/frontend-design
```

**What it enables:** UI/UX patterns and best practices. Teaches Claude to build better-looking interfaces with stronger design sensibility.

**Verification:** The plugin loads automatically. No additional configuration needed.

---

## Configuration Notes

**MCP servers vs. plugins:** MCP servers (context7, Figma) are installed using `claude mcp add` which writes to `~/.claude.json`. Plugins (Cloudflare, Playwright, frontend-design) are installed from within Claude Code using `/plugin marketplace add` which registers them in settings files.

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
