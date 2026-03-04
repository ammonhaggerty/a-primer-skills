# Troubleshooting Reference

Symptom-based troubleshooting for common setup issues. Each entry follows the format: **Symptom** -> **Cause** -> **Fix**.

---

## 1. "command not found: brew" after installing Homebrew

**Cause:** Homebrew is installed but its bin directory is not in PATH. This is especially common on Apple Silicon Macs where Homebrew installs to `/opt/homebrew/` instead of `/usr/local/`.

**Fix:** Add the Homebrew eval line to the shell profile.

For zsh (`~/.zshrc`):
```bash
echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.zshrc
source ~/.zshrc
```

For bash (`~/.bashrc`):
```bash
echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.bashrc
source ~/.bashrc
```

Verify with `brew --version` after sourcing.

---

## 2. "command not found: node"

**Cause:** Node.js is not installed, or it is installed via a version manager (nvm/fnm) but the shell session was not initialized.

**Fix:**

Install Node.js (pick one method):
- **macOS with Homebrew:** `brew install node`
- **macOS/Linux with nvm:** `curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.1/install.sh | bash` then restart terminal and run `nvm install --lts`
- **Windows with winget:** `winget install OpenJS.NodeJS.LTS`
- **Any platform:** Download from https://nodejs.org (LTS version)

If nvm is installed but node is not found, run:
```bash
nvm install --lts
nvm use --lts
```

---

## 3. npm install -g permission errors

**Cause:** The default npm global directory is owned by root or otherwise not writable by the current user. This happens with system-installed Node.js.

**Fix (Option A — set a user-owned prefix):**
```bash
mkdir -p ~/.npm-global
npm config set prefix '~/.npm-global'
```

Then add to `~/.zshrc` (or `~/.bashrc`):
```bash
export PATH="$HOME/.npm-global/bin:$PATH"
```

Source the profile and retry:
```bash
source ~/.zshrc
```

**Fix (Option B — use nvm):** nvm installs Node.js in the user's home directory, so global installs just work without permission issues. This is the cleanest long-term fix.

---

## 4. "Not authenticated" from Wrangler

**Cause:** Wrangler OAuth token has expired, was revoked, or was never set up.

**Fix:**
```bash
wrangler login
```

This opens a browser for Cloudflare OAuth. Complete the authentication flow. Verify with:
```bash
wrangler whoami
```

Successful output shows the account name and account ID.

---

## 5. MCP server not responding

**Cause:** The server may need re-authorization, or the configuration may be invalid.

**Fix:**

First, check if the server needs re-authorization:
1. Type `/mcp` inside Claude Code
2. Select the server that needs attention
3. Follow the browser prompts to re-authorize

If re-authorization doesn't help, check the configuration:
```bash
claude mcp list
claude mcp get <server-name>
```

If the server is misconfigured, remove and re-add it:
```bash
claude mcp remove <server-name>
claude mcp add --scope user <server-name> -- <command>
```

After any changes, restart the Claude Code session.

---

## 6. Port 8787 already in use

**Cause:** Another Wrangler dev process or a different application is already listening on port 8787.

**Fix (macOS/Linux):**
```bash
# Find what is using the port
lsof -i :8787

# Kill the process (replace <PID> with the actual process ID from lsof output)
kill <PID>
```

**Fix (Windows PowerShell):**
```powershell
netstat -ano | findstr :8787
# Note the PID in the last column
taskkill /PID <PID> /F
```

Alternatively, start Wrangler on a different port:
```bash
wrangler dev --port 8788
```

---

## 7. Git commits fail with "Please tell me who you are"

**Cause:** Git requires `user.name` and `user.email` to be configured before making commits. This is a one-time setup.

**Fix:**
```bash
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
```

Use the email associated with the GitHub account for proper commit attribution. Verify with:
```bash
git config --global user.name
git config --global user.email
```

---

## 8. wrangler deploy fails with account error

**Cause:** Not logged in to Cloudflare, or logged in to the wrong account.

**Fix:**

Check current authentication:
```bash
wrangler whoami
```

If not authenticated or wrong account, re-login:
```bash
wrangler login
```

If multiple accounts exist, verify the correct account is shown in `wrangler whoami` output before deploying.

---

