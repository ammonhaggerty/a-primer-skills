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
- GitHub CLI (`gh`) — installed

Present findings in plain English. Use a clear list showing what is ready and what is missing or needs configuration. Example format:

```
Here is what I found on your system:

  [check] Node.js v22.4.1 — ready
  [check] npm 10.8.1 — ready
  [check] Git 2.43.0 — installed, but name and email not configured
  [x] Homebrew — not found
  [x] Wrangler — not found
  [x] GitHub CLI — not found
```

This step is diagnostic only. Make no changes yet. Let the user absorb the picture before proceeding.

### Step 3: Account Setup

Two free accounts are needed before we install tools. Both take about two minutes to create. These are prerequisites — everything downstream depends on them.

**Check for existing accounts first:**

- Run `wrangler whoami` (if wrangler is installed) to check Cloudflare
- Run `gh auth status` (if gh is installed) to check GitHub

Skip any account that is already connected.

**Cloudflare account:**

If not already authenticated, ask:

"Before we install the rest of the tools, you'll need two free accounts. First up: Cloudflare — this is where your app will live on the internet. It's free to create and takes about two minutes."

"Open **https://dash.cloudflare.com/sign-up** in your browser, create your account, and let me know when you're done."

Wait for confirmation. Note: we will connect this account to the terminal in a later step — for now, just creating the account is enough.

**GitHub account:**

If not already authenticated, ask:

"Next: GitHub — this backs up your code to the cloud and tracks your project history. Also free, also about two minutes."

"Open **https://github.com/signup** in your browser, create your account, and let me know when you're done. Remember the email you use — we'll connect it to your code in the next step."

Wait for confirmation.

If both accounts already exist, skip this step entirely with: "You're already connected to both Cloudflare and GitHub — moving on."

### Step 4: Tool Installation

Work through missing tools one at a time. Skip anything already present and working. For each missing tool, follow this exact sequence:

1. Explain what it is and why it is needed, in one plain-English sentence
2. Show the command that will run and describe what it does
3. Ask permission before executing
4. Install using the platform-appropriate method from `references/environment-checks.md`
5. Verify installation succeeded by running the check command and confirming the output
6. Celebrate briefly — "Node.js is installed and ready" — before moving on

**Installation order:** Homebrew first (macOS, if missing), then Node.js, then Git, then GitHub CLI, then Wrangler. This order matters because later tools depend on earlier ones — Wrangler requires npm, which comes with Node.js, which on Mac is easiest to install through Homebrew.

**Tool explanations to use:**

- **Homebrew** — "A package manager for Mac. It installs developer tools with a single command instead of downloading installers from websites."
- **Node.js** — "The engine that runs JavaScript outside a browser. Nearly every web development tool depends on it."
- **Git** — "Version control. It tracks every change to your code so nothing is ever lost and you can always undo mistakes."
- **GitHub CLI** — "The command-line tool for GitHub. It lets Claude create repositories, push your code, and manage your projects."
  - **Mac:** `brew install gh`
  - **Linux:** Follow the instructions at https://github.com/cli/cli/blob/trunk/docs/install_linux.md
  - **Windows:** `winget install --id GitHub.cli`
- **Wrangler** — "The Cloudflare command-line tool. It deploys your projects to the internet."

**Git configuration:** After confirming Git is installed, check whether `user.name` and `user.email` are configured globally. If either is missing, ask the user for their name and email address. If they created a GitHub account in Step 3, remind them: "Use the same email you used for GitHub — this links your code changes to your GitHub profile." Configure with `git config --global`. Explain: "Git labels every change with your name and email — think of it as signing your work."

**Git HTTPS preference:** Also configure Git to use HTTPS instead of SSH for GitHub. This prevents authentication errors when installing plugins and cloning repositories, since most beginners do not have SSH keys set up:
```bash
git config --global url."https://github.com/".insteadOf "git@github.com:"
```
This is a silent, one-time configuration. No explanation is needed unless the user asks — just run it as part of Git setup.

If any installation fails, consult `references/troubleshooting.md` for the specific error symptom before attempting alternative approaches.

### Step 5: Account Connections

Connect the accounts created in Step 3 to the terminal tools installed in Step 4. Both connections are required.

**GitHub authentication:**

Check with `gh auth status`. If not authenticated:

"Now let's connect GitHub to your terminal. I'll start the login process — a browser window will open where you'll sign in to GitHub and authorize the connection."

Run `gh auth login`. Walk the user through the interactive prompts:
- Select **GitHub.com**
- Select **HTTPS** as the preferred protocol
- When asked to authenticate, choose **Login with a web browser**
- Follow the browser prompts to authorize

Verify with `gh auth status`. If it shows the user's GitHub username, confirm: "GitHub is connected as [username]."

**Cloudflare authentication:**

Check with `wrangler whoami`. If not authenticated:

"Now let's connect Cloudflare. A browser window will open — sign in with the Cloudflare account you just created and click 'Allow'."

