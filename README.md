# My Claude Skills

Personal collection of Claude Code skills and commands.

## Claude Code Discovery Chain

Claude Code walks **up the full directory tree** from your working directory, merging all `.claude/` folders it finds. It does NOT stop at the first one.

### Load order

```
~/.claude/                          ← Global (always loaded)
  ↑
/home/user/project-group/.claude/   ← Parent (merged in)
  ↑
/home/user/project-group/app/.claude/  ← Current dir (merged in, highest priority)
```

Deeper directories take precedence on conflicts.

### What gets discovered

| Item | Location | Scope |
|------|----------|-------|
| Settings | `.claude/settings.json` | Per-directory |
| Memory | `CLAUDE.md`, `CLAUDE.local.md` | Per-directory |
| Commands (legacy) | `.claude/commands/*.md` | Per-directory |
| Skills (current) | `.claude/skills/<name>/SKILL.md` | Per-directory |

### Key quote from docs

> "Claude Code reads memories recursively: starting in the cwd, Claude Code recurses up to (but not including) the root directory `/` and reads any CLAUDE.md or CLAUDE.local.md files it finds."

### References

- [Memory](https://code.claude.com/docs/en/memory.md) — directory traversal behavior
- [Skills](https://code.claude.com/docs/en/skills.md) — skill discovery and structure
- [Settings](https://code.claude.com/docs/en/settings.md) — configuration scopes and precedence
