---
name: project-resume
description: Use this skill at the start of every Claude Code session, or whenever
  the user says "resume", "continue", "pick up where we left off", "where were we",
  or starts a session in a project directory. Reads CLAUDE.md and ROADMAP.md to
  orient the session, identify the next task, run safety checks, and begin work
  without requiring the user to repeat context. Also triggers for "start working",
  "let's continue", or any implicit continuation of prior work.
---

# Project Resume

Orient every session using CLAUDE.md and ROADMAP.md as the single source of truth.
Four standing rules apply to every session regardless of task type.

## Session Start (run every time, in order)

### 1. Load context

Read CLAUDE.md, ROADMAP.md, `.claude/references/git-conventions.md`, run `git log --oneline -10`, and note the current branch.

### 2. Find resume point

Check `## Current Session State` in ROADMAP.md, then scan task markers:

1. First `[~]` task — resume from its note
2. If none, first `[ ]` in the lowest active phase
3. If none, tell the user all tasks are complete/blocked and ask what to do

### 3. Print session brief

```
------------------------------------------------------------
Session Resume
------------------------------------------------------------
Branch       : <branch>
Active Phase : <phase from ROADMAP.md>
Resuming     : <task ID and name>
Stopped At   : <note, or "fresh start">
Last Commit  : <hash and subject>
------------------------------------------------------------
```

### 4. Branch check (mandatory)

Print this and wait for an explicit answer before writing any code or committing:

```
Branch check:
  Current branch: <branch>
  (1) Continue on this branch
  (2) Create a new feature branch
  (3) Create a new fix branch
```

## Standing Rules

### R1 — Never push without consent

Commit freely after each task. Never run `git push` automatically. After committing say: "Ready to push -- confirm when you'd like me to." Prior-session consent does not carry over.

### R2 — Docker pre-flight

Before any `docker` / `docker compose` command, run:

```bash
docker info > /dev/null 2>&1 && echo "RUNNING" || echo "NOT RUNNING"
```

If NOT RUNNING, stop and ask the user to start Docker. Do not retry or start it yourself.

### R3 — Brainstorm before new features

Applies to tasks introducing new behaviour (pages, APIs, services, UI components, anything marked feature). Does not apply to bug fixes with known cause, chores, or tasks with full implementation steps in ROADMAP.md.

Before coding, produce this and wait for sign-off:

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

### R4 — Update ROADMAP.md after every task

**On task completion:**
1. Marker `[ ]`/`[~]` to `[x]`, append `*(commit: <hash>)*`
2. Mark next task `[~]`
3. Update `## Current Session State`
4. Commit code + ROADMAP.md together
5. Do not push (R1)

**On context-limit exit:**
1. Set current task to `[~]` with a note on what was done and what remains
2. Update `## Current Session State`
3. Commit all in-progress changes + ROADMAP.md

## Resume Command

```bash
claude --add-dir ~/.claude/skills --dangerously-skip-permissions -p "resume"
```

Or interactively:

```bash
claude --add-dir ~/.claude/skills --dangerously-skip-permissions
# then type: resume
```
