---
name: primer-setup
description: "This skill should be used when the user asks to \"set up my development environment\", \"configure my tools\", \"install development tools\", \"set up for the AI Primer\", \"get started with the primer\", or mentions needing to set up Node.js, Git, Wrangler, MCP servers, or Cloudflare for the first time. Guides complete beginners through one-time environment setup for full-stack AI product development."
---

# Primer Setup

## Overview

One-time guided setup for the AI Primer workflow. Designed for users who may have never opened a terminal. Progressive: present one step at a time, confirm before acting, verify after each change. Idempotent: detect what is already installed and skip completed steps. Safe to re-run at any point — nothing gets duplicated or overwritten.

This skill complements the AI Primer guidebook (Parts 2-3) and prepares the user's machine with every tool, connection, and configuration needed to build and ship full-stack products with Claude.

---

## Design Principles

Five principles govern how this setup operates. Follow all of them throughout the process.

### Never assume terminal familiarity

Explain every concept in plain English before executing any command. When a command runs, describe what it does and what the output means. Treat the terminal as a new environment the user is seeing for the first time. Avoid jargon; when technical terms are unavoidable, define them immediately.

### One step at a time

Present a single action, explain it, ask permission, execute it, verify the result, then move on. Never batch multiple installs into one step. Never run ahead while the user is still absorbing the previous result. Patience is the default.

### Idempotent

Detect what is already installed, configured, and working. Skip those steps entirely with a brief confirmation: "Git is already installed and configured — moving on." If something is installed but broken, fix it rather than re-installing. If something is partially configured, complete the configuration rather than starting over.

### Honest

Clearly distinguish what Claude handles automatically from what requires the user to act. Account creation, OAuth flows, and browser-based sign-ins cannot be completed by Claude — say so plainly. Never pretend to do something that requires human interaction. State what will happen, what the user needs to do, and when to come back.

### Platform-aware

Detect the operating system first and tailor every instruction accordingly. Mac-first instructions with Windows and Linux alternatives where relevant. Consult `references/environment-checks.md` for platform-specific detection commands and expected outputs.

---

## Setup Flow

Work through these steps in order. Each step includes detection logic to skip work that is already done.

### Step 1: Workspace Check

Detect the current working directory. Evaluate whether it is an appropriate workspace location.

If the user is in their home directory, Downloads, Desktop, or another unexpected location, explain the workspace convention: `~/Development` is a single folder where all projects live. It is the place to always launch Claude from. Having one consistent workspace means Claude can see all projects and understand the full development environment. Every project created during the primer will live inside this folder.

If `~/Development` does not exist, offer to create it. Explain what the `mkdir` command does before running it — it creates a new folder, nothing more. Guide the user to navigate into `~/Development` before proceeding with the rest of setup.

If the user is already in `~/Development` or a subdirectory of it, confirm the location and move on.

### Step 2: Environment Detection

Run detection commands from `references/environment-checks.md` to survey the current state of the machine. Check all of the following:

- Operating system (macOS, Linux, or Windows)
- Node.js — installed and version is v18 or higher
- npm — installed (ships with Node.js)
- Git — installed and global user.name/user.email configured
- Homebrew — installed (macOS only)
- Wrangler — installed

Present findings in plain English. Use a clear list showing what is ready and what is missing or needs configuration. Example format:

```
Here is what I found on your system:

  [check] Node.js v22.4.1 — ready
  [check] npm 10.8.1 — ready
  [check] Git 2.43.0 — installed, but name and email not configured
  [x] Homebrew — not found
  [x] Wrangler — not found
```

This step is diagnostic only. Make no changes yet. Let the user absorb the picture before proceeding.

### Step 3: Tool Installation

Work through missing tools one at a time. Skip anything already present and working. For each missing tool, follow this exact sequence:

1. Explain what it is and why it is needed, in one plain-English sentence
2. Show the command that will run and describe what it does
3. Ask permission before executing
4. Install using the platform-appropriate method from `references/environment-checks.md`
5. Verify installation succeeded by running the check command and confirming the output
6. Celebrate briefly — "Node.js is installed and ready" — before moving on

**Installation order:** Homebrew first (macOS, if missing), then Node.js, then Git, then Wrangler. This order matters because later tools depend on earlier ones — Wrangler requires npm, which comes with Node.js, which on Mac is easiest to install through Homebrew.

**Tool explanations to use:**

- **Homebrew** — "A package manager for Mac. It installs developer tools with a single command instead of downloading installers from websites."
- **Node.js** — "The engine that runs JavaScript outside a browser. Nearly every web development tool depends on it."
- **Git** — "Version control. It tracks every change to your code so nothing is ever lost and you can always undo mistakes."
- **Wrangler** — "The Cloudflare command-line tool. It deploys your projects to the internet."

