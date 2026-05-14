# Claude Code Project Configuration

## Core Directives

1. **Token Efficiency First:** Use claude-mem for history, graphify for structure.
2. **Modular Execution:** Defer to ecc skills and caveman for execution.
3. **Highest Industry Standards:** Strict typing, clean architecture, modularity.
4. Read and update master Roadmap from/to `Roadmap.md`. Do not commit changes to this file alone.
5. Default code directories: `./src` for source, `./tests` for tests — update per project.

## Documentation Standards

For full documentation conventions, read `.claude/references/documentation-standards.md`.

Critical rules (always apply):
- No icons or emojis in documentation or code unless explicitly stated
- Kebab-case for documentation file names (e.g. `git-conventions.md`)
- Markdown format for all code documentation

## Git Policy

For branch naming conventions, commit message format, and task status markers, read `.claude/references/git-conventions.md`.

Critical rules (always apply):
- No Claude references in commits
- **NEVER run `git push` automatically.** Always ask the user before pushing
- Follow the official style guide for your language
- Use linters and formatters appropriate for the tech stack

## Coding Behavior (Karpathy Principles)

Full guidelines in `karpathy-guidelines` skill.

- **Think before coding:** State assumptions explicitly. Ask before guessing.
- **Simplicity first:** Minimum code. No speculative features or abstractions.
- **Surgical changes:** Touch only what the task requires. Match existing style.
- **Goal-driven execution:** Define verifiable success criteria before starting.

## Code Review Strategy

- **Always first:** `ecc:<language>-review` (~5% cost)
- **Only when needed:** `/superpowers:requesting-code-review`, `/superpowers:security-auditor` (~15% cost)
- **When to use Superpowers:** Auth, payments, data handling, major refactors, pre-production

## On-Demand Skills and Agents

Loaded only when their trigger conditions are met.

| Trigger | What to Load |
|---------|-------------|
| New feature task | Brainstorm before coding (project-resume skill R3) |
| Docker / container task | `docker-dev` skill + `.claude/references/docker-services.md` |
| Code review | `.claude/agents/code-reviewer.md` or `/ecc:code-review` |
| Security audit | `.claude/agents/security-reviewer.md` or `/superpowers:security-auditor` |
| Session start / resume | `project-resume` skill — handles session brief, branch check, ROADMAP tracking |
| Git commit / branching | `.claude/references/git-conventions.md` |
| Creating / editing docs | `.claude/references/documentation-standards.md` |
| Architecture decisions | `adr-manager` skill — auto-captures all architectural decisions |

## Token Management

- Run `/compact` after each phase completion or 10 turns
- Use `/clear` between unrelated tasks
- Check `/cost` periodically
- Language rules are path-triggered — they only load when editing matching files
