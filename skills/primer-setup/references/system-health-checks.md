# System Health Checks Reference

Diagnostic commands to verify the system is ready for setup. Run all of these before installing any tools. Each check is marked as **blocker** (must fix before proceeding) or **warning** (note and continue).

---

## Admin Access — BLOCKER

The user needs admin access to install Homebrew, Xcode Command Line Tools, and global npm packages.

**Check (macOS):**
```bash
groups $(whoami) | grep -q admin && echo "HAS_ADMIN" || echo "NO_ADMIN"
```

**Successful result:** Output includes `HAS_ADMIN`.

**If no admin access:** Stop setup. Tell the user: "Your account doesn't have admin access, which is needed to install developer tools. If this is a work or shared computer, ask your IT department or the computer's owner to add your account to the admin group."

---

## Temp Directory Permissions — BLOCKER

macOS creates per-user temp directories on login. Fresh user accounts sometimes don't have these set up until the first restart. Without a writable temp directory, Claude Code, Wrangler, Playwright, and npm will all fail with cryptic permission errors.

**Check what the shell is using:**
```bash
echo "TMPDIR=$TMPDIR"
```

**Check what macOS says it should be:**
```bash
getconf DARWIN_USER_TEMP_DIR
```

**Successful result:** Both commands return the same user-specific path, like `/var/folders/k5/abc123def/T/`.

**Failure indicator:** `$TMPDIR` shows `/var/folders/zz/zyxvpxvq6csfxvn_n0000000000000/T/` — this is the system fallback directory, not writable by normal users. This happens when a user account was created as Standard and later upgraded to Admin, or when the per-user temp directory wasn't properly initialized.

**Verify write access:**
```bash
touch "$TMPDIR/primer-health-check-$$" && rm "$TMPDIR/primer-health-check-$$" && echo "TMPDIR_WRITABLE" || echo "TMPDIR_NOT_WRITABLE"
```

**If TMPDIR is the system fallback or not writable:** Fix it by setting the correct value in the shell profile. First, get the correct path:
```bash
getconf DARWIN_USER_TEMP_DIR
```

If that returns a valid user-specific path (not the `zz` fallback), add it to the shell profile:
```bash
echo 'export TMPDIR=$(getconf DARWIN_USER_TEMP_DIR)' >> ~/.zshrc
source ~/.zshrc
```

Then verify:
```bash
echo $TMPDIR
touch "$TMPDIR/primer-health-check-$$" && rm "$TMPDIR/primer-health-check-$$" && echo "TMPDIR_WRITABLE" || echo "TMPDIR_NOT_WRITABLE"
```

If `getconf DARWIN_USER_TEMP_DIR` also returns the `zz` fallback, try forcing macOS to create the per-user directory:
```bash
sudo launchctl kickstart -k system/com.apple.dirhelper
```
Then log out and back in. If still broken, restart the Mac.

Tell the user in plain English: "Your system's temporary directory isn't set up correctly. This sometimes happens with accounts that were created as a standard account and later upgraded to admin. I'm going to fix it by adding one line to your shell profile — this tells your terminal where to put temporary files."

---

## Home Directory Write Access — BLOCKER

Verify the user can create files and directories in their home directory and the workspace location.

**Check:**
```bash
touch ~/primer-health-check-$$ && rm ~/primer-health-check-$$ && echo "HOME_WRITABLE" || echo "HOME_NOT_WRITABLE"
```

**If `~/Development` exists, also check:**
```bash
touch ~/Development/primer-health-check-$$ && rm ~/Development/primer-health-check-$$ && echo "WORKSPACE_WRITABLE" || echo "WORKSPACE_NOT_WRITABLE"
```

**If not writable:** Stop setup. Tell the user: "Your home directory or workspace isn't writable. This is unusual and may indicate a permissions issue with your user account. Try restarting your Mac. If it persists, you may need to check your account permissions in System Settings → Users & Groups."

---

## Network Connectivity — BLOCKER

Setup needs to download tools from the internet. Check that key services are reachable.

**Check (test all three):**
```bash
curl -s --max-time 5 -o /dev/null -w "%{http_code}" https://github.com
curl -s --max-time 5 -o /dev/null -w "%{http_code}" https://registry.npmjs.org
curl -s --max-time 5 -o /dev/null -w "%{http_code}" https://dash.cloudflare.com
```

**Successful result:** HTTP status 200 (or 301/302) for all three.

**If any fail:** Tell the user which service is unreachable. Common causes: no internet connection, corporate firewall/VPN blocking access, DNS issues. Suggest: "Check your internet connection and try again. If you're on a corporate network, you may need to disconnect from VPN or ask IT to allow access to github.com, npmjs.org, and cloudflare.com."

---

## Disk Space — WARNING

Developer tools, npm packages, and browser binaries (Playwright) need several gigabytes.

**Check:**
```bash
df -h ~ | tail -1 | awk '{print $4}'
```

**Healthy:** At least 5GB free.

**If less than 5GB:** Warn the user: "You have less than 5GB of free disk space. The setup needs to download developer tools and browser binaries which can take a few gigabytes. Consider freeing some space before continuing, or the install may fail partway through."

---

## Shell — INFORMATIONAL

Detect the shell for PATH configuration instructions.

**Check:**
```bash
echo $SHELL
```

**Expected results:**
- `/bin/zsh` — Default on modern macOS. Profile file is `~/.zshrc`.
- `/bin/bash` — Older macOS or Linux. Profile file is `~/.bashrc`.

This determines which profile file to modify when adding Homebrew to PATH or fixing other environment issues.

---

## macOS Version — INFORMATIONAL

**Check:**
```bash
sw_vers -productVersion
```

**Expected:** macOS 12 (Monterey) or newer. Older versions may have compatibility issues with current tool versions.
