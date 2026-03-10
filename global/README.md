# Global Claude Configuration

This folder contains global configuration files and skills for Claude Code.

## Files

| File | Source | Purpose |
|------|--------|---------|
| `CLAUDE.md` | `~/.claude/CLAUDE.md` | Global Claude-specific instructions |
| `LLM.md` | `~/.claude/LLM.md` | Base development guidelines for all LLM tools |
| `skills/project-resume/SKILL.md` | `~/.claude/skills/project-resume/` | Session resume skill |

## Installation

Copy these files to your global Claude config directory:

```bash
cp global/CLAUDE.md ~/.claude/CLAUDE.md
cp global/LLM.md ~/.claude/LLM.md
cp -r global/skills/project-resume ~/.claude/skills/
```

---

## Additional Skills to Install

The following skills are referenced in the system but must be installed separately from [Superpowers](https://superpowers.club) or their respective sources:

### Superpowers Skills (install via Superpowers)

| Skill | Description |
|-------|-------------|
| `superpowers:using-superpowers` | Core skill dispatch and routing |
| `superpowers:brainstorming` | Brainstorm before creative/feature work |
| `superpowers:writing-plans` | Write implementation plans |
| `superpowers:executing-plans` | Execute multi-step plans with review checkpoints |
| `superpowers:systematic-debugging` | Structured debugging workflow |
| `superpowers:test-driven-development` | TDD workflow |
| `superpowers:requesting-code-review` | Request code review on completion |
| `superpowers:receiving-code-review` | Handle incoming code review feedback |
| `superpowers:verification-before-completion` | Verify before claiming work is done |
| `superpowers:finishing-a-development-branch` | Complete and integrate a dev branch |
| `superpowers:using-git-worktrees` | Isolated git worktree management |
| `superpowers:dispatching-parallel-agents` | Parallel agent task dispatch |
| `superpowers:subagent-driven-development` | Execute plans with subagents |
| `superpowers:writing-skills` | Create/edit skills |

### Other Built-in Skills (included with Claude Code Superpowers)

| Skill | Description |
|-------|-------------|
| `docker-dev` | Docker/container lifecycle management |
| `commit` | Smart git commit |
| `review` | Code review |
| `test` | Smart test runner |
| `session-start` | Start coding session |
| `session-end` | End coding session |
| `make-it-pretty` | UI polish |
| `format` | Auto format code |
| `simplify` | Code quality review |
| `security:security-audit` | Security audit |
| `frontend-design:frontend-design` | Premium frontend design |
| `claude-api` | Build apps with Anthropic SDK |
| `skill-creator:skill-creator` | Create/optimize skills |
