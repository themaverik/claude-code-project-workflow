# Claude Code Development Workflow Guide

Day-to-day reference for development sessions using Claude Code. For initial setup (global config, project scaffolding, language rule sets, skill reference), see `project-setup-guide.md`.

---

## Table of Contents

1. [Architecture Overview](#1-architecture-overview)
2. [Starting a Session](#2-starting-a-session)
3. [Development -- New Feature](#3-development----new-feature)
4. [Development -- Bug Fix](#4-development----bug-fix)
5. [UI Development](#5-ui-development)
6. [Docker and Infrastructure](#6-docker-and-infrastructure)
7. [Testing](#7-testing)
8. [Code Review](#8-code-review)
9. [Security Review](#9-security-review)
10. [Git Workflow](#10-git-workflow)
11. [Ending a Session](#11-ending-a-session)
12. [Resuming a Session](#12-resuming-a-session)
13. [Context Optimization](#13-context-optimization)
14. [Quick Skill Reference](#14-quick-skill-reference)

---

## 1. Architecture Overview

### Configuration Layers

```
~/.claude/CLAUDE.md                    Global (every project, every message)
~/.claude/rules/common/                Global flat rules (every file type)
  coding-style.md
  code-review.md
  security.md
  testing.md
  hooks.md
  patterns.md
  git-workflow.md
  development-workflow.md              (path-triggered: *.java, *.ts, *.py, *.go, *.rs)
  performance.md                       (path-triggered: same as above)
~/.claude/rules/                       Global path-triggered rules
  doc-standards.md                     (*.md, docs/**)
  docker.md                            (infrastructure/**, docker-compose*.yml)
  git-policy.md                        (.gitignore, .git/**, *.sh)
~/.claude/skills/                      Global skills (plugin-loaded)
  project-resume/
  adr-manager/
  doc-sync/
  docker-dev/
  karpathy-guidelines/
  knowledge-ingest/
  knowledge-query/
  pr-description/
  changelog-roadmap-policy.md

project/CLAUDE.md                      Project-level (always loaded)
project/.claude/rules/                 Language rule sets (path-triggered)
  java/, typescript/, python/, etc.
project/.claude/skills/                Project-level skills (plugin-loaded)
project/.claude/references/            On-demand reference files
  git-conventions.md
  documentation-standards.md
  docker-services.md
project/.claude/agents/                Project-level agents (explicit invoke)
  code-reviewer.md
  security-reviewer.md
```

### Loading Behavior

| Layer | When Loaded | Context Cost |
|-------|-------------|--------------|
| `~/.claude/CLAUDE.md` | Every message | ~100 lines |
| `~/.claude/rules/common/*.md` | Every message | ~9 files × ~40 lines |
| `project/CLAUDE.md` | Every message | ~50 lines |
| Language rules (e.g. `java/`) | On matching file edit | ~5 files × ~40 lines |
| Path-triggered flat rules | On matching file edit | ~40 lines each |
| Skill metadata | Every message | ~2 lines per skill |
| Skill body | On trigger only | 0 until triggered |
| Reference files | On demand only | 0 until read |
| Agent definitions | On explicit invocation | 0 until invoked |

### Precedence

```
Project CLAUDE.md > Global CLAUDE.md > Default Claude behavior
```

---

## 2. Starting a Session

### Launch

```bash
claude
```

Skills load via plugins automatically — no `--add-dir` flag needed.

### What Happens (project-resume skill)

1. Reads `CLAUDE.md`, `Roadmap.md` (or `CHANGELOG.md` + `Roadmap.md` for post-MVP projects), `git-conventions.md`, runs `git log --oneline -10`
2. Finds resume point: first `[~]` task → first `[ ]` task
3. Prints session brief:

```
------------------------------------------------------------
Session Resume
------------------------------------------------------------
Branch       : feature/user-authentication
Active Phase : Phase 2 -- Core Features
Resuming     : 2.3 User dashboard
Stopped At   : API done, UI pending
Last Commit  : f7a1ae7 fix(portal): resolve login timeout
ADR Records  : 3 (docs/adr/)
Knowledge    : synced 2026-04-30
------------------------------------------------------------
ADR capture  : ACTIVE (adr-manager -- decisions logged automatically)
------------------------------------------------------------
```

4. Branch check (waits for answer before any code):

```
Branch check:
  Current branch: feature/user-authentication
  (1) Continue on this branch
  (2) Create a new feature branch
  (3) Create a new fix branch
```

### Standing Rules (enforced every session)

| Rule | Behaviour |
|------|-----------|
| R1 -- Never push without consent | Commits freely. Never pushes. Says "Ready to push" after each commit. Prior-session consent does not carry over. |
| R2 -- Docker pre-flight | Runs `docker info` before any container command. Stops if Docker is not running. |
| R3 -- Brainstorm before new features | Requires brainstorm + sign-off before coding any new behaviour. Skipped for bug fixes, chores, tasks with full steps in Roadmap.md. |
| R4 -- Update Roadmap.md after every task | Marks tasks `[x]` with commit hash. Advances next task to `[~]`. Commits code + Roadmap.md together. |

---

## 3. Development -- New Feature

### Step 1: Brainstorm (auto via R3)

```
Feature Brainstorm: <name>
------------------------------------------------------------
Goal            : What problem does this solve?
Approaches      : 2-3 options
Recommended     : Which and why
Trade-offs      : What the choice gives up
Affected files  : Modules that change
Risks           : What could go wrong
Open questions  : Needs clarification?
------------------------------------------------------------
```

Waits for user sign-off. `adr-manager` automatically converts the brainstorm into an ADR and commits it with the feature code — no user action needed.

### Step 2: Plan (complex features)

```
/superpowers:writing-plans
```

Produces a phased implementation plan before any code is written.

### Step 3: Implement

- Follow Karpathy principles (loaded via `karpathy-guidelines` skill): think first, surgical changes, minimum code
- Language rules load automatically when editing matching files
- Files under 800 lines (preferably 200-400)

### Step 4: After each task

1. Roadmap.md: `[ ]` → `[x]` with `*(commit: <hash>)*`
2. Next task marked `[~]`
3. `## Current Session State` updated
4. Code + Roadmap.md committed together
5. "Ready to push -- confirm when you'd like me to."

---

## 4. Development -- Bug Fix

### Step 1: Systematic debugging

```
/superpowers:systematic-debugging
```

Reproduces the issue, identifies root cause, verifies the fix. No brainstorm needed for bugs with a known cause.

### Step 2: Fix and test

- Write regression test if applicable
- Fix the bug
- Verify the fix passes

### Step 3: Commit

```
fix(scope): description of what was fixed
```

---

## 5. UI Development

### Trigger

The `frontend-design` skill activates for UI components, pages, or applications.

### What Happens

- Bold aesthetic direction chosen (not generic AI patterns)
- Distinctive typography, colour, motion
- Uses existing stack conventions (React, TailwindCSS, shadcn/ui, etc.)
- Production-grade, accessible components
- `web/` rules load automatically when editing frontend files

---

## 6. Docker and Infrastructure

### Trigger

The `docker-dev` skill activates when mentioning docker, containers, compose, spin up, tear down, logs, restart, rebuild.

### Pre-flight (always first)

```bash
docker info > /dev/null 2>&1 && echo "RUNNING" || echo "NOT RUNNING"
```

If not running, Claude stops and asks the user to start Docker.

### Data Protection Rules

Claude will always ask for explicit confirmation before:
- Removing volumes (`docker compose down -v`, `docker volume rm/prune`)
- Pruning containers, images, or system
- Rebuilding database or storage containers

Prior-session consent does not carry over.

### Project-Specific Config

If the project has `.claude/references/docker-services.md`, Claude reads it for compose file paths, service map, ports, and project-specific workflows.

---

## 7. Testing

### ECC Test Skills

```bash
/ecc:go-test
/ecc:rust-test
/ecc:kotlin-test
/ecc:flutter-test
```

### TDD Enforcement

```
/superpowers:test-driven-development
```

Enforces write-tests-first (RED → GREEN → REFACTOR). Required minimum coverage: 80%.

### Verification Before Completion

```
/superpowers:verification-before-completion
```

Runs actual verification commands and shows output before claiming anything is done.

---

## 8. Code Review

### Strategy: ECC first, Superpowers for critical code

| Scenario | Command | Cost |
|----------|---------|------|
| Daily code review | `/ecc:java-review`, `/ecc:typescript-review`, `/ecc:python-review`, `/ecc:go-review`, etc. | ~5% |
| Auth / payments / data handling | `/superpowers:requesting-code-review` | ~15% |
| Pre-production validation | `/superpowers:requesting-code-review` | ~15% |
| Major architecture change | `/superpowers:requesting-code-review` | ~15% |

### Project Agent

Each project has `.claude/agents/code-reviewer.md`. Invoked via `/review` or after feature completion.

Evaluates: correctness, code quality, performance, security, maintainability, API design.

Output format:
```
Code Review: <branch or feature>
------------------------------------------------------------
Files reviewed : [count]
Issues found   : [count]
Verdict        : APPROVE / REQUEST CHANGES / COMMENT
------------------------------------------------------------
```

Severity: **BLOCKER** → **MAJOR** → **MINOR** → **NIT**. Only blockers prevent merge.

---

## 9. Security Review

### Trigger

Before deployment, or on demand.

### Command

```
/superpowers:requesting-code-review   ← for auth, payments, sensitive data
/ecc:security-review                  ← general security scan
```

### Project Agent

Each project has `.claude/agents/security-reviewer.md` tailored to its stack.

Output format:
```
Security Review: [Project Name]
------------------------------------------------------------
CRITICAL : [count]
HIGH     : [count]
MEDIUM   : [count]
LOW      : [count]
------------------------------------------------------------
```

---

## 10. Git Workflow

### Branch Naming (from `.claude/references/git-conventions.md`)

| Type | Pattern | Example |
|------|---------|---------|
| Feature | `feature/<description>` | `feature/user-auth` |
| Bug fix | `fix/<description>` | `fix/login-timeout` |
| Hotfix | `hotfix/<description>` | `hotfix/security-patch` |
| Refactor | `refactor/<scope>` | `refactor/database-layer` |
| Docs | `docs/<topic>` | `docs/api-endpoints` |

### Commit Format

```
<type>(<scope>): <subject>
```

Types: `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `build`, `ci`, `chore`, `revert`

### Critical Rules

- No Claude references in commits
- Never push without explicit user confirmation
- Prior-session push consent does not carry over

### Task Markers in Roadmap.md

| Marker | Status |
|--------|--------|
| `[ ]` | Pending |
| `[~]` | In Progress (always add note on what remains) |
| `[x]` | Completed (always add commit hash) |
| `[!]` | Blocked (always add reason) |

### Branch Completion

```
/superpowers:finishing-a-development-branch
```

Guides through merge / PR / cleanup when all branch tasks are done.

### PR Description

```
/pr-description
```

Or auto-triggers on "create a PR", "write a PR", "summarise changes for a pull request".

---

## 11. Ending a Session

### Normal End

1. All completed tasks marked `[x]` with commit hashes in Roadmap.md
2. Any in-progress task marked `[~]` with precise notes on what remains
3. `## Current Session State` updated
4. ADR sweep: any uncaptured decisions? (`adr-manager` handles most automatically)
5. Final commit (code + Roadmap.md + any ADRs)
6. If ADRs were written this session: `doc-sync` skill consolidates to `docs/Knowledge-Base.md`
7. "Ready to push -- confirm when you'd like me to."

### Context Limit Hit (automatic)

1. Current task updated to `[~]` with exact stopping point
2. `## Current Session State` updated
3. All changes committed
4. Next session picks up from the `[~]` marker

---

## 12. Resuming a Session

```bash
claude
# type: resume
```

Or headless:

```bash
claude --dangerously-skip-permissions -p "resume"
```

The `project-resume` skill runs the full session start sequence. The `[~]` marker tells Claude exactly where to pick up.

---

## 13. Context Optimization

### Always Loaded

| Source | Approx Lines |
|--------|-------------|
| `~/.claude/CLAUDE.md` | ~100 |
| `~/.claude/rules/common/` (9 files) | ~360 |
| `project/CLAUDE.md` | ~50 |
| Skill metadata (all skills) | ~30 |
| **Total baseline** | **~540** |

### On-Demand (0 lines until triggered)

| Source | Trigger | Lines |
|--------|---------|-------|
| Language rules (e.g. `java/`) | Editing matching file | ~200 per set |
| `doc-standards.md` | Editing `*.md` files | ~40 |
| `docker.md` | Editing docker files | ~40 |
| `project-resume` body | Session start / "resume" | ~120 |
| `adr-manager` body | Architecture decisions | ~80 |
| `docker-dev` body | Docker tasks | ~72 |
| `karpathy-guidelines` body | Any coding task | ~60 |
| `git-conventions.md` | Committing / branching | ~93 |
| `documentation-standards.md` | Editing docs | ~50 |
| `docker-services.md` | Docker tasks (project) | ~30 |
| `code-reviewer` agent | Code review | ~90 |
| `security-reviewer` agent | Security audits | ~120 |
| Superpowers skills | Explicit `/superpowers:*` | ~200 each |
| ECC skills | Explicit `/ecc:*` | Varies |

### How to Keep It Optimized

- **Project CLAUDE.md stays slim** — only critical directives. Skills and reference files handle detail.
- **No duplication** — if a skill has the content, don't repeat it in CLAUDE.md.
- Run `/compact` after each phase or ~10 turns. Run `/clear` between unrelated tasks.

---

## 14. Quick Skill Reference

### Auto-triggered (no command needed)

| Natural language | Skill |
|-----------------|-------|
| "resume", "continue", "where were we" | `project-resume` |
| "log this decision", "save this ADR", tech stack choices | `adr-manager` |
| "sync docs", "update knowledge base" | `doc-sync` |
| "spin up", "bring up docker", "tear down containers" | `docker-dev` |
| "ingest specs", "what does the spec say about X" | `knowledge-ingest` |
| "what did we decide about X", "any ADR for Y" | `knowledge-query` |
| Any coding, reviewing, or refactoring task | `karpathy-guidelines` |
| "create a PR", "write a PR", "summarise changes" | `pr-description` |

### ECC Skills (invoke with /ecc:<skill>)

```
# Code review
/ecc:java-review
/ecc:typescript-review
/ecc:python-review
/ecc:go-review
/ecc:kotlin-review
/ecc:rust-review

# Build and test
/ecc:java-build    /ecc:go-build    /ecc:rust-build    /ecc:flutter-build
/ecc:go-test       /ecc:rust-test   /ecc:kotlin-test   /ecc:flutter-test

# Planning
/ecc:plan          /ecc:feature-dev
/ecc:prp-prd       /ecc:prp-plan    /ecc:prp-implement  /ecc:prp-commit

# Quality
/ecc:code-review   /ecc:security-review   /ecc:verify   /ecc:docs

# Session
/ecc:save-session  /ecc:resume-session   /ecc:context-budget
```

### Superpowers (invoke with /superpowers:<skill> — ~10-15% cost each)

```
/superpowers:brainstorming               ← before building any new feature
/superpowers:writing-plans               ← turn a spec into a phased plan
/superpowers:test-driven-development     ← enforce write-tests-first
/superpowers:systematic-debugging        ← structured bug investigation
/superpowers:verification-before-completion ← evidence before claiming done
/superpowers:requesting-code-review      ← auth, payments, pre-production
/superpowers:finishing-a-development-branch ← merge/PR/cleanup when done
/superpowers:writing-skills              ← create a new skill
```

### Claude-mem

```
/claude-mem:do              ← general memory operation
/claude-mem:mem-search      ← search memory for a topic
/claude-mem:smart-explore   ← deep explore memory around a theme
/claude-mem:knowledge-agent ← answer a question using memory as context
/claude-mem:make-plan       ← create a plan stored in memory
/claude-mem:timeline-report ← chronological report of project activity
```
