---
name: doc-sync
description: >
  Consolidates all project docs into Knowledge-Base.md. Triggers on "sync docs",
  "consolidate docs", "update knowledge base", or automatically at session end
  when ADRs were added or knowledge-ingest was run. Reads ADRs, ROADMAP,
  CHANGELOG, CLAUDE.md, and any previously ingested spec/plan extractions.
user-invocable: true
allowed-tools: Read Bash(find *) Bash(ls *) Bash(date *)
---

# Doc Sync

Produces and maintains `docs/Knowledge-Base.md`. Generated, never manually authored.

## Triggers
- "sync docs" / "consolidate docs" / "update knowledge base"
- `/doc-sync`
- Auto: session end where 1+ ADRs added OR knowledge-ingest was run this session
- Auto: every 5th new ADR milestone

## Source Discovery

```!
echo "=== Core docs ==="
ls -la CLAUDE.md ROADMAP.md CHANGELOG.md 2>/dev/null
echo "=== ADR registry ==="
ls docs/adr/ 2>/dev/null | wc -l
echo "=== Ingested knowledge ==="
ls docs/ 2>/dev/null
echo "=== Spec/plan files ==="
find docs/ .specstory/ -name "*.md" 2>/dev/null | grep -E "(superpower|spec|plan|feature|plugin)" | head -20
echo "=== Current date ==="
date +%Y-%m-%d
```

## Output: docs/Knowledge-Base.md

```markdown
# Knowledge Base
> Auto-generated. Do not edit directly.
> Last synced: <date> | ADRs: <n> | Sources: <list>

---

## 1. Project Identity
<From CLAUDE.md — name, purpose, stack, key constraints, team conventions>

---

## 2. Architecture Decisions

| ID | Title | Status | Date | Source | Supersedes |
|----|-------|--------|------|--------|------------|
<Full ADR registry table from docs/adr/index.md>

### By Domain

#### Infrastructure & DevOps
#### Data & Storage  
#### Auth & Security
#### Services & Integration
#### Frontend
#### Cross-cutting

---

## 3. Domain Knowledge
<From knowledge-ingest runs — feature specs, business rules, constraints>
<Grouped by feature area, each section tagged with source file>

---

## 4. Implementation History
<From ROADMAP.md phases — completed with task summary and commit refs>

| Phase | Description | Status | Key Commits |
|-------|-------------|--------|-------------|

---

## 5. Current State
<From ROADMAP.md "Current Session State" + CHANGELOG.md [Unreleased]>

---

## 6. Open Decisions
<ADRs with Status: Proposed + open questions from knowledge-ingest>

| ID/Question | Source | Blocking? |
|-------------|--------|-----------|

---

## 7. Tech Debt & Future Implications
<From ADR Consequences ⚠️ 🔄 markers + ROADMAP deferred items>

---

## 8. Docs Index

| Document | Purpose | Last Updated |
|----------|---------|-------------|
| CLAUDE.md | Dev guidelines | |
| ROADMAP.md | Feature tracking | |
| docs/adr/ | Decision records | |
| docs/Knowledge-Base.md | This file | |
```

## Sync Rules

- **Non-destructive**: Update changed sections only. Preserve structure.
- **Domain Knowledge section**: merge with existing, tag new entries with source+date
- **Commit**: `docs: sync Knowledge-Base.md — <n> ADRs, <n> specs, <phase>`
- **Never block coding**: run after all code commits at session end
- **Print diff summary** after sync (ADRs indexed, specs merged, open decisions count)
