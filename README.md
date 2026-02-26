# a-primer-skills

Claude Code plugin for the [AI Primer](https://github.com/ammonhaggerty/ai-coding-primer) guidebook. Takes beginners from first terminal session to deployed full-stack project.

## Install

Requires Node.js and Claude Code already installed.

```bash
npx degit ammonhaggerty/a-primer-skills ~/.claude/plugins/a-primer-skills
claude --plugin-dir ~/.claude/plugins/a-primer-skills
```

Then tell Claude:

> "Use the primer setup skill to set up my development environment."

## What's Inside

**`primer-setup` skill** — One-time environment setup. Installs Node.js, Git, Wrangler. Connects Cloudflare. Configures MCP servers (context7, Cloudflare, Playwright, Figma). Installs workflow plugins (superpowers, frontend-design). Progressive, idempotent, beginner-friendly.

**`primer-workflow` skill** — Passive workflow guidance. Tips for brainstorming, building features, using Figma with Claude, and debugging. Surfaces automatically at the right moment.

**`/a-new-project` command** — Scaffolds a new project with Hono + DaisyUI + HTMX + Alpine.js on Cloudflare Workers (D1 + R2). Generates CLAUDE.md, initializes Git/GitHub, starts local dev server.

## The Stack

Projects created with `/a-new-project` use:

- **Hono** — Web framework
- **Tailwind CSS + DaisyUI** — Styling (CDN, no build step)
- **HTMX + Alpine.js** — Interactivity (CDN, no build step)
- **Cloudflare Workers** — Hosting
- **D1** — SQLite database
- **R2** — File storage

## Part of the AI Primer

This plugin is the hands-on companion to the [AI Primer guidebook](https://github.com/ammonhaggerty/ai-coding-primer) — an open-source guide for non-developers learning to build full-stack products with AI.
