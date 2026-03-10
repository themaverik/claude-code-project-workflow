# Claude Code Development Workflow Guide

Complete reference for the development cycle using Claude Code with global skills, project-level configuration, and context-optimized on-demand loading.

---

## Table of Contents

1. [Architecture Overview](#1-architecture-overview)
2. [Global Setup (One-Time)](#2-global-setup-one-time)
3. [Project Setup](#3-project-setup)
4. [Starting a Session](#4-starting-a-session)
5. [Development -- New Feature](#5-development----new-feature)
6. [Development -- Bug Fix](#6-development----bug-fix)
7. [UI Development](#7-ui-development)
8. [Docker and Infrastructure](#8-docker-and-infrastructure)
9. [Testing](#9-testing)
10. [Code Review](#10-code-review)
11. [Security Review](#11-security-review)
12. [Git Workflow](#12-git-workflow)
13. [Ending a Session](#13-ending-a-session)
14. [Resuming a Session](#14-resuming-a-session)
15. [Context Optimization](#15-context-optimization)
16. [Installed Plugins and Skills](#16-installed-plugins-and-skills)

---

## 1. Architecture Overview

### Configuration Layers

```
~/.claude/CLAUDE.md                    Global (all projects)
~/.claude/skills/                      Global skills (all projects)
  project-resume/SKILL.md               Session resume + task tracking
  docker-dev/SKILL.md                   Docker pre-flight + data protection
~/.agents/skills/                      Global agent skills
  browser-use/SKILL.md                  Browser automation

project/CLAUDE.md                      Project-level (always loaded)
project/.claude/references/            On-demand reference files
  git-conventions.md                    Branch/commit/task markers
  docker-services.md                    Project-specific docker config
project/.claude/agents/                Project-level agents
  security-reviewer.md                  Security audit subagent
```

### Loading Behavior

| Layer | When Loaded | Context Cost |
|-------|-------------|--------------|
| `~/.claude/CLAUDE.md` | Every message | ~100 lines (always) |
| `project/CLAUDE.md` | Every message | ~130 lines (always) |
| Skill metadata (name + description) | Every message | ~2 lines per skill (always) |
| Skill body | On trigger only | 0 lines until triggered |
| Reference files | On demand only | 0 lines until read |
| Agent definitions | On explicit invocation | 0 lines until invoked |

**Total always-loaded context: ~230 lines** (everything else is on-demand).

### Precedence

```
Project CLAUDE.md > Global CLAUDE.md > Default Claude behavior
```

Project-level instructions override global instructions when they conflict.

---

## 2. Global Setup (One-Time)

These files are created once and apply to every project.

### ~/.claude/CLAUDE.md

Controls Claude's interaction style across all projects:

- Concise responses, clarification-first approach
- No Claude references in git commits
- No auto-generated documentation without asking
- Ask before deleting files, making breaking changes, or updating dependencies
- Minimal code generation first, expand on request

### ~/.claude/skills/project-resume/SKILL.md

**Triggers:** Session start, "resume", "continue", "pick up where we left off", "start working"

**What it does:**
1. Reads CLAUDE.md, ROADMAP.md, and git-conventions.md
2. Runs `git log --oneline -10` to check recent commits
3. Finds resume point (first `[~]` task, or first `[ ]`)
4. Prints a session brief with branch, phase, task, and last commit
5. Asks a mandatory branch check before any work begins
6. Enforces four standing rules (see below)

**Standing Rules (enforced every session):**

| Rule | What It Does |
|------|-------------|
| R1 -- Never push without consent | Commits freely, never pushes. Says "Ready to push" after each commit. Prior-session consent does not carry over. |
| R2 -- Docker pre-flight | Checks Docker is running before any container command. Stops and asks user if not running. |
| R3 -- Brainstorm before new features | Requires a brainstorm analysis with approaches, trade-offs, risks before any new feature code. Waits for user sign-off. |
| R4 -- Update ROADMAP.md after every task | Marks tasks `[x]` with commit hash, advances next task to `[~]`, commits code + ROADMAP.md together. On context-limit exit, saves progress with precise notes. |

### ~/.claude/skills/docker-dev/SKILL.md

**Triggers:** Docker, containers, compose, "spin up", "bring up", "tear down", logs, restart, rebuild, health checks

**What it does:**
1. Runs pre-flight check (`docker info`) before any Docker command
2. Stops and waits for user if Docker is not running
3. Provides common Docker Compose commands
4. Looks for project-level `.claude/references/docker-services.md` for project-specific config
5. Enforces data protection rules

**Data Protection Rules:**
- Never remove volumes with database/file references without user confirmation
- Never prune containers, images, or volumes without user confirmation
- Never rebuild database/storage containers without user confirmation
- Displays affected containers/volumes and asks explicitly
- Prior-session consent does not carry over

---

## 3. Project Setup

When creating a new project:

### Step 1: Initialize

```bash
mkdir my-project && cd my-project
git init
```

### Step 2: Create .gitignore

Before any commits, create a `.gitignore` appropriate for the tech stack.

### Step 3: Create project CLAUDE.md

Copy from `templates/CLAUDE.md` and customize:
- Documentation standards
- Git policy (critical rules only -- details go in references)
- Code style guidelines for the project's languages
- On-demand skills and agents table

### Step 4: Create reference files

```bash
mkdir -p .claude/references .claude/agents
```

Copy from templates and customize:
- `.claude/references/git-conventions.md` -- branch naming, commit format, task markers
- `.claude/references/docker-services.md` -- project-specific services, ports, compose files (if applicable)
- `.claude/agents/security-reviewer.md` -- tailored to the project's auth/stack (if applicable)

### Step 5: Create ROADMAP.md

Copy from `templates/ROADMAP.md`. Structure your work into phases and tasks:

```markdown
# Project Roadmap

## Current Session State
Active task: None (new project)

## Phase 1 -- Foundation
- [ ] Set up project structure
- [ ] Configure database
- [ ] Implement authentication

## Phase 2 -- Core Features
- [ ] User dashboard
- [ ] API endpoints
```

### Step 6: Initial commit

```bash
git add .
git commit -m "chore: initialize project structure"
```

### Step 7: Create working branch

```bash
git checkout -b feature/initial-setup
```

---

## 4. Starting a Session

### Launch Claude Code

```bash
claude --add-dir ~/.claude/skills
```

### What Happens Automatically (project-resume skill)

1. **Load context** -- Reads CLAUDE.md, ROADMAP.md, git-conventions.md, runs `git log --oneline -10`

2. **Find resume point** -- Scans for first `[~]` task, then first `[ ]`

3. **Print session brief:**
   ```
   ------------------------------------------------------------
   Session Resume
   ------------------------------------------------------------
   Branch       : feature/user-authentication
   Active Phase : Phase 2 -- Core Features
   Resuming     : 2.3 Employee dashboard
   Stopped At   : API integration done, UI pending
   Last Commit  : f7a1ae7 fix(portal): fix React #310 error
   ------------------------------------------------------------
   ```

4. **Branch check (mandatory, waits for answer):**
   ```
   Branch check:
     Current branch: feature/user-authentication
     (1) Continue on this branch
     (2) Create a new feature branch
     (3) Create a new fix branch
   ```

No code is written until the user answers.

---

## 5. Development -- New Feature

### Step 1: Brainstorm (mandatory for new features)

Triggers automatically via project-resume R3. Claude produces:

```
Feature Brainstorm: Employee Leave Request Form
------------------------------------------------------------
Goal            : Allow employees to submit leave requests
Approaches      : 1. Server actions  2. API route + react-query  3. Form modal
Recommended     : Option 2 (consistent with existing patterns)
Trade-offs      : Extra API route vs simpler server action
Affected files  : frontends/hr-portal/app/leave/*, services/business/...
Risks           : Date validation edge cases
Open questions  : Approval workflow needed?
------------------------------------------------------------
```

Waits for user sign-off before any code.

**Skipped for:**
- Bug fixes with a known root cause
- Chore tasks (dependency updates, config changes, doc edits)
- Tasks with full implementation steps already in ROADMAP.md

### Step 2: Plan (multi-step tasks)

For complex features, the `writing-plans` skill activates to create a step-by-step implementation plan before coding.

### Step 3: Implement

- Code follows project CLAUDE.md style rules
- Files kept under 300 lines
- Language-specific conventions followed
- Comments explain WHY, not WHAT

### Step 4: After each task

1. ROADMAP.md updated: `[ ]` -> `[x]` with `*(commit: abc1234)*`
2. Next task marked `[~]`
3. `## Current Session State` updated
4. Code + ROADMAP.md committed together
5. "Ready to push -- confirm when you'd like me to."

---

## 6. Development -- Bug Fix

### Step 1: Systematic debugging

The `systematic-debugging` skill triggers to:
- Reproduce the issue
- Identify root cause
- Verify the fix

### Step 2: Fix and test

- Write regression test if applicable
- Fix the bug
- Verify the fix passes

### Step 3: Commit

No brainstorm needed for bug fixes with a known cause. Commit with:

```
fix(scope): description of the fix
```

---

## 7. UI Development

### Trigger

The `frontend-design` skill activates when building UI components, pages, or applications.

### What Happens

- Bold aesthetic direction chosen (not generic AI patterns)
- Distinctive typography, color, motion
- Uses existing stack conventions (React, TailwindCSS, shadcn/ui, etc.)
- Production-grade, accessible components

### Browser Testing (browser-use skill)

```bash
browser-use open http://localhost:3000          # open dev server
browser-use state                                # inspect elements
browser-use screenshot dashboard.png             # capture result
browser-use click 5                              # interact with element
```

---

## 8. Docker and Infrastructure

### Trigger

The global `docker-dev` skill activates when mentioning docker, containers, compose, etc.

### Pre-flight (always first)

```bash
docker info > /dev/null 2>&1 && echo "RUNNING" || echo "NOT RUNNING"
```

If not running, Claude stops and asks the user to start Docker.

### Data Protection

Claude will always ask for explicit confirmation before:
- Removing volumes (`docker compose down -v`, `docker volume rm/prune`)
- Pruning containers, images, or system (`docker system/container/image prune`)
- Rebuilding database or storage containers
- Removing containers with mounted volumes

### Project-Specific Config

If the project has `.claude/references/docker-services.md`, Claude reads it for:
- Compose file paths
- Service map (containers, ports, health endpoints)
- Project-specific workflows (dev vs full stack)
- Access credentials for dev tools (pgAdmin, Keycloak admin, etc.)

---

## 9. Testing

### Trigger

The `/test` skill or implementing a feature (auto via TDD skill).

### Backend

```bash
cd services/<service> && mvn test           # Java/Spring Boot
cd services/<service> && pytest              # Python
```

### Frontend

```bash
cd frontends/<app> && npm run lint           # lint
cd frontends/<app> && npm run type-check     # TypeScript check
cd frontends/<app> && npm test               # unit tests
```

### Verification Before Completion

The `verification-before-completion` skill ensures Claude runs actual verification commands and shows output before claiming anything is done. No "it should work" -- evidence first.

---

## 10. Code Review

### Trigger

After completing features, before merging branches, or on demand via `/review` or `/code-review:code-review`.

### Project-Level Agent

Each project can have a `.claude/agents/code-reviewer.md` for structured review. The agent evaluates:

- **Correctness** -- Logic, edge cases, error handling, return types
- **Code Quality** -- Naming, DRY, dead code, file size, import order
- **Performance** -- N+1 queries, unnecessary loops, blocking operations
- **Security** -- Hardcoded secrets, input validation, SQL injection, XSS
- **Maintainability** -- Readability, consistent patterns, test coverage
- **API Design** -- Naming, breaking changes, response consistency

### Output Format

```
Code Review: <branch or feature>
------------------------------------------------------------
Files reviewed : [count]
Issues found   : [count]
Verdict        : APPROVE / REQUEST CHANGES / COMMENT
------------------------------------------------------------
```

Findings are ranked: BLOCKER > MAJOR > MINOR > NIT. Only blockers prevent merge.

### Plugin Alternative

The `/review` and `/code-review:code-review` plugin skills provide code review without needing the project-level agent file. The agent template adds project-specific review criteria.

---

## 11. Security Review

### Trigger

Ask for a security review, or invoke before deployment.

### Project-Level Agent

Each project can have a `.claude/agents/security-reviewer.md` tailored to its stack. The agent:
- Scans for hardcoded secrets
- Checks authentication and authorization config
- Reviews CORS, CSP, input validation
- Checks for XSS, SQL injection, open endpoints
- Reviews Docker security (non-root users, exposed ports)
- Checks dependency CVEs

### Output Format

```
Security Review: <Project>
------------------------------------------------------------
CRITICAL : [count] issues requiring immediate fix
HIGH     : [count] issues to fix before production
MEDIUM   : [count] recommended improvements
LOW      : [count] minor hardening suggestions
------------------------------------------------------------
```

Also available: `/security:security-audit` plugin for broader checks.

---

## 12. Git Workflow

### Branch Naming

Read from `.claude/references/git-conventions.md` on demand:

| Type | Pattern | Example |
|------|---------|---------|
| Feature | `feature/<description>` | `feature/user-auth` |
| Bug fix | `fix/<description>` | `fix/login-timeout` |
| Hotfix | `hotfix/<description>` | `hotfix/security-patch` |
| Refactor | `refactor/<scope>` | `refactor/database-layer` |
| Docs | `docs/<topic>` | `docs/api-endpoints` |
| Experiment | `experiment/<name>` | `experiment/new-algorithm` |

### Commit Messages

Conventional Commits format:

```
<type>(<scope>): <subject>
```

Types: `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `build`, `ci`, `chore`, `revert`

### Critical Rules (always enforced)

- No Claude references in commits (no Co-Authored-By, no "Generated with Claude Code")
- Use `themaverick` as the commit author name
- Never push without explicit user confirmation
- Prior-session push consent does not carry over

### Task Tracking in ROADMAP.md

| Marker | Status |
|--------|--------|
| `[ ]` | Pending |
| `[~]` | In Progress (with notes on what remains) |
| `[x]` | Completed (with commit hash) |
| `[!]` | Blocked (with reason) |

### Branch Completion

The `finishing-a-development-branch` skill activates when all tasks on a branch are done, presenting options:
1. Merge to main
2. Create PR
3. Cleanup

---

## 13. Ending a Session

### Normal End

1. All completed tasks marked `[x]` with commit hashes in ROADMAP.md
2. Any in-progress task marked `[~]` with precise notes
3. `## Current Session State` updated in ROADMAP.md
4. Final commit made (code + ROADMAP.md)
5. "Ready to push -- confirm when you'd like me to."

### Context Limit Hit (automatic)

1. Current task updated to `[~]` with exact stopping point
2. `## Current Session State` updated
3. All changes committed (code + ROADMAP.md)
4. Next session picks up automatically from the `[~]` marker

---

## 14. Resuming a Session

### Interactive

```bash
claude --add-dir ~/.claude/skills
# type: resume
```

### Headless

```bash
claude --add-dir ~/.claude/skills --dangerously-skip-permissions -p "resume"
```

Both trigger the project-resume skill, which runs the full session start sequence. The `[~]` marker and note from the previous session tell Claude exactly where to pick up.

---

## 15. Context Optimization

### Design Principle

Only load what is needed for the current task. Everything else stays on disk until triggered.

### Always Loaded (~230 lines total)

| Source | Lines | Why Always |
|--------|-------|-----------|
| `~/.claude/CLAUDE.md` | ~100 | Interaction style, boundaries |
| `project/CLAUDE.md` | ~130 | Code style, critical git rules, skill routing |
| Skill metadata | ~2 each | Trigger matching only |

### On-Demand (0 lines until triggered)

| Source | Trigger | Lines When Loaded |
|--------|---------|-------------------|
| project-resume skill body | Session start / "resume" | ~117 |
| docker-dev skill body | Docker/container tasks | ~72 |
| browser-use skill body | Browser automation tasks | ~547 |
| git-conventions.md | Committing / branching | ~93 |
| documentation-standards.md | Creating / editing docs | ~50 |
| docker-services.md | Docker tasks (project-specific) | ~78 |
| code-reviewer agent | Code review | ~90 |
| security-reviewer agent | Security audits | ~120 |
| Plugin skills (commit, review, test, etc.) | On invocation | Varies |

### How to Keep It Optimized

1. **Project CLAUDE.md stays slim** -- Only code style rules and critical safety rules. Everything else points to skills or reference files.
2. **No duplication** -- If a skill already has the content (brainstorm format, docker pre-flight, session tracking), don't repeat it in CLAUDE.md.
3. **Reference files for detailed formats** -- Branch naming, commit types, task markers live in `.claude/references/` and are read only when needed.
4. **Agents for specialized tasks** -- Security review, performance analysis, etc. are separate files invoked explicitly.

---

## 16. Installed Plugins and Skills

### Global Skills (user-created)

| Skill | Location | Trigger |
|-------|----------|---------|
| project-resume | `~/.claude/skills/` | Session start, "resume", "continue" |
| docker-dev | `~/.claude/skills/` | Docker, containers, compose, infrastructure |
| browser-use | `~/.agents/skills/` | Browser automation, web testing |

### Plugin Skills (installed via plugins)

| Skill | Trigger | Type |
|-------|---------|------|
| `commit` | `/commit` | Git commit with conventions |
| `review` | `/review` | Code review |
| `test` | `/test` | Smart test runner |
| `frontend-design` | UI tasks, `/frontend-design:frontend-design` | Premium UI development |
| `session-start` | `/session-start` | Start coding session |
| `session-end` | `/session-end` | End coding session |
| `brainstorming` | Auto on new features | Explore intent before implementation |
| `writing-plans` | Multi-step tasks | Step-by-step implementation plan |
| `executing-plans` | Follow a plan | Execute with review checkpoints |
| `verification-before-completion` | Before claiming done | Evidence before assertions |
| `systematic-debugging` | Bugs / test failures | Structured debugging |
| `finishing-a-development-branch` | Branch complete | Merge / PR / cleanup options |
| `code-review` | `/review`, `/code-review:code-review`, or agent | Code quality, correctness, performance review |
| `security-audit` | `/security:security-audit` | Broad security checks |
| `dispatching-parallel-agents` | 2+ independent tasks | Parallel execution |

---

## Workflow Diagram

```
Project Setup
  Create CLAUDE.md + ROADMAP.md + .gitignore + references/
  Initial commit on main
  Create working branch
       |
       v
Start Session (project-resume skill)
  Load context -> Find resume point -> Print brief -> Branch check
  Wait for user answer
       |
       v
  +------------------+--------------------+
  |                  |                    |
  v                  v                    v
New Feature        Bug Fix             Chore/Doc
  |                  |                    |
  v                  v                    |
Brainstorm       Debug skill             |
  |                  |                    |
  v                  v                    |
Plan (if complex)  Fix + test            |
  |                  |                    |
  v                  v                    v
Implement --------+------------ Implement/Edit
  |                                       |
  +--- UI work? -> frontend-design        |
  |                                       |
  +--- Docker? -> docker-dev skill        |
  |      Pre-flight -> Data protection    |
  |      Project docker-services.md       |
  |                                       |
  +--- Browser? -> browser-use skill      |
  |                                       |
  v                                       v
Test (verification-before-completion)
  Run actual commands, show output
       |
       v
Commit (conventional commits from git-conventions.md)
  Update ROADMAP.md [x] with hash
  Mark next task [~]
  "Ready to push -- confirm?"
       |
       v
  +--- More tasks? -> Loop back
  |
  v
Code Review (code-reviewer agent or /review)
  Correctness, quality, performance, security
       |
       v
Branch Complete (finishing-a-development-branch)
  Merge / PR / Cleanup
       |
       v
Security Review (optional, before deploy)
  security-reviewer agent
       |
       v
End Session
  Update ROADMAP.md -> Commit -> Await push confirmation
       |
       v
Resume (next session)
  project-resume picks up from [~] marker
```
