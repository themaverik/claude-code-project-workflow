---
name: knowledge-ingest
description: >
  Reads superpowers docs, specs, feature plans, and plugin docs to extract
  structured knowledge into Knowledge-Base.md and feed decisions into ADR
  capture. Triggers on "ingest specs", "read superpowers", "scan plans",
  "what does the spec say about X", "load feature docs", or "/knowledge-ingest".
  Also triggers when user references a specific spec/plan file by name.
  Never preloaded — always on-demand only.
disable-model-invocation: false
user-invocable: true
allowed-tools: Read Bash(find *) Bash(ls *) Bash(wc *)
---

# Knowledge Ingest

Reads docs/superpowers/, docs/specs/, docs/plans/, and .specstory/ on demand.
Never loaded at session start — invoked only when needed.

## Trigger Phrases
- "ingest specs" / "read superpowers" / "scan plans"
- "what does the spec say about X"
- "load the feature docs"
- "check the plan for X"
- `/knowledge-ingest [optional: specific file or topic]`

## Step 1 — Discover Sources

```!
echo "=== Superpowers/Specs/Plans ==="
find docs/ -name "*.md" 2>/dev/null | grep -E "(superpower|spec|plan|feature|plugin|design)" | head -30
find .specstory/ -name "*.md" 2>/dev/null | head -20
find . -maxdepth 2 -name "superpowers*.md" -o -name "*-spec.md" -o -name "*-plan.md" 2>/dev/null | head -20
echo "=== File sizes (token estimate) ==="
find docs/ .specstory/ -name "*.md" 2>/dev/null | xargs wc -l 2>/dev/null | sort -rn | head -20
```

Print discovery summary and ask which files to ingest if $ARGUMENTS is empty.
If $ARGUMENTS contains a topic or filename, filter to matching files only.

## Step 2 — Read and Extract

For each selected file, extract into four buckets:

**Decisions** — explicit or implicit architectural choices
- Pattern: "we use X", "chosen approach is Y", "decided to Z", any design described as settled
- Feed directly to adr-manager for ADR creation (do not duplicate if ADR exists)

**Constraints** — hard limits, non-negotiables
- Pattern: "must not", "cannot", "out of scope", "not supported"
- Add to `docs/Knowledge-Base.md` → Constraints section

**Open Questions** — unresolved design points
- Pattern: "TBD", "to be decided", "open question", "not yet determined"
- Add to `docs/Knowledge-Base.md` → Open Decisions section

**Domain Knowledge** — business rules, data models, integration specs
- Everything else that isn't a decision or constraint
- Add to `docs/Knowledge-Base.md` → Domain Knowledge section by feature area

## Step 3 — ADR Handoff

For each **Decision** extracted:

1. Check `docs/adr/index.md` for existing coverage
2. If not covered → call adr-manager to create ADR with:
   - `Source: spec:<filename>` or `plan:<filename>`
   - Context reconstructed from the spec narrative
   - Status: `Accepted` if feature is built, `Proposed` if still planned
3. If partially covered → note gap in existing ADR as a comment

## Step 4 — Update Knowledge-Base.md

Append extracted knowledge to `docs/Knowledge-Base.md` under:

```markdown
## Domain Knowledge

### <Feature Area from spec filename>
*Sourced from: <filename> — <date ingested>*

#### Constraints
- <constraint>

#### Business Rules
- <rule>

#### Integration Points
- <integration>

#### Open Questions
| Question | Raised In | Status |
|----------|-----------|--------|
```

Do not overwrite existing Knowledge-Base content — append or update matching sections only.

## Step 5 — Ingest Summary

```
Knowledge Ingest Complete
------------------------------------------------------------
Files read       : <n> (<list>)
Decisions found  : <n> → sent to adr-manager
Constraints      : <n> → added to Knowledge-Base
Open questions   : <n> → added to Knowledge-Base
Domain entries   : <n> → added to Knowledge-Base
New ADRs created : <n>
------------------------------------------------------------
```

## Token Budget Note

Read files **selectively** — do not dump entire spec files into context.
- Read file → extract structured data → discard raw content
- If a file is >300 lines, read in sections, extract per section
- Store extracted data in Knowledge-Base, not in conversation
