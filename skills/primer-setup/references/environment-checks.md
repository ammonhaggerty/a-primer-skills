# Environment Checks Reference

Detection commands for every tool the primer-setup skill needs to verify. For each tool: the check command, what success looks like, minimum version (if any), and common failure causes.

---

## OS Detection

**Command:**
```bash
uname -s
```

**Successful result:**
- `Darwin` = macOS
- `Linux` = Linux

**Windows detection (PowerShell):**
```powershell
$env:OS
# Returns "Windows_NT"
```

**Common issues:** None. These commands are built into every OS.

---

## Node.js

**Minimum version:** v18 LTS

**Command:**
```bash
node --version
```

**Successful result:** `v18.x.x` or higher (e.g., `v20.11.0`, `v22.4.1`)

**Also check for version managers:**
```bash
# nvm
nvm --version
# Successful: 0.39.x or similar

# fnm
fnm --version
# Successful: fnm 1.x.x or similar
```

**Common failure causes:**
- `command not found: node` — Node.js not installed, or installed via nvm/fnm but the shell session was not restarted after install.
- Version below v18 — Installed long ago and never updated. Upgrade via nvm (`nvm install --lts`) or reinstall.
- nvm installed but `node` not found — Run `nvm install --lts` to install a Node.js version, then `nvm use --lts`.
- PATH issue on Mac — If installed via Homebrew, `/opt/homebrew/bin` may not be in PATH (see Homebrew section).

---

## Git

**Command:**
```bash
git --version
```

**Successful result:** `git version 2.x.x` (any recent 2.x version is fine)

**Check global config:**
```bash
git config --global user.name
git config --global user.email
```

**Successful result:** Returns the user's name and email. Empty output means not configured.

**Common failure causes:**
- `command not found: git` — On macOS, running `git` for the first time triggers Xcode Command Line Tools install. Let it complete and retry.
- Name/email not configured — Commits will fail. Fix with `git config --global user.name "Name"` and `git config --global user.email "email"`.

---

## Wrangler (Cloudflare CLI)

**Command:**
```bash
wrangler --version
```

**Successful result:** `wrangler x.x.x` (e.g., `wrangler 3.60.0`)

**Check authentication:**
```bash
wrangler whoami
```

**Successful result:** Shows account name and account ID.

**Common failure causes:**
- `command not found: wrangler` — Not installed. Install with `npm install -g wrangler`.
- "Not authenticated" from `wrangler whoami` — Run `wrangler login` to authenticate via browser.
- Installed but not found — Global npm bin directory not in PATH. Check with `npm config get prefix` and verify that `<prefix>/bin` is in PATH.

---

## Homebrew (macOS only)

**Command:**
```bash
brew --version
```

**Successful result:** `Homebrew 4.x.x` (e.g., `Homebrew 4.2.10`)

**Common failure causes:**
- `command not found: brew` — Either not installed, or (on Apple Silicon Macs) `/opt/homebrew/bin` is not in PATH.
- **Apple Silicon PATH fix:** Homebrew on M1/M2/M3 Macs installs to `/opt/homebrew/` instead of `/usr/local/`. Add the following to `~/.zshrc`:
  ```bash
  eval "$(/opt/homebrew/bin/brew shellenv)"
  ```
  Then restart the terminal or run `source ~/.zshrc`.
- On Intel Macs, Homebrew installs to `/usr/local/bin/` which is normally already in PATH.

---

## npm

**Command:**
```bash
npm --version
```

**Successful result:** `10.x.x` or similar (ships with Node.js)

**Common failure causes:**
- `command not found: npm` — npm is bundled with Node.js. If Node.js is installed but npm is missing, the Node.js installation may be corrupt. Reinstall Node.js.
- Permission errors on `npm install -g` — See troubleshooting reference for EACCES fixes.

---

## MCP Configuration Location

**Config file path:**
```
~/.claude/settings.json
```

This is where MCP server configurations (like context7) are stored under the `mcpServers` key.

**Check if the directory exists:**
```bash
ls -la ~/.claude/
```

**Check current MCP config:**
```bash
cat ~/.claude/settings.json
```

**Successful result:** A valid JSON file. If `mcpServers` key exists, it contains configured MCP servers. The file may not exist yet if Claude Code was just installed.

**Common failure causes:**
- Directory does not exist — Claude Code may not have been run yet. Run `claude` once to initialize.
- File contains invalid JSON — Usually from manual editing. Validate with a JSON linter or look for trailing commas, missing quotes.

---

## Installed Plugins

**Check installed plugins:**
```bash
claude /plugin list
```

**Successful result:** Lists all installed plugins, including marketplace plugins like `anthropic/cloudflare` and `anthropic/playwright`.

**Common failure causes:**
- Plugin not listed — Not yet installed. Install marketplace plugins with `claude /plugin marketplace add <name>`.
- Plugin listed but not working — May need re-authentication or reinstallation. Remove with `claude /plugin remove <name>` and reinstall.
