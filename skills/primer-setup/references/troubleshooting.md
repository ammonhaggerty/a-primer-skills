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

**Cause:** Usually a JSON syntax error in `~/.claude/settings.json`. JSON does not allow trailing commas, single quotes, or comments.

**Fix:**

Open the config file and validate the JSON:
```bash
cat ~/.claude/settings.json | python3 -m json.tool
```

If python3 reports an error, it will indicate the line number. Common mistakes:
- Trailing comma after the last item in an object or array
- Missing closing brace or bracket
- Single quotes instead of double quotes
- Unescaped special characters in strings

After fixing the JSON, restart the Claude Code session.

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

## General Debugging Tips

- **Restart the terminal** after modifying shell profiles (`.zshrc`, `.bashrc`). Many "command not found" issues resolve with a fresh shell session.
- **Check PATH** with `echo $PATH` to verify expected directories are included.
- **Use `which <command>`** to find where a command is installed, or confirm it is not found.
- **Re-source the profile** without restarting: `source ~/.zshrc` (or `source ~/.bashrc`).
- **Read the error message carefully** — it usually contains the exact file path or config key causing the issue.