Important: The `wrangler login` command has a 2-minute timeout. Since the user already created their account in Step 3, this should be quick. If the user did not create an account yet, direct them to dash.cloudflare.com/sign-up first.

Run `wrangler login` with a 3-minute timeout. After the user confirms the OAuth flow is complete, verify with `wrangler whoami`.

If the login timed out, offer to try again. If verification fails, consult `references/troubleshooting.md` for authentication issues.

### Step 6: MCP Server Configuration

Install MCP servers using `claude mcp add` CLI commands. These write to `~/.claude.json` and persist across all sessions automatically. Consult `references/mcp-server-configs.md` for exact commands.

**Required — install both:**

- **context7** — "This lets me pull the latest documentation for any library instead of relying on potentially outdated training data." Run: `claude mcp add --scope user context7 -- npx -y @upstash/context7-mcp@latest`

- **Cloudflare** — "This lets me search Cloudflare docs and help manage your cloud services directly." Run: `claude mcp add --transport http --scope user cloudflare https://mcp.cloudflare.com/mcp`. Authentication happens automatically via OAuth the first time it is used — no separate login needed. **Important:** When the OAuth authorization page appears in the browser, it shows three permission levels: Read Only (default), Workers Full Access, and DNS Full Access. Tell the user: "When the Cloudflare authorization page opens, **click 'Workers Full Access'** before clicking Allow. The default is Read Only, which won't let me create or manage your projects."

**Optional:**

- **Figma** — Ask: "Do you use Figma for design? If so, I can connect to your Figma files and generate code from your designs." If yes, run: `claude mcp add --transport http --scope user figma https://mcp.figma.com/mcp`. After installing, the user will need to authenticate via `/mcp` inside Claude Code on next restart. If no, skip.

Verify with `claude mcp list` to confirm context7 and cloudflare appear.

### Step 7: Marketplace Plugin Installation

Install plugins from the official Anthropic marketplace. The user needs to first add the marketplace, then install plugins from it. These are slash commands the user must type directly — they cannot be run via bash because they have interactive prompts.

Tell the user: "I need you to type a few commands to install plugins. These are slash commands — type them exactly as shown, starting with the `/`. For each one, when it asks about install scope, choose 'Install for you (user scope)'. After all three are installed, just type 'all done' or 'done' and I'll take it from there."

**First, add the official marketplace:**
```
/plugin marketplace add anthropics/claude-plugins-official
```

**Then install each plugin, one at a time:**

1. **Playwright** — "This lets me open a browser, test your app, and debug issues autonomously — when something looks wrong, I can see what you see."
   ```
   /plugin install playwright@claude-plugins-official
   ```
   Choose "Install for you (user scope)".

2. **frontend-design** — "This teaches me UI/UX patterns and helps me build better-looking interfaces."
   ```
   /plugin install frontend-design@claude-plugins-official
   ```
   Choose "Install for you (user scope)".

3. **superpowers** — "This gives me enhanced workflow skills — brainstorming, debugging, planning, and more."
   ```
   /plugin install superpowers@claude-plugins-official
   ```
   Choose "Install for you (user scope)".

After all three are installed, run `npx playwright install` to download browser binaries. Note this download may take a few minutes.

Explain the concept behind plugins and skills: "Skills are playbooks that teach me specific workflows. Without them, I am a generalist. With them, I know your preferred workflow and can be more effective at each part of building — from brainstorming to debugging to deployment."

### Step 8: Completion Summary

Present a clear checklist of everything that was set up, using checkmarks for completed items. Include the tool versions where relevant. Include account connections:

```
Accounts:
  [check] GitHub — connected as [username]
  [check] Cloudflare — connected as [account name]

Tools:
  [check] Node.js v22.x
  [check] Git — configured as [name] <[email]>
  [check] GitHub CLI
  [check] Wrangler

MCP Servers:
  [check] context7
  [check] Cloudflare

Plugins:
  [check] Playwright (with browsers)
  [check] frontend-design
  [check] superpowers
```

Tell the user to restart Claude to pick up the new configuration: "To restart, type `/exit` (or press Ctrl+C), then type `claude` to start a fresh session. This is like refreshing a browser — Claude picks up all the new tools and plugins when it starts fresh."

Remind the user how to start their first project: "After restarting, type `/a-new-project` to create your first project."

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
- Wrangler and GitHub authentication failures
- MCP server configuration issues
- MCP server re-authorization (expired tokens)
- Port conflicts

---

## Reference Files

Three reference files support this skill. Load them as needed during setup.

- `references/environment-checks.md` — Platform-specific commands for detecting installed tools and configurations. Use during Step 2 (detection) and Step 4 (installation verification).
- `references/mcp-server-configs.md` — Exact configurations and install commands for MCP servers and plugins. Use during Step 6 (MCP setup) and Step 7 (plugin installation).
- `references/troubleshooting.md` — Symptom-based troubleshooting for common setup issues. Consult whenever an installation or configuration step fails.
