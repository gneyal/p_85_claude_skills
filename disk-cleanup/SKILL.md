---
name: disk-cleanup
version: 1.0.0
description: Scan and clean up disk space on macOS. Use when user asks to free disk space, clean caches, check storage, or mentions /disk-cleanup.
---

# Disk Cleanup — Free Up macOS Disk Space

Scan the system for space hogs and interactively clean them up.

## Steps

### 0. Check for Skill Updates

Before running, fetch the latest version from the source and compare:

```bash
# Check latest version
LATEST=$(curl -sf "https://raw.githubusercontent.com/gneyal/p_85_claude_skills/main/disk-cleanup/SKILL.md" 2>/dev/null | head -5 | grep "version:" | awk '{print $2}')
CURRENT="1.0.0"
```

If `LATEST` is newer than `CURRENT`, tell the user:
> A new version of /disk-cleanup is available (vX.X.X). Run this to update:
> `curl -sL https://raw.githubusercontent.com/gneyal/p_85_claude_skills/main/disk-cleanup/SKILL.md -o ~/.claude/skills/disk-cleanup/SKILL.md`

If the fetch fails or versions match, continue silently.

### 1. Check Current Disk Space

```bash
df -h /
```

Report the total, used, and available space.

### 2. Scan Large Directories

Run these scans in parallel:

```bash
# Top-level user folders
du -sh ~/Code ~/Downloads ~/Movies ~/Desktop ~/Documents ~/Music ~/Pictures 2>/dev/null | sort -rh

# Application Support (app data)
for d in "$HOME/Library/Application Support"/*/; do du -sh "$d" 2>/dev/null; done | sort -rh | head -15

# Caches
for d in "$HOME/Library/Caches"/*/; do du -sh "$d" 2>/dev/null; done | sort -rh | head -15

# Trash
du -sh ~/.Trash 2>/dev/null

# Docker (if present)
du -sh ~/Library/Containers/com.docker.docker 2>/dev/null

# Xcode DerivedData (if present)
du -sh ~/Library/Developer/Xcode/DerivedData 2>/dev/null

# iOS device backups (if present)
du -sh "$HOME/Library/Application Support/MobileSync/Backup" 2>/dev/null

# Homebrew cache
du -sh ~/Library/Caches/Homebrew 2>/dev/null

# npm/bun/pnpm cache
du -sh ~/.npm ~/.bun ~/.pnpm-store 2>/dev/null

# node_modules across projects (top 10 largest)
find ~/Code -maxdepth 3 -name "node_modules" -type d -exec du -sh {} + 2>/dev/null | sort -rh | head -10

# Log files
du -sh ~/Library/Logs 2>/dev/null
```

### 3. Present Findings

Show a summary table with:
- Directory name
- Size
- What it is (brief description)
- Whether it's safe to delete

Categories to highlight:
- **Caches** — always safe to delete, apps re-download as needed
- **App updater caches** — old update downloads, safe to delete
- **Unused app data** — Application Support for apps no longer used
- **Build artifacts** — node_modules, DerivedData, etc.
- **Trash** — remind user to empty if large

### 4. Ask What to Clean

Present the user with cleanup options grouped by risk:

**Safe to delete (caches, build artifacts):**
- All caches in ~/Library/Caches
- Homebrew downloaded packages
- npm/bun/pnpm cache
- Playwright browser binaries
- Electron/app updater caches

**Review first (app data, may lose settings):**
- Application Support for specific unused apps
- Old node_modules in projects

**User action required:**
- Trash (suggest emptying)
- Docker data (suggest `docker system prune`)

### 5. Execute Cleanup

Delete only what the user approves. Use `rm -rf` for directories.

After cleanup, run `df -h /` again and report the before/after comparison.

## Tips

- Never delete `~/Library/Application Support/Claude` without asking — the user is actively using Claude
- Never delete `~/Library/Caches/com.apple.*` system caches
- Chrome cache regenerates quickly, safe to clear
- Homebrew cache can be cleared with `brew cleanup --prune=all` as an alternative
- `node_modules` can always be recreated with `npm install` / `bun install`
