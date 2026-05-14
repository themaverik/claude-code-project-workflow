---
paths:
  - ".gitignore"
  - ".git/**"
  - "**/*.sh"
---

# Git Policy

Read `.claude/references/git-conventions.md` for full conventions.

## Critical Rules

- Never run `git push` without explicit user confirmation — no exceptions
- No AI tool attribution in commits (no "Generated with Claude Code" or "Co-Authored-By: Claude")
- Use `themaverik` as commit author
- Commit message type: use `build` for skills, hooks, MCP config, settings changes
- Follow Conventional Commits — subject max 50 chars, imperative mood, no period
- Create `.gitignore` before first commit if missing
- Confirm before any force push or rebase on shared branches

## New Project Gitignore Strategy

Always ask before creating ignore files:
1. `.gitignore` — committed, visible to everyone. Use for open source or shared teams.
2. `.git/info/exclude` — local only, never pushed. Use for proprietary repos and personal tooling.
3. Both — `.gitignore` for shared rules, `.git/info/exclude` for personal exclusions.

Recommended for werkflow/werkflow-erp: option 3.
- `.gitignore`: node_modules/, target/, dist/, .env* (!.env.example), *.log
- `.git/info/exclude`: LLM.md, CLAUDE.md, .claude/

## Task Status Markers (ROADMAP.md)

- `[ ]` Pending
- `[~]` In progress — always note what remains
- `[x]` Completed — always note the commit hash
- `[!]` Blocked — always note the reason and what resolves it
