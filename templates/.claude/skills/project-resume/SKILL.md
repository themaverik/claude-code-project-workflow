---
name: project-resume
description: Use this skill at the start of every Claude Code session, or whenever
  the user says "resume", "continue", "pick up where we left off", "where were we",
  or starts a session in a project directory. Reads CLAUDE.md, ROADMAP.md, and CHANGELOG.md
  (if post-MVP) to orient the session, identify the next task, run safety checks,
  and begin work without requiring the user to repeat context. Also triggers for
  "start working", "let's continue", or any implicit continuation of prior work.
  Integrates with adr-manager skill (passive ADR capture) and doc-sync skill
  (end-of-session knowledge base sync).
---

# Project Resume

Orient every session using project context as the single source of truth.

**Active companion skills** (always on, no trigger needed):
- `adr-manager` — passively captures every architectural decision made during the session
- `doc-sync` — consolidates all docs into Knowledge_Base.md at session end if ADRs were logged
- `changelog-roadmap-policy` — governs change tracking based on project maturity

**Lazy skills** (load only when triggered — zero cost otherwise):
- `knowledge-ingest` — reads superpowers docs, specs, plans on demand; feeds decisions to adr-manager
- `docker-dev` — docker/compose operations

**Note:** This skill works with the `changelog-roadmap-policy` rule. Change tracking depends on project maturity:
- **Pre-MVP/Pre-Release:** Use ROADMAP.md only
- **Post-MVP/Post-Release:** Use ROADMAP.md (features) + CHANGELOG.md (all changes)

Four standing rules apply to every session regardless of task type.

## Session Start (run every time, in order)

### 1. Load context

Read CLAUDE.md, `.claude/references/git-conventions.md`, run `git log --oneline -10`, and note the current branch.

Then check project status to determine what to read:
- **If CHANGELOG.md exists:** Project is post-MVP/post-release. Read both CHANGELOG.md (for recent changes) and ROADMAP.md (for future planned features).
- **If only ROADMAP.md exists:** Project is pre-MVP/pre-release. Read ROADMAP.md for all development progress.

Also check ADR state:
- Count files in `docs/adr/` if directory exists
- Check if `docs/Knowledge_Base.md` exists and note last sync date
- Note both in session brief (see step 3)

### 2. Find resume point

**For pre-MVP projects (ROADMAP.md only):**
Check `## Current Session State` in ROADMAP.md, then scan task markers:
1. First `[~]` task — resume from its note
2. If none, first `[ ]` in the lowest active phase
3. If none, tell the user all tasks are complete/blocked and ask what to do

**For post-MVP projects (ROADMAP.md + CHANGELOG.md):**
Check `## Current Session State` in ROADMAP.md (if it exists):
1. First `[~]` task in planned features — resume from its note
2. If none, first `[ ]` in lowest planned phase
3. If none, ask user what feature to start next (use ROADMAP as planning tool, track changes in CHANGELOG.md)

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
ADR Records  : <count in docs/adr/ or "none yet">
Knowledge    : <"synced <date>" or "not yet generated">
------------------------------------------------------------
ADR capture  : ACTIVE (adr-manager — decisions logged automatically)
Specs/plans  : lazy — say "ingest specs" or "/knowledge-ingest" to load
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

**On sign-off:** The `adr-manager` skill automatically converts this brainstorm into an ADR. No user action needed — the ADR is written and committed with the feature code (see R4). Do not ask for permission.

### R4 — Update ROADMAP.md and/or CHANGELOG.md after every task

**For pre-MVP projects (ROADMAP.md only):**

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

---

**For post-MVP projects (ROADMAP.md + CHANGELOG.md):**

**On task completion (any change: feature, bugfix, update):**
1. Update CHANGELOG.md under `[Unreleased]` section with the change (Added/Fixed/Changed/Removed)
2. If feature adds to roadmap, update ROADMAP.md planned phases
3. Commit code + CHANGELOG.md together
4. Do not push (R1)

**On context-limit exit:**
1. Document what was completed in CHANGELOG.md `[Unreleased]`
2. Update ROADMAP.md if feature affects planned phases
3. Commit all in-progress changes + CHANGELOG.md and/or ROADMAP.md

**Reference:** See `changelog-roadmap-policy` skill for full change tracking rules and CHANGELOG format.

## Session End Routine

Before the final commit of any session, run this sequence in order:

1. **ADR sweep** (adr-manager) — any uncaptured decisions this session?
2. **Spec sweep** — were any spec/plan/superpowers files read this session that contain decisions not yet in ADR? If yes → flag for `/adr-ingest` or capture inline.
3. **R4** — mark tasks, update ROADMAP/CHANGELOG.
4. **Doc sync** (doc-sync) — if 1+ ADRs added OR knowledge-ingest ran this session.
5. **Single commit** — code + ADRs + ROADMAP/CHANGELOG + Knowledge_Base.md.
6. **Print end summary:**

```
------------------------------------------------------------
Session End
------------------------------------------------------------
Tasks completed  : <n>
ADRs logged      : <n> (<ADR-NNN, ...>)
Specs ingested   : <n files> (or "none this session")
ROADMAP updated  : Yes / No
Knowledge_Base   : Synced / Skipped
Ready to push    : confirm when ready
------------------------------------------------------------
```

## Resume Command

```bash
claude --add-dir ~/.claude/skills --dangerously-skip-permissions -p "resume"
```

Or interactively:

```bash
claude --add-dir ~/.claude/skills --dangerously-skip-permissions
# then type: resume
```