**Git configuration:** After confirming Git is installed, check whether `user.name` and `user.email` are configured globally. If either is missing, ask the user for their name and email address, then configure Git with `git config --global`. Explain: "Git labels every change with your name and email — think of it as signing your work. No account is needed for this. If you have or plan to create a GitHub account, use the same email here — it links your work to your profile. Not sure? Use whatever email you like. This is easy to change later with one command."

If any installation fails, consult `references/troubleshooting.md` for the specific error symptom before attempting alternative approaches.

### Step 4: Cloudflare Account Connection

Run `wrangler whoami` to check existing authentication.

**If already authenticated:** Confirm the account name and skip ahead. "You are already connected to Cloudflare as [account name]."

**If not authenticated:** Explain that this step requires the user to act — Claude cannot log in for them. Then run `wrangler login`, which opens a browser window for Cloudflare OAuth. Walk through what will happen:

"A browser window will open. Sign in to your Cloudflare account — or create a free account if you do not have one yet. Authorize the connection when prompted. Then come back here and let me know when it is done."

After the user confirms the OAuth flow is complete, verify with `wrangler whoami`. If verification fails, consult `references/troubleshooting.md` for authentication issues.

### Step 5: MCP Server & Plugin Configuration

Consult `references/mcp-server-configs.md` for exact configurations and install commands. Install and configure the following in order.

**Required — install all three:**

- **context7** — "This lets me pull the latest documentation for any library instead of relying on potentially outdated training data." Configure as an MCP server in `~/.claude/settings.json` using the exact JSON from the reference file. Merge into the existing `mcpServers` object; do not overwrite other servers.

- **Cloudflare** — "This lets me search Cloudflare docs and help manage your cloud services directly." Install as a marketplace plugin. Requires Wrangler to be authenticated (completed in Step 4).

- **Playwright** — "This lets me open a browser, test your app, and debug issues autonomously — when something looks wrong, I can see what you see." Install as a marketplace plugin. After installing, run `npx playwright install` to download the browser binaries. Note that this download may take a few minutes.

**Optional:**

- **Figma** — Ask: "Do you use Figma for design? If so, I can connect to your Figma files and generate code from your designs." If yes, walk through Figma MCP setup using the instructions in the reference file — this requires the user to generate a personal access token in their Figma account settings. If no, skip.

For each server or plugin: explain what it enables in one sentence, install or configure it, and verify it works. After all MCP servers and plugins are configured, restart the Claude Code session so the new tools become available. Explain that restarting is like refreshing a browser — it picks up the new configuration.

### Step 6: Plugin & Skill Installation

Install workflow plugins from the marketplace:

- **superpowers** — "This teaches me structured workflows for brainstorming ideas, debugging problems, planning features, and working with parallel agents."
- **frontend-design** — "This teaches me UI/UX patterns and helps me build better-looking interfaces."

Explain the concept behind plugins and skills: "Skills are playbooks that teach me specific workflows. Without them, I am a generalist. With them, I know your preferred workflow and can be more effective at each part of building — from brainstorming to debugging to deployment."

Verify each plugin installed successfully by checking the plugin list. If a plugin does not appear, consult `references/mcp-server-configs.md` for the correct install command and try again.

### Step 7: Completion Summary

Present a clear checklist of everything that was set up, using checkmarks for completed items. Include the tool versions where relevant.

Remind the user how to start their first project: "Create your first project by typing `/a-new-project`."

Suggest optional upgrades that are not required but can improve the experience:

- **Ghostty** — "A faster, cleaner terminal app for Mac. Free. Download from ghostty.org."
- **VS Code** — "A code editor for reading what Claude builds. Free. Download from code.visualstudio.com. Not required, but helpful for understanding your project over time."

Congratulate briefly and orient toward next steps. Keep it short — the user is ready to build, and the best way to learn is by doing.

---

## Troubleshooting

If any step fails during setup, consult `references/troubleshooting.md` for the specific symptom before trying alternative approaches. The troubleshooting reference is organized by symptom — match the error message or behavior to the corresponding entry for the targeted fix.

Common categories covered:

- "Command not found" errors after installation
- Permission errors during global npm installs
- Wrangler authentication failures
- MCP server configuration issues
- Port conflicts

---

## Reference Files

Three reference files support this skill. Load them as needed during setup.

- `references/environment-checks.md` — Platform-specific commands for detecting installed tools and configurations. Use during Step 2 (detection) and Step 3 (installation verification).
- `references/mcp-server-configs.md` — Exact configurations and install commands for MCP servers and plugins. Use during Step 5 (MCP setup) and Step 6 (plugin installation).
- `references/troubleshooting.md` — Symptom-based troubleshooting for common setup issues. Consult whenever an installation or configuration step fails.
