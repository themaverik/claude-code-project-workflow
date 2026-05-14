---
name: adr-manager
description: >
  Captures architectural decisions automatically. Triggers on brainstorm
  sign-off, plan mode decisions, "log this as ADR", "record this decision",
  tech stack choices, trade-off acceptances, or "why did we choose X".
  Also scans superpowers docs, specs, and plans for implicit decisions on
  "/adr-ingest" or "scan specs for decisions". Never waits for the user to ask.
user-invocable: true
allowed-tools: Read Bash(find *) Bash(ls *) Bash(cat *)
---

# ADR Manager

Captures decisions automatically. Runs passively every session.

## Decision Detection

**Auto-capture (no user prompt needed):**
- R3 brainstorm sign-off → immediately draft ADR from brainstorm output
- "let's go with X" / "we'll use X" / "we're going with X"
- Tech stack or library chosen over alternatives
- Structural pattern decided (schema-per-service, REST over events, etc.)
- Trade-off explicitly accepted ("we'll live with X for now")
- Any plan-mode decision that locks in an approach

**Explicit triggers:**
- "log this as ADR" / "record this decision" / "add to ADR"
- "why did we choose X" → check if ADR exists, create if missing
- `/adr-ingest` or "scan specs for decisions" → run knowledge source scan (see below)

**Never log:** bug fixes, routine task completions, style choices, already-captured decisions.

## Knowledge Source Scan (`/adr-ingest`)

When invoked via `/adr-ingest` or "scan specs for decisions":

```!
find docs/ .specstory/ -name "*.md" 2>/dev/null | head -40
find . -name "superpowers*.md" -o -name "*-spec.md" -o -name "*-plan.md" -o -name "*.spec.md" 2>/dev/null | head -20
```

Read the discovered files. For each file, extract:
1. Any explicit decisions ("we chose X", "decided to use Y", "going with Z")
2. Any implicit decisions (a design described without alternatives — the choice itself is the decision)
3. Any constraints or trade-offs stated as facts ("X is not supported", "we avoid Y because")

For each extracted decision:
- Check `docs/adr/index.md` — skip if already captured
- Draft a new ADR using the spec/plan as the Context source
- Set `Status: Accepted` if the feature is implemented, `Proposed` if still in design
- Note in the ADR: `*(sourced from: <filename>)*`

Print a scan summary before writing:
```
Knowledge Source Scan
------------------------------------------------------------
Files scanned    : <n>
Decisions found  : <n>
Already captured : <n>
New ADRs to write: <n> (<titles>)
------------------------------------------------------------
Proceed? (yes to write all / review to show each first)
```

Wait for confirmation before writing ADR files.

## ADR File Format

Directory: `docs/adr/ADR-NNN-<kebab-slug>.md`
Next number: count existing `docs/adr/ADR-*.md` files + 1, zero-padded to 3 digits.

```markdown
# ADR-<NNN>: <Title>

**Status**: <Proposed | Accepted | Deprecated | Superseded by ADR-XXX>
**Date**: <YYYY-MM-DD>
**Session**: <ROADMAP phase or task context>
**Source**: <brainstorm | spec:<filename> | plan:<filename> | session>

## Context
<What forced this decision? What constraints existed?>

## Options Considered

| Option | Pros | Cons |
|--------|------|------|
| <A> | | |
| <B> | | |

## Decision
**Chosen: <Option>**
<Why this won. What tipped the scale.>

## Consequences
- ✅ <Positive>
- ⚠️ <Trade-off accepted>
- 🔄 <Future implication>

## Supersedes
<ADR-XXX or None>
```

## ADR Index

Keep `docs/adr/index.md` current. Regenerate table on every write:

```markdown
# ADR Registry
| ID | Title | Status | Date | Source | Supersedes |
|----|-------|--------|------|--------|------------|
```

## Integration with R3 Brainstorm

On brainstorm sign-off → immediately:
1. Draft ADR from: Goal→Context, Approaches→Options, Recommended+Trade-offs→Decision, Risks→Consequences
2. Write `docs/adr/ADR-NNN-<slug>.md`
3. Update `docs/adr/index.md`
4. Include in same commit as feature code

**Do not ask. Just do it.**

## Session-End Sweep

Before final commit, check: any decisions made this session without an ADR?
Also check: were any superpowers/spec files read this session that contain decisions not yet captured?

Print sweep summary (see project-resume skill for full session-end format).

## Rules

- **R-ADR1 Immutable**: Never edit an accepted ADR's Decision. New decision = new ADR with `Supersedes`.
- **R-ADR2 Commit together**: ADR commits with the code it relates to. Never standalone unless retroactive.
- **R-ADR3 No interruption**: Write silently. Only prompt if decision is genuinely ambiguous.
- **R-ADR4 Source always noted**: Every ADR records where the decision came from (brainstorm / spec file / session).
