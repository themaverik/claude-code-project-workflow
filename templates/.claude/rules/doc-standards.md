---
paths:
  - "**/*.md"
  - "docs/**"
---

# Documentation Standards

Read `.claude/references/documentation-standards.md` for full conventions.

## Critical Rules

- No icons or emojis unless explicitly stated
- Exception: status indicators in README.md and ROADMAP.md only
  (done, in progress, not started, new release, declined)
- File naming: kebab-case for config/reference files, hyphenated Title-Case for docs/ files
- Exception: README.md, ROADMAP.md, CHANGELOG.md, LICENSE, CONTRIBUTING.md keep standard names
- Exception: ADR files follow MADR standard — `ADR-NNN-<kebab-slug>.md` (uppercase prefix)
- Document titles: Title Case inside the file
- `docs/Knowledge-Base.md` is auto-generated — never edit directly
- `docs/adr/index.md` is auto-maintained — never edit directly
- README.md must stay minimal: what it is, how to run it, ports, credentials only
