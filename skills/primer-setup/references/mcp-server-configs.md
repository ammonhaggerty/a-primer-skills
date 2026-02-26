# MCP Server Configurations Reference

Exact configurations and install commands for each MCP server used in the primer. Some are configured as MCP servers in `~/.claude/settings.json`, others are installed as plugins from the marketplace.

---

## context7

**Type:** MCP server (configured in settings)

**What it enables:** Live documentation lookup. Claude can fetch current docs for any library or framework instead of relying on training data. Eliminates outdated API references.

**Configuration in `~/.claude/settings.json`:**
```json
{
  "mcpServers": {
    "context7": {
      "command": "npx",
      "args": ["-y", "@upstash/context7-mcp@latest"]
    }
  }
}
```

**Prerequisites:** Node.js and npm must be installed. npx (included with npm) downloads and runs the server automatically.

**Verification:** Ask Claude: "Use context7 to look up the Hono framework documentation." Claude should be able to fetch and reference live documentation.

**Common issues:**
- Server fails to start — Usually a Node.js/npm issue. Verify `npx` works by running `npx --version`.
- JSON syntax error in settings.json — Validate the file. Check for trailing commas or missing braces.

---

## Cloudflare

**Type:** Marketplace plugin

**Install command:**
```bash
claude /plugin marketplace add anthropic/cloudflare
```

**What it enables:** Direct interaction with Cloudflare services. Claude can manage Workers, D1 databases, KV namespaces, R2 buckets, and other Cloudflare resources without leaving the conversation.

**Prerequisites:** Wrangler must be installed and authenticated.
```bash
npm install -g wrangler
wrangler login
```

The `wrangler login` command opens a browser for Cloudflare OAuth authentication. Complete the flow and return to the terminal.

**Verification:** Ask Claude: "Use the Cloudflare tools to list my Workers." Claude should be able to query the Cloudflare account.

**Common issues:**
- "Not authenticated" — Run `wrangler login` to re-authenticate.
- Plugin not working after install — Restart the Claude Code session.
- Account selection — If multiple Cloudflare accounts exist, may need to set the active account via the Cloudflare MCP tools.

---

## Playwright

**Type:** Marketplace plugin

**Install command:**
```bash
claude /plugin marketplace add anthropic/playwright
```

**What it enables:** Browser automation. Claude can navigate to URLs, take screenshots, click elements, fill forms, and inspect pages. Useful for testing web apps, debugging UI issues, and visual verification.

**Prerequisites:** Playwright browsers need to be installed after adding the plugin:
```bash
npx playwright install
```

This downloads Chromium, Firefox, and WebKit browsers. It can take a few minutes.

**Verification:** Ask Claude: "Use Playwright to navigate to https://example.com and take a screenshot." Claude should open a browser, navigate, and return a screenshot.

**Common issues:**
- Browser not installed — Run `npx playwright install` to download browser binaries.
- Plugin installed but tools not available — Restart the Claude Code session.

---

## Figma (optional)

**Type:** MCP server (typically pre-configured in Claude settings)

**What it enables:** Design-to-code workflow. Claude can inspect Figma files, read component properties, extract design tokens, and generate code that matches designs.

**How it is configured:** Figma MCP is typically set up through Claude's settings UI, not manually. It requires a Figma personal access token.

**To generate a Figma personal access token:**
1. Open Figma and go to account settings.
2. Scroll to "Personal access tokens."
3. Generate a new token with appropriate scopes.
4. Add the token to Claude's Figma MCP configuration.

**Verification:** Ask Claude: "Use Figma to get metadata for this file" and provide a Figma file URL. Claude should be able to read the file's structure.

**Common issues:**
- Token expired — Generate a new personal access token in Figma settings.
- "Unauthorized" errors — Token may lack the required scopes. Regenerate with full read access.
- Not configured — This is optional. Skip if the user does not use Figma.

---

## Configuration Notes

**MCP servers vs. plugins:** MCP servers (like context7) are configured as JSON in `~/.claude/settings.json` under the `mcpServers` key. Plugins (like Cloudflare and Playwright) are installed via the `claude /plugin marketplace add` command and managed separately.

**Merging into existing settings:** When adding context7 to `settings.json`, merge it into the existing `mcpServers` object. Do not overwrite other servers already configured. If `mcpServers` does not exist yet, create it.

**Restarting after changes:** After modifying `~/.claude/settings.json` or installing plugins, restart the Claude Code session for changes to take effect.
