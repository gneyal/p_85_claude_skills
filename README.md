# Claude Code Skills

A collection of custom skills for [Claude Code](https://claude.com/claude-code).

## Available Skills

### /disk-cleanup

Scan and clean up disk space on macOS. Finds caches, unused app data, build artifacts, and other space hogs — then lets you choose what to delete.

**Install:**

```bash
mkdir -p ~/.claude/skills/disk-cleanup && curl -sL https://raw.githubusercontent.com/gneyal/claude-skills/main/disk-cleanup/SKILL.md -o ~/.claude/skills/disk-cleanup/SKILL.md
```

Restart Claude Code. Type `/disk-cleanup` and go.