## 9. D1 database creation fails

**Cause:** Not authenticated with Cloudflare, or the account does not have permissions for D1.

**Fix:**

Verify authentication:
```bash
wrangler whoami
```

If not authenticated:
```bash
wrangler login
```

Then retry creating the database:
```bash
wrangler d1 create <database-name>
```

If still failing, check that the Cloudflare account has Workers paid plan or is within free tier limits.

---

## 10. "EACCES" errors on macOS

**Cause:** File permission issue. Most commonly caused by npm's global directory being owned by root, or an incomplete Homebrew installation.

**Fix for npm EACCES:**
```bash
mkdir -p ~/.npm-global
npm config set prefix '~/.npm-global'
```

Add to `~/.zshrc`:
```bash
export PATH="$HOME/.npm-global/bin:$PATH"
```

Then `source ~/.zshrc` and retry.

**Fix for Homebrew EACCES:**
```bash
sudo chown -R $(whoami) /opt/homebrew
```

If the error is on `/usr/local` (Intel Mac):
```bash
sudo chown -R $(whoami) /usr/local/lib /usr/local/bin
```

---

## 11. "MCP server failed" message on startup

**Cause:** One or more MCP servers need to be re-authorized. Services like Cloudflare and Figma use short-lived authorization tokens that expire periodically. When they expire, Claude shows "MCP server failed" at startup. This is normal and easy to fix.

**Fix:**

1. Type `/mcp` and press Enter
2. You'll see a list of your MCP servers — the ones needing attention will be marked
3. Select the server that needs re-authorization (e.g., Cloudflare or Figma)
4. A browser window will open — sign in and approve the connection

**If it doesn't work the first time:** Some services (like Cloudflare) have a separate login and authorization step. If you're not already signed into the service in your browser, the authorization may fail silently. Sign into the service first (e.g., go to dash.cloudflare.com and log in), then try `/mcp` again.

This will happen from time to time — it's not a sign that anything is broken, just that the authorization token expired.

---

## 12. Repeated "permission denied: /var/folders/zz/" errors

**Cause:** On macOS, each user gets a private temp directory at `/var/folders/XX/XXXXX/T/`. When a brand-new user account hasn't been properly initialized (typically because the Mac hasn't been restarted since the account was created), the system falls back to `/var/folders/zz/zyxvpxvq6csfxvn_n0000000000000/T/` — a system-level temp directory that normal users can't write to. This causes cascading permission errors in Claude Code, Wrangler, and other tools.

**Symptoms:**
- `zsh:1: permission denied: /var/folders/zz/.../claude-XXXX-cwd` after every command
- `EACCES: permission denied, mkdir '/var/folders/zz/.../miniflare-...'` when running `wrangler dev`
- Commands appear to succeed but always show "Error: Exit code 1"

**Fix:** Restart the Mac. This triggers macOS to create the per-user temp directory. After restart, check with:
```bash
echo $TMPDIR
```
It should show something like `/var/folders/k5/abc123/T/` (user-specific), not `/var/folders/zz/zyxvpxvq6csfxvn_n0000000000000/T/` (system fallback).

If the problem persists after restart, log out and back in, or try:
```bash
sudo launchctl kickstart -k system/com.apple.dirhelper
```

---

## 13. Uninstalling the AI Primer Skills plugin

If you want to remove the a-primer-skills plugin completely, you need to do both steps — uninstall the plugin and remove the marketplace source:

```
/plugin uninstall a-primer-skills@a-primer
/plugin marketplace remove ammonhaggerty/a-primer-skills
```

The first command removes the plugin itself. The second removes the marketplace entry so it no longer appears in your plugin list. If you only do the first step, the marketplace reference remains and Claude may still reference the plugin.

---

## General Debugging Tips

- **Restart the terminal** after modifying shell profiles (`.zshrc`, `.bashrc`). Many "command not found" issues resolve with a fresh shell session.
- **Check PATH** with `echo $PATH` to verify expected directories are included.
- **Use `which <command>`** to find where a command is installed, or confirm it is not found.
- **Re-source the profile** without restarting: `source ~/.zshrc` (or `source ~/.bashrc`).
- **Read the error message carefully** — it usually contains the exact file path or config key causing the issue.
